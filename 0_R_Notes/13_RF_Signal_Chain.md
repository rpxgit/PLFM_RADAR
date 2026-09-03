---
type: reverse-engineering-note
status: active
domain: hardware
confidence: medium
canonical: true
---

# RF Signal Chain Reconstruction & Performance Model

> **Related notes:** [[02_System_Architecture]] · [[03_Hardware]] · [[05d_Beamformer_ADAR1000]] · [[08_Component_Inventory]]
> **Document type:** Canonical RF Architecture and Budget
> **Status:** Active (Phase 0 Evidence)

---

## 1. Top-Level RF Architecture

The AERIS-10 system uses a Pulsed Linear Frequency Modulated (PLFM) architecture, characterized by digital IF generation followed by analog upconversion to X-Band (~10.5 GHz). 

The key components of the RF/Mixed-Signal chain are:
* **Chirp Generator:** FPGA (XC7A50T) + DAC (AD9708) -> Analog PLFM IF
* **Local Oscillator:** ADF4382 Synthesizer (Fixed CW LO)
* **Mixers:** LTC5552 (Up-Mixer & Down-Mixer)
* **Beamformer:** ADAR1000 (4-channel Phase/Gain control, Qty 4)
* **T/R Front-End:** ADTR1107 (Contains LNA, PA, and internal T/R switch, Qty 16)
* **RF Switching:** M3SWA2-34DR+ (SPDT Switches)

---

## 2. Reconstructed TX Chain

The Transmit chain routes the IF signal from the DAC, upconverts it to X-band, steers it, and radiates it.

| Stage | Component | MPN | Freq Range | Function / Evidence | Gain / Loss | P1dB / Power | Interface |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Waveform Gen** | FPGA | XC7A50T | Baseband | Generates PLFM sequence (`plfm_chirp_controller`) | - | - | Digital |
| **IF DAC** | DAC | AD9708 | IF | Converts digital PLFM to analog IF (`dac_interface_single.v`) | - | 0 dBm (Typ) | Parallel / RF Out |
| **Up-Mixer** | Mixer | LTC5552 | 3-20 GHz | Upconverts IF to X-Band (10.5 GHz). LO from ADF4382. | ~ -10 dB | 10 dBm | RF Coax |
| **Beamformer** | ADAR1000 | ADAR1000 | 8-16 GHz | Applies phase & gain shifts (`ADAR1000_Manager.cpp`) | ~ +15 dB | ~ 10 dBm | SPI / RF Coax |
| **Front-End (Nexus)**| T/R Module | ADTR1107 | 6-18 GHz | Contains PA and T/R switch. Drives antenna directly. | ~ +25 dB | ~ 25 dBm | RF Coax |
| **Bypass Switch** | SPDT Switch | M3SWA2-34DR+ | DC-4 GHz? | (Hypothesized) Routes TX signal to GaN PA in Extended variant. | ~ -1.5 dB | - | GPIO |
| **GaN PA (Ext)** | Power Amp | QPA2962 | 2-18 GHz | 10W PA for Extended variant (`RF_PA.sch`) | ~ +22 dB | 40 dBm (10W)| RF Coax |
| **Antenna** | Patch / WG | Custom | 10.5 GHz | Patch Array (Nexus) or Waveguide (Ext) | +21 dBi (Est)| - | Air |

**Note on SPDT Switches:** The BOM contains 17x `M3SWA2-34DR+`. One is highly likely used at the ADAR1000 common port to switch between the TX Up-Mixer and RX Down-Mixer. The other 16 are likely used to bypass the unidirectional GaN PA during receive on the Extended variant.

---

## 3. Reconstructed RX Chain

The Receive chain captures echoes, passes them through the beamformer, downconverts them, and digitizes the signal.

