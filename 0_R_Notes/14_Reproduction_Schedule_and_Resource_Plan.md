---
type: reverse-engineering-note
status: active
domain: system
confidence: mixed
canonical: true
---

# Reproduction Schedule and Resource Plan

> **Related notes:** [[10_Reverse_Engineering_Plan]] · [[09_Unknowns_and_Hypotheses]] · [[08_Component_Inventory]]
> **Document type:** Master Reproduction Resource Model
> **Status:** Living Document

This document defines the engineering effort, timeline, equipment, budget, and personnel required to reproduce the AERIS-10 radar system from the current repository state into a fully validated physical system.

---

## 1. Reproduction Scenarios

### Scenario A — Minimum Functional Replica
**Goal:** Demonstrate the core architecture and obtain a functioning radar-like system using justified engineering substitutions where necessary.
* **Scope:** 3km Nexus variant only.
* **Substitutions:** Use COTS evaluation boards for the AD9523-1, ADF4382, and ADAR1000 if custom RO4350B PCB fabrication fails or passives cannot be recovered. Use off-the-shelf patch antennas.

### Scenario B — High-Fidelity Replica
**Goal:** Reproduce the original architecture, components, PCBs, firmware, FPGA, RF chain, antenna, and software as closely as evidence permits.
* **Scope:** 3km Nexus variant.
* **Substitutions:** None permitted. Requires exact recovery of `BOM_Main_Board.xlsx` (UNK-001) and precise RO4350B dielectric stackup matching.

### Scenario C — Production-Reproducible Replica
**Goal:** Enable an independent engineer/team to reproduce the system repeatedly using documented manufacturing, programming, calibration, and validation processes.
* **Scope:** 3km Nexus and 20km Extended variants.
* **Requirements:** Full mechanical CAD recovery (UNK-002), automated test scripts, thermal profiling, and complete RF calibration tables.

---

## 2. Work Breakdown Structure (WBS)

1. **Phase 0: Evidence Closure**
   1. Extract Binary BOMs (RE-001)
   2. Extract Mechanical CAD (RE-002)
   3. Resolve Power Specs (RE-003)
2. **Phase 1: Digital Reconstruction**
   1. MCU Firmware Reconstruction (RE-004)
   2. FPGA RTL Synthesis & Timing Closure (RE-005)
   3. Host Software Setup
3. **Phase 2: Hardware Procurement & Fabrication**
   1. Component Sourcing (Passives, Active RF ICs, GaN PAs)
   2. PCB Fabrication (10-layer RO4350B Main Board, Antenna, PA boards)
   3. Mechanical CNC Machining (Waveguide, Enclosure, Heatsink)
   4. SMT Assembly
4. **Phase 3: Hardware Bring-Up**
   1. Power Subsystem Sequencing (1.0V -> 1.8V -> 3.3V)
   2. Clock Subsystem Synchronization (AD9523-1)
   3. MCU/FPGA JTAG Activation
5. **Phase 4: RF Bring-Up & Calibration**
   1. LO/Mixer Chain Validation
   2. GaN PA Negative Bias Calibration (DAC5578)
   3. ADAR1000 Phase/Gain Beam Table Calibration
6. **Phase 5: System Integration & Validation**
   1. Software/Hardware Loop Validation
   2. RF EIRP and Phase Noise Characterization
   3. Field Testing
   4. Independent Reproduction Documentation

---

## 3. Task-Level Resource Requirements

| Task | Role | Skills | Equipment | Materials | Software | External Service | Duration | Dependencies |
| ---- | ---- | ------ | --------- | --------- | -------- | ---------------- | -------- | ------------ |
| **BOM Extraction (RE-001)** | Systems Eng | Data Parsing | PC | None | Python/Pandas | None | 1–3 days | None |
| **FPGA Synthesis (RE-005)** | FPGA Eng | Verilog, Timing | PC | None | Xilinx Vivado | None | 1–2 weeks | None |
| **PCB Fabrication** | Hardware Eng | Altium/Gerber | None | RO4350B/FR4 | CAM350 | PCB Fab House | 3–5 weeks | RE-001, RE-003 |
| **SMT Assembly** | Hardware Eng | DFM, SMT | None | PCB, Components | None | PCBA House | 2–4 weeks | PCB Fab |
| **Power Bring-up** | Hardware Eng | Debugging | DMM, Oscilloscope | PCBA | None | None | 1–2 weeks | SMT Assembly |
| **RF Bring-up** | RF Eng | RF Theory | Spectrum Analyzer | PCBA, Cables | None | None | 2–3 weeks | Power Bring-up |
| **Antenna Match** | RF Eng | Microwave | VNA | Antenna PCBA | None | None | 1–2 weeks | SMT Assembly |

---

## 4. Human Resources

Determine the smallest credible team (1–3 people).

