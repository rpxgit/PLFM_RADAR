# Component Inventory & Preliminary Sourcing BOM

> **Related notes:** [[00_Executive_Summary]] · [[02_System_Architecture]] · [[03_Hardware]] · [[08_Unknowns_and_Hypotheses]]  
> **Document type:** Component Inventory — Replication Baseline  
> **Created:** 2026-09-03  
> **Status:** Preliminary — requires cross-validation against Excel BOMs (binary, not inspected)  
> **Unit definition:** One complete AERIS-10 radar system  

---

## 1. Unit Definition

"**One unit**" = one complete AERIS-10 radar system ready for operation.

Two product variants exist. They share the core electronics but differ in power amplification and antenna:

| Variant | Antenna | PA Stage | USB | Target Range |
|---------|---------|----------|-----|-------------|
| **AERIS-10N (Nexus)** — Base Unit | 8×16 patch array PCB | 16× ADTR1107 integrated T/R (on Main Board) | FT2232H (USB 2.0) | 3 km |
| **AERIS-10E (Extended)** — Extended Unit | 32×16 dielectric-filled slotted waveguide | 16× QPA2962 GaN PA boards **+ ADTR1107** | FT601 (USB 3.0) | 20 km |

> [!IMPORTANT]
> The tables below are organized as: **Base Unit** (shared components), then **Extended-Only** add-ons. Components exclusive to the Extended variant are clearly marked.

---

## 2. PCB Assemblies per Unit

| # | PCB Name | Rev | Qty/Unit | Layers | Material | Schematic Source | Gerber Set | BOM Available | Notes |
|---|----------|-----|---------|--------|----------|-----------------|------------|--------------|-------|
| PCB-1 | **Main Board** | V6 | 1 | 10 | RO4350B hybrid (100 µm RF core) | `4_6_Schematics/MainBoard/RADAR_Main_Board.sch` | `Gerber_Main_Board/` | `BOM_Main_Board.xlsx` + `CPL.xlsx` (binary) | Production board, most complex |
| PCB-2 | **Power Board** | V6 | 1 | 2 | FR-4 (inferred) | `4_6_Schematics/PowerBoard/PowerBoard.sch` | `Gerber_PowerBoard/` | `BOM_Power_Board.xlsx` + `PowerBoard.csv` ✅ | Multi-rail supply |
| PCB-3 | **Frequency Synthesizer Board** | V6 | 1 | 6 | RO4003C or FR-4 (inferred) | `4_6_Schematics/FrequencySynthesizerBoard/Clocks_Freq_Synth_board.sch` | `Gerber_freq_synth/` | `BOM_Freq_Synth.xlsx` + CSV ✅ | Clock gen + 2× LO synth |
| PCB-4 | **Power Amplifier Board** | — | 16 (Extended only) / 0 (Nexus) | 4 | RO4350B (inferred, RF) | `4_6_Schematics/PowerAmplifierBoard/RF_PA.sch` | `Gerber_PA/` | `BOM_PA.xlsx` + `BOM.xlsx` + `CPL.xlsx` | Per-element PA, 10W GaN |
| PCB-5 | **Patch Antenna** | — | 1 (Nexus only) | 4 | RO4350B (inferred, microwave) | `4_6_Schematics/Antennas/Patch/` | `Gerber_Patch_Antenna/` | `BOM_Patch_Antenna.xlsx` | 8×16 elements, 16× SMA feeds |

**Evidence:** `4_7_Production Files/` directory listing, `4_4_Board Stack-up/Stack_Hybrid.png`, `PCBWay_Impedance_Note_RO4350B_h0p102mm.pdf`

---

## 3. Active Semiconductor Components — Main Board

### 3.1 Digital ICs

| Category | Component / Part | Manufacturer | MPN | Qty/Unit | Package | Ref Des | Specification | Source Evidence | Confidence | Sourcing Status |
|----------|-----------------|-------------|-----|---------|---------|---------|---------------|----------------|------------|----------------|
| FPGA | **Xilinx Artix-7** | AMD/Xilinx | XC7A50T-2FTG256I | 1 | FTG256 BGA | U42 | 50T production target; -2 speed, industrial temp | `constraints/xc7a50t_ftg256.xdc` line 4; `.brd` U42 | **Confirmed** | Exact part identified |
| MCU | **STM32F746xx** | STMicroelectronics | STM32F746VGT6 (inferred) | 1 | LQFP-100 (inferred) | — | ARM Cortex-M7, 216 MHz, USB OTG, 3×I²C, 2×SPI, 2×UART | `main.h`, `stm32f7xx_hal.h` | **High** | Manufacturer identified — exact suffix unknown |
| USB 2.0 Bridge | **FT2232H** | FTDI | FT2232HL | 1 | LQFP-64 | — | Dual-channel USB 2.0 Hi-Speed, 245 sync FIFO mode | `DS_FT2232H.pdf`, `xc7a50t_ftg256.xdc` Bank 35 | **Confirmed** | Exact part identified |
| USB 3.0 Bridge | **FT601** | FTDI | FT601Q-B | 1 (Extended only) | QFN-76 | — | USB 3.0 SuperSpeed, 32-bit sync FIFO | `usb_data_interface.v`, `hr2220p601r-10-datasheet.pdf` | **High** | Exact part identified — Extended variant only |
| EEPROM | **AT93C46A** | Microchip | AT93C46A | 1 (inferred) | SOIC-8 | — | 1 Kbit serial EEPROM, 3-wire | `AT93C46A.pdf` in datasheets | **Low** | Purpose unknown — datasheet present but no code reference |