| Stage | Component | MPN | Freq Range | Function / Evidence | Gain / Loss / NF | Interface |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Antenna** | Patch / WG | Custom | 10.5 GHz | Captures radar echoes | +21 dBi (Est) | Air |
| **Bypass Switch** | SPDT Switch | M3SWA2-34DR+ | DC-4 GHz? | Bypasses GaN PA to route Rx to ADTR1107 LNA. | ~ -1.5 dB | RF Coax |
| **Front-End LNA** | T/R Module | ADTR1107 | 6-18 GHz | Internal LNA amplifies received echo | ~ +20 dB / 2.5 dB NF | RF Coax |
| **Beamformer** | ADAR1000 | ADAR1000 | 8-16 GHz | Applies phase/gain weights. Routes to common Rx port. | ~ +10 dB / 5 dB NF | SPI / RF Coax |
| **Common T/R Sw**| SPDT Switch | M3SWA2-34DR+ | DC-4 GHz? | Routes common RF to Down-Mixer. | ~ -1.5 dB | GPIO |
| **Down-Mixer** | Mixer | LTC5552 | 3-20 GHz | Downconverts X-Band to IF | ~ -10 dB / 10 dB NF | RF Coax |
| **IF Amp** | RF Amp | AD8352 | DC-2.2 GHz | Drives differential inputs of the ADC | ~ +15 dB / 5 dB NF | RF Traces |
| **ADC** | ADC | AD9484 | IF | Digitizes at 400 MSPS (`ad9484_interface_400m.v`) | - | 8-bit LVDS |

---

## 4. Chirp Generation Resolution

Previous assumptions posited that the ADF4382 synthesizer autonomously generated the FMCW chirp ramp. This has been **refuted** by evidence found in the FPGA RTL (`radar_transmitter.v` and `plfm_chirp_controller.v`).

**Mechanism:**
1. The STM32 MCU toggles the `stm32_new_chirp` GPIO line to start an acquisition frame.
2. The FPGA catches this edge and triggers the internal `plfm_chirp_controller` module.
3. The FPGA generates a Polyphase Linear FM (PLFM) digital waveform and streams it to the AD9708 DAC.
4. The DAC outputs an analog modulated IF chirp.
5. The ADF4382, configured by the MCU over SPI (`adf4382a_manager.c`), acts purely as a fixed-frequency CW Local Oscillator (LO) to the LTC5552 mixers, upconverting the IF chirp to X-Band.

---

## 5. RF & Power Budget Estimates

*These budgets are calculated from typical component specifications and represent a theoretical baseline. Actual PCB insertion losses and matching network variations will affect final values.*

### TX Budget (Nexus Variant - per channel)
* **DAC Output:** 0 dBm
* **LTC5552 Mix:** -10 dB (Conversion Loss)
* **ADAR1000 TX:** +15 dB (Gain)
* **ADTR1107 PA:** +25 dB (Gain)
* **Total TX Power to Antenna (per channel):** `0 - 10 + 15 + 25 = 30 dBm (1 Watt)`

### TX Budget (Extended Variant - per channel)
* **DAC Output:** 0 dBm
* **LTC5552 Mix:** -10 dB
* **ADAR1000 TX:** +15 dB
* **ADTR1107 TX:** +15 dB (Operating backed-off)
* **M3SWA2 Switch:** -1.5 dB
* **QPA2962 PA:** +22 dB (Gain)
* **Total TX Power to Antenna (per channel):** `~ 40 dBm (10 Watts)`

---

## 6. Noise & Sensitivity Model

Estimating the receiver cascaded Noise Figure (NF) using Friis' formula for the Nexus variant:

$F_{total} = F_1 + \frac{F_2 - 1}{G_1} + \frac{F_3 - 1}{G_1 G_2} + \dots$

