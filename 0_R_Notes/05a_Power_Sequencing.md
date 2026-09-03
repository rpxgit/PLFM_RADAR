# Power Sequencing

> **Related notes:** [[05_Firmware_MCU]]
> **Status:** Read-only Forensic Snapshot

---

## 1. Overview

The MCU firmware implements a strict, hardcoded power sequencing routine during the boot initialization phase in `main.cpp`. It ensures that sensitive components (like the AD9523 Clock Generator and the FPGA) are powered up in the correct voltage order before any configuration is sent to them over SPI.

---

## 2. AD9523 Power Sequence

The AD9523 provides the reference clocks for the entire system (FPGA, DAC, ADC, Synthesizers).

| Step | Action | GPIO Pin (Macro) | Target | Delay | Evidence |
| ---- | ------ | ---------------- | ------ | ----: | -------- |
| 1 | Assert Reset | `AD9523_RESET_Pin` (LOW) | AD9523 | None | `main.cpp:1433` |
| 2 | Enable 1.8V | `EN_P_1V8_CLOCK_Pin` (HIGH) | AD9523 Core | 100 ms | `main.cpp:1437` |
| 3 | Enable 3.3V | `EN_P_3V3_CLOCK_Pin` (HIGH) | AD9523 I/O | 100 ms | `main.cpp:1440` |
| 4 | Release Reset| `AD9523_RESET_Pin` (HIGH) | AD9523 | 100 ms | `main.cpp:1443` |

*The configuration function `configure_ad9523()` is only called after this sequence completes.*

---

## 3. FPGA Power Sequence

The FPGA requires its core, auxiliary, and I/O voltages to come up in a specific order to prevent latch-up and ensure correct configuration loading.

| Step | Action | GPIO Pin (Macro) | Target | Delay | Evidence |
| ---- | ------ | ---------------- | ------ | ----: | -------- |
| 1 | Enable 1.0V | `EN_P_1V0_FPGA_Pin` (HIGH) | FPGA Core (VCCINT) | 100 ms | `main.cpp:1477` |
| 2 | Enable 1.8V | `EN_P_1V8_FPGA_Pin` (HIGH) | FPGA Aux (VCCAUX) | 100 ms | `main.cpp:1480` |
| 3 | Enable 3.3V | `EN_P_3V3_FPGA_Pin` (HIGH) | FPGA I/O (VCCO) | 100 ms | `main.cpp:1483` |

---

## 4. Exceptions and Unknowns

*   **RF and PA Power:** The initialization sequence in `main.cpp` for the RF frontend (ADAR1000, ADF4382, PA Bias) does not explicitly show GPIO power enables in the same block as the Clock and FPGA. They might be enabled implicitly via their respective manager classes or later in the sequence.
*   **OCXO Warmup:** Before any power sequencing occurs, the MCU waits 180 seconds (3 minutes) for the Oven-Controlled Crystal Oscillator (OCXO) to stabilize, feeding the watchdog timer (`HAL_IWDG_Refresh`) continuously during this period.
