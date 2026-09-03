---
type: reverse-engineering-note
status: active
domain: hardware
confidence: high
canonical: true
---

# 12. Power Architecture

> **Related notes:** [[03b_Power_Board]], [[05a_Power_Sequencing]]
> **Status:** Reconstructed from `Power Management V6.xlsx`

## 1. Primary Inputs (Bench Setup)

The AERIS-10 system employs a split power architecture due to the extreme disparity between logic and RF PA requirements.

*   **AERIS-10N (Nexus Variant):** Requires a standard **12V DC** supply. The Power Board utilizes 21x TPS562208 buck converters (4.5V - 17V input range) to derive all system logic and RF front-end rails.
*   **AERIS-10E (Extended Variant):** Requires a dual supply setup:
    *   **12V DC** for the Main Board / Digital Logic.
    *   **22V DC** at >32 Amps (~700W) for the 16x QPA2962 GaN Power Amplifiers.
*   **Mechanical Actuation:** The stepper motor driver (TBS6600) requires a **20V** supply (rated for 9-42V).

## 2. Main Board Power Tree (12V Input)

The 12V input is stepped down and inverted to create a highly complex localized power tree.

| Rail | Nominal Voltage | Source | Consumers | Current | Sequencing | Confidence |
| ---- | --------------: | ------ | --------- | ------: | ---------- | ---------- |
| `+3V3_XO` | +3.3V | Buck | OCXO, VCXO | 1250 mA | None | High |
| `+3V3_CLOCK` | +3.3V | Buck | AD9523 Clock Synth | 250 mA | Yes | High |
| `+1V8_CLOCK` | +1.8V | LDO | AD9523 Clock Synth | 250 mA | Yes | High |
| `+5V0_LO` | +5.0V | Buck | ADF4382 Synthesizer | 270 mA | None | High |
| `+3V3_LO` | +3.3V | Buck | ADF4382 Synthesizer | 580 mA | None | High |
| `+3V3` | +3.3V | Buck | STM32 MCU, Sensors | 250 mA | None | High |
| `+3V3_FPGA` | +3.3V | Buck | XC7A50T I/O | 220 mA | Yes | High |
| `+1V8_FPGA` | +1.8V | Buck | XC7A50T Aux | 72 mA | Yes | High |
| `+1V0_FPGA` | +1.0V | Buck | XC7A50T Core | 30 mA | Yes | High |
| `+3V3_AN` | +3.3V | Buck | ADC/DAC, Mixers | 150 mA | None | High |
| `+3V4` | +3.4V | LDO | M3SWA2 RF Switches | ~3 mA | None | High |
| `-3V4` | -3.4V | LM2662 Inverter | M3SWA2 RF Switches | 1.8 mA | None | High |
| `+3V3_ADAR` | +3.3V | Buck | ADAR1000 Beamformers | 2800 mA | Yes | High |
| `-5V0_ADAR` | -5.0V | LM2662 Inverter | ADAR1000 Beamformers | 240 mA | Yes | High |
| `+5V0_PA` | +5.0V | Buck | ADTR1107 VDD_PA | 4000 mA | Yes | High |
| `-3V3_SW` | -3.3V | LM2662 Inverter | ADTR1107 VSS_SW | 0.2 mA | Yes | High |

## 3. Negative Rail Generation

The contradiction between positive-only schematics and negative-bias requirements is fully resolved. The Power Board utilizes 5x **LM2662MX** Switched Capacitor Voltage Converters to generate the `-5.0V`, `-3.4V`, and `-3.3V` rails locally from their respective positive counterparts.

## 4. Extended PA Power Tree (22V Input)

The Microwave PA Boards are completely isolated from the logic power plane.

| Rail | Nominal Voltage | Source | Consumers | Current | Sequencing | Confidence |
| ---- | --------------: | ------ | --------- | ------: | ---------- | ---------- |
| `+22V0` | +22V | Direct Input | QPA2962 Drain | 32,000 mA | YES | High |
| `VG` | -4.0V to -1.2V | DAC/OpAmp | QPA2962 Gate | 160 mA | YES | High |
| `+5V5_PA` | +5.5V | Buck | OPA4703 OpAmp | 100 mA | YES | High |
| `-5V5_PA` | -5.5V | LM2662 | OPA4703 OpAmp | 100 mA | YES | High |