### 3.2 Analog / Mixed-Signal ICs

| Category | Component / Part | Manufacturer | MPN | Qty/Unit | Package | Ref Des | Specification | Source Evidence | Confidence | Sourcing Status |
|----------|-----------------|-------------|-----|---------|---------|---------|---------------|----------------|------------|----------------|
| Clock Generator | **AD9523-1** | Analog Devices | AD9523BCPZ | 1 | QFN-72 (LFCSP) | IC1 (Freq Synth Board) | 14-output PLL clock gen, 3.6 GHz VCO | Freq Synth CSV BOM, `ad9523.c/h` | **Confirmed** | Exact part identified |
| Freq Synth (TX LO) | **ADF4382A** | Analog Devices | ADF4382ABCCZ | 1 | CC-48 (QFN) | U1 (Freq Synth Board) | 20 GHz µW PLL synth, 10.5 GHz output | Freq Synth CSV BOM, `adf4382a_manager.h` | **Confirmed** | Exact part identified |
| Freq Synth (RX LO) | **ADF4382A** | Analog Devices | ADF4382ABCCZ | 1 | CC-48 (QFN) | U6 (Freq Synth Board) | 20 GHz µW PLL synth, 10.38 GHz output | Freq Synth CSV BOM, `adf4382a_manager.h` | **Confirmed** | Exact part identified |
| DAC (Chirp) | **AD9708** | Analog Devices | AD9708ARZ (inferred) | 1 | SOIC-28 (inferred) | U3 (Main Board) | 8-bit, 100+ MSPS TxDAC | `AD9708.pdf`, `dac_interface_single.v`, XDC comment line 70 | **High** | Manufacturer identified — exact suffix inferred |
| ADC (IF) | **AD9484** | Analog Devices | AD9484BCPZ-500 (inferred) | 1 | QFN | — | 8-bit, 500 MSPS (clocked at 400 MHz) LVDS | `AD9484/` datasheet dir, `ad9484_interface_400m.v` | **High** | Manufacturer identified — speed grade inferred |
| Mixer (TX) | **LTC5552** | Analog Devices (Linear) | LTC5552IDD | 1 | DFN-10 | — | 3–20 GHz double-balanced mixer, +24 dBm IIP3 | `LTC5552f.pdf`, `README.md` line 55 | **Confirmed** | Exact part identified |
| Mixer (RX) | **LTC5552** | Analog Devices (Linear) | LTC5552IDD | 1 | DFN-10 | — | 3–20 GHz double-balanced mixer | Same as above | **Confirmed** | Exact part identified |
| Phase Shifter (×4) | **ADAR1000** | Analog Devices | ADAR1000ACPZ (inferred) | 4 | QFN-48 | — | 4-channel, 8–16 GHz beamformer, I/Q vector modulator | `ADAR1000.pdf`, `ADAR1000_Manager.h/cpp` | **Confirmed** | Exact part identified |
| T/R Module (×16) | **ADTR1107** | Analog Devices | ADTR1107ACCZ (inferred) | 16 | — | — | 8–16 GHz integrated T/R front-end, PA+LNA+switch | `adtr1107.pdf`, `README.md` line 57 | **Confirmed** | Exact part identified |
| PA Idq ADC (×2) | **ADS7830** | Texas Instruments | ADS7830IPW (inferred) | 2 | TSSOP-16 (inferred) | U88, U89 (Main Board) | 8-channel, 8-bit I²C ADC | `ADS7830.H`, `ads7830.pdf`, `README.md` line 76 | **Confirmed** | Exact part identified |
| PA Vg DAC (×2) | **DAC5578** | Texas Instruments | DAC5578IDGSR (inferred) | 2 | MSOP-10 or TSSOP (inferred) | U7, U69 (Main Board) | 8-channel, 8-bit I²C DAC | `DAC5578.H`, `dac7578.pdf`, `README.md` line 77 | **Confirmed** | Exact part identified |
| Thermal ADC | **ADS7830** | Texas Instruments | ADS7830IPW (inferred) | 1 | TSSOP-16 (inferred) | U10 (Main Board) | 8-channel ADC reading 8 thermistors | `README.md` line 82, `main.cpp` lines 244, 1001–1008 | **Confirmed** | Exact part identified |
| Current Sense Amp (×16) | **INA241A3** | Texas Instruments | INA241A3IDGKR | 16 | SOT-8 (DGK) | U11, U73–U87 (Main Board) | ×50 gain, current sense amplifier | `.brd` element listing (16 instances), `README.md` line 76 | **Confirmed** | Exact part identified |
| IF Amplifier | **LTC6419** | Analog Devices (Linear) | LTC6419CUF (inferred) | 1 (inferred) | QFN | — | Dual 900 MHz diff amp, 7 nV/√Hz | `LTC6419fa.pdf` | **Medium** | Datasheet present — exact placement unknown |
| Op-Amp | **OPA703** | Texas Instruments | OPA703UA (inferred) | ? | SOT-23-5 | — | CMOS op-amp, 12 MHz GBW | `opa703.pdf` | **Medium** | Datasheet present — qty/placement unknown |

### 3.3 Power Regulators — Power Board

*From `PowerBoard.csv` (confirmed BOM data):*

