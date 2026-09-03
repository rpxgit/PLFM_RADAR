---
type: reverse-engineering-note
status: active
domain: fpga
confidence: mixed
canonical: true
---

# FPGA Constraints and Target Analysis

> **Related notes:** [[04_Firmware_FPGA]]
> **Status:** Read-only Forensic Snapshot

---

## 1. Target Device Identification

### Documentation vs Reality Contradiction
*   **Documentation Claim:** Prior constraints and general project READMEs occasionally implied a 100T or 200T FPGA target.
*   **Actual Implementation Evidence:** `constraints/xc7a50t_ftg256.xdc` and the wrapper `radar_system_top_50t.v` explicitly override this. The physical board uses the **Artix-7 XC7A50T**.

### Production Target
*   **Family:** Xilinx Artix-7
*   **Part Number:** XC7A50T-2FTG256I
*   **Package:** FTG256 (256-ball BGA)
*   **Speed Grade:** -2
*   **Temperature Grade:** Industrial (I)
*   **Evidence:** `xc7a50t_ftg256.xdc` lines 4-11.

---

## 2. I/O Bank Configuration

The constraint file explicitly maps VCCO voltages to specific physical banks, aligning perfectly with the sub-circuits they drive.

| Bank | VCCO | IOSTANDARD | Primary Functions | Evidence |
| ---- | ---: | ---------- | ----------------- | -------- |
| **0** | 3.3V | LVCMOS33 | JTAG, Flash CS | `xc7a50t_ftg256.xdc` |
| **14** | 2.5V | LVDS_25 | ADC inputs (400MHz LVDS) | `xc7a50t_ftg256.xdc` |
| **15** | 3.3V | LVCMOS33 | DAC, SPI 3.3V (STM32), Mixer enables | `xc7a50t_ftg256.xdc` |
| **34** | 1.8V | LVCMOS18 | ADAR1000 SPI/Control (Level shifted) | `xc7a50t_ftg256.xdc` |
| **35** | 3.3V | LVCMOS33 | FT2232H USB 2.0 FIFO (15 signals) | `xc7a50t_ftg256.xdc` |

---

## 3. Physical & DRC Constraints

The repository reveals several hardware-level issues that were resolved via DRC waivers in the implementation scripts (`build_50t_test.tcl`).

*   **BIVC-1 Waiver (Bank 14 VCCO Mismatch):** Bank 14 receives LVDS signals (requiring `LVDS_25`), but the `adc_pwdn` output shares the bank. Because 7-series input buffers are VCCO-independent, the DRC was demoted to a Warning safely.
*   **NSTD-1 / UCIO-1 Waiver (Unconstrained Ports):** The top-level module `radar_system_top.v` contains ports for a 32-bit FT601 USB 3.0 controller. The 50T wrapper leaves these unconnected because the 50T board uses the 8-bit FT2232H instead. These unmapped ports are demoted to Warnings.
*   **PLIO-9 Clock Pin Workaround:** The FT2232H CLKOUT was routed to a Negative (N-type) MRCC pin instead of a Positive (P-type) pin. The FPGA correctly uses the `IBUFG`, but a warning waiver was required.

---

## 4. Replication Impact

**CRITICAL:** The XC7A50T in the FTG256 package has severe I/O limitations (only 170 user I/Os, and this design uses almost all of them across 4 different voltage domains). 
Any hardware replication attempt **must** strictly adhere to this exact pin mapping and voltage assignment, as the PCB cannot easily absorb pin swaps across voltage domains.

The constraints completely validate the finding that the production AERIS-10 board is heavily I/O constrained and relies on the FT2232H (USB 2.0) rather than the FT601 (USB 3.0) due to lack of available pins.


## Related Notes

- [[04_Firmware_FPGA]]
- [[03a_Main_Board]]
