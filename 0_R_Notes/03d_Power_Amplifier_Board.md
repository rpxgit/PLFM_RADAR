# Power Amplifier Board

> **Related notes:** [[03_Hardware]] · [[13_RF_Signal_Chain]]
> **Status:** Read-only Forensic Snapshot

---

## 1. Board Identity

*   **PCB Name:** `RF_PA`
*   **Domain:** RF Power Amplification
*   **Variant Target:** AERIS-10E (Extended) ONLY
*   **Quantity per Unit:** 16
*   **Function:** Boosts the output of the ADTR1107 front-end to 10W per channel before feeding the slotted waveguide array.

---

## 2. Schematic Analysis

| Block | Components | Inputs | Outputs | Interface | Function | Evidence | Confidence |
| ----- | ---------- | ------ | ------- | --------- | -------- | -------- | ---------- |
| **Power Amplifier** | QPA2962 | RF In (from ADTR1107) | RF Out (to Waveguide) | RF (SMA/SMP) | 10W GaN RF Amplification | `README.md`, `.sch` | Confirmed |
| **Current Sense** | 5 mΩ Shunt (On PA) + INA241A3 (On Main) | Vd Current | Analog V_Idq | Analog | Idq monitoring for calibration | `README.md` | Confirmed |

---

## 3. Signal Flow

```text
[Main Board ADTR1107] ──► RF IN ──► [QPA2962] ──► RF OUT ──► [Waveguide Antenna]
```

---

## 4. Hardware ↔ Firmware Correlation

*   The QPA2962 requires precise gate biasing to set the quiescent current ($I_{dq}$).
*   Because GaN characteristics vary over temperature and manufacturing, the Main Board MCU dynamically adjusts $V_g$ via the DAC5578 and measures the resulting $I_{dq}$ via the ADS7830.
*   This closed-loop calibration relies on a 5mΩ shunt resistor on the PA board.

---

## 5. Replication-Critical Features

*   **GaN Power Sequencing:** $V_g$ (negative) must be applied before $V_d$ (high positive).
*   **Thermal Management:** 16x 10W GaN amplifiers will dissipate significant heat. The mechanical integration of heat sinks and cooling fans is critical.
*   **Shunt Value:** The 5mΩ shunt cannot be changed without updating the firmware's calibration math.

---

## 6. Unknowns & Next Steps

| ID | Unknown | Why It Matters | Evidence Available | Required Investigation |
| -- | ------- | -------------- | ------------------ | ---------------------- |
| PAB-01 | Bias Routing | How are the individual Vg and Idq sense lines routed from the Main Board to the 16 PA boards? | `.sch` | Parse Main and PA board schematics. |
| PAB-02 | Physical Integration | How do 16 boards physically mate with the Main Board and the Waveguide? | `.dwg` | Parse mechanical drawings. |
