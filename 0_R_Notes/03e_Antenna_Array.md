---
type: reverse-engineering-note
status: active
domain: hardware
confidence: mixed
canonical: true
---

# Antenna Array

> **Related notes:** [[03_Hardware]] · [[13_RF_Signal_Chain]]
> **Status:** Read-only Forensic Snapshot

---

## 1. Subsystem Identity

*   **Domain:** RF Radiation and Spatial Filtering
*   **Function:** Transmits radar pulses into free space and receives reflections. Enables electronic beam steering (phased array).

---

## 2. Variant Configurations

The AERIS-10 product relies on two completely distinct antenna configurations depending on the target variant.

### AERIS-10N (Nexus Variant)
*   **Type:** Microstrip Patch Antenna Array
*   **Array Size:** 8x16 elements
*   **Range:** 3 km
*   **Interface:** Connects directly to the ADTR1107 front-ends on the Main Board.
*   **Evidence:** `Patch_Anetnna_16_8.sch`, `Patch_Anetnna_16_8.brd`

### AERIS-10E (Extended Variant)
*   **Type:** Dielectric-Filled Slotted Waveguide Array
*   **Array Size:** 32x16 slots
*   **Range:** 20 km
*   **Interface:** Connects to the 16x Power Amplifier (QPA2962) boards.
*   **Evidence:** `DFSWA.dwg`, `Dielectric_Filled_Waveguide_Array_Spec.docx`

---

## 3. Steering and Actuation

*   **Electronic Scanning (Elevation/Azimuth):** ±45° steering is handled by the ADAR1000 phase shifters on the Main Board, manipulating the phase of the 16 channels feeding the array.
*   **Mechanical Scanning (Azimuth):** 360° physical rotation is handled by a stepper motor and slip-ring assembly, controlled by the MCU.

---

## 4. Replication-Critical Features

*   **Dielectric Substrate (Patch):** The microstrip patch array performance relies entirely on the precise dielectric constant ($D_k$) and thickness of the PCB material. (Specific material needs verification).
*   **Mechanical Tolerances (Waveguide):** The slotted waveguide array's resonant frequency (10.5 GHz) is extremely sensitive to CNC machining tolerances.
*   **Phase Matching:** The physical connections (cables/traces) between the Main Board/PA Boards and the antenna feed points must be exactly phase-matched.

---

## 5. Unknowns & Next Steps

| ID | Unknown | Why It Matters | Evidence Available | Required Investigation |
| -- | ------- | -------------- | ------------------ | ---------------------- |
| ANT-01 | Patch Stack-up | What is the exact PCB substrate for the Patch Antenna? | `4_4_Board Stack-up` | Verify if stack-up documentation exists for the Patch board. |
| ANT-02 | Waveguide Integration | How do the 16 PA boards mechanically and electrically mate to the waveguide feeds? | `DFSWA.dwg` | Parse DWG CAD files. |
| ANT-03 | Stepper Motor / Slip-ring | What are the exact part numbers for the mechanical actuation system? | `README.md`, `SlipRing.dwg` | Parse DWG CAD files. |


## Related Notes

- [[03_Hardware]]
- [[05d_Beamformer_ADAR1000]]
