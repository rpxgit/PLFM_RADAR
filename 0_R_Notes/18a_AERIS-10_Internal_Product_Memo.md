---
type: internal-memo
status: active
domain: strategy
audience: non-technical
canonical: true
---

# AERIS-10 — Internal Product & Replication Memo
**Classification:** Internal / Strategy  
**Prepared for:** Decision Makers, Program Leadership  
**Based on:** Technical Reverse-Engineering Assessment, Phase 0

---

## 1. Executive Summary

AERIS-10 is a **10.5 GHz phased-array radar system** — a device that sends a precisely controlled radio signal, listens for its reflection off objects, and uses the timing, frequency, and directional information in that echo to determine *where* something is and *how fast* it is moving. Think of it as an intelligent, electronically steered "radio eye" that can continuously scan a defined area, detect targets at significant range, and stream that detection data to a connected computer in real time.

The design is **open-source**, built around commercially available components from Analog Devices and Xilinx, and comes in two variants: a 3 km short-range model and a 20 km long-range model. The current evidence indicates it is in late-alpha / pre-production stage, authored by ABAC INDUSTRY (Morocco), and released under open hardware and software licences.

---

## 2. The Problem

### Problem Statement

**Organizations requiring autonomous or remote sensing capability** need **compact, affordable, electronically steerable radar** because **passive optical sensors fail at night, in fog, or against small low-signature targets**, but existing approaches face **high cost, limited availability, foreign supply-chain dependency, and inadequate customization for specific mission profiles**.

The source material does not identify a specific end-customer segment for AERIS-10. The product's exact intended market is currently inferred from its capability profile. However, the design — multi-kilometre detection range, electronic beam steering, digital target processing, and a software-configurable interface — is directly relevant to any operator requiring medium-range situational awareness from a system they can own, modify, and integrate without ongoing vendor dependency.

---

## 3. The Solution

AERIS-10 addresses the above through several key design decisions:

- **Software-defined waveform:** The radar chirp is generated digitally inside a programmable processor and then converted to a radio signal. This means the waveform can be reconfigured in software without changing any hardware.
- **Electronic beam steering:** Instead of mechanically rotating an antenna, the system shifts the direction of the transmitted beam electronically in milliseconds. Scanning is fast, silent, and repeatable.
- **Separated data and control paths:** High-speed radar data flows directly from the sensor to the host computer over USB, bypassing the system controller. The controller handles only initialization and safety. This design prevents throughput bottlenecks and enables real-time data rates.
- **On-board digital signal processing:** A dedicated programmable processor (an FPGA — a chip optimized for parallel mathematical operations) performs target detection locally, with no cloud or network dependency.
- **Software-configurable control:** A Python dashboard on a connected PC provides live visualization, beam control, and system configuration.

The result is a fully transparent system — the hardware, firmware, signal processing, and interface are all documented and modifiable.

---

## 4. How It Works — In 5 Steps

**1. Generate:** A precisely shaped radio signal (a frequency-swept chirp) is created digitally and converted to a 10.5 GHz radio wave.

**2. Aim:** Phase-shifting electronics steer the direction of the transmitted beam — no mechanical rotation required for the primary scan.

**3. Transmit & Receive:** The amplified signal is radiated toward the target area. When it reflects off an object, the returning echo is captured, amplified, and digitized.

**4. Process:** The on-board digital processor compares the transmitted and received signals to extract range, and optionally velocity (Doppler), then applies an adaptive detection algorithm to identify real targets above the noise floor.

**5. Display / Control:** Processed detections stream via USB to the host computer, where the operator sees a live range/Doppler map and can adjust system parameters in real time.

---

## 5. Product Snapshot

| Attribute | AERIS-10 |
| :--- | :--- |
| Radar type | Pulsed linear frequency-modulated (LFM) — *Confirmed* |
| Frequency class | ~10.5 GHz (X-Band) — *Confirmed* |
| Architecture | Phased array, electronically steered ±45° — *Confirmed* |
| Known variants | Nexus (3 km, PCB patch antenna) / Extended (20 km, waveguide) — *Confirmed* |
| Approx. range classes | 3 km / 20 km — *Documented claims; not yet independently measured* |
| Processing | Programmable digital processor (FPGA) on-board — *Confirmed* |
| Host interface | USB 2.0 (Nexus) / USB 3.0 (Extended), Python GUI — *Confirmed* |
| Physical architecture | 5 PCBs + antenna array + mechanical enclosure — *Confirmed* |
| Current reproduction state | Digital stack fully mapped; physical fabrication blocked pending BOM/CAD extraction |

---

---

## 6. Why This Is Strategically Interesting

Reproducing AERIS-10 offers the following **potential** strategic advantages. These are opportunities, not guaranteed outcomes:

