# Hardware Architecture Master Document

> **Related notes:** [[02_System_Architecture]] · [[01_Component_Inventory_and_Sourcing]] · [[09_Unknowns_and_Hypotheses]] · [[12_Power_Architecture]] · [[13_RF_Signal_Chain]]
> **Sub-assemblies:** [[03a_Main_Board]] · [[03b_Power_Board]] · [[03c_Frequency_Synthesizer_Board]] · [[03d_Power_Amplifier_Board]] · [[03e_Antenna_Array]]
> **Document type:** Canonical Hardware Integration Map  
> **Status:** Read-only Forensic Snapshot

---

## 1. Physical Hardware Hierarchy

The AERIS-10 product relies on a distributed multi-board architecture. 

| Assembly | Function | Quantity / Unit | Parent Assembly | Evidence | Confidence |
| -------- | -------- | --------------: | --------------- | -------- | ---------- |
| **AERIS-10 System** | Complete Product | 1 | N/A | `README.md` | Confirmed |
| ├── **Main Board** | System Controller, FPGA, Mixed Signal, RF Transceiver | 1 | System | `.sch`, `.brd` | Confirmed |
| ├── **Power Board** | Power Regulation and Sequencing | 1 | System | `.sch`, `.brd` | Confirmed |
| ├── **Freq Synth Board** | Clock and LO Generation | 1 | System | `.sch`, `.brd` | Confirmed |
| ├── **PA Board** | 10W GaN Amplification | 16 (Extended Variant Only) | System | `.sch`, `.brd` | Confirmed |
| ├── **Antenna Array** | RF Radiation | 1 | System | `.dwg`, `.brd` | Confirmed |
| └── **Electromechanical** | Enclosure, Stepper, Fans, Slip-ring | 1 | System | `.dwg` | Confirmed |

---

## 2. Board-Level Evidence Collection

| Board | Schematic | PCB | BOM | Gerbers | Netlist | Firmware Evidence | Manufacturing Evidence | Completeness |
| ----- | --------- | --- | --- | ------- | ------- | ----------------- | ---------------------- | ------------ |
| **Main Board** | `RADAR_Main_Board.sch` | `RADAR_Main_Board.brd` | `BOM_Main_Board.xlsx` (Binary) | Present (`4_7_Production Files`) | Inferred from Eagle | `main.cpp`, `main.h`, `radar_system_top.v` | `CPL.xlsx` | High |
| **Power Board** | `PowerBoard.sch` | `PowerBoard.brd` | `BOM_Power_Board.xlsx` (Binary) | Present (`4_7_Production Files`) | Inferred from Eagle | `main.cpp` (Power Sequencing) | `CPL.xlsx` | High |
| **Freq Synth** | `Clocks_Freq_Synth_board.sch` | `Clocks_Freq_Synth_board.brd` | `BOM_Freq_Synth.xlsx` (Binary) | Present (`4_7_Production Files`) | Inferred from Eagle | `adf4382a_manager.cpp` | `CPL.xlsx` | High |
| **PA Board** | `RF_PA.sch` | `RF_PA.brd` | `BOM_PA.xlsx` (Binary) | Present (`4_7_Production Files`) | Inferred from Eagle | `README.md` (Bias Control) | `CPL.xlsx` | High |
| **Antenna (Nexus)** | `Patch_Anetnna_16_8.sch` | `Patch_Anetnna_16_8.brd` | `BOM_Patch_Antenna.xlsx` (Binary) | Present (`4_7_Production Files`) | Inferred from Eagle | None | `CPL.xlsx` | High |

---

## 3. Cross-Board Interconnect

