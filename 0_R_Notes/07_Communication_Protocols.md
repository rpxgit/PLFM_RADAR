---
type: reverse-engineering-note
status: active
domain: protocol
confidence: mixed
canonical: true
---

# System Communication Protocols

> **Related notes:** [[02_System_Architecture]] · [[07a_USB_Packet_Protocol]] · [[07b_SPI_Bus_Map]] · [[07c_I2C_Bus_Map]] · [[07d_STM32_FPGA_GPIO_Interface]]
> **Document type:** Master Communication Architecture Map
> **Status:** Read-only Forensic Snapshot

---

## 1. Protocol Architecture Summary

The PLFM-RADAR platform uses a split-plane communication architecture:
1.  **High-Speed Data Plane:** Host PC ↔ FT2232H (USB) ↔ FPGA. All DSP output is streamed directly to the host without MCU bottlenecking.
2.  **Supervisory Control Plane:** MCU ↔ RF / Clocking peripherals (via SPI/I²C). The MCU configures analog biases, synchronizes LO generation, and establishes power sequencing.

```mermaid
flowchart LR
    HOST["Host GUI (Python)"]
    USB["FTDI FT2232H (USB 2.0)"]
    MCU["STM32 (Microcontroller)"]
    FPGA["Artix-7 FPGA"]
    
    SPI1["SPI1"]
    SPI4["SPI4 (no_os)"]
    I2C1["I²C1"]
    I2C2["I²C2"]
    
    ADAR["ADAR1000 (Beamformers)"]
    SYNTH["ADF4382A (Synth)"]
    CLK["AD9523 (Clock)"]
    DAC["DAC5578 (PA Bias)"]
    ADC["ADS7830 (Telemetry)"]

    HOST <-->|Bulk Endpoint (FIFO)| USB
    USB <-->|245 FIFO parallel| FPGA
    
    MCU -.->|Power/Reset GPIO| FPGA
    MCU <-->|SPI1| SPI1
    MCU <-->|SPI4| SPI4
    MCU <-->|I2C1| I2C1
    MCU <-->|I2C2| I2C2
    
    SPI1 --> ADAR
    SPI4 --> SYNTH
    SPI4 --> CLK
    I2C1 --> DAC
    I2C2 --> ADC
```

---

## 2. Interface Inventory

| Interface ID | Source | Destination | Physical Layer | Protocol | Direction | Purpose | Evidence | Confidence |
| ------------ | ------ | ----------- | -------------- | -------- | --------- | ------- | -------- | ---------- |
| `INT-USB-01` | Host PC | FT2232H | USB 2.0 | FTDI Bulk | Bidirectional | High-speed data / commands | `radar_protocol.py` | Confirmed |
| `INT-FIFO-01`| FT2232H | FPGA | Parallel | 245 Synchronous | Bidirectional | Streaming radar bins | `radar_protocol.py` | Confirmed |
| `INT-SPI-01` | STM32 | ADAR1000 x4 | SPI (4-wire) | SPI Mode 0 | Bidirectional | Beamformer phase/gain config | `main.h`, `ADAR1000_Manager.cpp` | Confirmed |
| `INT-SPI-02` | STM32 | AD9523/ADF4382A | SPI (4-wire) | SPI Mode 0 (10MHz) | Bidirectional | Clock and LO synthesis | `main.cpp`, `adf4382a_manager.c` | Confirmed |
| `INT-I2C-01` | STM32 | DAC5578 x2 | I²C | 7-bit, 400kHz | Write-Only | PA gate voltage biasing | `main.cpp`, `DAC5578.H` | Confirmed |
| `INT-I2C-02` | STM32 | ADS7830 x3 | I²C | 7-bit, 400kHz | Bidirectional | Voltage/Current telemetry | `main.cpp`, `ADS7830.H` | Confirmed |
| `INT-GPIO-01`| STM32 | FPGA | Digital I/O | 3.3V CMOS | Bidirectional | Saturation signaling / Sync | `main.h` | High |

---

## 3. Sub-Protocol Summaries

*   **USB Packet Protocol ([[07a_USB_Packet_Protocol]]):** 
    Commands are 32-bit big-endian words `[Opcode, 0x00, ValHigh, ValLow]`. Responses are 11-byte data frames (`0xAA` header, `0x55` footer) or 26-byte status frames (`0xBB` header, `0x55` footer).
*   **SPI Bus Map ([[07b_SPI_Bus_Map]]):** 
    SPI1 is used for raw HAL block transfers to the ADAR1000s. SPI4 leverages Analog Devices `no_os` hardware abstraction to interface with the AD9523 clock generator and ADF4382A synthesizers.
*   **I²C Bus Map ([[07c_I2C_Bus_Map]]):** 
    I2C1 isolates the critical DAC5578 PA bias controllers (addresses 0x48/0x49). I2C2 handles telemetry polling (ADS7830 at 0x48/0x49/0x4A).