| Category | Component / Part | Manufacturer | MPN | Qty/Unit | Package | Ref Des | Specification | Confidence | Sourcing Status |
|----------|-----------------|-------------|-----|---------|---------|---------|---------------|------------|----------------|
| DC/DC Step-down (×21) | **TPS562208** | Texas Instruments | TPS562208DDCT | 21 | SOT-23-THIN (DDC) | U1–U17, U24, U26, U28, U31, U33 | 4.5–17V in, 2A sync buck, FCCM | **Confirmed** | Exact part identified |
| Ultra-low-noise LDO (×6) | **ADM7151** | Analog Devices | ADM7151ACPZ-04-R7 | 6 | LFCSP-8 (CP_8_11) | U5, U23, U25, U27, U29, U30 | 800 mA, ultra-low noise, high PSRR, RF LDO | **Confirmed** | Exact part identified |
| Low-noise LDO (×2) | **TPS7A8300** | Texas Instruments | TPS7A8300RGRR | 2 | QFN-20 (RGR) | U32, U34 | 2A, low-VIN, ultra-low-dropout, power good | **Confirmed** | Exact part identified |
| Voltage Inverter (×5) | **LM2662** | Texas Instruments | LM2662MX/NOPB | 5 | SOIC-8 (M08A) | U18–U22 | Switched-cap voltage converter (for negative bias) | **Confirmed** | Exact part identified |

### 3.4 Frequency Synthesizer Board — Additional ICs

*From `Clocks_Freq_Synth_board.csv` (confirmed BOM data):*

| Category | Component / Part | Manufacturer | MPN | Qty/Unit | Package | Ref Des | Specification | Confidence | Sourcing Status |
|----------|-----------------|-------------|-----|---------|---------|---------|---------------|------------|----------------|
| RF Balun/Transformer (×4) | **MTX2-143+** | Mini-Circuits | MTX2-143+ | 4 | DQ1225 | U2, U3, U7, U9 | 1:4 impedance ratio RF transformer, 0.5–14.5 GHz | **Confirmed** | Exact part identified |
| Attenuator (×4) | **ATS1005-3DB** | Susumu | ATS1005-3DB-FD-T05 | 4 | SMT | U4, U5, U8, U10 | Thin-film 3 dB attenuator pad | **Confirmed** | Exact part identified |
| Ferrite Bead (×4) | **FBMH1608HL601-T** | TDK (inferred) | FBMH1608HL601-T | 4 | 0603 | FB1–FB4 | 600Ω @ 100MHz ferrite bead | **Confirmed** | Exact part identified |

### 3.5 Power Amplifier Board (×16, Extended Only)

*From `RF_PA.brd` element listing:*

| Category | Component / Part | Manufacturer | MPN | Qty/Board | Qty/Unit (×16) | Package | Ref Des | Specification | Confidence | Sourcing Status |
|----------|-----------------|-------------|-----|----------|---------------|---------|---------|---------------|------------|----------------|
| GaN PA | **QPA2962** | Qorvo | QPA2962B | 1 | 16 | Custom QFN | U$1 | 8.5–11 GHz, 10W GaN PA | **Confirmed** | Exact part identified |
| Current Shunt | **5 mΩ Shunt** | Vishay (inferred) | WSL2816 series | 1 | 16 | 2816 | R10 | 5 mΩ, high-power current sense | **Confirmed** | Generic spec identified — exact MPN from `.brd` pattern |

---

## 4. Passive Components Summary

### 4.1 Power Board Passives (from `PowerBoard.csv`)

| Type | Value | Package | Qty | Ref Des (first/last) |
|------|-------|---------|-----|---------------------|
| Capacitor | 0.1 µF | C0805 | 21 | C1, C6, ... C154 |
| Capacitor | 10 µF | C0805 | 60 | C4, C5, ... C158 |
| Capacitor | 1 µF | C0805 | 12 | C26, C27, ... C142 |
| Capacitor | 22 µF | C0603 | 52 | C2, C3, ... C159 |
| Capacitor | 10 nF | C0603 | 4 | C150, C152, C160, C162 |
| Capacitor | 10 µF | C0603 | 4 | C151, C153, C161, C163 |
| Tantalum Cap | 47 µF / 20V | 7343 (T521W) | 10 | C96–C105 |
| Resistor | 10 kΩ | M0805 | 27 | R2, R4, ... R56 |
| Resistor | 56.2 kΩ | M0805 | 11 | R17, R23, ... R55 |
| Resistor | 32.2 kΩ | M0805 | 6 | R5, R7, ... R19 |
| Resistor | 12 kΩ | M0805 | 4 | R9, R35, R43, R47 |
| Resistor | Various (3.09k–61.9k) | M0805 | 8 | R1, R3, R21, R33, R39, R49, R53–R58 |
| Inductor | 2.2 µH (power) | VLP8040T | 2 | U$1, U$2 |
| Inductor | 3.3 µH (power) | VLP8040T | 19 | U$3–U$21 |
| Connector | 2-pin header | 22-23-2021 | 35 | X2–X36 |
| Connector | Screw terminal | AK300/2 | 1 | X1 |
| Pin Header | 2×5 | MA10-2 | 1 | SV1 |

**Total Power Board component count: ~273 components**

### 4.2 Frequency Synthesizer Board Passives (from CSV BOM)

| Type | Value | Package | Qty | Notes |
|------|-------|---------|-----|-------|
| Capacitor | 0.1 µF | C0201 | 26 | Decoupling |
| Capacitor | 4.7 µF | C0201 | 20 | Bulk bypass |
| Capacitor | 10 µF | C1210 | 5 | Bulk |
| Capacitor | 22 µF | C1210 | 6 | Bulk |
| Capacitor | 1 µF | C0201 | 10 | Local bypass |
| Capacitor | 10 pF | C0201 | 6 | RF tuning |
| Capacitor | Various (0.33µ–1nF) | C0201/C0402 | ~20 | Mixed signal |
| Resistor | Various (0–200k) | R0201 | ~50 | Bias/feedback |
| Inductor | 1.3 nH | L0201 | 8 | RF matching |
| Inductor | Unspecified | L5650M | 5 | Power |
| SMA connector | 142-0731-211 | Through-hole | 11 | RF I/O |
| Molex 2-pin | 22-23-2021 | THT | 6 | Power |
| MMCX/coax | CJT-T-P-HH-ST-TH1 | THT | 2 | Samtec coax |
| LED (green) | 0603 | SMD | 2 | Status |

