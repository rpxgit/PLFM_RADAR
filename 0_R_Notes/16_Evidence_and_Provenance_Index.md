---
type: reverse-engineering-note
status: active
domain: system
confidence: high
canonical: true
---

# Evidence and Provenance Index

> **Related notes:** [[09_Unknowns_and_Hypotheses]] · [[15_Reproduction_Readiness_and_Gate_Review]] · [[10_Reverse_Engineering_Plan]]
> **Document type:** Master Evidence Provenance Registry
> **Status:** Living Document

This document establishes the strict chain of custody for all technical conclusions in the AERIS-10 reverse-engineering project, answering: *Where did we get this information, and how strong is the evidence?*

---

## 1. Evidence Classification

* **E0 — Direct Primary Evidence:** Actual source code, schematic, PCB, BOM, CAD, captured data, test result, etc.
* **E1 — Strong Corroboration:** Multiple independent primary sources agree.
* **E2 — Project Documentation:** Explicit documentation without direct implementation confirmation.
* **E3 — Derived Evidence:** Strong technical inference from available evidence.
* **E4 — Hypothesis:** Plausible but unverified.
* **E5 — Missing Evidence:** The required source does not currently exist or cannot be accessed.

---

## 2. Evidence Registry

| Evidence ID | Source Type | Repository Path | Source Location | Subject | What It Establishes | Confidence | Related Notes |
| ----------- | ----------- | --------------- | --------------- | ------- | ------------------- | ---------- | ------------- |
| **EVID-001** | Schematic | `Hardware/Schematics/PowerBoard.sch` | Top Level | Power Topology | Presence of LM2662 inverters providing -5V | E0 | `03` |
| **EVID-002** | C++ Source | `Firmware/STM32/src/main.cpp` | `main()` loop | MCU Control | SPI/I2C opcode structures for AD9523, ADF4382 | E0 | `05`, `07` |
| **EVID-003** | Python | `Software/GUI/radar_protocol.py` | `read_fifo()` | Host/FPGA Data | GUI directly reads FT2232H high-speed FIFO | E0 | `06`, `07` |
| **EVID-004** | Verilog | `FPGA/RTL/top.v` | I/O declarations | FPGA Architecture | Pin mappings for 50T/200T target | E0 | `04` |
| **EVID-005** | Documentation | `README.md` | Hardware Section | Power Constraints | Claims positive-rails only | E2 (Contradicted) | `09` |
| **EVID-006** | Project File | `FPGA/Vivado/AERIS_10.xpr` | Target Property | FPGA Target | XC7A50T vs XC7A200T split | E3 | `04` |
| **EVID-007** | Python | `Software/GUI/main.py` | PyQt6 Imports | GUI Architecture | App is built on PyQt6 | E0 | `06` |
| **EVID-008** | Binary Data | `Hardware/BOM/BOM_Main_Board.xlsx` | Unparsable | Passives | Unknown | E5 | `08` |
| **EVID-009** | Binary Data | `Hardware/Mechanical/*.dwg` | Unparsable | Enclosure | Unknown | E5 | `03` |
| **EVID-010** | Schematic | `Hardware/Schematics/PowerBoard.sch` | DC Input | Input Spec | Connector exists, but label is missing | E5 | `03`, `12` |

---

## 3. Claim Registry

| Claim ID | Technical Claim | Evidence ID(s) | Confidence | Related Unknown | Related Task | Status |
| -------- | --------------- | -------------- | ---------- | --------------- | ------------ | ------ |
| **CLM-001** | GaN PAs (QPA2962) require negative pinch-off bias | EVID-001 | High | None | Hardware Bring-up | Verified |
| **CLM-002** | Host GUI bypasses STM32 for high-speed ADC data | EVID-002, EVID-003 | High | None | Digital Integration | Verified |
| **CLM-003** | AERIS-10 uses a split 50T/200T FPGA architecture | EVID-004, EVID-006 | Medium | UNK-005 | RE-005 | Accepted |
| **CLM-004** | The system input DC voltage is strictly defined | EVID-010 | Low | UNK-010 | RE-003 | Unverified |
| **CLM-005** | The MCU is an STM32F746 | EVID-002 | Medium | UNK-003 | RE-004 | Inferred |

---

## 4. Evidence → Claim → Task

```text
EVID-001 (PowerBoard.sch LM2662 presence)
   ↓
CLM-001 (GaN PAs require negative pinch-off bias)
   ↓
CON-001 (Contradicts README.md EVID-005)
   ↓
RE-003 (Review PowerBoard.sch)
   ↓
Resolution: Hardware schematic E0 overrides documentation E2.
```

```text
EVID-008 (Unparsable BOM_Main_Board.xlsx)
   ↓
CLM-006 (Exact values of 500+ passives are unknown)
   ↓
UNK-001 (BOM passives missing)
   ↓
RE-001 (Parse BOM via Python/Pandas locally)
   ↓
Resolution: Pending Execution.
```

---

## 5. Replication-Critical Evidence

The following primary evidence is absolutely critical; without it, faithful reproduction would be significantly impaired:

* **FPGA Top-Level RTL (`EVID-004`):** Defines the exact pinout mapping to the physical hardware.
* **Firmware Main Loop (`EVID-002`):** Defines the precise initialization sequence for the RF chain.
* **Host Protocol Bindings (`EVID-003`):** Defines the USB FTDI opcodes required to command the radar.
* **Power Board Schematic (`EVID-001`):** Defines the critical RF bias safety interlocks.

---

## 6. Weak / Inferential Evidence

Important conclusions currently supported primarily by inference or weak documentation:

