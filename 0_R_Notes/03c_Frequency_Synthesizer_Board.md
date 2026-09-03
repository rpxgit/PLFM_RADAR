# Frequency Synthesizer Board

> **Related notes:** [[03_Hardware]] · [[13_RF_Signal_Chain]]
> **Status:** Read-only Forensic Snapshot

---

## 1. Board Identity

*   **PCB Name:** `Clocks_Freq_Synth_board`
*   **Domain:** Timing and Local Oscillator Generation
*   **Function:** Generates phase-aligned digital clocks for the DSP pipeline and high-frequency Local Oscillators (LO) for the RF mixers.

---

## 2. Schematic Analysis

| Block | Components | Inputs | Outputs | Interface | Function | Evidence | Confidence |
| ----- | ---------- | ------ | ------- | --------- | -------- | -------- | ---------- |
| **Clock Gen** | AD9523-1 | Reference / MCU SPI | Digital Clocks | SPI, Diff Pairs | Generates clocks for FPGA, ADC, DAC | `README.md`, `.sch` | Confirmed |
| **RF Synthesizer** | ADF4382 (x2) | AD9523 Clock, MCU SPI | LO RF Signal | SPI, SMA | Generates TX and RX LOs for up/down conversion | `README.md`, `.sch` | Confirmed |

---

## 3. Signal Flow

```text
Reference Clock (On-board or External)
      │
      ▼
   AD9523-1 ──► [Clock Out] ──► (To Main Board: FPGA, ADC, DAC)
      │
      ▼
   ADF4382 ──► [LO Out] ──► (To Main Board: LTC5552 Mixers)
```

---

## 4. Hardware ↔ Firmware Correlation

*   The STM32F746 configures both the AD9523 and ADF4382 via SPI over board-to-board headers.
*   Firmware must initialize the AD9523 first to provide reference clocks before initializing the ADF4382 or the FPGA.

---

## 5. Replication-Critical Features

*   **Phase Alignment:** The traces leaving the AD9523 must be strictly length-matched to ensure phase coherence across the DAC and ADC, which is critical for pulse compression and Doppler processing.
*   **Impedance Control:** RF LO traces from the ADF4382 must be strictly 50Ω.
*   **Noise Isolation:** The board must be shielded from the switching noise of the Power Board.

---

## 6. Unknowns & Next Steps

| ID | Unknown | Why It Matters | Evidence Available | Required Investigation |
| -- | ------- | -------------- | ------------------ | ---------------------- |
| FSB-01 | Reference Clock | Does it use a TCXO, OCXO, or external reference? | `.sch` | Parse schematic. |
| FSB-02 | Physical Routing | Are the clocks routed via cables (SMA/u.FL) or board-to-board headers? | `.sch`, `.dwg` | Parse schematic/mechanicals. |
