---
type: reverse-engineering-note
status: active
domain: system
confidence: high
canonical: true
---

# Systems Reproduction Readiness & Technical Gate Review

> **Related notes:** [[00_Executive_Summary]] · [[08_Component_Inventory]] · [[09_Unknowns_and_Hypotheses]] · [[10_Reverse_Engineering_Plan]] · [[14_Reproduction_Schedule_and_Resource_Plan]]
> **Document type:** Master Go/No-Go Gate Framework
> **Status:** Living Document

This document is a strict, evidence-first **go/no-go framework** designed to prevent the premature expenditure of resources on hardware fabrication, RF activation, or system integration while critical uncertainties remain unresolved.

---

## 1. Readiness Model

The physical and digital reproduction of the AERIS-10 system must pass through the following sequential readiness levels:

* **R0 — Evidence Ready:** Repository and documentation evidence is sufficiently understood to begin reconstruction planning.
* **R1 — Digital Reconstruction Ready:** MCU, FPGA, host software, and protocol reconstruction is sufficiently understood for build/testing.
* **R2 — Hardware Fabrication Ready:** BOM, PCB, mechanical, and manufacturing evidence is sufficient to authorize fabrication.
* **R3 — Hardware Bring-Up Ready:** Power, clocks, reset, programming, and basic instrumentation are sufficiently understood for safe bring-up.
* **R4 — RF Activation Ready:** RF chain, biasing, clocking, interlocks, and measurement setup are sufficiently understood for RF activation.
* **R5 — System Integration Ready:** Hardware, firmware, FPGA, communications, and host software can be integrated.
* **R6 — Calibration Ready:** Required calibration mechanisms and measurement infrastructure are sufficiently understood.
* **R7 — Functional Replica Ready:** The system demonstrates intended core operation.
* **R8 — Performance Validation Ready:** The system can be characterized against defined technical metrics.
* **R9 — Reproduction Ready:** An independent second build can be performed using the resulting documentation.

---

## 2. Gate Criteria

| Gate | Entry Criteria | Required Evidence | Blocking Unknowns | Required Resources | Exit Criteria | Status |
| ---- | -------------- | ----------------- | ----------------- | ------------------ | ------------- | ------ |
| **R0** | Repo accessible | Code, Schematics, Layouts | None | Engineering time | Knowledge base mapped | **PASS** |
| **R1** | R0 Exit | HDL, C++ Source, Makefiles/Projects | UNK-003, UNK-005 | Vivado, CubeIDE | Bitstream & ELF compiled | **CONDITIONAL** |
| **R2** | R1 Exit | Exact BOM, CAD files, Gerber files | UNK-001, UNK-002, UNK-010 | PCBA Fab, CNC Fab | Purchase Orders issued | **BLOCKED** |
| **R3** | R2 Exit | Power schematics, Bring-up procedure | UNK-010 | DMM, Oscilloscope | Rails verified safely | NOT ASSESSED |
| **R4** | R3 Exit | PA bias sequences, LO clock config | None | Spectrum Analyzer | Bias verified, LO present | NOT ASSESSED |
| **R5** | R4 Exit | Protocol opcodes, GUI source | None | Host PC, JTAG | End-to-end data flow | NOT ASSESSED |
| **R6** | R5 Exit | Phase/Gain tables, Calibration scripts | Thermistor Specs (UNK-007) | VNA | Calibration saved | NOT ASSESSED |
| **R7** | R6 Exit | Baseline LFM parameters | None | Test targets | Target detected | NOT ASSESSED |
| **R8** | R7 Exit | Benchmark specifications | None | RF Anechoic Chamber | Range/Resolution verified | NOT ASSESSED |
| **R9** | R8 Exit | Complete assembly/test manual | None | Documentation Eng | System reproducible | NOT ASSESSED |

---

## 3. Current Gate Assessment

