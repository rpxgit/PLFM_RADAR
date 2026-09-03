# SPI Bus Map

> **Related notes:** [[07_Communication_Protocols]] · [[03_Hardware]] · [[05_Firmware_MCU]]
> **Status:** Read-only Forensic Snapshot

---

## 1. SPI Architecture Overview

The system utilizes two primary SPI buses originating from the STM32 microcontroller to configure the RF and clocking peripherals. The SPI buses strictly use software-controlled Chip Select (CS) GPIOs to address multiple devices on the same bus.

---

## 2. SPI Instance Map

### SPI1: Beamformer Bus
*   **Controller:** STM32 (`hspi1`)
*   **Mode:** Master, Mode 0 (CPOL=0, CPHA=0)
*   **Data Width:** 8-bit
*   **Speed / Timings:** 1000ms timeout in `HAL_SPI_TransmitReceive`.
*   **Driver:** Direct STM32 HAL (`HAL_SPI_Transmit`, `HAL_SPI_TransmitReceive`) used within `ADAR1000_Manager.cpp`.

| Device | Purpose | CS GPIO | Pins |
| ------ | ------- | ------- | ---- |
| ADAR1000 (1) | TX/RX Beamformer | `PA0` (ADAR_1_CS_3V3_Pin) | SCK/MOSI/MISO standard to SPI1 |
| ADAR1000 (2) | TX/RX Beamformer | `PA1` (ADAR_2_CS_3V3_Pin) | SCK/MOSI/MISO standard to SPI1 |
| ADAR1000 (3) | TX/RX Beamformer | `PA2` (ADAR_3_CS_3V3_Pin) | SCK/MOSI/MISO standard to SPI1 |
| ADAR1000 (4) | TX/RX Beamformer | `PA3` (ADAR_4_CS_3V3_Pin) | SCK/MOSI/MISO standard to SPI1 |

### SPI4: Clock & Synthesizer Bus
*   **Controller:** STM32 (`hspi4`), abstracted via Analog Devices `no_os` SPI layer (`platform_noos_stm32.c`).
*   **Mode:** Master, Mode 0 (CPOL=0, CPHA=0)
*   **Data Width:** 8-bit, MSB First
*   **Speed:** 10 MHz (10000000 Hz) defined in `init_param.spi_init.max_speed_hz`.
*   **Driver:** ADI `no_os_spi` (wraps STM32 HAL).

| Device | Purpose | CS GPIO | Handshake/Feedback |
| ------ | ------- | ------- | ------------------ |
| AD9523 | Master Clock Generator | `PF7` (AD9523_CS_Pin) | STATUS0 (`PF8`), STATUS1 (`PF9`), SYNC (`PF5`) |
| ADF4382A (TX) | 10.5 GHz LO Generator | `PG14` (ADF4382_TX_CS_Pin) | LKDET (`PG11`) |
| ADF4382A (RX) | 10.38 GHz LO Generator | `PG10` (ADF4382_RX_CS_Pin) | LKDET (`PG6`) |

---

## 3. Transaction Analysis & Invariants

### 3.1. ADAR1000 Transactions (SPI1)
*   **Transaction Format:** 24-bit instruction (R/W bit, 15-bit address, 8-bit data).
*   **Implementation:** The MCU driver asserts the software CS (low), calls `HAL_SPI_TransmitReceive()` with a 3-byte array, and de-asserts CS (high).
*   **Invariant:** The ADAR1000 driver uses a manual GPIO assertion sequence, bypassing hardware NSS.

### 3.2. ADF4382A / AD9523 Transactions (SPI4)
*   **Transaction Format:** Managed internally by the ADI `no_os` library structure. The MCU sets up a `no_os_spi_init_param` passing `&hspi4` into the `extra` field, allowing the cross-platform ADI drivers to execute native SPI frames.
*   **Invariant:** EZSync synchronization is triggered via SPI on the ADF4382A devices to achieve exact phase alignment across multiple LO generators.

---

## 4. Unknowns / Required Investigation

| ID | Unknown | Impact | Next Steps |
| -- | ------- | ------ | ---------- |
| SPI-01 | FPGA SPI Configuration | The FPGA might be configured via an SPI interface (or parallel), but it is not evident on SPI1 or SPI4 in the current MCU mapping. Is the FPGA bitstream loaded via SPI flash or MCU GPIO? | Map MCU pins to FPGA config pins (`PROGRAM_B`, `DONE`, `INIT_B`). |