| Component | NF (dB) | Gain (dB) | Linear F | Linear G | Cumulative NF (dB) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **ADTR1107 LNA** | 2.5 | 20.0 | 1.77 | 100.0 | **2.50** |
| **ADAR1000 RX** | 5.0 | 10.0 | 3.16 | 10.0 | **2.59** |
| **Switch** | 1.5 | -1.5 | 1.41 | 0.70 | **2.60** |
| **LTC5552 Mix**| 10.0 | -10.0 | 10.00 | 0.10 | **2.97** |
| **AD8352 IF Amp**| 5.0 | 15.0 | 3.16 | 31.6 | **5.45** |
| **AD9484 ADC** | ~15 (Quant) | - | ~31.6 | - | **UNKNOWN** |

**System Sensitivity Model:**
* **Cascaded RF NF:** ~5.5 dB (Excluding ADC Quantization noise).
* **Thermal Noise Floor:** $kT = -174$ dBm/Hz.
* **Assumed Bandwidth:** ~100 MHz IF Bandwidth.
* **Noise Floor:** $-174 + 10\log_{10}(10^8) + 5.5 = -88.5$ dBm.

---

## 7. Radar Equation Assessment

Using the theoretical parameters in the performance model (White Paper Appendix H):

$R_{\max} = \left[ \frac{P_t G_t G_r \lambda^2 \sigma}{(4\pi)^3 kT_0BF_n L SNR_{\min}} \right]^{1/4}$

**Parameters:**
* $\lambda$: 0.0285 m (10.5 GHz)
* $\sigma$: 1 m² (Drone RCS)
* $P_t$: 16 W (Nexus - 1W x 16), 160 W (Extended - 10W x 16)
* $G_t = G_r$: 21 dBi (Nexus Patch), 32 dBi (Extended Waveguide assumption)
* $L$ (System Losses): 3 dB (Assumed)
* $SNR_{\min}$: 10 dB (Standard detection threshold)
* $NF_{sys}$: 5.5 dB
* $B$: 100 MHz (Uncompressed), assuming Processing Gain offsets $B$ to effective detection bandwidth.

**Theoretical Plausibility:**
> **Engineering model — not measured performance.**

* **Nexus Variant (16W total, 21 dBi Gain):** Theoretical max range $\approx 2 - 3$ km for a 1 m² target. The marketed "3 km class" is physically plausible but highly dependent on optimized DSP integration gain.
* **Extended Variant (160W total, 32 dBi Gain):** Theoretical max range $\approx 15 - 20$ km. The "20 km class" marketing claim is mathematically supported by the brute force addition of sixteen 10W GaN PAs and a high-gain waveguide.

---

## 8. Calibration Requirements

The system demands extreme precision in phase and gain to achieve coherent beamforming.

1. **PA Bias Calibration:** The Extended variant's GaN PAs (QPA2962) require closed-loop $I_{dq}$ (quiescent drain current) tuning. The MCU sets the negative gate bias ($V_g$) via `DAC5578`, applies drain voltage ($V_d$), and adjusts $V_g$ until the correct current is read across a 5mΩ shunt by the `ADS7830` ADC. The MCU firmware must contain logic for `kPaBiasIdqCalibration` (0x0D).
2. **Phase / Beam Steering Calibration:** The `ADAR1000_Manager` relies on look-up tables (`VM_I`, `VM_Q`) for setting 2.8125-degree phase increments. Real-world array imperfections necessitate an external anechoic chamber calibration to map these ideal values to corrected phase/gain coefficients.
3. **Timing Synchronization:** `stm32_new_chirp` triggers the PLFM sequence. The ADC sampling window must be precisely synchronized to this trigger. 

**Repository Calibration Status:**
The `ADAR1000_Manager.cpp` establishes the framework for setting phase and bias. However, an *actual* automated anechoic chamber calibration procedure or pre-calculated factory calibration matrix (EEPROM) is currently **missing** from the repository, implying the system boots with uncalibrated theoretical phase weights.

---

## Related Notes
- [[05d_Beamformer_ADAR1000]]
- [[08_Component_Inventory]]
- [[18_AERIS-10_Product_White_Paper]]