* **Reverse-Engineering Lead / Hardware Engineer:** (Required full-time). Owns system architecture, unblocks binary evidence, verifies PCB layouts, manages external PCBA/Fab houses, and performs power bring-up.
* **Firmware / FPGA Engineer:** (Required part-time). Recompiles STM32 HAL, synthesizes Vivado bitstreams, tests FT2232H Python GUI bindings.
* **RF / Test Engineer:** (Required part-time). Conducts VNA S-parameter measurements, phase noise analysis, and creates ADAR1000 calibration tables. 

*Note: For Scenario A, a single senior cross-functional engineer could cover all roles. Scenario C requires dedicated RF and FPGA domain experts due to strict calibration requirements.*

---

## 5. Equipment Requirements

### Required Equipment

#### Digital
* **Must Own:** 4-Channel Mixed Signal Oscilloscope (>500 MHz bandwidth for clock inspection), STM32 ST-LINK V2/V3, Xilinx Platform Cable USB II (or Digilent JTAG).
* **Optional:** High-speed Logic Analyzer.

#### RF
* **Must Own or Rent:** Spectrum Analyzer (Minimum 12 GHz bandwidth), Vector Network Analyzer (VNA) (Minimum 12 GHz).
* **Must Own:** Low-loss SMA/SMP cables, precision attenuators, DC blocks.
* **Optional / Rent:** RF Signal Generator, RF Power Meter.

#### Electrical
* **Must Own:** Bench Power Supply (Programmable, current-limiting), precision Digital Multimeter (DMM).
* **Optional:** FLIR/Thermal Camera (for GaN PA heat dissipation validation).

#### Mechanical
* **Must Own:** Digital calipers.
* **Outsource:** CNC milling machines.

---

## 6. Manufacturing Resources

* **PCB Fabrication:** **Outsource**. The Main Board is a 10-layer mixed-dielectric (RO4350B/FR4) impedance-controlled board. Requires professional fabrication.
* **PCB Assembly (SMT):** **Outsource**. 500+ passives, BGA FPGAs, and QFN RF ICs require machine pick-and-place and reflow.
* **CNC Machining:** **Outsource**. Waveguide and heatsink require professional milling.
* **Cable/Harness Assembly:** **In-house**. Crimp and assemble ribbon/coax cables manually.

---

## 7. Software Infrastructure

* **FPGA Toolchain:** Xilinx Vivado (Requires validation if WebPACK is sufficient for XC7A50T).
* **MCU Toolchain:** STM32CubeIDE / GCC ARM.
* **Host Software:** Python 3.x, PyQt6, pyftdi.
* **Simulation (Optional):** SymbiYosys (Formal Verification), CST Microwave Studio / HFSS (Antenna modeling).

---

## 8. Material Requirements

See `[[08_Component_Inventory]]` for the canonical BOM.

* **Semiconductors:** XC7A50T FPGA, STM32F746 MCU, AD9523-1, ADF4382, ADAR1000, QPA2962, DAC5578.
* **Passives:** 500+ exact value resistors/capacitors/inductors (Currently blocked by UNK-001).
* **PCB Material:** Rogers RO4350B high-frequency laminate.
* **Mechanical:** Custom machined aluminum heatsink/waveguide, Stepper motor, Slip-ring (UNK-004).

---

## 9. Schedule Model

### Parallelizable Work
- **Phase 0 & 1:** MCU/FPGA/Software compilation and code review can happen in parallel with BOM/CAD extraction and component sourcing.
- **Phase 2:** PCB Fab and CNC machining can occur concurrently.

### Sequential Work
```text
Evidence Closure (UNK-001, UNK-002)
      │
 ┌────┼─────────┐
 ↓    ↓         ↓
BOM  FPGA      MCU
 │    │         │
 ↓    └────┬────┘
PCB        ↓
 │     Digital Integration
 ↓          │
Assembly ───┤
            ↓
       Power/Clock Bring-up (Critical)
            ↓
       RF Bring-up
            ↓
       Calibration & System Test
```

---

## 10. Critical Path

1. **Extract Binary BOM (`BOM_Main_Board.xlsx`)** (Zero Slack)
2. **Order Components / Fab 10-Layer RO4350B PCB** (Bottleneck: Fab Lead Times)
3. **PCBA SMT Assembly** (Bottleneck: Assembly Lead Times)
4. **Power Bring-Up** (Zero Slack - RF ICs will burn out if sequenced incorrectly)
5. **RF Bring-Up & Calibration**

*Reasoning: We cannot fab the board without knowing the exact BOM footprints. We cannot assemble the board without the exact passive values. Hardware lead times strictly dominate the schedule.*

---

## 11. Timeline Scenarios

### Aggressive (Scenario A / Experienced Team)
* **Duration:** 8–10 weeks.
* **Assumptions:** Binary files extract cleanly, components are in-stock, PCB fab/assembly takes 4 weeks, zero hardware respins required.