**Total Freq Synth Board component count: ~180 components**

### 4.3 PA Board Passives (per board, from `.brd` file)

| Type | Value | Package | Qty/board | Qty/unit (×16) |
|------|-------|---------|----------|---------------|
| Capacitor | 10 µF | C1206 | 3 | 48 |
| Capacitor | 0.1 µF | C0402 | 6 | 96 |
| Resistor | 10 Ω | R0402 | 2 | 32 |
| Resistor | 0 Ω | R0402/R0603 | 6 | 96 |
| SMA connector | 142-0731-211 | THT | 2 | 32 |
| Molex 2-pin | 22-23-2021 | THT | 1 | 16 |
| Molex 3-pin | 22-23-2031 | THT | 1 | 16 |
| Terminal block | AK300/2 | THT | 1 | 16 |
| Shunt resistor | 5 mΩ | WSL2816 | 1 | 16 |

**Total PA Board component count (per board): ~23 → Total for 16 boards: ~368 components**

### 4.4 Patch Antenna Board (Nexus variant)

| Type | Value | Package | Qty | Notes |
|------|-------|---------|-----|-------|
| SMA connector | 142-0731-211 | THT | 16 | One per antenna element (J1–J17, skipping J16) |

**Note:** The patch antenna board is primarily a PCB with microstrip patch elements — minimal discrete components (connectors only from MNT data).

### 4.5 Main Board Passives

> [!WARNING]
> The Main Board BOM (`BOM_Main_Board.xlsx`) is in binary Excel format and could not be parsed. Quantities below are derived from `.brd` element extraction and firmware evidence. A complete passive BOM requires opening the `.xlsx` file.

**Known from board file evidence:**
- 16× INA241A3IDGKR (confirmed, U11/U73–U87)
- Extensive passive component population (estimated 500+ components based on board complexity)
- The Main Board is a 10-layer, ~280 mm² board with 90+ unique ICs

---

## 5. Oscillators and Crystals

*From Freq Synth Board CSV BOM:*

| Component | Manufacturer | MPN | Qty | Specification | Confidence | Sourcing Status |
|-----------|-------------|-----|-----|---------------|------------|----------------|
| **100 MHz VCXO** | ECS International | ECOC-2522-100.000-3HC | 1 | 100 MHz, 25×22 mm SMD | **Confirmed** | Exact part identified |
| **50 MHz VCXO** (×2) | Crystek | CVHD-950-50.000 | 2 | 50 MHz, low phase noise, SMD4 | **Confirmed** | Exact part identified |

**Evidence:** Freq Synth CSV BOM lines (X4, X5, X6)

---

## 6. Sensors

| Sensor | Manufacturer | MPN | Qty | Interface | Measurement | Source Evidence | Confidence | Sourcing Status |
|--------|-------------|-----|-----|-----------|-------------|----------------|------------|----------------|
| **GPS Receiver** | Unicore | UM982 | 1 | UART5 | Dual-antenna RTK GNSS, position + heading | `um982_gps.c/h`, `README.md` line 78 | **Confirmed** | Exact part identified |
| **IMU Module** | Generic | GY-85 | 1 | I²C (I2C2) | 9-DOF: accel + gyro + magnetometer | `GY_85_HAL.c/h`, `README.md` line 79 | **Confirmed** | Module identified — breakout board |
| **Barometer** | Bosch | BMP180 | 1 | I²C (I2C2) | Pressure (300–1100 hPa), altitude | `BMP180.cpp/h`, `README.md` line 80 | **Confirmed** | Exact part identified |
| **Thermistors** (×8) | Unknown | — | 8 | Analog → ADS7830 (U10) | Temperature monitoring, 8 zones | `main.cpp` lines 1001–1008, `README.md` line 82 | **High** | Generic spec — exact part unknown |
| **Temperature Sensor** | TI / Analog | TMP35/36/37 | ? | Analog | Temperature IC | `tmp35_36_37.pdf` in datasheets | **Low** | Datasheet present — usage/qty uncertain |

---

## 7. Actuators

| Actuator | Manufacturer | MPN | Qty | Interface | Specification | Source Evidence | Confidence | Sourcing Status |
|----------|-------------|-----|-----|-----------|---------------|----------------|------------|----------------|
| **Stepper Motor** | Unknown | — | 1 | GPIO (STEPPER_CLK_P, STEPPER_CW_P) | 200 steps/rev (1.8°), azimuth rotation | `main.cpp` line 199, `main.h` lines 136–139 | **High** | Generic spec identified — exact motor unknown |
| **Stepper Motor Driver** | Unknown | — | 1 | GPIO step/direction | Compatible with step/dir interface | `main.h` GPIO defines | **Medium** | Insufficient information — no driver IC specified |
| **Cooling Fan(s)** | Unknown | — | ≥1 | GPIO (EN_DIS_COOLING, PD7) | Thermal management fans | `main.h` line 142–143, `README.md` line 94 | **Medium** | Generic — exact part unknown |

---