| Source Board | Destination Board | Connector | Signals | Power | Clock | RF | Control | Evidence | Confidence |
| ------------ | ----------------- | --------- | ------- | ----- | ----- | -- | ------- | -------- | ---------- |
| **Power Board** | **Main Board** | TBD Header | No | Yes | No | No | Enables | `.sch` | High |
| **Freq Synth** | **Main Board** | SMA/SMP & Header | SPI | No | Yes | LO | Yes | `README.md` | Confirmed |
| **Main Board** | **PA Board (x16)** | SMA/SMP & Header | Vg, Idq sense | Vd | No | RF | No | `README.md` | Confirmed |
| **Main / PA Board** | **Antenna Array** | SMA/SMP | No | No | No | RF | No | `README.md` | Confirmed |

*(Note: Exact connector part numbers and mechanical stacking arrangements require extraction from the CAD/DWG files).*

---

## 4. Net / Signal Classification

### Power
*   **Source:** Power Board.
*   **Critical Nets:** Main DC input, 3.3V Digital, 1.8V Digital, PA Drain ($V_d$), PA Gate ($V_g$).

### Clock / RF
*   **Source:** Frequency Synthesizer Board.
*   **Critical Nets:** FPGA/ADC/DAC system clocks (differential LVDS/PECL). LO signals (10.5 GHz X-band).

### Digital Control
*   **Source:** Main Board (MCU).
*   **Critical Nets:** SPI buses to Freq Synth and Phase Shifters. I2C buses to ADCs and DACs. GPIO for RF switching and power enables.

### High-Speed Digital Data
*   **Source:** Main Board (FPGA/ADC/DAC).
*   **Critical Nets:** 400MHz LVDS from ADC to FPGA. Parallel CMOS from FPGA to DAC. FTDI USB FIFO to Host.

---

## 5. Hardware ↔ Firmware ↔ FPGA Correlation

| Hardware Feature | MCU Firmware Evidence | FPGA Evidence | Expected Function |
| ---------------- | --------------------- | ------------- | ----------------- |
| **ADC (AD9484)** | None | `tb_ad9484_xsim.v` | 400MSPS Digitization |
| **DAC (AD9708)** | None | `radar_system_top.v` | IF Chirp generation |
| **ADAR1000** | `ADAR1000_Manager.cpp` | None | RF Beam steering |
| **ADF4382** | `adf4382a_manager.cpp` | None | LO synthesis |
| **USB Interface** | `radar_protocol.py` (CDC) | `radar_system_top_50t.v` | Host communications |
| **Power/Bias** | `main.cpp` | None | GaN Vg calibration, Thermal limits |

---

## 6. Mechanical & Thermal Integration

*   **Enclosure:** Defined by `Enclosure.dwg`. Contains all PCBs.
*   **Thermal Management:** The 16x 10W GaN PAs in the Extended variant require massive heat dissipation. 8x thermistors are measured by the MCU, triggering a cooling fan (`EN_DIS_COOLING`). Heat sinks are mechanically defined in the `.dwg` files.
*   **Actuation:** A stepper motor and slip-ring allow 360° continuous rotation of the radar assembly (`SlipRing.dwg`).

---

## 7. Manufacturing Readiness

| Assembly | Design Complete? | Manufacturing Files? | BOM? | Assembly Data? | Critical Unknowns | Readiness |
| -------- | ---------------- | -------------------- | ---- | -------------- | ----------------- | --------- |
| **Main Board** | Yes | Yes (Gerbers) | Yes (Binary `.xlsx`) | Yes (`CPL.xlsx`) | Component MPNs hidden in binary BOM | Blocked |
| **Power Board** | Yes | Yes (Gerbers) | Yes (Binary `.xlsx`) | Yes (`CPL.xlsx`) | Power sequencing and exact specs | Blocked |
| **Freq Synth** | Yes | Yes (Gerbers) | Yes (Binary `.xlsx`) | Yes (`CPL.xlsx`) | Component MPNs hidden in binary BOM | Blocked |
| **PA Board** | Yes | Yes (Gerbers) | Yes (Binary `.xlsx`) | Yes (`CPL.xlsx`) | Component MPNs hidden in binary BOM | Blocked |
| **Antennas** | Yes | Yes / No | Yes (Nexus only) | No | Missing stack-up for waveguide manufacturing | Blocked |

