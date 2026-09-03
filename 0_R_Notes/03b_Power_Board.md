---
type: reverse-engineering-note
status: active
domain: hardware
confidence: mixed
canonical: true
---

# Power Board

> **Related notes:** [[03_Hardware]] · [[12_Power_Architecture]]
> **Status:** Read-only Forensic Snapshot

---

## 1. Board Identity

*   **PCB Name:** `PowerBoard`
*   **Domain:** Power Regulation and Sequencing
*   **Layer Count:** Unknown (Requires gerber/board parsing)
*   **Function:** Accepts main DC input and generates all required regulated voltage rails for the radar system. Sequences power under MCU control.

---

## 2. Power Architecture & Sequencing

| Rail | Nominal Voltage | Source | Load(s) | Enable / Control | Evidence | Confidence |
| ---- | --------------: | ------ | ------- | ---------------- | -------- | ---------- |
| **Main Input** | TBD | External | Regulators | Master Switch | `PowerBoard.sch` | High |
| **Digital Logic** | 3.3V, 1.8V | Main | MCU, FPGA, USB | MCU GPIO Seq | `README.md` | High |
| **RF Bias (LNA)** | TBD | Main | ADTR1107 | MCU GPIO Seq | `README.md` | High |
| **GaN PA Drain (Vd)** | High (e.g. 24V) | Main | QPA2962 | MCU GPIO Seq | `README.md` | Confirmed |
| **GaN PA Gate (Vg)** | Negative | Main | QPA2962 | MCU GPIO Seq | `README.md` | Confirmed |

*(Note: Exact voltages require parsing `Power Management V6.xlsx` or `PowerBoard.sch`)*

---

## 3. Component Roles

*   **Regulators / Converters:** (Specific ICs pending BOM/schematic extraction).
*   **Load Switches:** Enables precise timing control of downstream loads.
*   **Protection Circuitry:** Over-current/over-voltage protection (Inferred).

---

## 4. Hardware ↔ Firmware Correlation

*   The STM32F746 MCU on the Main Board enforces the power-up sequence.
*   The Power Board provides the raw rails, but the Main Board MCU decides *when* those rails reach the critical loads, especially the GaN PAs.

---

## 5. Replication-Critical Features

*   **GaN PA Protection:** The negative gate voltage ($V_g$) rail MUST be active and stable before the high-current drain voltage ($V_d$) is applied. The Power Board and MCU must work in perfect synchronization to avoid destroying the QPA2962 amplifiers.
*   **RF Noise:** Switching regulators on the Power Board must be heavily filtered before reaching the Frequency Synthesizer and RF Front-End, otherwise phase noise will degrade radar performance.

---

## 6. Unknowns & Next Steps

| ID | Unknown | Why It Matters | Evidence Available | Required Investigation |
| -- | ------- | -------------- | ------------------ | ---------------------- |
| PB-01 | Main Input Specs | What is the required voltage and current capability of the external power supply? | `PowerBoard.sch` | Parse schematic. |
| PB-02 | Power Sequencing Matrix | What is the exact millisecond sequencing required for boot? | `Power Management V6.xlsx` | Extract Excel data. |


## Related Notes

- [[03_Hardware]]
- [[05a_Power_Sequencing]]


### Reconstructed Hardware Topology (EVID-PWR1)
The Power Board utilizes a distributed Point-of-Load (PoL) architecture driven by a 12V input:
*   **Buck Converters (Qty 21):** `TPS562208` (4.5V-17V input, 2A output). Forms the primary step-down network for all 3.3V, 5V, 1.8V, and 1.0V rails.
*   **LDO Regulators (Qty 6):** `ADM7151` (800mA Ultralow Noise). Used exclusively for critical RF/Clock rails (AD9523, VCO).
*   **Negative Inverters (Qty 5):** `LM2662MX` Switched Capacitor Voltage Converters. Resolves the missing negative bias capability.
