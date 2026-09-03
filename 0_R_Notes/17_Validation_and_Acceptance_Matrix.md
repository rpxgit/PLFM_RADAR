---
type: reverse-engineering-note
status: active
domain: system
confidence: high
canonical: true
---

# Validation and Acceptance Matrix

> **Related notes:** [[02_System_Architecture]] · [[07_Communication_Protocols]] · [[15_Reproduction_Readiness_and_Gate_Review]] · [[10_Reverse_Engineering_Plan]]
> **Document type:** Master Acceptance Test Specification
> **Status:** Living Document

This document defines exactly how we will prove that the AERIS-10 replica is functionally, technically, and behaviorally equivalent to the original reconstruction target. It establishes objective acceptance criteria without inventing unverified performance specifications.

---

## 1. Validation Philosophy

The replica will be evaluated across six distinct levels of equivalence. It is understood that not all levels are currently measurable until physical baselines are established.

* **Functional Equivalence:** Does the replica perform the intended core radar functions (LFM chirp, receive, digitize, CFAR)?
* **Interface Equivalence:** Does it communicate exactly using the expected USB, SPI, I2C, and GPIO protocols?
* **Architectural Equivalence:** Does it reproduce the split host-bypassed high-speed data plane and slow MCU control plane?
* **Behavioral Equivalence:** Does it sequence power, clocks, and thermal interlocks similarly under comparable conditions?
* **Performance Equivalence:** Does it achieve comparable measurable RF and DSP performance (Target range, resolution, phase noise)?
* **Manufacturing Reproducibility:** Can another independent unit be built successfully using our generated documentation?

---

## 2. Validation Hierarchy

Testing must proceed sequentially to isolate faults.

* **V0 — Component:** Individual ICs (MCU, FPGA, PLL) function nominally.
* **V1 — Board:** Power rails, resets, and physical connectivity are correct.
* **V2 — Subsystem:** Isolated functional blocks (e.g., AD9523 clock tree) operate correctly.
* **V3 — Interface:** Communication buses (SPI/I2C/USB) transmit valid payloads.
* **V4 — Digital Integration:** Firmware, FPGA RTL, and Host GUI operate together.
* **V5 — RF Integration:** LO, mixers, and PAs amplify and transmit the digital waveform.
* **V6 — System:** The complete assembly detects physical targets.
* **V7 — Performance:** The system is characterized against metric baselines (e.g., max range).
* **V8 — Independent Reproduction:** A third party successfully builds the system.

---

## 3. Master Validation Matrix

| Test ID | Level | Domain | Requirement | Test Method | Equipment | Expected Result | Acceptance Criterion | Evidence | Related Unknown | Status |
| ------- | ----- | ------ | ----------- | ----------- | --------- | --------------- | -------------------- | -------- | --------------- | ------ |
| **VAL-001** | V1 | Power | Negative PA Bias | Scope probe on PA Vg | Oscilloscope | -5V present before Vd | Vg strictly leads Vd | Scope Trace | None | Open |
| **VAL-002** | V3 | USB | High-Speed FIFO | PyFTDI test script | PC | 100MB/s throughput | Zero dropped packets | Script Output | None | Open |
| **VAL-003** | V4 | MCU | AD9523 Init | SPI Logic capture | Logic Analyzer | PLL locks | LOCK pin asserts high | LA Capture | None | Open |
| **VAL-004** | V6 | System| Point Target | Corner reflector at 100m | GUI | Clear peak in Range bin | SNR > 15dB | GUI Screenshot | None | Open |
| **VAL-005** | V7 | RF | Output Power | Measure Tx port | Spectrum Analyzer | TBD | Measurement required — baseline not yet established | SA Trace | None | Open |

*(Note: Matrix will be expanded as physical bring-up proceeds).*

---

## 4. Component Validation (V0)

