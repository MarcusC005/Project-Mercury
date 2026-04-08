<div align="center">

# ⚡ Mercury
### GPU-Class PDN Test Platform

*A fabrication-valid, industry-representative power delivery network test board targeting 500 A peak current at 1.0 V — designed to operate within the electrical regime of RTX 4090-class discrete GPU power delivery systems.*

---

**🏆 PCBWay International PCB Design Competition — 11th Place + Popular Design Prize**

---

![Board](docs/images/3D%20Real%20Shot.PNG)

</div>

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
| ![Top](docs/images/3D%20Front%20RT.PNG) | ![Bottom](docs/images/3D%20Back%20RT.PNG) |

| Side View | All Layers Composite |
|:---:|:---:|
| ![Side](docs/images/3D%20Side%20View.PNG) | ![All Layers](docs/images/All%20Layers.PNG) |

| L1 — 12V + VCORE Distribution | MLCC Carpet (Load Region) |
|:---:|:---:|
| ![Layer 1](docs/images/Layer%201.PNG) | ![MLCC](docs/images/Inner%20Chip%20Area.PNG) |

| DrMOS Phase Row | 12V and VCORE Planes |
|:---:|:---:|
| ![DrMOS](docs/images/PCB%20View%20DrMOS%20Phases.PNG) | ![Power Planes](docs/images/PCB%20View%2012V%20and%20VCORE.PNG) |

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
│       ├── 3D Front RT.PNG
│       ├── 3D Front Nm.PNG
│       ├── 3D Back RT.PNG
│       ├── 3D Back Nm.PNG
│       ├── 3D Real Shot.PNG
│       ├── 3D Side View.PNG
│       ├── All Layers.PNG
│       ├── Layer 1.PNG
│       ├── Layer 8.PNG
│       ├── Inner Chip Area.PNG
│       ├── PCB View DrMOS Phases.PNG
│       └── PCB View 12V and VCORE.PNG
├── gerbers/
│   └── (Gerber and drill files)
└── centroid/
    └── (Component placement files)
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
[LinkedIn](https://www.linkedin.com/in/marcus-cropley-938582370/) · [GitHub](https://github.com/MarcusC005)

---

<div align="center">
<sub>Mercury — GPU-Class PDN Test Platform · Revision 1.0</sub>
</div>