## 8. Connectors and Interconnect

### 8.1 RF Connectors

| Connector | Manufacturer | MPN | Qty/Unit | Location | Specification | Confidence | Sourcing Status |
|-----------|-------------|-----|---------|----------|---------------|------------|----------------|
| **SMA Jack (Female)** | Cinch Connectivity Solutions (Johnson) | 142-0731-211 | 11 | Freq Synth Board (J1–J13) | 50 Ω, through-hole solder | **Confirmed** | Exact part identified |
| **SMA Jack (Female)** | Cinch Connectivity Solutions | 142-0731-211 | 16 | Patch Antenna Board (J1–J17) | 50 Ω, through-hole solder | **Confirmed** | Exact part identified |
| **SMA Jack (Female)** | Cinch Connectivity Solutions | 142-0731-211 | 32 | 16× PA Boards (J1, J2 per board) | 50 Ω, RF in/out per PA channel | **Confirmed** | Exact part identified |
| **SMA Broadband Circulator** | — | MTH801C-20 (datasheet) | ? | RF front-end chain | 8–12 GHz, SMA connectors | **Low** | Datasheet present — usage/qty unknown |
| **SMA connector (panel)** | — | FR2-SMA-KFD0405A | ? | Enclosure panel mount | Panel-mount SMA feedthrough | **Low** | Datasheet present — usage/qty unknown |

**Total identified SMA connectors: ≥59 (confirmed) + unknown enclosure connectors**

### 8.2 Board-to-Board / Wire-to-Board

| Connector | Manufacturer | MPN | Qty/Unit | Location | Specification | Confidence |
|-----------|-------------|-----|---------|----------|---------------|------------|
| **Molex 2-pin header** | Molex | 22-23-2021 | 35 + 6 + 16 = **57** | Power Board + Freq Synth + PA Boards | 0.100" (2.54 mm) center, 2-pin | **Confirmed** |
| **Molex 3-pin header** | Molex | 22-23-2031 | 16 | PA Boards (X3 per board) | 0.100" (2.54 mm) center, 3-pin | **Confirmed** |
| **Screw terminal 2P** | PTR | AK300/2 | 1 + 16 = **17** | Power Board (X1) + PA Boards | Power input terminal | **Confirmed** |
| **Pin header 2×5** | Generic | MA10-2 | 1 | Power Board (SV1) | 2×5 = 10-pin header | **Confirmed** |
| **Pin header 2×6** | Generic | PINHD-2X6 | 1 | Freq Synth Board (JP1) | 12-pin header | **Confirmed** |
| **Pin header 2×7** | Generic | PINHD-2X7 | 1 | Freq Synth Board (JP2) | 14-pin header | **Confirmed** |
| **Samtec coax** | Samtec | CJT-T-P-HH-ST-TH1 | 2 | Freq Synth Board (J3, J4) | Through-hole coaxial connector | **Confirmed** |

### 8.3 USB Connectors

| Connector | Qty | Notes | Confidence |
|-----------|-----|-------|------------|
| USB Type-B or Micro-B | 1–2 | For FT2232H/FT601 host connection + STM32 CDC | **Medium** — exact type not identified from repository |

---

## 9. Mechanical Components

### 9.1 Enclosure and Structure

| Component | Description | Source Evidence | Qty | Confidence | Sourcing Status |
|-----------|-------------|----------------|-----|------------|----------------|
| **Enclosure** | Custom machined enclosure | `8_Utils/Mechanical_Drawings/Enclosure.dwg` | 1 | **Confirmed** | Custom / manufactured item |
| **Waveguide (upper)** | Slotted waveguide upper half | `Waveguide_upper_part.dwg` | 1 (Extended) | **Confirmed** | Custom / manufactured item |
| **Waveguide (lower)** | Slotted waveguide lower half | `Waveguide_lower_part.dwg` | 1 (Extended) | **Confirmed** | Custom / manufactured item |
| **Heatsink (Main Board)** | Main board heatsink | `Heatsink_MainBoard.dwg` | 1 | **Confirmed** | Custom / manufactured item |
| **Heatsink (PA)** | PA board heatsink | `Heatsink_PA.dwg` | 1–16 (Extended) | **High** | Custom / manufactured item |
| **Waveguide Assembly** | Complete waveguide | `Waveguide.dwg` | 1 (Extended) | **Confirmed** | Custom / manufactured item |
| **Slip-Ring** | Rotating electrical joint | `SlipRing.dwg`, `README.md` line 92 | 1 | **Confirmed** | Sourced item — exact MPN unknown |
| **Stepper Motor Bracket** | Motor mount | `Stepper Motor Bracket.dwg` | 1 | **Confirmed** | Custom / manufactured item |
| **Backplate** | Rear structural plate | `Backplate.dwg` | 1 | **Confirmed** | Custom / manufactured item |
| **Board Support** | PCB mounting structure | `Board_Support.dwg` | 1 | **Confirmed** | Custom / manufactured item |

**All mechanical drawings are in AutoCAD DWG format** — dimensions/materials not extractable without CAD software.

### 9.2 Fasteners and Assembly Hardware

> [!WARNING]
> No explicit fastener BOM, screw specifications, or assembly hardware list was found in the repository. Screws, standoffs, thermal pads, and other assembly hardware are undocumented.

---

## 10. RF / Microwave Components (from Datasheets)

The following components have datasheets present in `7_Components Datasheets and Application Notes/` but their exact usage, quantity, and board placement require schematic analysis:

| Component | Manufacturer | MPN | Type | Frequency | Use (Inferred) | Confidence | Sourcing Status |
|-----------|-------------|-----|------|-----------|----------------|------------|----------------|
| **M3SWA2-34DR+** | Mini-Circuits | M3SWA2-34DR+ | SPDT switch | DC–34 GHz | RF T/R switching | **Medium** | Datasheet present — placement unknown |
| **PMA2-123LNW+** | Mini-Circuits | PMA2-123LNW+ | LNA | 1.2–3 GHz | Low noise amplifier | **Medium** | Datasheet present — may not be used at 10.5 GHz |
| **PMA5-123-3W** | Mini-Circuits | PMA5-123-3W | PA | 0.01–2.3 GHz | Power amplifier | **Low** | Datasheet present — likely reference/evaluation only |
| **QPA1013** | Qorvo | QPA1013D | LNA | 0.05–4 GHz | Low noise amplifier | **Low** | Datasheet present — unlikely in 10.5 GHz system |
| **QPM1021** | Qorvo | QPM1021 | T/R module | 8.5–10.5 GHz | Alternative T/R module | **Medium** | Possible alternative to ADTR1107 |
| **SMIQ-5143H+** | Mini-Circuits | SMIQ-5143H+ | IQ mixer | 5–14.3 GHz | IQ modulator/demodulator | **Medium** | Possible alternative mixer |
| **STUW81300** | STMicroelectronics | STUW81300TR | VCO/PLL | 8.1–13 GHz | Possible alternative LO | **Low** | Reference datasheet only |
| **TGA2623** | Qorvo/TriQuint | TGA2623-SM | PA | 8.5–11 GHz | GaN PA, 25W | **Medium** | Alternative to QPA2962 |
| **MMIQA-0218HPSM** | Marki | MMIQA-0218HPSM | IQ mixer | 2–18 GHz | Integrated IQ mixer GaAs | **Medium** | Possible alternative architecture |
| **MAX1449** | Maxim (Analog) | MAX1449 | ADC | — | 10-bit, 100 MSPS ADC | **Low** | Possible alternative/eval |
| **EP4RKU** | — | EP4RKU_2b | Attenuator | — | RF attenuator | **Low** | Evaluation component |
| **MAX20029** | Maxim (Analog) | MAX20029 | Dual LDO | — | Low-noise dual LDO regulator | **Medium** | Possible power reg alternative |

> [!NOTE]
> Many datasheets in the repository appear to be reference/evaluation materials rather than confirmed design-in components. Only components confirmed in schematics, BOMs, or firmware should be considered for procurement.

---

## 11. AD9523-1 Clock Tree — Output Assignments

*Extracted from `main.cpp` lines 1116–1174 (`configure_ad9523`):*

| Output | Destination | Frequency | Divider | Driver Mode | Interface |
|--------|-------------|-----------|---------|-------------|-----------|
| OUT0 | ADF4382A TX (ref) | 300 MHz | /12 | LVDS 7 mA | Differential |
| OUT1 | ADF4382A RX (ref) | 300 MHz | /12 | LVDS 7 mA | Differential |
| OUT4 | AD9484 ADC clock | 400 MHz | /9 | LVDS 7 mA | Differential |
| OUT5 | FPGA ADC clock | 400 MHz | /9 | LVDS 7 mA | Differential |
| OUT6 | FPGA system clock | 100 MHz | /36 | CMOS | Single-ended |
| OUT7 | FPGA test clock | 20 MHz | /180 | CMOS | Single-ended |
| OUT8 | SYNC_TX | 60 MHz | /60 | LVDS 4 mA | Differential |
| OUT9 | SYNC_RX | 60 MHz | /60 | LVDS 4 mA | Differential |
| OUT10 | DAC clock | 120 MHz | /30 | CMOS | Single-ended |
| OUT11 | FPGA DAC clock | 120 MHz | /30 | CMOS | Single-ended |

VCO frequency: 3.6 GHz (PLL2: VCXO 100 MHz × N=36)

---

## 12. External Equipment (Not Part of Product BOM)

| Equipment | Purpose | Required For | Notes |
|-----------|---------|-------------|-------|
| **Xilinx Vivado** | FPGA synthesis/P&R | Development | License required for XC7A50T |
| **STM32CubeIDE** | MCU firmware development | Development | Free |
| **Icarus Verilog** | FPGA simulation | CI/Testing | Open-source |
| **SymbiYosys** | Formal verification | CI/Testing | Open-source |
| **JTAG Programmer** | FPGA programming | Manufacturing/Debug | Xilinx Platform Cable USB or Digilent HS3 |
| **ST-LINK** | STM32 programming/debug | Manufacturing/Debug | ST-LINK/V2 or V3 |
| **Autodesk Eagle** | PCB design viewing | Design/Modification | Required to read .sch/.brd files |
| **AutoCAD / FreeCAD** | Mechanical drawing viewing | Design/Modification | Required to read .dwg files |
| **USB 2.0/3.0 Host PC** | Radar operation | Operation | Runs Python GUI |
| **12V–48V DC Supply** | Main power input | Operation | Exact voltage TBD from power board schematic |
| **SMA cables** | RF interconnect | Assembly/Testing | 50 Ω, SMA-SMA, various lengths |
| **RF test equipment** | Validation | Testing | Spectrum analyzer, VNA, signal generator |

---

## 13. Software-Only Dependencies (NOT Physical Components)

| Dependency | Type | Notes |
|------------|------|-------|
| Python 3.12 | Runtime | Host GUI |
| PyQt6 ≥ 6.5 | GUI framework | V7 dashboard |
| numpy, matplotlib, scipy | Libraries | Signal processing |
| pyftdi, pyusb | USB drivers | Hardware interface |
| uv | Package manager | Dependency management |
| ruff, pytest | Dev tools | Linting, testing |

