# Project-Mercury
A fabrication-valid, industry-representative power delivery network test board targeting 500 A peak current at 1.0 V — designed to operate within the electrical regime of RTX 4090-class discrete GPU power delivery systems.
<div align="center">

#  Mercury
### GPU-Class PDN Test Platform

*A fabrication-valid, industry-representative power delivery network test board targeting 500 A peak current at 1.0 V — designed to operate within the electrical regime of RTX 4090-class discrete GPU power delivery systems.*

---

**🏆 PCBWay International PCB Design Competition — 11th Place + Popular Design Prize**

---

![Board](docs/images/3D_Front.png)



---

## Overview

Mercury is a high-current power delivery network (PDN) test platform built to evaluate the physical and geometric limits of PCB-level power distribution under GPU-class transient conditions. The board targets a peak load current of approximately **500 A at 1.0 V**, with a maximum allowable voltage droop of **40 mV** — corresponding to a target impedance of **0.08 mΩ**.

This is not a concept board or a simplified academic exercise. It is a fabrication-valid design, laid out to industry-grade PCB standards, with a realistic 12-phase VRM architecture, a controlled MOSFET-based transient load system, and full engineering documentation.

The central finding of the project: **early transient droop in a PCB-level PDN is dominated by effective series inductance, not by total bulk capacitance.** Geometric decisions — plane adjacency, current path length, via density, and layer sandwiching — outweigh component value selection as the primary determinant of peak voltage droop. This is confirmed through LTspice parametric simulation and aligns directly with how industrial GPU power delivery teams approach board-level PDN design.

---

## Key Specifications

| Parameter | Value |
|---|---|
| Output Voltage (VCORE) | ~1.0 V |
| Peak Load Current | ~500 A |
| Maximum Voltage Droop | ≤ 40 mV |
| Target Impedance | ~0.08 mΩ |
| Input Supply | 12 V |
| Max Input Power | ~600 W (connector-limited) |
| VRM Phase Count | 12 |
| Switching Frequency | 400–500 kHz |
| Phase Inductor Value | ~150 nH |
| PCB Layer Count | 6 |
| Board Dimensions | 165.5 × 120 mm |
| Total Via Count | ~1,000 |
| VRM Controller | Renesas ISL68229 |
| Power Stage | MP86956-class DrMOS |
| MCU | STM32G030K6T6 |

---

## Architecture

The board is structured around three functional blocks:

**VRM Block**
Renesas ISL68229 12-phase PWM controller driving MP86956-class DrMOS power stages at 400–500 kHz. Each phase delivers approximately 40–50 A under rated load. PMBus telemetry is available via a 2×3 header with 0 Ω MCU isolation resistors, allowing firmware or external-tool access without bus contention.

**PDN Block**
Six-layer stackup with a dedicated VCORE plane (L3) sandwiched between two solid GND planes (L2 and L4), minimising loop inductance through plane adjacency. High-frequency decoupling is provided by an MLCC carpet (220 nF → 22 µF) directly beneath the load region, with 100 µF and 330 µF bulk capacitors placed progressively farther from the load centroid.

| Layer | Function | Electrical Role |
|---|---|---|
| L1 | Components + high-current pours | 12 V distribution, switching nodes, phase outputs |
| L2 | Solid ground plane | Primary return reference |
| L3 | VCORE plane | High-current distribution |
| L4 | Solid ground plane | Secondary return + field containment |
| L5 | Logic + 3.3 V routing | Low-current routing and control |
| L6 | Ground plane + placement | MLCC return and bottom copper continuity |

**Load Block (PADDLE)**
N-channel MOSFET banks controlled by STM32G030K6T6, providing repeatable current step injection up to approximately 150 A per bank. Current monitoring uses an INA181 sense amplifier on a low-side shunt. Multiple banks can be enabled simultaneously or staged to approach the full 500 A envelope.

---

## PCB Renders

<div align="center">

| Top Side | Bottom Side |
|:---:|:---:|
| ![Top](docs/images/3D_Front.png) | ![Bottom](docs/images/3D_Back.png) |

| All Layers Composite | L1 — 12V + VCORE |
|:---:|:---:|
| ![All Layers](docs/images/All_Layers.png) | ![Layer 1](docs/images/Layer_1.png) |

| MLCC Carpet (Load Region) | DrMOS Phase Row |
|:---:|:---:|
| ![MLCC](docs/images/Inner_Chip_Area.png) | ![DrMOS](docs/images/DrMOS_Phases.png) |

</div>

---

## Simulation

All simulations were performed in LTspice. Two analysis types were used:

- **AC impedance analysis** — 1 A injection at load node, impedance derived as Z = V/I across frequency. Distribution inductance modelled as series L_DIST + R_DIST between capacitor network and load.
- **Transient time-domain analysis** — step current source, parametric sweeps across distribution inductance, bulk capacitance value, and MLCC placement inductance.

**Key findings from simulation:**
- Removing approximately half the total bulk capacitance produced negligible change in first-edge droop
- Increasing distribution inductance produced proportional increase in peak droop for the same current step
- Series inductance is the binding constraint for sub-microsecond transient response — not charge storage
- The 220 nF MLCC tier contributes only marginally before depleting; the 1 µF tier is the effective high-frequency workhorse

> **Note:** Simulations use lumped circuit models and should be interpreted as conservative sensitivity indicators, not absolute performance predictors. Physical transient waveform validation was not completed in this revision.

---

## Documentation

Full engineering documentation is provided in the `/docs` directory.

| Document | Description |
|---|---|
| `Mercury_Master_Technical_Report.docx` | Complete 14-section technical report covering architecture, PDN theory, modelling, transient analysis, capacitor network design, stackup, via strategy, load modelling, layout implementation, limitations, and revision strategy |
| `Mercury_Executive_Summary.docx` | 2-page summary of project scope, key specifications, design decisions, and central findings — for fast review |
| `Mercury_Engineering_Reflections.docx` | Honest personal reflection on what worked, what didn't, what was learned, and what a second revision would change |

---

## Competition Result

Mercury was submitted to the **PCBWay International PCB Design Competition** and placed **11th overall**, also receiving the **Popular Design Prize**. It was the only design-only entry (no fabricated hardware) in the top results — competing directly against built and tested boards.

---

## Tools Used

- **KiCad** — schematic capture and PCB layout
- **LTspice** — PDN simulation (AC impedance + transient)
- **PCBWay** — fabrication target and competition platform

---

## Repository Structure

```
Mercury/
├── README.md
├── docs/
│   ├── Mercury_Master_Technical_Report.docx
│   ├── Mercury_Executive_Summary.docx
│   ├── Mercury_Engineering_Reflections.docx
│   └── images/
│       ├── 3D_Front.png
│       ├── 3D_Back.png
│       ├── 3D_Real_Shot.png
│       ├── 3D_Side_View.png
│       ├── All_Layers.png
│       ├── Layer_1.png
│       ├── Layer_8.png
│       ├── Inner_Chip_Area.png
│       ├── DrMOS_Phases.png
│       └── PCB_12V_VCORE.png
├── gerbers/
│   ├── Mercury_(GPU)-L1.gbr
│   ├── Mercury_(GPU)-GND1.gbr
│   ├── Mercury_(GPU)-VCORE.gbr
│   ├── Mercury_(GPU)-GND2.gbr
│   ├── Mercury_(GPU)-Logic+3V3.gbr
│   ├── Mercury_(GPU)-GND3.gbr
│   ├── Mercury_(GPU)-F_Silkscreen.gbr
│   ├── Mercury_(GPU)-B_Silkscreen.gbr
│   ├── Mercury_(GPU)-F_Mask.gbr
│   ├── Mercury_(GPU)-B_Mask.gbr
│   ├── Mercury_(GPU)-F_Paste.gbr
│   ├── Mercury_(GPU)-B_Paste.gbr
│   ├── Mercury_(GPU)-Edge_Cuts.gbr
│   ├── Mercury_(GPU)-PTH.drl
│   ├── Mercury_(GPU)-NPTH.drl
│   └── Mercury_(GPU)-job.gbrjob
├── centroid/
│   ├── Mercury_(GPU)-all.pos
│   ├── Mercury_(GPU)-top.pos
│   └── Mercury_(GPU)-bottom.pos
└── kicad/
    └── (KiCad project files)
```

---

## Scope and Limitations

This board is a design and analysis platform. The following limitations apply to this revision:

- Physical transient waveform measurements have not been performed. The 40 mV droop target has not been experimentally confirmed.
- VRM loop compensation was not formally derived.
- No formal thermal simulation was performed for either the DrMOS devices or the load MOSFETs.
- The load MOSFET topology (switched to GND) limits sustained duty cycle; a MOSFET-gated resistor bank architecture is identified as the preferred approach for Rev-A.

These are documented in full in the Master Technical Report (Section 12) and the Engineering Reflections document.

---

## Author

**Marcus Cropley**
Electronics and Computer Systems Engineering — RMIT University, Melbourne
https://www.linkedin.com/in/marcus-cropley-938582370/ · [GitHub](https://github.com/Marcusc05)

---

<div align="center">
<sub>Mercury — GPU-Class PDN Test Platform · Revision 1.0</sub>