---

## 8. Hardware Revision / Variant Matrix

| Hardware Item | Revision / Variant | Difference | Evidence | Replication Impact |
| ------------- | ------------------ | ---------- | -------- | ------------------ |
| **Antenna Array** | Nexus | 8x16 Patch Array | `.brd` / `.sch` | Lower range, simple PCB fabrication. |
| **Antenna Array** | Extended | 32x16 Waveguide | `.dwg` | High range, requires complex CNC machining. |
| **Power Amplifiers** | Nexus | Relies on Main Board ADTR1107 | `README.md` | Lower power, simplified power routing. |
| **Power Amplifiers** | Extended | Adds 16x QPA2962 PA Boards | `RF_PA.sch` | High thermal/power requirements, requires strict GaN sequencing. |
| **USB Controller** | 50T Production | FT2232H (USB 2.0) | `radar_system_top_50t.v` | Lower data throughput, 8-bit FIFO. |
| **USB Controller** | 200T Premium | FT601 (USB 3.0) | `radar_system_top.v` | Higher throughput, 32-bit FIFO. |

---

## 9. Replication-Critical Hardware

### Critical
*   **Main Board Stack-up:** 10-layer RO4350B with 100µm cores. Cannot be substituted without recalculating all RF impedance traces.
*   **GaN Protection Circuitry:** The 5mΩ shunt + 50x amplifier tied to the MCU closed-loop calibration. Must be replicated exactly.
*   **Phase-Matched Traces:** Clock distribution and RF LO traces.

### High
*   **Mechanical Tolerances:** The Waveguide antenna array (Extended variant) requires CNC machining to exact `.dwg` specifications.
*   **Thermal Interfaces:** Heat sinks and thermal paste for the QPA2962 amplifiers.

---

## 10. Hardware Unknowns

| ID | Unknown | Assembly | Why It Matters | Evidence Available | Required Investigation | Priority |
| -- | ------- | -------- | -------------- | ------------------ | ---------------------- | -------- |
| HW-01 | Main Board BOM | Main Board | 500+ passives must be identified for assembly | `BOM_Main_Board.xlsx` | Parse Excel binary. | Critical |
| HW-02 | Physical Board Mating | All | Must know how 16 PA boards attach to Main Board & Waveguide | `.dwg` files | Parse Mechanical CAD. | High |
| HW-03 | Waveguide Materials | Antenna | Exact dielectric fill material for the slotted waveguide | `Dielectric_Filled_Waveguide_Array_Spec.docx` | Read DOCX spec sheet. | High |

---

## 11. Final Hardware Assessment

### Hardware We Understand
We possess high confidence in the overall physical hierarchy, the board-to-board functional boundaries, the MCU/FPGA/RF topology, and the variant differences (Nexus vs Extended). 

### Hardware We Can Reproduce
We possess the complete Eagle `.sch`/`.brd` files and Gerber manufacturing outputs for all PCBs.

### Hardware Requiring Further Investigation
We are blocked on the passive component specifications due to the binary `.xlsx` BOM format, and the exact physical interconnect/housing dimensions locked in the `.dwg` files.

### Manufacturing Gaps
Assembly cannot commence without the parsed BOMs. Waveguide fabrication cannot commence without extracting the dielectric fill material specifications.

### Highest-Risk Hardware Dependencies
The tight coupling between the Main Board's MCU calibration firmware and the physical shunt resistor values on the PA Boards.

### Recommended Next Hardware Investigation
*   **Investigation:** Extract binary `.xlsx` BOM files.
*   **Why it matters:** It is impossible to manufacture or purchase components for the PCBs without the passive part specifications.
*   **Required evidence:** `4_7_Production Files/Gerber_*/*.xlsx`
*   **Expected output:** A flat, human-readable list of all reference designators and exact MPNs.