### Realistic (Scenario B)
* **Duration:** 16–20 weeks.
* **Assumptions:** Procurement delays for exotic RF ICs (ADAR1000, QPA2962), one mandatory PCB respin (Revision B) due to minor footprint/RF matching errors, normal iteration on firmware.

### Conservative (Scenario C)
* **Duration:** 6–9 months.
* **Assumptions:** High-fidelity calibration requires multiple field tests, CNC machining requires iteration, FPGA requires reverse-engineering of missing proprietary Xilinx IP.

---

## 12. Cost Model

* **Engineering Labor:** *Estimated* (1-3 Full Time Equivalents for 3-6 months).
* **Components:** *Estimated* ($2k - $4k per board due to exotic RF ICs).
* **PCB Fab & Assembly (Prototype Qty):** *Estimated* ($3k - $6k per run for 10-layer Rogers).
* **Mechanical Fabrication:** TBD — requires sourcing quote (Based on `.dwg` complexity).
* **Test Equipment:**
  * Basic (Oscilloscope, Logic Analyzer): $1k - $3k (Buy).
  * RF (VNA, Spectrum Analyzer): $1k - $3k/month (Rent).
* **Contingency:** 100% markup on PCB/Component costs to allow for 1 mandatory respin.

---

## 13. Equipment Acquisition Strategy

### Minimum Equipment Configuration (Functional Replica)
* **Buy:** DMM, Bench Supply, ST-Link, Xilinx JTAG, 4-Ch Oscilloscope.
* **Rent:** Basic Spectrum Analyzer (to verify LO presence).

### Full Characterization Equipment (High-Fidelity)
* **Rent:** 12+ GHz VNA (for antenna return loss), High-End Spectrum Analyzer (for phase noise and spurious emissions).
* *Trade-off:* Buying a 12 GHz VNA/SA costs >$50k. Renting costs ~$2k/month. For a 3-month RF bring-up phase, renting is mandatory unless existing lab access is secured.

---

## 14. Iteration Allowance

Do not assume first-pass hardware success.

## Iteration Risk
* **High Risk:** RF PCB Layout. Microstrip impedance mismatch or cross-talk is highly likely on a dense 10.5 GHz 10-layer board. **Budget for 1 full PCB respin.**
* **Medium Risk:** Power Sequencing. GaN PAs are notoriously fragile if Vg (bias) and Vd (drain) are sequenced incorrectly. **Budget for sacrificing 1-2 PA boards during initial bring-up.**
* **Low Risk:** MCU/FPGA/GUI software bugs. Can be iterated via JTAG/USB with zero material cost.

---

## 15. Resource Bottlenecks

1. **RF Equipment (VNA/SA):** Extremely expensive to rent; schedule is blocked if rentals are delayed.
2. **Obsolete / Exotic Components:** ADAR1000 and QPA2962 may have 26-52 week lead times depending on global supply chain.
3. **Specialized RF Fabrication:** Finding a PCBA house capable of handling Rogers RO4350B reliably at prototype quantities.

---

## 16. Procurement Lead-Time Dependencies

The following canonical components from `[[08_Component_Inventory]]` govern the schedule:
* **ADAR1000 (Beamformer):** Exact MPN required. *Long-lead candidate.*
* **QPA2962 (GaN PA):** Exact MPN required. *Long-lead candidate.*
* **STM32F746xx:** Exact suffix (UNK-003) required to determine physical footprint before ordering.
* **Passives:** Currently blocked by UNK-001.

---

## 17. Reproduction Gates

### Gate 0 — Evidence Closure
* **Entry:** Current state.
* **Exit:** `BOM_Main_Board.xlsx` and `*.dwg` successfully parsed into plaintext.

### Gate 1 — Digital Build Reproducible
* **Exit:** FPGA `.bit` and MCU `.elf` compile cleanly from source with zero missing IP.

### Gate 2 — Hardware Fabrication Authorized
* **Exit:** BOM sourced, gerbers sent to Fab, CNC files sent to machine shop.

### Gate 3 — Power Bring-Up Ready
* **Exit:** PCBA received. Short-circuit tests passed. 1.0V/1.8V/3.3V/5V/-5V rails measure correctly *before* RF ICs are populated/enabled.

### Gate 4 — RF Activation Ready
* **Exit:** MCU successfully sequences DAC5578 to provide -5V pinch-off bias to PA gates before drain voltage is applied.

### Gate 5 — Functional Replica
* **Exit:** Host Python GUI visualizes ADC data indicating basic LFM chirps.

---

## 18. Minimum Team

* **Functional Replica:** 1 Senior Systems/Hardware Engineer.
* **High-Fidelity Replica:** 1 Hardware Engineer + 1 RF Engineer (part-time).
* **Production-Reproducible Replica:** 1 Hardware Eng + 1 RF Eng + 1 Test/Documentation Eng.

