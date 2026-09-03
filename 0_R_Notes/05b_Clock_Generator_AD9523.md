# AD9523 Clock Generator Configuration

> **Related notes:** [[05_Firmware_MCU]] · [[04c_Clock_Domains]]
> **Status:** Read-only Forensic Snapshot

---

## 1. Overview

The Analog Devices AD9523 provides the master clock distribution for the radar system. The STM32 configures it via SPI using the Analog Devices `no_os` driver stack (`ad9523.c`). Configuration occurs strictly after the OCXO warmup (3 minutes) and the dedicated clock power sequencing.

---

## 2. Clock Output Mapping

According to the firmware source (`main.cpp:1448`), the AD9523 is configured to output the following frequencies. These form the critical clock domains for the system.

| Channel | Frequency | Format | Destination | Phase Alignment | Evidence |
| ------- | --------- | ------ | ----------- | --------------- | -------- |
| **0** | 300 MHz | LVDS | ADF4382 TX Synthesizer Reference | Master | `main.cpp` |
| **1** | 300 MHz | LVDS | ADF4382 RX Synthesizer Reference | Aligned with CH 0 | `main.cpp` |
| **4** | 400 MHz | LVDS | AD9484 ADC Clock | Master | `main.cpp` |
| **5** | 400 MHz | LVDS | FPGA ADC Interface | Aligned with CH 4 | `main.cpp` |
| **6** | 100 MHz | LVCMOS | FPGA System / DSP Clock | - | `main.cpp` |
| **7** | 20 MHz | LVCMOS | FPGA Test Clock | - | `main.cpp` |
| **8** | 60 MHz | LVDS | ADF4382 TX Sync | Master | `main.cpp` |
| **9** | 60 MHz | LVDS | ADF4382 RX Sync | Aligned with CH 8 | `main.cpp` |
| **10** | 120 MHz | LVCMOS | FPGA DAC Interface | Master | `main.cpp` |
| **11** | 120 MHz | LVCMOS | Unused/Aux | Aligned with CH 10 | `main.cpp` |

---

## 3. Initialization Details

1.  **Driver Initialization:** Handled by `configure_ad9523()` which wraps the `ad9523_setup()` function from the `no_os` library.
2.  **Failure Handling:** If the AD9523 fails to initialize or lock its PLL to the OCXO reference, the STM32 enters an infinite halt loop (`main.cpp:1468`). The system cannot operate without a locked clock tree.
3.  **Phase Alignment:** Channels are explicitly grouped into phase-aligned pairs (e.g., TX and RX synthesizers receive identical, phase-matched 300 MHz and 60 MHz references). This is critical for coherent FMCW radar operation.
