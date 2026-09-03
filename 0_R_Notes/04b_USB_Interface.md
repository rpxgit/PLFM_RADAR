---
type: reverse-engineering-note
status: active
domain: fpga
confidence: mixed
canonical: true
---

# FPGA USB Interface

> **Related notes:** [[04_Firmware_FPGA]] · [[06_Software_GUI]]
> **Status:** Read-only Forensic Snapshot

---

## 1. USB Architecture Topology

The AERIS-10 system uses a direct FPGA-to-Host USB architecture for high-speed radar data, bypassing the MCU entirely. The MCU maintains a separate, low-speed USB CDC connection.

```text
  [FPGA DSP Pipeline]
           │
           ▼
[usb_data_interface_ft2232h.v] ──► [FT2232H IC] ──► USB 2.0 ──► [Host PC GUI]
```

---

## 2. Interface Variants

The repository contains RTL for two different USB targets, selected via a `USB_MODE` parameter.

| Parameter | Controller | Speed | Bus Width | Clock | Physical Board | Evidence |
| --------- | ---------- | ----- | --------- | ----- | -------------- | -------- |
| `USB_MODE=0` | FT601 | USB 3.0 | 32-bit | 100 MHz | 200T Premium | `usb_data_interface.v` |
| `USB_MODE=1` | FT2232H | USB 2.0 | 8-bit | 60 MHz | 50T Production | `usb_data_interface_ft2232h.v` |

**Confirmed Implementation:** The 50T constraint files and wrapper explicitly use `USB_MODE=1` and the FT2232H controller.

---

## 3. FPGA Implementation Details

### Data Flow
1.  **Input:** The USB interface accepts structured packets from the `radar_receiver_final` (range profiles, doppler bins, and targets).
2.  **Buffering:** Data is written into an asynchronous FIFO (`clk_100m` write domain).
3.  **FTDI State Machine:** The read side (`60 MHz` domain) monitors the FT2232H `TXE#` (Transmit Enable / FIFO not full) flag.
4.  **Output:** When `TXE#` is low, the FPGA drives 8-bit data onto the FT2232H bus and strobes `WR#`.

### Host Contract
The FPGA simply streams raw bytes. It is entirely up to the Python GUI to find the frame headers and unpack the CFAR targets and Range/Doppler matrices. The FPGA does not implement a USB MAC/PHY; it relies on the FTDI chip's "245 Synchronous FIFO" mode.

---

## 4. Replication Impact

The FT2232H in Synchronous FIFO mode theoretically peaks at ~40 MB/s. Given that raw ADC data is 400 MSPS (800 MB/s), **the FPGA cannot stream raw radar data over USB 2.0 in real-time**. 
Therefore, the FPGA's internal DSP pipeline (DDC, Matched Filter, CFAR) is strictly required to decimate the data down to a bandwidth the FT2232H can handle. Raw data taps (`dbg_adc_i/q`) exist in RTL but can likely only be captured via JTAG ILA or drastically slowed down sweeps.


## Related Notes

- [[04_Firmware_FPGA]]
- [[07a_USB_Packet_Protocol]]
- [[06_Software_GUI]]