---

## 19. Timeline Summary

| Scenario | Team | Estimated Duration | Critical Path | Major Assumptions | Main Risks |
| -------- | ---- | -----------------: | ------------- | ----------------- | ---------- |
| Functional | 1 | 8–12 weeks | PCB Fab & SMT Assembly | COTS substitutions allowed | RF matching fails |
| High Fidelity | 2 | 16–20 weeks | PCBA -> Power -> RF | All BOM recovered | PA burnout, Long-lead ICs |
| Production | 3 | 6–9 months | RF Calibration & Test | Full CAD recovered | Certification / Tolerances |

---

## 20. Resource Summary

| Resource | Minimum | Recommended | Full Characterization |
| -------- | ------- | ----------- | --------------------- |
| **Engineers** | 1 (Cross-functional) | 2 (HW + RF) | 3 (HW + RF + Test) |
| **RF Equipment** | Basic SA (Rent) | VNA + SA (Rent) | VNA + SA + Power Meter (Own/Rent) |
| **Digital Equipment** | DMM, ST-Link | 4-Ch Scope, JTAG | Logic Analyzer, Thermal Imager |
| **Manufacturing** | COTS EVBs | Prototyping PCBA | ITAR/ISO Certified PCBA |
| **Software/Tools** | Vivado WebPACK | Vivado Standard | CST Microwave Studio |
| **External Services** | Cheap PCB Fab | Rogers/RF PCBA | RF Anechoic Chamber Testing |

---

## 21. What We Can Do Now (Immediate Low-Cost Work)

We can proceed immediately without hardware fabrication, expensive equipment, or component procurement by executing the Phase 0 and Phase 1 tasks from `[[10_Reverse_Engineering_Plan]]`:

1. **RE-001:** Extract the Main BOM locally using Python/Pandas.
2. **RE-002:** Extract the CAD Data locally.
3. **RE-004 / RE-005:** Compile the STM32 Firmware and synthesize the FPGA RTL to verify toolchain integrity.

---

## 22. What We Must Buy / Build / Outsource

### Buy
* **Electronic Components:** All ICs and passives (Standard supply chain).
* **Debug Tools:** ST-Link, Xilinx JTAG, Oscilloscope, DMM (Required for bench bring-up).
* **Test Equipment Rental:** VNA and Spectrum Analyzer (Buying is cost-prohibitive for prototypes).

### Build
* **Cable Harnesses & Assembly:** Manually crimping and integrating boards in-house ensures careful staged power-up.
* **Test Fixtures:** 3D print simple jigs to hold the antenna and waveguide during RF testing.

### Outsource
* **PCB Fabrication & SMT Assembly:** 10-layer Rogers RO4350B boards with BGA/QFN components cannot be fabricated or soldered reliably by hand.
* **CNC Machining:** Waveguide and enclosure demand precision metal milling.

---

## 23. Decision Points

1. **Component Obsolescence:** If ADAR1000 or QPA2962 are unavailable, do we redesign the RF front-end (adds 3-4 months) or pause the project?
2. **BOM Recovery Failure:** If `BOM_Main_Board.xlsx` is permanently corrupted, do we abandon the High-Fidelity replica and attempt a Scenario A build using COTS evaluation boards?
3. **FPGA Licensing:** If Vivado Synthesis requires proprietary RF IP, do we rewrite the open-source equivalents or purchase the Xilinx license?

---

## 24. Recommended Reproduction Strategy

1. **Recommended Scenario:** Scenario B (High-Fidelity Replica) for the 3km Nexus variant. It balances realistic hardware costs while proving the core LFM phased array architecture.
2. **Minimum Credible Team:** 2 Engineers (1 Hardware/Firmware + 1 RF).
3. **Immediate Resource Requirements:** A workstation with Python, Vivado, and STM32CubeIDE to close Gate 0 and Gate 1.
4. **Critical Equipment:** 4-Channel Oscilloscope (Buy), 12GHz VNA (Rent).
5. **Major External Services:** RF-capable PCBA house.
6. **Critical Path:** Evidence Extraction -> Hardware Procurement -> PCBA -> Staged Power/RF Bring-Up.
7. **Main Uncertainties:** Unparsed binary BOM passives (UNK-001).
8. **Recommended Contingency:** 100% PCB/Component budget markup to absorb 1 mandatory PCB respin (Revision B) and the likely loss of 1-2 GaN PA boards during bias testing.
9. **Pre-Fabrication Conditions (Gate 2):** FPGA bitstream must compile flawlessly, and exact DC input voltage (UNK-010) must be confirmed.
10. **Reproduction Success Criteria:** The host Python GUI successfully visualizes range/doppler targets using the newly fabricated hardware array.
