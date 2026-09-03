---
type: reverse-engineering-note
status: active
domain: hardware
confidence: mixed
canonical: true
---

# Main Board

> **Related notes:** [[03_Hardware]] · [[01_Component_Inventory_and_Sourcing]]
> **Status:** Read-only Forensic Snapshot

---

## 1. Board Identity

*   **PCB Name:** `RADAR_Main_Board`
*   **Domain:** Digital Control, FPGA DSP, and RF Transceiver
*   **Layer Count:** 10 layers (Inferred from `RADAR_Main_Board.L1` to `.L10` gerbers)
*   **Stack-up:** Hybrid RO4350B (Confirmed by `Stack_Hybrid.png` and `PCBWay_Impedance_Note_RO4350B_h0p102mm.pdf`)
*   **Function:** Serves as the central hub of the radar system, orchestrating power, processing radar signals, generating IF chirps, and up/down converting RF signals.

---

## 2. Schematic Analysis

| Block | Components | Inputs | Outputs | Interface | Function | Evidence | Confidence |
| ----- | ---------- | ------ | ------- | --------- | -------- | -------- | ---------- |
| **System Controller** | STM32F746 | Host USB, Sensors | SPI/I2C/GPIO | USB, SPI, I2C | Power sequencing, peripheral config | `RADAR_Main_Board.sch`, `main.cpp` | Confirmed |
| **DSP Engine** | XC7A50T-FTG256 | ADC, MCU GPIO | DAC, USB FIFO | LVDS, Parallel | Radar signal processing | `RADAR_Main_Board.sch`, `radar_system_top.v` | Confirmed |
| **Waveform Gen** | AD9708 (DAC) | FPGA Parallel | IF Output | CMOS Parallel | IF Chirp generation | `RADAR_Main_Board.sch` | High |
| **Data Capture** | AD9484 (ADC) | IF Input | FPGA LVDS | 400MHz LVDS | IF Digitization | `RADAR_Main_Board.sch` | High |
| **Mixer Stage** | LTC5552 (x2) | DAC IF, LO | ADC IF, RF Out | RF | Up and Down conversion | `README.md` | Confirmed |
| **Beamforming** | ADAR1000 (x4) | LO / RF | RF Elements | SPI, RF | 16-channel phase/gain control | `RADAR_Main_Board.sch` | Confirmed |
| **RF Front-End** | ADTR1107 (x16) | ADAR1000 RF | Antenna RF | RF | LNA / Low-power PA | `README.md` | Confirmed |
| **PA Calibration** | DAC5578 (x2), ADS7830 (x2), INA241A3 | MCU I2C | Vg Control, Idq | I2C, Analog | Closed-loop GaN gate bias | `README.md`, `main.cpp` | Confirmed |

---

## 3. Component Roles

*   **STM32F746:** Acts as the master state machine. It does not touch high-speed radar data.
*   **XC7A50T:** Pure data processor. Triggers off MCU signals and pushes data to the host.
*   **FT2232H / FT601:** Handles USB bridging. FT2232H provides USB 2.0 (50T board), FT601 provides USB 3.0 (200T premium board).
*   **LTC5552:** Mixes the low-frequency DAC output (IF) with the ADF4382 LO to reach 10.5 GHz (X-band).
*   **ADAR1000:** Splits the single TX/RX RF chain into 16 independent elements with controllable phase shifts for electronic beam steering.

---

## 4. Hardware ↔ Firmware Correlation

| Hardware Feature | Firmware Evidence | Expected Function | Confidence |
| ---------------- | ----------------- | ----------------- | ---------- |
| **I2C DACs (0x48, 0x49)** | `README.md` / `main.cpp` | Set Vg for 16 PA channels | Confirmed |
| **I2C ADCs (0x48, 0x4A)** | `README.md` / `main.cpp` | Read Idq for 16 PA channels | Confirmed |
| **Thermal ADC (0x4A?)** | `README.md` | Read 8 thermistors | Confirmed |
| **SPI Buses** | `ADAR1000_Manager.cpp` | Configure ADAR1000 | Confirmed |
| **GPIO Toggle** | `radar_system_top.v` (`stm32_new_chirp`) | Trigger radar pulse | Confirmed |
| **USB CDC** | `radar_protocol.py` | Host telemetry and control | Confirmed |

---

## 5. Replication-Critical Features

*   **PCB Stack-up:** The 10-layer RO4350B stack-up with 100µm core is mandatory for maintaining impedance across the 10.5 GHz RF traces. Substitution will destroy RF performance.
*   **ADC LVDS Routing:** The 400MHz differential pairs between AD9484 and XC7A50T require exact length matching.
*   **PA Bias Loop:** The 5mΩ shunt resistors and 50x INA241A3 amplifiers must be identical, as the MCU calibration math relies on these fixed hardware ratios.

---

## 6. Unknowns & Next Steps

| ID | Unknown | Why It Matters | Evidence Available | Required Investigation |
| -- | ------- | -------------- | ------------------ | ---------------------- |
| MB-01 | FPGA Configuration | How does the FPGA load its bitstream (Flash SPI vs MCU)? | Schematics | Parse `.sch` for flash ICs. |
| MB-02 | USB Hubbing | Does the board have an onboard USB hub, or two separate USB ports? | `README.md` | Parse `.sch` for USB connectors/hubs. |


## Related Notes

- [[03_Hardware]]
- [[04_Firmware_FPGA]]
- [[05_Firmware_MCU]]
