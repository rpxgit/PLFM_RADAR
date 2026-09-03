# STM32 ↔ FPGA GPIO Interface

> **Related notes:** [[07_Communication_Protocols]] · [[04_Firmware_FPGA]] · [[05_Firmware_MCU]]
> **Status:** Read-only Forensic Snapshot

---

## 1. Interface Overview

Unlike typical Software-Defined Radios where the MCU streams data to/from the FPGA over a wide parallel bus (e.g., FMC) or high-speed SPI, the architecture of this radar platform isolates the MCU from the high-speed data plane. 

The primary host data path is **Host PC ↔ FT2232H (USB) ↔ FPGA**.
The MCU acts as a **supervisory control plane**, managing power sequencing, RF clock synthesis, and beamformer bias calibration.

The direct communication interface between the STM32 and the FPGA is limited to a few specific digital I/O lines used for synchronization, fault indication, or state signaling.

---

## 2. GPIO Signal Map

Based on `main.h` pin definitions, the following digital lines directly connect the STM32 and the Artix-7 FPGA:

| MCU Pin | FPGA Interface | Direction | Function | Active Level | Evidence / Confidence |
| ------- | -------------- | --------- | -------- | ------------ | --------------------- |
| `PD13` | `FPGA_DIG5_SAT`| FPGA → MCU | Saturation / Over-range indicator from the DSP pipeline. | High | Inferred from `SAT` suffix and standard radar DSP usage. (Confidence: Medium) |
| `PD14` | `FPGA_DIG6`    | Unknown   | General purpose signaling. | Unknown | `main.h` (Confidence: Low) |
| `PD15` | `FPGA_DIG7`    | Unknown   | General purpose signaling. | Unknown | `main.h` (Confidence: Low) |

### Power Sequencing Signals
While not strictly a communication protocol, the MCU controls the FPGA's power rails. These must be sequenced correctly before the FPGA will boot from its configuration memory.

| MCU Pin | Signal | Direction | Function |
| ------- | ------ | --------- | -------- |
| `PE7` | `EN_P_1V0_FPGA` | MCU → Power IC | Enables FPGA core logic rail (1.0V). |
| `PE8` | `EN_P_1V8_FPGA` | MCU → Power IC | Enables FPGA auxiliary rail (1.8V). |
| `PE9` | `EN_P_3V3_FPGA` | MCU → Power IC | Enables FPGA I/O bank rail (3.3V). |

---

## 3. Configuration & Boot Architecture

**Invariant:** The STM32 **does not** configure the FPGA dynamically (e.g., via SelectMAP or slave SPI). The absence of `PROGRAM_B`, `INIT_B`, `DONE`, and `CCLK` pins on the STM32 GPIO map strongly indicates the FPGA boots independently from a dedicated SPI Flash memory chip upon receiving power.

---

## 4. Unknowns / Required Investigation

| ID | Unknown | Impact | Next Steps |
| -- | ------- | ------ | ---------- |
| GPIO-01 | DIG6 & DIG7 Function | Two digital pins exist with unknown semantics. They may be used by the MCU to trigger a radar frame, or by the FPGA to signal frame completion (Interrupt). | Analyze `main.cpp` for GPIO EXTI interrupt handlers attached to `PD14` and `PD15`. |
| GPIO-02 | Clock Synchronization | Is there a clock signal shared directly between the MCU and FPGA? | Verify if `AD9523` provides independent clocks to both, eliminating the need for a direct MCU-FPGA clock line. |
