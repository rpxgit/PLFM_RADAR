---
type: reverse-engineering-note
status: active
domain: mcu
confidence: mixed
canonical: true
---

# ADAR1000 Beamformer Configuration

> **Related notes:** [[05_Firmware_MCU]]
> **Status:** Read-only Forensic Snapshot

---

## 1. Overview

The ADAR1000 active antenna beamforming chips are responsible for electronic steering of the phased array. The STM32 controls an array of 4 ADAR1000 devices.

---

## 2. Initialization and Architecture

*   **Manager Class:** The firmware abstracts the entire array behind the C++ `ADAR1000Manager` class (`ADAR1000_Manager.cpp`).
*   **SPI Bus:** The devices are programmed via SPI.
*   **Initialization Loop:** `main.cpp` initializes the array in a sequential loop, checking for basic communication. If the loop fails, it triggers a fatal error and halts (`main.cpp:372`).

---

## 3. Beam Steering and Gain Control

*   **Vector Loading:** The firmware sets custom beam patterns using `adarManager.setCustomBeamPattern16()`. It explicitly differentiates between TX patterns and RX patterns (`ADAR1000Manager::BeamDirection::TX` vs `RX`).
*   **Automatic Gain Control (AGC):** An outer-loop AGC class (`ADAR1000_AGC.h`) runs on the STM32 to dynamically adjust the ADAR1000 Variable Gain Amplifiers (VGAs) based on signal strength.

---

## 4. Diagnostics and Fault Handling

The STM32 continuously monitors the ADAR1000 chips for health:

1.  **Temperature:** It reads the internal temperature sensor of each chip (`main.cpp:588`).
2.  **Errors:** If communication drops or temperature exceeds a threshold, the firmware generates specific error codes: `ERROR_ADAR1000_COMM` or `ERROR_ADAR1000_TEMP`.
3.  **Emergency Stop:** While a temperature fault might trigger a system shutdown to prevent damage, unit tests (`test_gap3_overtemp_emergency_stop.c`) verify that simple communication errors do *not* automatically trigger a hard emergency stop, allowing the system to attempt recovery.


## Related Notes

- [[05_Firmware_MCU]]
- [[03a_Main_Board]]
- [[03e_Antenna_Array]]
