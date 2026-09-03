# Host Software & GUI Architecture

> **Related notes:** [[02_System_Architecture]] · [[04_Firmware_FPGA]] · [[04b_USB_Interface]]
> **Status:** Read-only Forensic Snapshot

---

## 1. Executive Summary

The repository contains multiple iterations of a Python-based host control and visualization dashboard for the radar. The software is responsible for issuing high-level configurations to the radar (via opcodes), recording high-speed data streams, and visualizing range-doppler matrices in real time. The software communicates almost exclusively over a dedicated high-speed USB bridge (FTDI FT2232H) directly into the FPGA, bypassing the MCU for heavy data lifting.

---

## 2. GUI Version Inventory

The repository contains several historical and active versions of the software located in `9_Firmware/9_3_GUI/`.

| GUI File | Framework | Status | Description / Relationship |
| -------- | --------- | ------ | -------------------------- |
| `GUI_V5.py` | Tkinter | Legacy | Historical monolith GUI. |
| `GUI_V6.py` | Tkinter | Deprecated | Attempted FT601 (USB 3.0) support. Hardware does not support this. |
| `GUI_V65_Tk.py` | Tkinter | Development | Board bring-up GUI. Used for initial FT2232H testing and HDF5 recording. |
| `GUI_V7_PyQt.py` | PyQt6 | **Production** | Modern, modularized dashboard drawing from the `v7/` package. Uses hardware acceleration and a clear MVC architecture. |

*Note: The remainder of this document focuses on the canonical V7 architecture.*

---

## 3. Host Software Architecture

The V7 software utilizes a modular, multi-threaded architecture to decouple the high-speed USB data acquisition from the Qt rendering thread.

```mermaid
flowchart TB
    USER["Operator (GUI)"]
    DASH["RadarDashboard (v7/dashboard.py)"]
    WORK["Processing Workers (v7/workers.py)"]
    PROTO["Protocol Layer (radar_protocol.py)"]
    USB["FTDI Driver (pyftdi)"]
    HW["Hardware (FT2232H -> FPGA)"]

    USER <--> DASH
    DASH -->|Config Opcodes| PROTO
    PROTO -->|Raw USB Writes| USB
    USB <--> HW
    
    HW -->|Raw USB Reads| USB
    USB -->|Byte Streams| PROTO
    PROTO -->|Parsed Frames| WORK
    WORK -->|Qt Signals (numpy arrays)| DASH
```

### Application Entry Points
*   **Main Application:** `GUI_V7_PyQt.py` serves as the entry script, instantiating the `RadarDashboard` from `v7/dashboard.py`.
*   **Acquisition Thread:** `radar_protocol.py` spawns a `RadarAcquisition` thread that continuously polls the USB interface to prevent FIFO overflow on the hardware.

---

## 4. USB Transport & Device Abstraction

*   **Driver:** The software uses `pyftdi` in **245 Synchronous FIFO mode** to interface with the FT2232H (`radar_protocol.py`, `FT2232HConnection`).
*   **Endpoints:** The FTDI chip acts as a transparent bridge. The software writes commands to Channel A and reads massive data blocks (64 KB chunks) from the same channel.
*   **Abstraction:** The `FT2232HConnection` class abstracts the hardware, providing a uniform `read()` and `write()` interface for the protocol layer.

---

## 5. Protocol & Data Reception Pipeline

The software uses a strict binary protocol to communicate with the FPGA.

### Command Structure (Host → FPGA)
Commands are defined in `Opcode(IntEnum)` within `radar_protocol.py`. These perfectly match the Verilog `usb_cmd_opcode` cases (0x01 to 0xFF).
*   **Example Commands:** `RADAR_MODE (0x01)`, `DETECT_THRESHOLD (0x03)`, `LONG_CHIRP (0x10)`, `CFAR_ENABLE (0x25)`, `AGC_ENABLE (0x28)`.

### Data Reception (FPGA → Host)
1.  **Transport:** `RadarAcquisition` thread reads 64 KB chunks from the FTDI buffer.
2.  **Framing:** The `RadarProtocol` class searches for specific header (`0xAA`) and footer (`0x55`) bytes to align frames.
3.  **Deserialization:** Parsed into Python `RadarFrame` (target matrices) or `StatusResponse` objects.
4.  **Processing:** Emitted via PyQt signals (`_on_frame_ready`) to the GUI thread for visualization.
5.  **Recording:** Parallelly written to disk in HDF5 format by the `DataRecorder` class if enabled.

---

## 6. GUI Component Map (V7)

The V7 dashboard utilizes a tabbed interface (`v7/dashboard.py`):

| UI Component | Backend Function | Hardware Effect |
| ------------ | ---------------- | --------------- |
| **Main Tab** | Real-time target lists and scatter plots. | None (Read-only visualization). |
| **Map Tab** | `RangeDopplerCanvas` (matplotlib) rendering heatmaps. | None. |
| **FPGA Control** | Sliders/spinboxes for chirp/CFAR settings. | Sends opcodes (e.g., 0x24 for CFAR mode) directly to FPGA registers. |
| **AGC Monitor** | Visualizes AGC state. | Toggles FPGA digital AGC tracking. |
| **Diagnostics** | Displays device health and hardware logs. | Triggers `STATUS_REQUEST (0xFF)`. |

---

## 7. Build & Deployment Status

*   **Language:** Python 3.12+
*   **Dependencies:** `requirements_v7.txt` (PyQt6, numpy, matplotlib, pyftdi).
*   **Status:** The application is highly portable and can be run locally assuming the FTDI drivers are installed. It also contains a mocked `FT2232HConnection` mode for UI testing without the radar hardware attached.

---

## 8. Replication-Critical Behavior

1.  **pyftdi Synchronous FIFO Configuration:** The exact configuration (`set_bitmode(0xFF, PyFtdi.BitMode.SYNCFF)`) is mandatory. If standard asynchronous mode is used, bandwidth will collapse and the FPGA FIFOs will overflow.
2.  **Opcode Mapping:** The Python `Opcode(IntEnum)` is strictly coupled to the Verilog `radar_system_top.v` state machine. Any changes to one must be mirrored in the other.
3.  **Thread Priorities:** The USB acquisition thread must not be blocked by the GUI rendering (matplotlib), or data loss will occur.

---

## 9. Unknowns & Next Investigations

| ID | Unknown | Required Investigation | Priority |
| -- | ------- | ---------------------- | -------- |
| GUI-01 | MCU Command Interface | Is the MCU controlled exclusively via the FPGA (acting as a master proxy), or does it have a separate USB CDC port (`USBHandler.cpp`) that the GUI is ignoring? | High |
| GUI-02 | Calibration Routines | Does the GUI perform PA Bias or Beamforming calibration, or is that hardcoded in the MCU firmware? | Medium |