Tests prioritized for replication-critical components:
* **FPGA (XC7A50T):** JTAG chain responds, configuration memory flashes, DONE pin asserts.
* **STM32F746:** SWD chain responds, firmware flashes, HSE oscillator starts.
* **AD9523-1 (Clock):** SPI interface responds to ID read, PLL1 and PLL2 lock.
* **ADF4382 (Synth):** SPI interface responds, outputs 10.5 GHz CW tone.
* **ADAR1000 (Beamformer):** SPI interface responds, bias DACs output voltage.
* **QPA2962 (GaN PA):** Idq quiescent current matches datasheet under -5V bias.

---

## 5. Board-Level Validation (V1)

### Main Board
* All switching regulators provide correct output voltages (1.0V, 1.8V, 3.3V) with <50mV ripple.
* No thermal runaway on FPGA/MCU under reset state.

### Power Board (LM2662 Inverter)
* Output generates clean -5V rail.
* Current limit trips correctly if PA gate is shorted.

### Antenna Assembly
* VNA confirms S11 (Return Loss) is <-10dB at 10.5 GHz.

---

## 6. Digital Validation (V4)

### MCU
* **Boot:** Vector table loads, clocks initialize via HAL.
* **State Machine:** Enters IDLE state, waits for Host USB command.
* **Peripherals:** SPI1 (Synthesizer), SPI2 (Beamformer), I2C1 (Thermal) initialize.

### FPGA
* **Configuration:** Boot from SPI Flash succeeds.
* **CDC:** Clock Domain Crossing between 100MHz logic and FT2232H 60MHz FIFO clock is stable.
* **DSP Pipeline:** Test pattern generator outputs expected digital chirp.
* **ADC Input:** Captures valid JESD204B/LVDS data from receiver.

---

## 7. Protocol Validation (V3)

* **USB Framing:** `radar_protocol.py` successfully parses the custom header/footer magic bytes.
* **Commands:** Host can read/write the 32-bit control register space.
* **SPI Transactions:** MCU SPI writes match the timing diagrams in Analog Devices datasheets.
* **MCU↔FPGA GPIO:** Hardware interlocks (e.g., TX_ENABLE) assert with <1ms latency.

---

## 8. RF Validation (V5)

The following metrics require establishing a baseline measurement from a known-good unit or initial physical prototype. **Target numbers will not be invented.**

* **Output Power (EIRP):** *Measurement required — baseline not yet established.*
* **Center Frequency:** 10.5 GHz.
* **Bandwidth (LFM Chirp):** *Measurement required — baseline not yet established.*
* **Phase Noise:** *Measurement required — baseline not yet established.*
* **Isolation (Tx/Rx):** *Measurement required — baseline not yet established.*
* **Antenna Gain:** *Measurement required — baseline not yet established.*

---

## 9. Calibration Validation (V6)

* **Calibration Execution:** ADAR1000 phase/gain sweeping script executes without crashing.
* **Coefficient Storage:** Matrices successfully write to Non-Volatile Memory.
* **Coefficient Loading:** FPGA applies correct phase shifts upon beam-steer command.
* **Repeatability:** Phase values remain consistent across power cycles.
* **Unknowns:** *Exact algorithm for calculating the phase correction table is currently undocumented and requires physical trial/error.*

---

## 10. End-to-End Validation (V6)

Observable evidence must be collected at every boundary of the data path:

```text
Host (Command sent via GUI)
 ↓ (Verified by PyFTDI log)
USB
 ↓ (Verified by Logic Analyzer)
MCU / FPGA
 ↓ (Verified by FPGA ILA / JTAG)
RF (Waveform generated)
 ↓ (Verified by Spectrum Analyzer)
Antenna (Radiated)
 ↓ (Physical environment)
Target / Stimulus (Corner Reflector)
 ↓
RX (Echo received)
 ↓ (Verified by Oscilloscope on ADC lines)
FPGA DSP (CFAR applied)
 ↓ (Verified by FPGA ILA)
USB
 ↓ (Verified by PC network capture)
Host Visualization (Verified by GUI Screenshot)
```

---

## 11. Performance Characterization (V7)

To be populated with empirical targets once a prototype is built:

* **Detection Range:** TBD.
* **Range Resolution:** TBD.
* **Doppler Behavior:** TBD.
* **Angular Resolution:** TBD.
* **Beam Steering Speed:** TBD.
* **Power Consumption:** TBD.
* **Thermal Stability:** TBD.

---

## 12. Regression Validation

The following automated or checklist tests must run after changes:
* **Firmware Changes:** Re-run V3 SPI logic captures to ensure timing wasn't broken.
* **FPGA Changes:** Re-run V3 USB FIFO throughput test.
* **Hardware Revisions:** Re-run V1 Power sequencing test (Mandatory).
* **Calibration Changes:** Re-run V6 Point Target detection.

---

## 13. Evidence Requirements

A test cannot be marked `PASS` merely because it was executed. Acceptable evidence includes:

* Automated test script logs (`.txt`, `.xml`).
* Logic analyzer captures (`.sal`).
* Oscilloscope screen captures (`.png`).
* Spectrum Analyzer / VNA CSV traces.
* High-resolution photographs of physical integration.
* Compiled bitstream/ELF hashes.

---

## 14. Unknown ↔ Validation Traceability

| Unknown | Validation Test | What It Proves | Required Evidence | Resolution |
| ------- | --------------- | -------------- | ----------------- | ---------- |
| **UNK-010** (DC Power Specs) | V1 Power Test | Validates that applying the chosen DC voltage does not exceed thermal/current limits | DMM/Oscilloscope log | Pending Hardware |
| **UNK-003** (STM32 Suffix) | V0 Component Test | Validates that the chosen LQFP/BGA package physically mates to the PCB footprint | PCB Assembly photograph | Pending CAD |
| **UNK-005** (FPGA Boot) | V4 Digital Test | Proves the FPGA successfully loads the bitstream from SPI Flash upon power-up | DONE pin scope trace | Pending Hardware |

---

## 15. Acceptance Levels

* **Functional Acceptance:** Minimum operation. The system powers up, connects to USB, and transmits/receives an RF chirp.
* **High-Fidelity Acceptance:** Architecture matches the target. All specific ICs (AD9523, ADAR1000) are utilized exactly as designed in the original repo.
* **Performance Acceptance:** Measured characteristics (EIRP, SNR, Range) meet or exceed the empirically established baselines.
* **Reproducibility Acceptance:** An independent second unit succeeds in meeting Performance Acceptance using only our generated documentation.

---

## 16. Replica Acceptance Checklist

* [ ] **BOM:** All passives mapped and sourced.
* [ ] **PCB:** 10-layer RO4350B board fabricated successfully.
* [ ] **Mechanical:** Enclosure and waveguide CNC machined.
* [ ] **Power:** Negative bias interlock verified.
* [ ] **Clock:** AD9523 PLL locked.
* [ ] **MCU:** Firmware boots and handles USB interrupts.
* [ ] **FPGA:** Bitstream synthesized and loaded.
* [ ] **Protocols:** GUI commands correctly actuate hardware.
* [ ] **Software:** Python visualization renders FFT data.
* [ ] **RF:** PA output verified on Spectrum Analyzer.
* [ ] **Antenna:** VNA S11 match verified.
* [ ] **Calibration:** Phase/gain tables generated and loaded.
* [ ] **Testing:** System detects 100m corner reflector.
* [ ] **Documentation:** Build manual complete.
* [ ] **Independent Reproduction:** *Target not yet established.*

---

## 17. Final Validation Status

**Current Status:** **IMPOSSIBLE TO VALIDATE CURRENTLY**

The project is at Phase 0. No hardware exists to validate.

### Five Most Important Validation Gaps:
1. Cannot validate digital integration (V4) without physical FPGA/MCU hardware.
2. Cannot establish RF performance baselines (V7) without a functioning original unit or completed prototype.
3. Cannot validate power sequences (V1) without resolving UNK-010 (Input power specs).
4. Cannot validate antenna performance (V5) without `.dwg` waveguide extraction (UNK-002).
5. Cannot validate beamforming calibration (V6) without understanding the undocumented phase algorithm.
