# AERIS-10 Reverse Engineering — Current Project Status

## Meta
- **Date:** 2026-09-03
- **Phase:** 0 (Evidence Extraction)
- **Status:** Blocked on binary file parsing (`.xlsx`, `.dwg`)
- **Canonical Hub:** `[[00_Executive_Summary]]`
- **Execution Plan:** `[[10_Reverse_Engineering_Plan]]`

## System Identity
- 10.5 GHz LFM phased array radar.
- Variants: 3km Nexus (Patch/1W), 20km Extended (Waveguide/10W).
- Hybrid architecture: Host Python GUI <--> FT2232H <--> FPGA (High Speed) | STM32F746 (OOB Control).

## Subsystem Readiness
- **Digital Protocols:** Complete (SPI/I2C/USB/GPIO mapped).
- **FPGA RTL:** Complete (Synthesis verification pending).
- **MCU Firmware:** Complete (Wait on exact MCU MPN).
- **RF/Hardware:** Blocked (Missing component values and dimensions).

## Critical Blockers & Unknowns
1. **UNK-001 / RE-001:** `BOM_Main_Board.xlsx` extraction required for passives.
2. **UNK-002 / RE-002:** Mechanical `.dwg` extraction required for chassis.
3. **UNK-010 / RE-003:** Input DC constraints (12V vs 24V vs 48V) required from `PowerBoard.sch`.

## Instructions for Future AI Assistant
You are stepping into a heavily documented, evidence-based reverse engineering project.
1. DO NOT assume unknown values; read `[[09_Unknowns_and_Hypotheses]]`.
2. Start your work by pulling the tasks labeled `RE-001` and `RE-002` from `[[10_Reverse_Engineering_Plan]]`.
3. If writing code to parse binary files, use the `scratch/` directory.
