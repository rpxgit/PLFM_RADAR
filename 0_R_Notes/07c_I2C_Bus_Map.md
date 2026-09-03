---
type: reverse-engineering-note
status: active
domain: protocol
confidence: mixed
canonical: true
---

# I²C Bus Map

> **Related notes:** [[07_Communication_Protocols]] · [[03_Hardware]] · [[05_Firmware_MCU]]
> **Status:** Read-only Forensic Snapshot

---

## 1. I²C Architecture Overview

The system uses multiple I²C buses on the STM32 to monitor power sensors (voltages/currents), bias the RF power amplifiers, and interface with environmental sensors.

All STM32 I²C buses are initialized with `I2C_ADDRESSINGMODE_7BIT` and use a timing configuration of `0x00808CD2`.

---

## 2. I²C Instance Map

### I2C1: PA Bias DACs
*   **Controller:** STM32 (`hi2c1`)
*   **Purpose:** Controlling the RF Power Amplifier (PA) gate voltages via DACs.
*   **Devices:**
    *   **DAC5578 (1):** Address `0x48`. (PA Bias Bank 1)
    *   **DAC5578 (2):** Address `0x49`. (PA Bias Bank 2)
*   **Hardware Interlocks:** Independent of I²C, the DACs have a hardware `CLR` pin (`PB4`, `PB8`) acting as a fast emergency RF shutdown.

### I2C2: System Telemetry & Power Monitoring
*   **Controller:** STM32 (`hi2c2`)
*   **Purpose:** Reading power supply voltages, currents, and temperature telemetry.
*   **Devices:**
    *   **ADS7830 (1):** Address `0x48`.
    *   **ADS7830 (2):** Address `0x4A`.
    *   **ADS7830 (3):** Address `0x49`.
*   **Transaction:** 8-bit single-ended ADC reads.

### I2C3: Environmental Sensors (Inferred)
*   **Controller:** STM32 (`hi2c3`)
*   **Purpose:** Interfacing with environmental and positioning sensors.
*   **Devices (Deduced from driver files in repo):**
    *   **BMP180:** Barometric pressure / Temperature.
    *   **GY-85:** 9-DOF IMU (Accelerometer, Gyroscope, Magnetometer). Hardware interrupt pins mapped in `main.h` (`MAG_DRDY` on `PC6`, `ACC_INT` on `PC7`, `GYR_INT` on `PC8`).

---

## 3. Transaction Analysis & Invariants

### 3.1. DAC5578 (PA Bias)
*   **Transaction Format:** The MCU writes to specific DAC channels (1 through 8) to set negative gate bias voltages. 
*   **Protocol:** Standard I²C write sequence consisting of the 7-bit device address, a command byte (specifying channel and operation), followed by two data bytes (12-bit or 8-bit DAC value padded).
*   **Invariant:** The DAC outputs are held at 0V by the hardware `CLR` line during boot sequence, guaranteeing no RF emission until the I²C registers are safely initialized.

### 3.2. ADS7830 (Telemetry)
*   **Transaction Format:** The MCU writes a command byte to select the channel (0-7) and mode (single-ended), then reads back a single 8-bit data byte representing the ADC conversion result.

---

## 4. Unknowns / Required Investigation

| ID | Unknown | Impact | Next Steps |
| -- | ------- | ------ | ---------- |
| I2C-01 | Full Sensor Address Map | The explicit I²C addresses for the GY-85 IMU and BMP180 are abstracted inside the driver files and not evident in `main.cpp`. | Search `BMP180.cpp` and `GY_85_HAL.c` for hardcoded 7-bit addresses. |


## Related Notes

- [[07_Communication_Protocols]]
- [[05e_PA_Bias_Calibration]]
