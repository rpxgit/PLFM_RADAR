# AERIS-10 Reverse Engineering — Current Project Status

## Status Date
2026-09-03

## Project Objective
Perform a forensic reverse-engineering reconstruction of the open-source AERIS-10 pulse linear frequency modulated (LFM) phased array radar system to enable a fully reproducible physical and software system replica.

## Product/System Identity
AERIS-10 is a 10.5 GHz radar system existing in two variants: 3km Nexus (Patch antenna, 1W) and 20km Extended (Waveguide, 10W GaN). It uses a hybrid FPGA/MCU architecture where an FT2232H provides a high-speed data plane to a Python GUI, while an STM32F746 handles slow out-of-band management (SPI, I2C, GPIO) of RF ICs.

## Current Overall State
The digital architecture, firmware behavior, and communication protocols have been completely mapped. Hardware reproduction is currently blocked at Phase 0 due to missing binary file extractions (Excel BOM and AutoCAD DWG).

## Knowledge Base
A canonical 32-file Obsidian knowledge base (`0_R_Notes/`) is established and cross-linked. `[[00_Executive_Summary]]` serves as the primary navigation hub.

## Repository Understanding
Complete.

## Hardware Status
Partially Understood. Schematics map to digital structure, but unparsed binary `.xlsx` BOMs block physical component identification.

## FPGA Status
Mostly Understood. RTL pipeline is mapped and testbenches are present. Synthesis must be verified against the XC7A50T target to ensure no missing Xilinx IP.

## MCU Firmware Status
Mostly Understood. `main.cpp` driver usage is mapped. Needs verification of `.ioc` and linker script for complete rebuilding.

## Host Software Status
Mostly Understood. Python GUI (V7 PyQt6) and `radar_protocol.py` (FT2232H FIFO) are fully mapped.

## Communication Protocol Status
Complete. SPI, I2C, USB, and GPIO interfaces are mapped to physical pins and software opcodes.

## BOM Status
Blocked. `BOM_Main_Board.xlsx` requires external parsing.

## Power Architecture Status
Early Investigation. Rails are partially mapped but exact input DC constraints require schematic review (`RE-003`).

## RF Signal Chain Status
Partially Understood. Signal path is conceptually clear but RF measurements (phase noise, conversion loss) are missing.

## Calibration Status
Not Yet Investigated.

## Testing / Simulation Status
Mostly Understood. FPGA formal verification (SymbiYosys), co-simulations, and MCU CppUTest frameworks exist.

## Manufacturing Status
Blocked. CNC machine files (.dwg) require external parsing.

## Confirmed Facts
- GUI communicates directly with FPGA via USB FIFO (bypasses MCU).
- MCU controls power sequencing, ADAR1000 SPI, and DAC5578 PA bias.
- GaN PAs require -5V deep pinch-off bias prior to drain voltage activation.

## High-Confidence Conclusions
- The 50T board is the production target; 200T is the dev target.
- The system heavily uses the Analog Devices no-OS framework.

## Important Inferences
- The AD9523-1 provides the coherent reference clock to the entire DSP and LO chain.

## Active Hypotheses
- HYP-001: FPGA boots autonomously from SPI Flash.
- HYP-002: MCU has native debug USB CDC on FS/HS pins.

## Critical Unknowns
- UNK-001: `BOM_Main_Board.xlsx` content (passives).
- UNK-002: Mechanical `.dwg` dimensions.
- UNK-010: Power Supply input specifications.

## Replication Blockers
- Binary file extraction (BOM + CAD).

## Current Critical Path
1. Parse binary artifacts locally (`BOM_Main_Board.xlsx`, `.dwg`).
2. Fabricate bare PCBs and machine mechanical parts.
3. Perform staged power & clock hardware bring-up.

## Completed Investigation Work
- Protocol and Architecture mapping (Notes 01-08).
- Uncertainty and execution planning (Notes 09-10).

## Active Investigation Tasks
- RE-001: Extract Main BOM
- RE-002: Extract CAD Data
- RE-003: Resolve Power Specs
- RE-004: Verify STM32 MPN
- RE-005: Verify FPGA Build

## Next Recommended Actions
1. Execute `RE-001` (Parse `BOM_Main_Board.xlsx` locally).
2. Execute `RE-002` (Parse `*.dwg` locally).
3. Execute `RE-003` (Determine input DC voltage from `PowerBoard.sch`).

## Key Evidence Files
- `0_R_Notes/00_Executive_Summary.md`
- `0_R_Notes/10_Reverse_Engineering_Plan.md`
- `0_R_Notes/09_Unknowns_and_Hypotheses.md`

## Important Knowledge Base Links
- [[02_System_Architecture]]
- [[07_Communication_Protocols]]
- [[08_Component_Inventory]]

## Known Missing Artifacts
- STM32 `.ioc` / linker scripts.
- Measured RF performance data.
- System-level integration test results.

## Major Risks
- Main Board passive values unrecoverable from `.xlsx`.
- FPGA requires proprietary Xilinx IP not in repo.
- Missing clock phase noise data degrades SNR.

## Current Replication Readiness
Blocked at Phase 0 (Evidence Closure). Cannot proceed to Phase 2 (Hardware Fabrication) without resolving binary extractions.

## Instructions for Future AI Assistant
This project aims to reverse-engineer and replicate the AERIS-10 radar system from its source code repository. You are stepping into the project at Phase 0. 

**Knowledge Base Structure:**
- The canonical documentation lives in `0_R_Notes/`. 
- `[[00_Executive_Summary]]` is the navigation hub.
- `[[09_Unknowns_and_Hypotheses]]` tracks all uncertainties (`UNK-*`, `HYP-*`).
- `[[10_Reverse_Engineering_Plan]]` is the master execution roadmap tracking investigation tasks (`RE-*`). 

**Rules:**
- **DO NOT** assume unknowns are resolved unless explicitly proven by evidence.
- **DO NOT** make up electrical specifications, calibration vectors, or RF parameters.
- If you are asked to continue the investigation, your highest priority is to tackle the tasks in the "Next Recommended Actions" section above (specifically `RE-001` and `RE-002`), as these are P0 replication blockers.