*   **STM32↔FPGA GPIO ([[07d_STM32_FPGA_GPIO_Interface]]):**
    A minimal interface. The STM32 provides power sequencing (`EN_P_1V0_FPGA`, etc.) and monitors the DSP saturation flag (`FPGA_DIG5_SAT`). 

---

## 4. Cross-Layer Command/Data Flow

### Example Trace: Changing CFAR Threshold

| User Operation | Host Function | Protocol | MCU Handler | FPGA/RF Action | Hardware Effect | Evidence |
| -------------- | ------------- | -------- | ----------- | -------------- | --------------- | -------- |
| User moves "Threshold" slider in UI | `_send_fpga_cmd()` | `build_command(0x03, val)` | N/A (Bypasses MCU) | FPGA receives 32b word over FTDI FIFO | `usb_cmd_opcode == 8'h03`, updates `cfar_threshold_reg` | `radar_protocol.py`, `dashboard.py` |

### Example Trace: Beamformer Steering (Hypothesized)

| User Operation | Host Function | Protocol | MCU Handler | FPGA/RF Action | Hardware Effect | Evidence |
| -------------- | ------------- | -------- | ----------- | -------------- | --------------- | -------- |
| TBD (Not in GUI) | TBD | TBD | `ADAR1000Manager` writes registers over SPI1 | N/A (Bypasses FPGA) | ADAR1000 shifts phase of RF array | `ADAR1000_Manager.cpp` |

---

## 5. Protocol Invariants

| Invariant | Evidence | Why It Must Be Preserved | Confidence |
| --------- | -------- | ------------------------ | ---------- |
| **USB Bulk Chunk Size** | `FT2232HConnection.open()` sets 64KB | The FPGA streams raw bins rapidly. Small chunk sizes will block the Python thread context-switcher, overflowing the FTDI internal buffer. | Confirmed |
| **FTDI FIFO Mode** | `set_bitmode(0xFF, SYNCFF)` | The FPGA expects a synchronous 60MHz clock from the FT2232H. Standard asynchronous mode will corrupt the byte stream. | Confirmed |
| **DAC HW Clear** | `DAC_1_VG_CLR` in `main.h` | Holds PA bias DACs at 0V during I²C init. If ignored, the PAs will boot fully open and burn out instantly. | Confirmed |
| **Opcode Mapping** | `Opcode(IntEnum)` | Must exactly match Verilog cases. | Confirmed |

---

## 6. Compatibility Requirements

### Host Compatibility
To replace the host software, a new application MUST:
1. Implement PyFtdi (or native D2XX) 245 Synchronous FIFO mode.
2. Replicate the 11-byte frame parsing state machine exactly.
3. Be capable of continuously consuming ~20-30 MB/s of bulk data without GC pauses stalling the USB pipeline.

### Firmware Compatibility
To replace the MCU firmware, a new implementation MUST:
1. Initialize the AD9523 before the ADF4382A.
2. Sequence the power rails correctly to boot the FPGA.
3. Trigger the SPI-based EZSync command to the ADF4382A synthesizers to achieve phase coherence.

---

## 7. Verification Evidence

| Test | Interface | What It Validates | Location | Status |
| ---- | --------- | ----------------- | -------- | ------ |
| `smoke_test.py` | USB/FPGA | Validates USB command `0x30` triggers FPGA BIST, and parses `0xFF` response. | `9_3_GUI/smoke_test.py` | Implemented |
| `test_v7.py` | Host Protocol | Unit tests the `RadarProtocol` header/footer framing. | `9_3_GUI/test_v7.py` | Implemented |
| `ADAR1000Manager::verify()`| SPI1 | Reads back scratchpad registers to confirm SPI health. | `ADAR1000_Manager.cpp` | Implemented |
| `adf4382_spi_read()` | SPI4 | Polls `0x58` lock status register to confirm LO lock. | `adf4382a_manager.c` | Implemented |

---

## 8. Unknowns & Next Investigations

| ID | Unknown | Interface | Evidence | Impact | Required Investigation | Priority |
| -- | ------- | --------- | -------- | ------ | ---------------------- | -------- |
| PROT-01 | MCU Native USB | USB | `USBHandler.cpp` exists | Medium | Does the MCU expose a separate CDC port for beamformer/synth tuning that the GUI currently ignores? | High |
| PROT-02 | FPGA Config Flow | SPI / GPIO | Lack of MCU bitbang config pins | High | How does the FPGA receive its bitstream? Is there an undetected SPI Flash? | High |
| PROT-03 | Missing Telemetry | I²C | `BMP180` and `GY_85` drivers exist in source tree. | Low | Are the sensors physically populated, and what I²C addresses do they occupy? | Low |

---
*Created in read-only forensic mode. Derived directly from repository implementation evidence.*


## Related Notes

- [[02_System_Architecture]]
- [[07a_USB_Packet_Protocol]]
- [[07b_SPI_Bus_Map]]
- [[07c_I2C_Bus_Map]]
- [[07d_STM32_FPGA_GPIO_Interface]]
