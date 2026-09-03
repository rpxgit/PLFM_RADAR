---
type: reverse-engineering-note
status: active
domain: mcu
confidence: mixed
canonical: true
---

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


## Related Notes

- [[05_Firmware_MCU]]
- [[03b_Power_Board]]


## 2. RF Front-End (ADTR1107) Bias Sequencing

The ADTR1107 T/R chips on the Main Board have a strict bias-up and bias-down procedure enforced by the MCU via DACs:

**Bias-Up:**
1. Connect all GND pins.
2. Set VDD_SW to 3.3V.
3. Set VSS_SW to -3.3V.
4. Set CTRL_SW to 3.3V.
5. Set VGG_PA to -1.75V (Negative Gate Bias).
6. Set VDD_PA to 0V.
7. Set VGG_LNA to 0V.
8. Set VDD_LNA to 3.3V.
9. Apply RF Signal.
10. Apply +5V to VDD_PA.

**Bias-Down:**
1. Turn off RF Signal.
2. Set VDD_LNA to 0V.
3. Set CTRL_SW to 0V.
4. Set VSS_SW to 0V.
5. Set VDD_SW to 0V.

## 3. GaN PA (QPA2962) Bias Sequencing

The Extended Variant PA boards follow the strict GaN depletion-mode rules:

**Bias-Up:**
1. Set ID limit to 2840 mA.
2. Set VG to -4.0V (Pinch-off).
3. Set VD to +22V.
4. Adjust VG more positive until IDQ = 1680 mA.
5. Apply RF signal.

**Bias-Down:**
1. Turn off RF Signal.
2. Reduce VG to -4.0V (IDQ ~ 0 mA).
3. Set VD to 0V.
4. Turn off VD supply.
5. Turn off VG supply.
