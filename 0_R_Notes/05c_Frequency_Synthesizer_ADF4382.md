---
type: reverse-engineering-note
status: active
domain: mcu
confidence: mixed
canonical: true
---

# ADF4382 Frequency Synthesizer Configuration

> **Related notes:** [[05_Firmware_MCU]] · [[05b_Clock_Generator_AD9523]]
> **Status:** Read-only Forensic Snapshot

---

## 1. Overview

The radar utilizes the Analog Devices ADF4382 as its primary frequency synthesizer for generating the FMCW chirp waveform. Two devices exist (TX and RX synthesizers), both fed by phase-aligned 300 MHz reference clocks and 60 MHz sync clocks from the AD9523.

---

## 2. Firmware Control

*   **Libraries:** The firmware relies on `adf4382.c` (base Analog Devices driver) wrapped by `adf4382a_manager.c` to abstract the radar-specific chirp profiles.
*   **Interface:** SPI.
*   **Initialization:** Controlled from `main.cpp` via `ADF4382Manager`.

## 3. Clock References

From the AD9523 configuration:

| Input | Frequency | Source | Format |
| ----- | --------- | ------ | ------ |
| REFIN | 300 MHz | AD9523 (CH0/1) | LVDS |
| SYNC | 60 MHz | AD9523 (CH8/9) | LVDS |

## 4. Initialization & Runtime Behavior

1.  **PLL Initialization:** The STM32 initializes the ADF4382 devices over SPI. It must wait for the AD9523 clocks to become stable before programming the PLL registers.
2.  **Runtime Modulation (Corrected Hypothesis):** Initially, it was hypothesized that the ADF4382's internal ramp generator generated the FMCW chirp. **This is incorrect.** Evidence in `radar_transmitter.v` and `plfm_chirp_controller.v` confirms that the chirp is generated digitally in the FPGA (using PLFM) and converted to an analog IF via the AD9708 DAC. 
3. **True Role:** The ADF4382 acts exclusively as a fixed-frequency Local Oscillator (CW LO) for the LTC5552 up-mixers and down-mixers, stepping up the modulated IF chirp to the final X-Band frequency.

*Note: The exact center frequency and fractional-N parameters are passed from the host PC over USB, parsed by the MCU firmware, and written to the ADF4382 registers to set the operating band.*


## Related Notes

- [[05_Firmware_MCU]]
- [[03c_Frequency_Synthesizer_Board]]