- **Complete stack ownership:** The entire system is known and reconstructable — hardware, firmware, signal processing, and software. No vendor black box.
- **Customization without redesign:** Waveform, detection threshold, beam pattern, and control interface are all software-configurable. The same hardware platform can be adapted for different missions.
- **Derivative product potential:** The architecture is a foundation, not a ceiling. Alternative antenna arrays, frequency offsets, higher power stages, or integration with autonomous platforms are all natural extensions once the base is validated.
- **Know-how accumulation:** Reproducing this system develops end-to-end radar engineering competence — RF design, high-speed PCB, FPGA signal processing, embedded control, and RF measurement — with broad transferable value.
- **Technology sovereignty:** A domestically reproduced system removes foreign export controls, end-user certificate requirements, and supply-chain vulnerability from the operational picture.
- **Cost control at scale:** An open-architecture design, once validated, carries no licensing fees and can be manufactured and maintained domestically.

---

## 7. Replication Snapshot

### Timeline

> **Estimated replication timeline: approximately 4–5 months for a high-fidelity functional reproduction (Nexus / 3 km variant), subject to BOM/CAD extraction, component procurement lead times, one mandatory PCB revision, and RF validation — not a guarantee.**

### Budget

All figures are engineering estimates based on the technical reconstruction. Exact sourcing quotes are required before committing capital.

| Cost Category | Low | Expected | High | Notes |
| :--- | ---: | ---: | ---: | :--- |
| **Parts + PCB fabrication + assembly** | ₹4 L | ₹7 L | ₹12 L | Rogers RF laminate is expensive; includes 1 PCB respin |
| **Engineering / labor** | ₹3 L | ₹6 L | ₹10 L | 2 engineers × 4–5 months; rate-dependent |
| **Lab equipment rental** | ₹1 L | ₹2 L | ₹3.5 L | 12 GHz VNA + spectrum analyzer rental |
| **Contingency (RF risk)** | ₹1 L | ₹2 L | ₹4 L | GaN PA boards are high-risk at first power-up |

> **Indicative low-to-mid reproduction envelope: ₹9–17 lakh** for a high-fidelity Nexus (3 km) functional prototype.  
> *Assumes 2 engineers, 1 PCB respin, rented RF test equipment, outsourced PCB fabrication and mechanical machining. Excludes production tooling and regulatory certification.*

---

## 8. Resource Allocation

### Core Team (Minimum Practical — 2 People)

| Resource | Allocation | Role |
| :--- | :--- | :--- |
| Systems / Hardware Engineer | Full-time | Architecture, board bring-up, integration, procurement management |
| RF Engineer | Full-time | RF chain, antenna, PA bias calibration, beam-steering tables |

### Specialist / Outsourced Support

| Resource | Allocation | Role |
| :--- | :--- | :--- |
| FPGA / Digital Engineer | Part-time (~30%) | FPGA synthesis, DSP pipeline, USB interface |
| Embedded Engineer | Part-time (~20%) | MCU firmware compilation, peripheral initialization |
| PCB / Hardware Engineer | As needed | Board layout review, design-for-manufacture |
| Mechanical Engineer | As needed | Enclosure and antenna CNC review |
| Test / Validation | As needed | Acceptance testing, RF characterization |

*PCB fabrication (10-layer, RF-grade laminate), SMT assembly (500+ components, BGA), and CNC machining must be outsourced. These are not feasible in-house at prototype quantities.*

---

## 9. Replication Path

| Stage | What Happens |
| :--- | :--- |
| **Evidence extraction** | Parse BOM Excel and CAD files to recover all component values and mechanical dimensions |
| **Design reconstruction** | Compile FPGA bitstream and MCU firmware from source; verify digital stack integrity |
| **Procurement** | Source all ICs and passives; flag long-lead items (beamformer chips: 26-week lead risk) |
| **Fabrication** | Send Gerbers to RF-capable PCB house; CNC files to machining shop |
| **Bring-up** | Power sequencing validation before any RF IC is enabled |
| **RF activation** | LO and mixer chain verified; GaN PA bias calibrated under safe conditions |
| **Calibration** | Beam-steering phase/gain tables generated and loaded into memory |
| **Validation** | System detects a physical target; host GUI confirms range detection |

---

## 10. Current Blockers / Decision Gates

**Blockers to fabrication** (cannot order parts or PCBs until resolved):
- **BOM not extracted:** 500+ passive component values are locked inside a binary Excel file. Without these, PCB assembly is not possible.
- **CAD not extracted:** Mechanical dimensions for the waveguide and enclosure are locked inside AutoCAD files. Without these, CNC machining cannot begin.

