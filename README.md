# Current-Starved Ring Oscillator (CSRO)

Designing and simulating a **3-Stage Current-Starved Ring Oscillator (CSRO)**—often used as a Voltage-Controlled Oscillator (VCO)—in Cadence Virtuoso involves building a 12-transistor current-mirror biased inverter chain and running transient, frequency, power, and physical verification flows.

This project presents the complete design, schematic implementation, initial-condition transient simulation, performance evaluation, physical layout, and verification (DRC, LVS, and QRC/RCX parasitic extraction) of a **3-Stage Current-Starved Ring Oscillator** using **90 nm CMOS technology (`gpdk090`)** in Cadence Virtuoso IC 6.1.8.

---

## Table of Contents

1. [Theory of Operation](#theory-of-operation)
2. [Design Specifications & Component Breakdown](#design-specifications--component-breakdown)
3. [Mathematical Background](#mathematical-background)
4. [Schematic Design](#schematic-design)
5. [Transient Simulation & Initial Condition Setup](#transient-simulation--initial-condition-setup)
6. [Frequency & Power Analysis (ADE Explorer & ViVA XL)](#frequency--power-analysis-ade-explorer--viva-xl)
7. [Physical Layout](#physical-layout)
8. [Design Rule Check (DRC)](#design-rule-check-drc)
9. [Layout Versus Schematic (LVS) & RCX Extraction](#layout-versus-schematic-lvs--rcx-extraction)
10. [Step-by-Step Implementation Workflow](#step-by-step-implementation-workflow)
11. [Overall Performance Summary](#overall-performance-summary)
12. [Applications](#applications-of-current-starved-ring-oscillators)
13. [Tools and Technologies](#tools-and-technologies)
14. [Project Structure](#project-structure)
15. [Conclusion](#conclusion)
16. [Author & Acknowledgment](#author--acknowledgment)

---

## Theory of Operation

A conventional ring oscillator generates a periodic output waveform by connecting an odd number of inverter stages in a closed loop, but its oscillation frequency is fixed by process, voltage, and temperature (PVT) delays.

A **Current-Starved Ring Oscillator (CSRO)** overcomes this limitation by regulating the current available to charge and discharge the load capacitance of each inverter stage via current-mirror biasing transistors controlled by an external control voltage ($V_{ctrl}$):

- **Biasing and Current Mirror Stage (`NM0`, `NM1`, `PM0`):** The external control voltage $V_{ctrl}$ (`1.8 V`) drives the gate of bias input NMOS `NM0`, establishing a reference drain current $I_{ref}$ through diode-connected NMOS `NM1` and diode-connected PMOS `PM0`. This reference current is mirrored simultaneously across the upper PMOS current sources (`PM1`–`PM3`) and the lower NMOS current sinks (`NM2`–`NM4`).
- **Three-Stage Current-Starved Inverter Ring (`PM1`–`PM6`, `NM2`–`NM4`):** Three cascaded CMOS inverter stages (`PM4`/`NM2`, `PM5`/`NM3`, and `PM6`/`NM4`) are starved of unlimited supply current by the upper PMOS current sources and lower NMOS current sinks.
- **Closed-Loop Feedback & Oscillation Condition:** The output of the third inverter stage (`vosc`) feeds directly back to the input of the first stage. Because an odd number of inverting stages ($N = 3$) provides $180^\circ$ of DC phase inversion plus an additional $180^\circ$ of frequency-dependent RC phase shift (totaling $360^\circ$ with loop gain $\ge 1$), the circuit satisfies the Barkhausen criterion and sustains continuous oscillation.
- **Frequency Tuning Mechanism:** Varying $V_{ctrl}$ modulates the mirrored drain current $I_D$, directly controlling the slew rate and propagation delay $t_d$ of each inverter stage.

---

## Design Specifications & Component Breakdown

The 3-stage current-starved ring oscillator was designed and verified using the Generic 90 nm PDK (`gpdk090`) in Cadence Virtuoso.

| Parameter | Specification | Remarks / Instance Details |
| :--- | :--- | :--- |
| **Circuit Topology** | 3-Stage Current-Starved Ring Oscillator | Odd-stage feedback loop ($N = 3$) |
| **Technology Node** | 90 nm CMOS (`gpdk090`) | Standard 1 V thin-oxide devices |
| **Library / Cell Names** | `connectLib` / `csrosch` (`csro`) | `schematic`, `layout`, `av_extracted`, `maestro` |
| **Transistor Count** | 12 MOSFETs (7 PMOS + 5 NMOS) | Uniform sizing: $W = 120\text{ nm}$, $L = 100\text{ nm}$, $m = 1$ |
| **Supply Voltage ($V_{DD}$)** | $1.0\text{ V}$ | DC source `V0` (`vdc = 1 V`) |
| **Control Voltage ($V_{ctrl}$)** | $1.8\text{ V}$ | DC source `V1` (`vdc = 1.8 V`) |
| **Initial Condition** | `/vosc = 0 V` | Node set to break DC metastability and start oscillation |
| **Output Node** | `vosc` | Rail-to-rail periodic output signal |
| **Transient Stop Time** | $100\text{ ns}$ | `spectre` simulator (`moderate` accuracy) |
| **Oscillation Frequency ($f_{osc}$)** | **$13.2021\text{ GHz}$** ($\approx 13.2\text{ GHz}$) | Period $T \approx 75.75\text{ ps}$ |
| **Average Power ($P_{avg}$)** | **$115.353\ \mu\text{W}$** ($\approx 115.4\ \mu\text{W}$) | Evaluated over transient window |
| **Physical Verification** | DRC Clean & LVS Matched | 0 DRC errors, 0 LVS mismatches, QRC/RCX extracted |

### Transistor-Level Component Allocation

1. **Bias & Current-Mirror PMOS (`PM0`–`PM3`, 4 instances, `gpdk090_pmos1v`):** `PM0` acts as the diode-connected reference load, while `PM1`–`PM3` mirror $I_{ref}$ as upper current sources for the three inverter stages ($W = 120\text{ nm}, L = 100\text{ nm}, m = 1$).
2. **Inverter Pull-Up PMOS (`PM4`–`PM6`, 3 instances, `gpdk090_pmos1v`):** Switching pull-up transistors for Stages 1, 2, and 3 ($W = 120\text{ nm}, L = 100\text{ nm}, m = 1$).
3. **Bias & Current-Mirror NMOS (`NM0`–`NM1`, 2 instances, `gpdk090_nmos1v`):** `NM0` receives $V_{ctrl}$ at its gate and `NM1` acts as the diode-connected NMOS current-mirror reference ($W = 120\text{ nm}, L = 100\text{ nm}, m = 1$).
4. **Current-Starving Pull-Down NMOS (`NM2`–`NM4`, 3 instances, `gpdk090_nmos1v`):** Current-sink transistors limiting discharge current for Stages 1, 2, and 3 ($W = 120\text{ nm}, L = 100\text{ nm}, m = 1$).
5. **DC Sources & Pins:** `V0` ($V_{DD} = 1.0\text{ V}$), `V1` ($V_{ctrl} = 1.8\text{ V}$), supply pins `vdd`, `gnd`, and output pin `vosc`.

---

## Mathematical Background

### 1. Current-Starved Oscillation Frequency

The oscillation frequency $f_{osc}$ of an $N$-stage Current-Starved Ring Oscillator is governed by the mirrored drain current $I_D$, the number of stages $N$, the supply voltage $V_{DD}$, and the total effective capacitance per stage $C_{tot}$:

$$f_{osc} = \frac{I_D}{N \cdot C_{tot} \cdot V_{DD}}$$

In terms of single-stage propagation delay $t_d$, the oscillation frequency and period $T$ for $N = 3$ stages are:

$$T = 2 \cdot N \cdot t_d = 6 t_d \implies f_{osc} = \frac{1}{2 N t_d} = \frac{1}{6 t_d}$$

From the simulated oscillation frequency of $f_{osc} = 13.2021\text{ GHz}$:

$$T = \frac{1}{13.2021\text{ GHz}} \approx 75.75\text{ ps}$$

$$t_d = \frac{T}{6} \approx 12.62\text{ ps per stage}$$

### 2. Current-Starving Tuning Relationship

The control voltage $V_{ctrl}$ sets the gate-to-source overdrive of `NM0`, regulating the mirrored current $I_D$:

- **Lower $V_{ctrl}$** $\rightarrow$ Lower mirrored current $I_D$ $\rightarrow$ Slower node charging/discharging ($t_d \uparrow$) $\rightarrow$ **Lower $f_{osc}$**
- **Higher $V_{ctrl}$** $\rightarrow$ Higher mirrored current $I_D$ $\rightarrow$ Faster node charging/discharging ($t_d \downarrow$) $\rightarrow$ **Higher $f_{osc}$**

### 3. Average & Dynamic Power Consumption

Total average power dissipation is computed from the time-domain supply power waveform `:pwr`:

$$P_{avg} = \frac{1}{T_{sim}} \int_{0}^{T_{sim}} P(t)\,dt = V_{DD} \times I_{avg}$$

The dynamic switching component scales with total stage capacitance $C_{tot}$, supply voltage $V_{DD}$, and oscillation frequency $f_{osc}$:

$$P_{dynamic} = N \cdot C_{tot} \cdot V_{DD}^2 \cdot f_{osc} = I_D \cdot V_{DD}$$

---

## Schematic Design

The schematic (`connectLib/csrosch/schematic`) was captured in **Virtuoso Schematic Editor L** using 90 nm PDK devices (`gpdk090_pmos1v` and `gpdk090_nmos1v`), uniform sizing ($W = 120\text{ nm}, L = 100\text{ nm}, m = 1$), DC supply source `V0` ($1.0\text{ V}$), and control source `V1` ($1.8\text{ V}$).

**Schematic of the 3-Stage Current-Starved Ring Oscillator:**

![Schematic](Screenshot%20from%202026-11-03%2011-31-04.png)

---

## Transient Simulation & Initial Condition Setup

Because a symmetric ring oscillator can settle at a metastable DC operating point during transient startup, an explicit initial condition of **`0 V`** was assigned to node **`/vosc`** in Virtuoso ADE Explorer before launching the `spectre` transient simulation (`0` to `100 ns`, `moderate` accuracy).

**Initial Condition Setup (`/vosc = 0 V`) in ADE Explorer:**

![Initial Condition Setup](Screenshot%20from%202026-11-03%2011-44-36.png)

**Transient Response of Output Node `/vosc`:**

![Transient Simulation](Screenshot%20from%202026-11-03%2011-49-01.png)

The waveform confirms rapid startup and clean, sustained periodic oscillations between $\approx 0.07\text{ V}$ and $\approx 0.92\text{ V}$ under a $1.0\text{ V}$ supply.

---

## Frequency & Power Analysis (ADE Explorer & ViVA XL)

Scalar frequency and average power metrics were extracted directly in **Virtuoso ADE Explorer / Maestro** and **ViVA XL** using built-in SKILL calculator expressions.

### Calculator Expressions

**Oscillation Frequency:**
```skill
frequency(VT("/vosc"))

```

**Average Power Consumption:**

```skill
average(getData(":pwr" ?result "tran"))

```

### Simulation Measurements & Waveforms

| Metric | Expression | Measured Value |
| --- | --- | --- |
| **Oscillation Frequency** | `frequency(VT("/vosc"))` | **$13.2021\text{ GHz}$** ($13.2\text{ GHz}$ rounded) |
| **Average Power** | `average(getData(":pwr" ?result "tran"))` | **$115.353\ \mu\text{W}$** ($115.4\ \mu\text{W}$ rounded) |
| **ADE Design Variables** | `vdd = 1`, `vosc = 1.8` | $V_{DD} = 1.0\text{ V}$, $V_{ctrl} = 1.8\text{ V}$ |

**Oscillation Frequency Measurement (`13.2021 GHz`) on `/vosc`:**

**Transient Output Voltage (`/vosc`) and Instantaneous Power (`:pwr`) Waveforms:**

**ADE Explorer Scalar Outputs (`13.2 GHz` and `115.4 µW`):**

---

## Physical Layout

The physical layout (`csro/csro/layout`) was implemented in **Virtuoso Layout Suite XL** adhering to 90 nm CMOS (`gpdk090`) design rules:

* **Well & Substrate Engineering:** All 7 PMOS devices are grouped inside a shared `Nwell` connected to `vdd!` via N-tap contacts, while all 5 NMOS devices reside in the P-substrate tied to `gnd!` via P-tap contacts.
* **Multi-Layer Interconnect Routing:** Gate connections use `Poly`, local device routing uses `Metal1`, and cross-stage feedback (`Vosc`) and bias routing (`Vctrl`) utilize `Metal1`/`Metal2` with `Via1` transitions.

**Physical Layout of the 3-Stage Current-Starved Ring Oscillator:**

---

## Design Rule Check (DRC)

Physical DRC verification was executed using **Assura DRC** against the 90 nm PDK rule deck to validate minimum width, spacing, enclosure, and density constraints:

```text
No DRC errors found.
Total DRC violations: 0

```

**Assura DRC Verification Result:**

* Checked all active (`Oxide`), `Nwell`, `Poly`, implant (`Pimp`/`Nimp`), contact (`Cont`), `Metal1`, `Via1`, and `Metal2` layers against `gpdk090` design rules.
* Zero spacing, width, overlap, or floating-well violations (`No DRC errors found`).

---

## Layout Versus Schematic (LVS) & RCX Extraction

**Assura LVS** was run to compare the extracted layout netlist against the golden schematic (`csrosch`):

```text
*** Schematic and Layout Match

Extraction Information:
  0 cells have 0 mal-formed device problems
  0 cells have 0 label short problems
  0 cells have 0 label open problems

Comparison Information:
  0 cells have 0 Net mismatches
  0 cells have 0 Device mismatches
  0 cells have 0 Pin mismatches
  0 cells have 0 Parameter mismatches

Total DRC violations: 0

```

Following LVS sign-off, **Assura QRC (RCX)** was executed to generate the `av_extracted` view containing coupled parasitic resistances and capacitances for post-layout verification.

**Assura LVS Run Summary:**

**Assura LVS Debug Result (`Schematic and Layout Match`):**

---

## Step-by-Step Implementation Workflow

### Phase 1: Schematic Design & Pre-Layout Simulation

1. Launch Cadence Virtuoso IC 6.1.8 and create schematic cell `csrosch` inside library `connectLib` / `csro`.
2. Instantiate **7 PMOS (`pmos1v`)** and **5 NMOS (`nmos1v`)** transistors from `gpdk090` with $W = 120\text{ nm}$, $L = 100\text{ nm}$, and $m = 1$.
3. Wire the current-mirror bias network (`NM0`, `NM1`, `PM0`) to the three current-starved inverter stages (`PM1`–`PM6`, `NM2`–`NM4`), feeding output `vosc` back to the first inverter input.
4. Connect DC sources `V0 = 1 V` ($V_{DD}$) and `V1 = 1.8 V` ($V_{ctrl}$), place pins `vdd`, `gnd`, and `vosc`, and run **Check and Save**.
5. Open **ADE Explorer / Maestro**, choose `spectre`, apply an initial condition `/vosc = 0 V`, and run a `100 ns` transient analysis (`moderate` accuracy).
6. Evaluate `frequency(VT("/vosc"))` ($13.2021\text{ GHz}$) and `average(getData(":pwr" ?result "tran"))` ($115.353\ \mu\text{W}$) in ViVA XL.

### Phase 2: Physical Layout, DRC, LVS & QRC Extraction

1. Open **Virtuoso Layout Suite XL** from the schematic and generate all 12 MOSFET instances and I/O pins (`vdd!`, `gnd!`, `Vctrl`, `Vosc`).
2. Place PMOS devices in a shared `Nwell` with N-taps and NMOS devices in the P-substrate with P-taps.
3. Complete interconnections using `Poly`, `Metal1`, and `Metal2`.
4. Run **Assura $\rightarrow$ Run DRC** and verify `No DRC errors found`.
5. Run **Assura $\rightarrow$ Run LVS** and verify `Schematic and Layout Match` with zero mismatches.
6. Run **Assura $\rightarrow$ Run QRC (RCX)** to produce the `av_extracted` parasitic view.

---

## Overall Performance Summary

| Verification / Analysis Stage | Metric / Check | Result |
| --- | --- | --- |
| **Transient Simulation** | Oscillation Stability (`0–100 ns`) | Sustained periodic oscillation (`0.07 V` – `0.92 V`) |
| **Frequency Analysis** | Oscillation Frequency ($f_{osc}$) | **$13.2021\text{ GHz}$** ($13.2\text{ GHz}$) |
| **Power Analysis** | Average Power Dissipation ($P_{avg}$) | **$115.353\ \mu\text{W}$** ($115.4\ \mu\text{W}$) |
| **Physical DRC** | Assura Design Rule Check | **Clean (0 DRC violations)** |
| **Physical LVS** | Assura Layout Versus Schematic | **Matched (0 net/device/pin/parameter mismatches)** |
| **Parasitic Extraction** | Assura QRC / RCX | **`av_extracted` view generated** |

---

## Applications of Current-Starved Ring Oscillators

* **Voltage-Controlled Oscillators (VCOs)** in RF and mixed-signal front-ends
* **Phase-Locked Loops (PLLs)** and Delay-Locked Loops (DLLs)
* **On-Chip Clock Generation** and frequency synthesizers
* **Clock and Data Recovery (CDR)** circuits in high-speed serial links
* **Low-Power Integrated Timing References** and sensor readout interfaces

---

## Tools and Technologies

| Tool / Technology | Purpose |
| --- | --- |
| **Cadence Virtuoso IC 6.1.8** | Custom IC schematic capture and physical layout environment |
| **Virtuoso Schematic Editor L** | Transistor-level schematic design (`csrosch`) |
| **ADE Explorer / Maestro** | Testbench variable management, initial conditions, and simulation control |
| **Spectre Simulator** | High-accuracy transient circuit simulation (`0–100 ns`) |
| **ViVA XL & Calculator** | Waveform visualization, scalar frequency, and average power extraction |
| **Virtuoso Layout Suite XL** | Connectivity-driven physical layout implementation |
| **Assura DRC / LVS / QRC** | Physical design rule verification, schematic-to-layout matching, and RCX extraction |
| **Generic 90 nm PDK (`gpdk090`)** | 90 nm CMOS device models (`pmos1v`, `nmos1v`) and physical rule decks |

---

## Project Structure

```text
Current_Starved_Ring_Oscillator/
│
├── README.md                                  # Complete project documentation
├── 226EC1002 DIC LAB PROJECT REPORT.pdf       # Formal IIITDM Kurnool lab project report
│
├── CSRO_PVG/                                  # Cadence Virtuoso 90nm library & verification files
│   ├── csro/                                  # Library directory (schematic, layout, av_extracted, maestro)
│   ├── csro.drc / csro.lvsdb / csro.erc       # Assura DRC, LVS, and ERC verification databases
│   └── cds.lib                                # Cadence library definitions
│
└── Screenshots/                               # Simulation & physical verification captures
    ├── Screenshot from 2026-11-03 11-31-04.png
    ├── Screenshot from 2026-11-03 11-44-36.png
    ├── Screenshot from 2026-11-03 11-49-01.png
    ├── Screenshot from 2026-11-03 11-54-35.png
    ├── Screenshot from 2026-11-03 11-59-22.jpg
    ├── Screenshot from 2026-11-03 12-09-59.png
    ├── Screenshot from 2026-11-06 13-56-32.png
    ├── Screenshot from 2026-11-10 16-25-21.png
    ├── Screenshot from 2026-11-10 16-25-47.jpg
    └── Screenshot from 2026-11-10 16-25-52.png

```

---

## Conclusion

A 3-stage Current-Starved Ring Oscillator (CSRO) was successfully designed, simulated, and physically verified at the 90 nm CMOS technology node (`gpdk090`) in Cadence Virtuoso. Pre-layout transient simulation with an initial condition of `/vosc = 0 V` confirmed stable, sustained oscillation at **$13.2021\text{ GHz}$** with an average power dissipation of **$115.353\ \mu\text{W}$** for $V_{DD} = 1.0\text{ V}$ and $V_{ctrl} = 1.8\text{ V}$. The physical layout passed **Assura DRC** (`0` violations) and **Assura LVS** (`Schematic and Layout Match`) cleanly, followed by **QRC/RCX** parasitic extraction (`av_extracted`).

---

## Author & Acknowledgment

* **Author:** Thanakanti Praneeth
* **Roll Number:** 226EC1002
* **Programme:** M.Tech in VLSI & Microelectronics
* **Department:** Electronics and Communication Engineering
* **Institution:** Indian Institute of Information Technology, Design and Manufacturing (IIITDM), Kurnool, Andhra Pradesh
* **Course:** Digital IC Design Laboratory

> **Acknowledgment:** Sincere thanks to **Dr. P. Ranga Babu** (Associate Professor, Department of Electronics and Communication Engineering, IIITDM Kurnool) for his valuable guidance, technical mentorship, and support throughout this Digital IC Laboratory project.

```

```