* **R0 — PASS:** The digital architecture, communication protocols, and sub-assemblies have been rigorously mapped into `0_R_Notes`.
* **R1 — CONDITIONAL:** We possess the FPGA and MCU source code, but we cannot pass this gate until `RE-004` (STM32 MPN verification) and `RE-005` (Vivado synthesis verification) are executed to prove the code builds flawlessly without missing proprietary IP.
* **R2 — BLOCKED:** Hardware Fabrication is hard-blocked. We cannot authorize PCB assembly without extracting the exact passive component values via `RE-001` (BOM) and `RE-003` (Power Specs), nor machine the chassis without `RE-002` (CAD).
* **R3 through R9 — NOT ASSESSED:** These downstream gates remain inaccessible until R2 is passed.

---

## 4. Hard Gates vs Soft Gates

### Hard Gate
Must be strictly satisfied before proceeding due to high financial cost or risk of hardware damage.
* **R2 (Hardware Fabrication):** Cannot be bypassed. Ordering PCBs with missing BOM data guarantees a failed board.
* **R4 (RF Activation):** Cannot be bypassed. Activating the GaN PAs without correct negative biasing will immediately destroy the QPA2962 chips.

### Soft Gate
Can remain unresolved if an explicit mitigation exists.
* **R1 (Digital Reconstruction):** Soft gate. We can proceed with hardware fabrication (R2) even if there are minor firmware compilation bugs, because firmware can be flashed and debugged iteratively via JTAG on the finished board.

### Informational
Useful but does not block physical progression.
* **R0 (Evidence Ready):** Informational boundary. 

---

## 5. P0/P1 Unknown Gate Mapping

| Unknown | Description | Gate Affected | Severity | Resolution Task | Can Be Mitigated? | Gate Impact |
| ------- | ----------- | ------------- | -------- | --------------- | ----------------- | ----------- |
| **UNK-001** | Binary `BOM_Main_Board.xlsx` | **R2** (Fabrication) | P0 | RE-001 | No | **Hard Blocker** |
| **UNK-002** | Mechanical `.dwg` | **R2** (Fabrication) | P0 | RE-002 | No | **Hard Blocker** |
| **UNK-010** | Power Supply Specs | **R2** / **R3** (Bring-up) | P0 | RE-003 | No | **Hard Blocker** |
| **UNK-003** | STM32F746 suffix | **R2** (Fabrication) | P1 | RE-004 | No (Footprint needed) | **Hard Blocker** |
| **UNK-005** | FPGA Boot Mechanism | **R3** (Bring-up) | P1 | Schematic Review | Yes (Manual JTAG boot) | Soft Blocker |

---

## 6. Evidence Sufficiency

Evidence for clearing the gates is classified as follows:

* **Direct:** Schematics, source code, Gerber files. (Required for Hard Gates).
* **Corroborated:** Protocol behavior matched against Verilog state machines. (Acceptable for Soft Gates).
* **Inferred:** Assuming the FPGA boots from SPI Flash based on XC7A50T standards. *(Do not allow inference to satisfy a hard gate like R3 or R4).*
* **Missing:** `BOM_Main_Board.xlsx` contents. (Currently causing the R2 BLOCKED state).
* **Contradictory:** README claims positive-only rails; BOM lists LM2662 negative voltage inverters. *(Resolved in favor of Direct BOM evidence).*

---

## 7. Hardware Fabrication Authorization Gate (R2)

To commit to 10-layer PCB fabrication and CNC machining, the following must be unconditionally known and documented:

* **BOM:** 100% of passives (resistors, capacitors, inductors) assigned values, footprints, and tolerances.
* **PCB Files:** Gerbers matched to the exact BOM footprint list.
* **PCB Revision:** Base Nexus (Patch) vs Extended (Waveguide) explicitly selected.
* **Mechanical:** `.dwg` dimensions extracted and validated for Waveguide flange mating.
* **Power Architecture:** Input DC voltage and current capability defined.
* **Component Availability:** AD9523, ADF4382, ADAR1000, and QPA2962 located in stock.
* **Programming Strategy:** JTAG/SWD headers confirmed on layout for MCU and FPGA.
* **Known High-Risk Assumptions:** None permitted at this gate.