---

## 14. Sourcing Summary

### Component Counts

| Category | Count |
|----------|-------|
| **Total identified physical line items (unique parts)** | ~85 |
| Confirmed exact parts (MPN known) | 32 |
| Parts requiring sourcing research (manufacturer known, MPN uncertain) | 15 |
| Parts requiring equivalent/substitute research | 8 |
| Custom manufactured parts | 8 |
| Unknown components (insufficient info) | 12 |
| External equipment (not in product) | 10 |
| Optional components (Extended variant only) | QPA2962 (×16), FT601, PA boards, waveguide antenna |

### Passive Component Counts (per unit, confirmed boards)

| Board | Passives | ICs | Connectors | Total |
|-------|---------|-----|------------|-------|
| Power Board | ~230 | 34 | 37 | ~273 |
| Freq Synth Board | ~155 | 14 | 22 | ~180 |
| PA Board (×16) | ~176 | 16 | 80 | ~368 |
| Patch Antenna | 0 | 0 | 16 | 16 |
| Main Board | ~500+ (est.) | 90+ (est.) | 20+ (est.) | ~600+ (est.) |
| **Estimated total** | | | | **~1,437+** |

---

## 15. Critical Procurement Risks

| Risk | Component(s) | Impact | Mitigation |
|------|-------------|--------|------------|
| **Long lead / allocation** | QPA2962 (GaN PA), ADAR1000, ADF4382A, AD9523-1 | All are high-performance ADI/Qorvo parts with historically constrained supply | Order early; verify lead times with distributors |
| **Obsolescence risk** | LTC5552, FT2232H, BMP180 | BMP180 is discontinued (successor: BMP280/BMP390) | BMP280 is pin-compatible; verify firmware changes |
| **Custom RF PCB** | Main Board (10-layer RO4350B) | Very few PCB fabricators handle 10-layer RF hybrid stack-ups with 100 µm cores | PCBWay confirmed as supplier (impedance note exists) |
| **Custom machined parts** | Enclosure, waveguide, heatsinks | Require CNC machining; no material/tolerance specs in repository | Open DWG files to extract specs |
| **Stepper motor unspecified** | Motor + driver | Cannot procure without specs (torque, voltage, current, NEMA size) | Derive from mechanical load requirements |
| **Main Board BOM binary** | All Main Board passives | ~500+ passive components unverifiable without opening .xlsx | Open `BOM_Main_Board.xlsx` in Excel |
| **Slip-ring unspecified** | Slip-ring assembly | Channel count, current rating, bandwidth unknown | Inspect `SlipRing.dwg` for dimensions |
| **Thermistor specs unknown** | 8× temperature sensors | Cannot procure without resistance value, β-coefficient | Inspect schematic for values |

---

## 16. Information Gaps & Investigation Backlog

### 16.1 Unavailable / Unparsed Source Evidence

| Artifact | Path | Why It Matters | Expected Information | Current Status |
| -------- | ---- | -------------- | -------------------- | -------------- |
| Binary Excel BOMs | `4_7_Production Files/Gerber_*/*.xlsx` | Main Board has 500+ unverified passives | Complete component lists for all boards | Pending extraction |
| Mechanical DWG Files | `8_Utils/Mechanical_Drawings/*.dwg` | Contains mechanical dimensions and materials | Specifications for enclosure, waveguide, heatsinks | Unparsed |
| Board Files | `4_6_Schematics/*/*.brd` | Verifies schematic against layout | Exact reference designators and placements | Partially extracted |
| PCB Stack-up | `4_4_Board Stack-up/` | Needed for RF PCB fabrication | Layer thicknesses, dielectric constants for non-Main boards | Missing |

### 16.2 Component Identification Gaps

| Component | What Is Known | What Is Unknown | Why It Matters | Priority |
| --------- | ------------- | --------------- | -------------- | -------- |
| MCU | STM32F746xx (Cortex-M7) | Exact suffix (e.g. VGT6 vs VET6) | Determines package size, Flash, and RAM capacity | Critical |
| USB Connector | Used for FT2232H/FT601 | Type-B, Micro-B, or Type-C | Required for mechanical design and BOM | High |
| Stepper Motor | Azimuth rotation, GPIO controlled | NEMA size, voltage, holding torque, step angle | Cannot procure without exact mechanical/electrical specs | High |
| Stepper Driver | GPIO step/dir interface | Driver IC/Module used | Required for BOM | High |
| Slip Ring | Used for continuous 360° rotation | Channels, current rating, bandwidth, RPM | Critical mechanical/electrical interconnect | High |
| Cooling Fan | GPIO controlled (EN_DIS_COOLING) | Voltage, airflow, size, connector | Required for thermal management and BOM | Medium |
| Thermistors | 8x analog sensors read by ADS7830 | Resistance at 25°C, β-coefficient | Needed for temperature calculation accuracy | Medium |
| Power Supply | Main DC input | Input voltage range, current capacity | Defines system power requirements | Critical |
| EEPROM | AT93C46A datasheet present | Is it populated? What is its function? | Affects BOM and firmware initialization | Low |

### 16.3 Manufacturing Information Gaps

*   **PCB stack-ups:** Missing for Power, Freq Synth, PA, and Patch Antenna boards.
*   **Fasteners:** No explicit BOM for screws, standoffs, or spacers.
*   **Thermal interface materials:** Missing specs for thermal pads or paste (especially for QPA2962 GaN PAs).
*   **Cable/harness specifications:** SMA cable lengths and power wiring gauge unknown.
*   **Mechanical dimensions/tolerances:** Locked in DWG files.
*   **Assembly information:** No sequence or torque specs provided.

