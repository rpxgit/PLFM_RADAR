# PA Bias and Calibration

> **Related notes:** [[05_Firmware_MCU]]
> **Status:** Read-only Forensic Snapshot

---

## 1. Overview

Because the radar utilizes GaN Power Amplifiers (PAs), the bias voltage ($V_g$) must be carefully sequenced and calibrated to hit a specific quiescent current ($I_{dq}$) before RF drive is applied. This prevents catastrophic thermal runaway in the GaN devices.

The STM32 implements a closed-loop bias controller using I2C DACs (for setting voltage) and ADCs (for reading current).

---

## 2. Hardware Interfaces

| Role | Device | Interface | Address | Evidence |
| ---- | ------ | --------- | ------- | -------- |
| Gate Voltage ($V_g$) Control | DAC5578 (x2) | I2C | 0x48, 0x49 | `main.cpp:1845`, `DAC5578.h` |
| Quiescent Current ($I_{dq}$) Monitor | ADS7830 | I2C | - | `main.cpp`, `ADS7830.h` |

---

## 3. DAC Configuration & Safety

The DAC5578 is configured with strict safety defaults to protect the GaN PAs:

*   **Initialization:** `DAC5578_Init()` connects to two 8-channel DACs.
*   **Clear Code:** The firmware configures `DAC5578_SetClearCode(..., DAC5578_CLR_CODE_ZERO)` (`main.cpp:1865`). This ensures that if the hardware CLR pin is asserted, all DAC outputs immediately snap to 0V (which for a negative-bias GaN setup usually implies maximum pinch-off, though physical polarity inversion op-amps may be present on the PCB).
*   **Emergency Stop:** The MCU has direct GPIO control over the DAC CLR pins (`DAC5578_ActivateClearPin`). If an emergency stop is triggered (e.g. over-temperature or FPGA fault), the STM32 fires this pin to instantly kill the PA bias without relying on slow I2C transactions (`main.cpp:818`).
*   **LDAC Sync:** The firmware uses `DAC5578_SetupLDAC(..., 0xFF)` so that I2C writes can be synchronized across all channels simultaneously.

---

## 4. Calibration Loop

While the exact numeric algorithm requires deeper extraction, the structural loop is evident:
1.  The STM32 sets a target $V_g$ via the DAC5578 (`DAC5578_WriteAndUpdateChannelValue`).
2.  It reads the resulting $I_{dq}$ via the ADS7830.
3.  It iterates until the target current is achieved for each channel independently.