**Current Authorization Status:** **REJECTED.** Prevented by UNK-001, UNK-002, UNK-003, and UNK-010.

---

## 8. RF Activation Gate (R4)

Before enabling RF power/transmission, the following prerequisites must be met to prevent equipment destruction:

* **Negative PA Bias:** The MCU *must* demonstrate it can configure the DAC5578 to output the deep-pinch-off negative gate voltage (Vg) *before* the drain voltage (Vd) MOSFETs are enabled.
* **Drain Supply:** Current-limited bench supply confirmed holding steady under bias.
* **Clock Verification:** AD9523-1 outputs confirmed locked and stable via oscilloscope.
* **Beamformer Configuration:** ADAR1000 SPI writes confirmed via Logic Analyzer.
* **Antenna/Load:** Patch antenna or 50Ω dummy loads confirmed physically connected to all active RF ports.
* **Thermal Monitoring:** ADS7830 thermistor feedback loop verified active.

---

## 9. Calibration Gate (R6)

Before full phased array calibration can begin, the following must be understood:

* **Calibration Target:** Corner reflector or known RCS target positioned in far-field.
* **Measurement Setup:** VNA/SA synchronized with host GUI trigger.
* **Calibration Coefficients:** Understanding of how the ADAR1000 RAM stores phase/gain tables.
* **Storage:** Where calibration vectors are permanently stored (STM32 Flash vs EEPROM vs Host PC).
* **Software Interaction:** GUI opcode for pushing calibration arrays to hardware.

*Status:* **NOT ASSESSED.** Exact calibration sequence is undocumented in the current repo evidence.

---

## 10. Independent Reproduction Gate (R9)

The claim “This system is reproducible” can only be made when the following artifacts are generated and proven by an independent second build:

* Complete, plaintext BOM with verified MPNs.
* Verified manufacturing package (Gerbers, Drill files, CNC STEP files).
* Unambiguous firmware and FPGA build instructions (including specific toolchain versions).
* Host software setup guide.
* Hardware assembly and torque procedure (specifically for RF waveguides and heatsinks).
* JTAG programming procedure.
* Functional test and troubleshooting tree.

**Acceptance Criterion:** A third-party engineer, given only the generated R9 documentation package and funds, can build a working AERIS-10 radar without contacting the original developers.

---

## 11. Current Readiness Blockers

Ranked by severity and cost of proceeding prematurely:

1. **Unparsed Binary BOM (`BOM_Main_Board.xlsx`) [UNK-001]:** Absolute highest severity. We cannot buy parts or assemble the board. Attempting to guess passive values on a 10.5 GHz RF board guarantees failure.
2. **Undefined Power Constraints [UNK-010]:** Supplying 24V to a 12V system will immediately destroy the board. Must be resolved before R2.
3. **Unparsed Mechanical CAD (`*.dwg`) [UNK-002]:** Prevents manufacturing the enclosure and waveguide.
4. **Unverified STM32 Suffix [UNK-003]:** We cannot order the MCU without knowing the exact package, risking a PCB footprint mismatch.
5. **Unverified FPGA Synthesis [UNK-005]:** Risk of discovering proprietary/licensed Xilinx IP late in the build cycle.

---

## 12. Current Gate Decision

* **Highest Gate Currently Passed:** **R0 (Evidence Ready)**.
* **Next Gate:** **R1 (Digital Reconstruction Ready)** and **R2 (Hardware Fabrication Ready)**.
* **Conditions Required to Pass R2:** Successful extraction of the binary Excel BOMs and AutoCAD DWGs, plus verification of the power supply requirements via schematic review.
* **Current Blockers:** Lack of plaintext BOM, mechanical dimensions, and power input specifications.
* **Evidence Still Required:** Executing investigations `RE-001` through `RE-005`.
* **Fabrication Decision:** **DO NOT PROCEED WITH FABRICATION.** The project is strictly blocked at the R2 gate. Immediate engineering effort must be focused entirely on binary artifact extraction (RE-001/RE-002).