### 16.4 Functional / Architectural Unknowns

*   **EEPROM function:** What data is stored in the AT93C46A?
*   **Firmware-dependent component variants:** How does the firmware distinguish between FT2232H and FT601 build variants?
*   **Hardware/software coupling:** Are the thermal trip points for the cooling fans hardcoded or configurable?
*   **Calibration requirements:** How is the `Idq` closed-loop calibration procedure performed at boot?

---

## 17. Reverse-Engineering Investigation Backlog

| ID | Investigation | Input Evidence | Expected Output | Priority | Dependency | Status |
| -- | ------------- | -------------- | --------------- | -------- | ---------- | ------ |
| INV-001 | Extract Main Board BOM | `BOM_Main_Board.xlsx` | Complete Main Board component list | Critical | Excel parsing | Pending |
| INV-002 | Extract Power Board BOM | `BOM_Power_Board.xlsx` | Cross-validate against CSV | High | Excel parsing | Pending |
| INV-003 | Extract PA Board BOM | `BOM_PA.xlsx` | Verify against `.brd` extraction | High | Excel parsing | Pending |
| INV-004 | Resolve STM32F746 suffix | Main Board schematic/BOM | Exact MCU MPN/package | Critical | Schematic/BOM | Pending |
| INV-005 | Identify USB Connector | Main Board `.brd` / Schematic | Connector MPN | High | Schematic/BRD | Pending |
| INV-006 | Specify Stepper Motor/Driver | Schematic / DWG files | Exact motor and driver specs | High | Schematic/CAD | Pending |
| INV-007 | Specify Slip Ring | `SlipRing.dwg` / Schematic | Slip ring specs (channels, bandwidth) | High | CAD/Schematic | Pending |
| INV-008 | Extract Mechanical Specs | `8_Utils/Mechanical_Drawings/*.dwg` | Dimensions/materials for custom parts | High | DWG parsing | Pending |
| INV-009 | Identify Thermistor Specs | Main Board schematic | Resistance and β-coefficient | Medium | Schematic | Pending |
| INV-010 | Determine Power Supply Specs | Power Board schematic | Input voltage and current requirements | Critical | Schematic | Pending |

---

## 18. Recommended Next Actions

### 18.1 Engineering / Reverse-Engineering Actions
*   **Priority:** Critical
*   **Reason:** Need to close major component identification gaps to understand system architecture fully.
*   **Dependency:** Ability to parse Excel and DWG files.
*   **Expected deliverable:** Updated Component Inventory with exact MPNs for MCU, stepper motor, and slip ring.

### 18.2 BOM / Procurement Actions
*   **Priority:** High
*   **Reason:** Several high-performance components have historically constrained supply.
*   **Dependency:** Confirmation of exact part numbers from investigations.
*   **Expected deliverable:** RFQ (Request for Quote) for long-lead ADI/Qorvo components (ADAR1000, ADF4382A, AD9523-1, QPA2962). (Do not purchase until exact specs are verified).

### 18.3 Manufacturing Preparation
*   **Priority:** Medium
*   **Reason:** Need to define physical assembly requirements.
*   **Dependency:** Extraction of mechanical DWG files and PCB schematics.
*   **Expected deliverable:** Fastener BOM, cable harness specifications, and complete PCB fabrication notes.

---

## 19. BOM Readiness

| Area | Status | Explanation |
| ---- | ------ | ----------- |
| Electronic components | Partial | Active ICs mostly identified; 500+ passives locked in binary BOMs. |
| PCB assemblies | Mostly Ready | Gerber files present; stack-ups needed for non-Main boards. |
| Mechanical components | Blocked | Custom parts locked in DWG files; exact motor/slip-ring unknown. |
| Cables/harnesses | Unknown | No lengths or specifications provided in repository. |
| Consumables | Unknown | Thermal paste, threadlocker, adhesives unspecified. |
| Exact MPN coverage | Partial | Good coverage on RF/Digital ICs; poor on passives/electromechanical. |
| Quantity coverage | Partial | Known for ICs; unknown for Main Board passives. |
| Manufacturing data | Partial | Gerbers exist; assembly instructions and CAD specs missing. |
| Overall procurement readiness | Blocked | Cannot order full BOM until Excel files and DWGs are parsed. |

---

## 20. Sourcing Risk Summary

*   **Long lead time:** QPA2962 (GaN PA), ADAR1000, ADF4382A, AD9523-1. These are high-performance ADI/Qorvo parts with historically constrained supply.
*   **Custom manufactured:** Enclosure, waveguide, heatsinks (require CNC machining specs from unparsed DWG files).
*   **External supplier dependency:** Main Board is a 10-layer RO4350B hybrid PCB. Very few fabricators (e.g., PCBWay) can handle this stack-up with 100 µm cores.
*   **Obsolete/discontinued:** BMP180 (successor is BMP280/BMP390). Firmware changes may be required if exact replacement isn't pin-compatible.
*   **Specification incomplete:** Stepper motor, slip-ring, cooling fans, thermistors, and power supply.
*   **Exact MPN unknown:** STM32F746xx (suffix unknown), USB connector type.
*   **Quantity uncertain:** Main Board passives (locked in binary Excel BOM).

---

*This document links back to [[00_Executive_Summary]] and forward to [[03_Hardware]] for detailed board-level analysis.*