**Blockers to RF activation** (cannot safely power the RF chain until resolved):
- **Input power unconfirmed:** The DC input voltage (12V / 24V / 48V) has not been verified from the power board schematic. Incorrect voltage risks permanent hardware damage.

**Blockers to performance validation** (cannot claim performance metrics until resolved):
- **FPGA bitstream unverified:** The production digital design has not been compiled end-to-end from source to confirm it fits within the target chip without unsatisfied dependencies.
- **RF performance unmeasured:** All range figures (3 km / 20 km) are documented marketing claims. No measured output power, phase noise, or detection range data currently exists.

---

## 11. India Market Context

### Technical Opportunity

AERIS-10's capability profile is credibly relevant to the following Indian application areas:

- **Defense and border security:** Medium-range electronically steered radar for perimeter surveillance, drone detection, and coastal monitoring.
- **Autonomous systems:** Sensing payload integration on unmanned ground or aerial platforms.
- **Aerospace and research:** University laboratories, DRDO-adjacent programs, and private aerospace companies seeking affordable, modifiable radar test platforms.
- **Maritime and coastal applications:** Port security and small-vessel detection at 3–20 km range.
- **Industrial sensing:** Long-range object detection where optical sensors fail — dust, fog, darkness.

### Why India Is Relevant

India's defense indigenization programs (IDDM, iDEX, Make in India), a growing private defense and aerospace ecosystem, increasing domestic ITAR-sensitivity around foreign sensors, and a strong base of electronics engineering talent all create a relevant environment for building sovereign radar sensing capability. An open-architecture system avoids export license constraints, creates a domestic engineering foundation, and is extensible to derivative programs.

> **A reliable market-size estimate requires separate market research and is outside the scope of this technical replication assessment.** The technical opportunity is well-supported. Validated commercial opportunity requires independent analysis.

---

## 12. Strategic Options

**Option A — Replicate:** Reproduce the existing system faithfully.  
*Advantage:* Fastest path to a known, validated system.  *Disadvantage:* Inherits all original design constraints.

**Option B — Replicate + Improve:** Achieve functional reproduction first, then upgrade selected areas (power, bandwidth, antenna, processor).  
*Advantage:* Builds on a proven foundation without reinventing solved problems.  *Disadvantage:* Requires deeper RF expertise in the improvement phase.

**Option C — Use as Technology Reference:** Extract the architecture, algorithms, and component selections as a reference for an original design.  
*Advantage:* Full design freedom; no dependency on recovering the original BOM/CAD.  *Disadvantage:* Significantly higher design risk and timeline; loses benefit of validated hardware.

*The available evidence does not yet establish sufficient performance data to formally recommend one option. Option B is the natural extension of Option A and is the most common approach in programs of this type.*

---

## 13. Recommendation / Next Decision

> **Recommended immediate action: authorize the evidence-extraction sprint (BOM parsing, CAD extraction, power schematic review) before committing any fabrication capital.**

These three tasks represent **less than one week of engineering effort and near-zero direct cost**. They substantially reduce uncertainty across every major project variable:

- Exact parts list → accurate procurement cost and lead times
- Mechanical dimensions → accurate CNC quote and feasibility
- DC input voltage → safe hardware bring-up planning
- Combined → a significantly more reliable reproduction budget and timeline

Committing to PCB fabrication before this work is complete risks spending ₹4–7 lakh on hardware that cannot be assembled or safely powered.

The decision in front of the team is not "should we build the radar?" — it is: **authorize the evidence-extraction sprint, review the outputs, then make an informed go/no-go on fabrication.**

---

> ### AERIS-10 — Current Working Assessment
>
> **What it is:** A 10.5 GHz electronically steered phased-array radar (3 km and 20 km variants) with a fully open, documented hardware and software stack.
>
> **Problem:** Organizations need affordable, sovereign, software-configurable radar sensing capability without dependence on foreign supply chains or proprietary systems.
>
> **Solution:** AERIS-10 provides a complete, modifiable radar architecture — antenna to host display — where every layer is known, reproducible, and adaptable.
>
> **Replication timeline:** ~4–5 months for a high-fidelity Nexus (3 km) functional prototype; contingent on resolving current blockers.
>
> **Indicative low-to-mid reproduction budget:** ₹9–17 lakh (parts, fabrication, labor, lab access; 1 PCB respin included).
>
> **Core resources:** 2 engineers (hardware + RF), part-time FPGA/embedded support, outsourced PCB/CNC.
>
> **Current blockers:** Binary BOM and CAD files not yet extracted; DC input voltage and FPGA synthesis unverified; no measured RF performance data exists.
>
> **Immediate decision:** Authorize a low-cost evidence-extraction sprint (≤1 week) before committing fabrication capital.