* **STM32 Package (CLM-005):** We infer an STM32F746 from the firmware build targets, but the exact physical suffix (e.g., LQFP vs BGA) is unverified because the hardware PCB layout files have not been cross-referenced. Linked to `UNK-003`.
* **FPGA Target (CLM-003):** We infer the XC7A50T target from Vivado project file names, rather than a physical teardown or explicit top-level block. Linked to `UNK-005`.

---

## 7. Evidence Gaps

| Gap | Missing Evidence | Why Needed | Affected Claims | Affected Tasks | Priority |
| --- | ---------------- | ---------- | --------------- | -------------- | -------- |
| **GAP-001** | Parsed `BOM_Main_Board.xlsx` | 500+ passives required for SMT | CLM-006 | RE-001 | P0 |
| **GAP-002** | Parsed `.dwg` CAD files | Waveguide and Enclosure machining | CLM-007 | RE-002 | P0 |
| **GAP-003** | System DC Input Label | Required to safely apply bench power | CLM-004 | RE-003 | P1 |

---

## 8. Conflicting Evidence

* **CON-001:** `EVID-005` (`README.md`) states the system uses positive-rails only. `EVID-001` (`PowerBoard.sch`) explicitly contains an LM2662 negative voltage inverter.
  * **Resolution:** Direct Evidence (E0) overrides Project Documentation (E2). `README.md` is incorrect. See `[[09_Unknowns_and_Hypotheses]]`.

---

## 9. Evidence Navigation

* **Executive Summary:** [[00_Executive_Summary]]
* **Repository Map:** [[01_Repository_Map]]
* **Hardware:** [[03_Hardware]]
* **FPGA:** [[04_Firmware_FPGA]]
* **MCU:** [[05_Firmware_MCU]]
* **Software:** [[06_Software_GUI]]
* **Protocols:** [[07_Communication_Protocols]]
* **BOM:** [[08_Component_Inventory]]
* **Unknowns:** [[09_Unknowns_and_Hypotheses]]
* **Reverse Engineering Plan:** [[10_Reverse_Engineering_Plan]]
* **Readiness Gates:** [[15_Reproduction_Readiness_and_Gate_Review]]

---

## 10. Final Provenance Health

* **Strongest Evidence Domains:** Digital Architecture, Host GUI, Communications Protocol, MCU Firmware Logic. (Backed entirely by E0 source code).
* **Weakest Evidence Domains:** Mechanical Dimensions, Passive Component Selection, Physical PCB Footprints. (Blocked by E5 missing binary artifacts).
* **Major Claims Without Primary Evidence:** The physical package of the MCU and FPGA are currently inferred (E3).
* **Major Missing Artifacts:** `BOM_Main_Board.xlsx`, Mechanical `.dwg` files.
* **Replication-Critical Evidence Gaps:** The unparsed BOM prevents any hardware fabrication.

This index guarantees that no engineering assumption in the AERIS-10 project is treated as fact without a direct link to the underlying repository evidence.


### EVID-BOM1 — Main Board Excel Extraction
- **Source:** `Hardware/BOM/BOM_Main_Board.xlsx` (Parsed via Zip/XML Extraction)
- **Finding:** Contains 104 exact MPNs for the Main Board.
- **Key Discovers:** Proves the RF front-end utilizes 16x ADTR1107 T/R chips rather than QPA2962. Identifies LTC5552 mixers and AD9484 ADC. Confirms 500+ RF matching passives.


### EVID-CAD1 — Patch Antenna FDTD Report
- **Source:** `docs/AERIS_Antenna_Report.pdf`
- **Finding:** Provides the exact FDTD-tuned mechanical geometry for the 10.5 GHz patch antenna element.
- **Dimensions:** 9.545 mm x 7.401 mm on 0.102 mm RO4350B substrate. Fabricator must hold ±0.025 mm tolerance.



### EVID-BOM1 — Main Board Excel Extraction
- **Source:** `Hardware/BOM/BOM_Main_Board.xlsx` (Parsed via Zip/XML Extraction)
- **Finding:** Contains 104 exact MPNs for the Main Board.
- **Key Discovers:** Proves the RF front-end utilizes 16x ADTR1107 T/R chips rather than QPA2962. Identifies LTC5552 mixers and AD9484 ADC. Confirms 500+ RF matching passives.


### EVID-CAD1 — Patch Antenna FDTD Report
- **Source:** `docs/AERIS_Antenna_Report.pdf`
- **Finding:** Provides the exact FDTD-tuned mechanical geometry for the 10.5 GHz patch antenna element.
- **Dimensions:** 9.545 mm x 7.401 mm on 0.102 mm RO4350B substrate. Fabricator must hold ±0.025 mm tolerance.


### EVID-PWR1 — Power Management Matrix
- **Source:** `3_Power Management/Power Management V6.xlsx`
- **Finding:** Details the entire power tree, current budget, and exact gate-drain bias sequencing for both the ADTR1107 and QPA2962 RF devices.


### EVID-FPGA1 — FPGA Pure Verilog Architecture
- **Source:** `9_Firmware/9_2_FPGA/` (`xfft_16.v`, `fft_engine.v`, `nco_400m_enhanced.v`, `build_50t.tcl`)
- **Finding:** The design is completely devoid of Xilinx proprietary IP blocks. It utilizes a custom Verilog FFT engine and NCO, driven by a batch TCL build script targeting `xc7a50tftg256-2`.

### EVID-FPGA2 — Digital Chirp Generation
- **Source:** `9_Firmware/9_2_FPGA/radar_transmitter.v`
- **Finding:** Proves that the PLFM chirp is generated digitally in the FPGA via `plfm_chirp_controller_enhanced` and output through the AD9708 DAC, rather than autonomously by the ADF4382 synthesizer.
