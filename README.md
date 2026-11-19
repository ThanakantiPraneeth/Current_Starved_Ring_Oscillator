# Current-Starved Ring Oscillator (CSRO)

Designing and simulating a **3-Stage Current-Starved Ring Oscillator (CSRO)**—often used as a Voltage-Controlled Oscillator (VCO)—in Cadence Virtuoso involves building a 12-transistor current-mirror biased inverter chain and running transient, frequency, power, and physical verification flows.

This project presents the complete design, schematic implementation, initial-condition transient simulation, performance evaluation, physical layout, and verification (DRC, LVS, and QRC/RCX parasitic extraction) of a **3-Stage Current-Starved Ring Oscillator** using **90 nm CMOS technology (`gpdk090`)** in Cadence Virtuoso IC 6.1.8.

---

## Table of Contents

1. [Theory of Operation](#theory-of-operation)
2. [Design Specifications](#design-specifications)
3. [Mathematical Background](#mathematical-background)
4. [Schematic Design](#schematic-design)
5. [Transient Simulation](#transient-simulation)
6. [Frequency Analysis](#frequency-analysis)
7. [ADE Explorer and Calculator Measurements](#ade-explorer-and-calculator-measurements)
8. [Layout](#layout)
9. [Design Rule Check (DRC)](#design-rule-check-drc)
10. [Layout Versus Schematic (LVS)](#layout-versus-schematic-lvs)
11. [Step-by-Step Implementation Workflow](#step-by-step-implementation-workflow)
12. [Overall Performance Summary](#overall-performance-summary)
13. [Applications of Current-Starved Ring Oscillators](#applications-of-current-starved-ring-oscillators)
14. [Tools and Technologies](#tools-and-technologies)
15. [Project Structure](#project-structure)
16. [Conclusion](#conclusion)
17. [Author & Acknowledgment](#author--acknowledgment)

---

## Theory of Operation

A conventional ring oscillator generates a periodic output waveform by connecting an odd number of inverter stages in a closed loop, but it inherently has limited control over its oscillation frequency.

A **Current-Starved Ring Oscillator (CSRO)** improves upon this by limiting the current available to each inverter stage via current-mirror biasing transistors controlled by an external control voltage ($V_{ctrl}$):

- **Biasing and Current Mirror Stage (`NM0`, `NM1`, `PM0`):** The external control voltage $V_{ctrl}$ (`1.8 V`) drives the gate of bias input NMOS `NM0`, establishing a reference drain current $I_{ref}$ through diode-connected NMOS `NM1` and diode-connected PMOS `PM0`. This reference current is mirrored simultaneously across the upper PMOS current sources (`PM1`–`PM3`) and the lower NMOS current sinks (`NM2`–`NM4`).
- **Three-Stage Current-Starved Inverter Ring (`PM1`–`PM6`, `NM2`–`NM4`):** Three cascaded CMOS inverter stages (`PM4`/`NM2`, `PM5`/`NM3`, and `PM6`/`NM4`) are starved of unlimited supply current by the upper PMOS current sources and lower NMOS current sinks.
- **Closed-Loop Feedback & Oscillation Condition:** The output of the third inverter stage (`vosc`) feeds directly back to the input of the first stage. Because an odd number of inverting stages ($N = 3$) provides $180^\circ$ of DC phase inversion plus an additional $180^\circ$ of frequency-dependent RC phase shift (totaling $360^\circ$ with loop gain $\ge 1$), the circuit satisfies the Barkhausen criterion and sustains continuous oscillation.
- **Frequency Tuning Mechanism:** Varying $V_{ctrl}$ modulates the mirrored drain current $I_D$, directly controlling the slew rate and propagation delay $t_d$ of each inverter stage.

---

## Design Specifications

The current-starved ring oscillator was designed and verified using the Generic 90 nm PDK (`gpdk090`) in Cadence Virtuoso.

| Parameter | Specification | Remarks / Instance Details |
| :--- | :--- | :--- |
| **Circuit Type** | 3-Stage Current-Starved Ring Oscillator | Odd-stage feedback loop ($N = 3$) |
| **Technology** | 90 nm CMOS (`gpdk090`) | Standard 1 V thin-oxide devices |
| **Library / Cell Names** | `connectLib` / `csrosch` (`csro`) | `schematic`, `layout`, `av_extracted`, `maestro` |
| **Transistor Count** | 12 MOSFETs (7 PMOS + 5 NMOS) | Uniform sizing: $W = 120\text{ nm}$, $L = 100\text{ nm}$, $m = 1$ |
| **Supply Voltage ($V_{DD}$)** | $1.0\text{ V}$ | DC source `V0` (`vdc = 1 V`) |
| **Control Voltage ($V_{ctrl}$)** | $1.8\text{ V}$ | DC source `V1` (`vdc = 1.8 V`) |
| **Initial Condition** | `/vosc = 0 V` | Node set to initiate oscillation |
| **Output Node** | `vosc` | Rail-to-rail periodic output signal |
| **Simulation Tool** | Cadence Virtuoso IC 6.1.8 | ADE Explorer / Maestro & ViVA XL |
| **Simulator** | Spectre | Transient analysis (`0–100 ns`, `moderate` accuracy) |
| **Oscillation Frequency ($f_{osc}$)** | **$13.2021\text{ GHz}$** ($\approx 13.2\text{ GHz}$) | Period $T \approx 75.75\text{ ps}$ |
| **Average Power ($P_{avg}$)** | **$115.353\ \mu\text{W}$** ($\approx 115.4\ \mu\text{W}$) | Evaluated over transient window |
| **Physical Verification** | DRC, LVS, and QRC/RCX | 0 DRC errors, 0 LVS mismatches, `av_extracted` generated |

### Main Design Components

1. **Bias & Current-Mirror PMOS (`PM0`–`PM3`, 4 instances, `gpdk090_pmos1v`):** `PM0` acts as the diode-connected reference load, while `PM1`–`PM3` mirror $I_{ref}$ as upper current sources ($W = 120\text{ nm}, L = 100\text{ nm}, m = 1$).
2. **Inverter Pull-Up PMOS (`PM4`–`PM6`, 3 instances, `gpdk090_pmos1v`):** Switching pull-up transistors for Stages 1, 2, and 3 ($W = 120\text{ nm}, L = 100\text{ nm}, m = 1$).
3. **Bias & Current-Mirror NMOS (`NM0`–`NM1`, 2 instances, `gpdk090_nmos1v`):** `NM0` receives $V_{ctrl}$ at its gate and `NM1` acts as the diode-connected NMOS current-mirror reference ($W = 120\text{ nm}, L = 100\text{ nm}, m = 1$).
4. **Current-Starving Pull-Down NMOS (`NM2`–`NM4`, 3 instances, `gpdk090_nmos1v`):** Current-sink transistors limiting discharge current for Stages 1, 2, and 3 ($W = 120\text{ nm}, L = 100\text{ nm}, m = 1$).
5. **Supply Voltage Source (`V0`):** $V_{DD} = 1.0\text{ V}$ DC source.
6. **Control Voltage Source (`V1`):** $V_{ctrl} = 1.8\text{ V}$ DC source.
7. **Feedback & Output Node (`vosc`):** Third-stage output routed back to the first inverter input.

---

## Mathematical Background

### 1. Ring Oscillator Frequency

The oscillation frequency of an $N$-stage ring oscillator is approximately determined by:

$$f_{osc} = \frac{1}{2 N t_d}$$

For a three-stage ring oscillator ($N = 3$):

$$f_{osc} = \frac{1}{6 t_d}$$

The total oscillation period is:

$$T = 2 N t_d = 6 t_d \implies t_d = \frac{T}{2N}$$

### 2. Current-Starved Frequency & Delay Calculation

The oscillation frequency $f_{osc}$ of an $N$-stage Current-Starved Ring Oscillator is governed by the mirrored drain current $I_D$ and total stage node capacitance $C_{tot}$:

$$f_{osc} = \frac{I_D}{N \cdot C_{tot} \cdot V_{DD}}$$

From the simulated oscillation frequency of $f_{osc} = 13.2021\text{ GHz}$:

$$T = \frac{1}{13.2021\text{ GHz}} \approx 75.75\text{ ps}$$

$$t_d = \frac{T}{6} \approx 12.62\text{ ps per stage}$$

### 3. Current-Starving Principle

The control voltage $V_{ctrl}$ regulates the available charging and discharging current $I_D$ supplied to each inverter stage:

- Lower available current ($V_{ctrl} \downarrow$) $\rightarrow$ Higher propagation delay ($t_d \uparrow$) $\rightarrow$ Lower frequency ($f_{osc} \downarrow$).
- Higher available current ($V_{ctrl} \uparrow$) $\rightarrow$ Lower propagation delay ($t_d \downarrow$) $\rightarrow$ Higher frequency ($f_{osc} \uparrow$).

### 4. Power Consumption

The average power consumption is evaluated from the supply voltage and average supply current:

$$P_{avg} = \frac{1}{T_{sim}} \int_{0}^{T_{sim}} P(t)\,dt = V_{DD} \times I_{avg}$$

For the dynamic switching component:

$$P_{dynamic} = \alpha C_L V_{DD}^{2} f_{osc}$$

where $\alpha$ is the switching activity factor, $C_L$ is the effective load capacitance, $V_{DD}$ is the supply voltage, and $f_{osc}$ is the oscillation frequency.

---

## Schematic Design

The fundamental schematic of the CSRO (`connectLib/csrosch/schematic`) was built in Cadence Virtuoso Schematic Editor L using 90 nm CMOS technology (`gpdk090_pmos1v` and `gpdk090_nmos1v`), uniform sizing ($W = 120\text{ nm}, L = 100\text{ nm}, m = 1$), DC supply `V0` ($1.0\text{ V}$), and control voltage `V1` ($1.8\text{ V}$).

**Schematic of the Current-Starved Ring Oscillator:**

![Schematic](Screenshot%20from%202026-11-03%2011-31-04.png)

---

## Transient Simulation

Transient analysis was performed using Cadence ADE Explorer and the Spectre simulator (`0` to `100 ns`, `moderate` accuracy). To break initial DC metastability and initiate oscillation, an initial condition of `/vosc = 0 V` was configured in ADE Explorer.

The transient response observed at the oscillator output node (`vosc`) confirms rapid startup and sustained periodic oscillations between $\approx 0.07\text{ V}$ and $\approx 0.92\text{ V}$ under a $1.0\text{ V}$ supply.

**Initial Condition Setup (`/vosc = 0 V`) in ADE Explorer:**

![Initial Condition Setup](Screenshot%20from%202026-11-03%2011-44-36.png)

**Initial transient response of the oscillator:**

![Transient Simulation](Screenshot%20from%202026-11-03%2011-49-01.png)

---

## Frequency Analysis

The oscillation frequency was calculated from the output waveform using the ViVA XL Calculator.

The frequency measurement expression used was:

```skill
frequency(VT("/vosc"))

```

The calculator determines an oscillation frequency of **$13.2021\text{ GHz}$** ($13.2\text{ GHz}$ rounded) from the time-domain output waveform.

**Frequency analysis showing the oscillation frequency:**

---

## ADE Explorer and Calculator Measurements

The Cadence ViVA XL Calculator and ADE Explorer / Maestro were used to extract scalar oscillation frequency and average power metrics over the transient simulation window.

### Frequency Measurement Expression

```skill
frequency(VT("/vosc"))

```

### Average Power Measurement Expression

```skill
average(getData(":pwr" ?result "tran"))

```

### ADE Simulation Measurements

| Measurement | Expression / Variable | Observed Value |
| --- | --- | --- |
| **Oscillation frequency** | `frequency(VT("/vosc"))` | **$13.2021\text{ GHz}$** ($13.2\text{ GHz}$) |
| **Average power** | `average(getData(":pwr" ?result "tran"))` | **$115.353\ \mu\text{W}$** ($115.4\ \mu\text{W}$) |
| **ADE design variables** | `vdd = 1`, `vosc = 1.8` | $V_{DD} = 1.0\text{ V}$, $V_{ctrl} = 1.8\text{ V}$ |

**Transient Output Voltage (`/vosc`) and Instantaneous Power (`:pwr`) Waveforms:**

**ADE Explorer Scalar Outputs (`13.2 GHz` and `115.4 µW`):**

---

## Layout

After completing the schematic-level design and simulations, the circuit (`csro/csro/layout`) was implemented in Cadence Virtuoso Layout Suite XL using 90 nm CMOS (`gpdk090`) design rules:

* All 7 PMOS devices grouped inside a shared `Nwell` connected to `vdd!` via N-tap contacts.
* All 5 NMOS devices placed in the P-substrate tied to `gnd!` via P-tap contacts.
* Interconnect routing using `Poly`, `Metal1`, and `Metal2` layers with `Via1` transitions for supply, ground, control voltage (`Vctrl`), internal inverter nodes, and feedback (`Vosc`).

**Physical layout of the Current-Starved Ring Oscillator:**

---

## Design Rule Check (DRC)

Design Rule Check (DRC) was performed using **Assura DRC** to verify that the physical layout satisfies the 90 nm fabrication design rules.

The supplied DRC screenshot confirms:

```text
No DRC errors found.
Total DRC violations: 0

```

**DRC verification result:**

* Physical layout checked against the 90 nm CMOS (`gpdk090`) design rules across `Nwell`, `Oxide`, `Poly`, `Pimp`/`Nimp`, `Cont`, `Metal1`, `Via1`, and `Metal2` layers.
* No reported DRC errors (`No DRC errors found`).
* Total reported DRC violations: 0.

---

## Layout Versus Schematic (LVS)

Layout Versus Schematic (LVS) verification was performed using **Assura LVS** to verify exact correspondence between the physical layout and the original schematic (`csrosch`).

The supplied LVS run summary reports:

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

Following LVS verification, **Assura QRC (RCX)** was run to generate the `av_extracted` parasitic netlist view.

**LVS run summary:**

**Final LVS verification:**

---

## Step-by-Step Implementation Workflow

### 1. Schematic Design & Pre-Layout Simulation

1. Launch Cadence Virtuoso IC 6.1.8 and open library `connectLib` / `csro`. Create a new schematic cell view named `csrosch`.
2. Place 7 PMOS (`pmos1v`) and 5 NMOS (`nmos1v`) transistors from `gpdk090`, setting all dimensions to $W = 120\text{ nm}$, $L = 100\text{ nm}$, and $m = 1$.
3. Wire the current-mirror biasing network (`NM0`, `NM1`, `PM0`) to the three cascaded current-starved inverter stages (`PM1`–`PM6`, `NM2`–`NM4`), connecting output `vosc` back to the first inverter input.
4. Connect DC sources `V0 = 1 V` ($V_{DD}$) and `V1 = 1.8 V` ($V_{ctrl}$), add pins `vdd`, `gnd`, and `vosc`, and run **Check and Save**.
5. Open Virtuoso ADE Explorer / Maestro, select the `spectre` simulator, set an initial condition of `0 V` on `/vosc`, and configure a transient analysis (`tran`) from `0` to `100n` with `moderate` accuracy.
6. Add output expressions `frequency(VT("/vosc"))` and `average(getData(":pwr" ?result "tran"))`, run the simulation, and verify sustained oscillation at $13.2021\text{ GHz}$ with $115.353\ \mu\text{W}$ average power.

### 2. Physical Layout, DRC, LVS & Parasitic Extraction

1. Launch **Virtuoso Layout Suite XL** from the schematic view and generate all transistor instances along with `vdd!`, `gnd!`, `Vctrl`, and `Vosc` pins.
2. Group all PMOS devices inside a shared `Nwell` region with N-tap substrate ties, and place NMOS devices in the P-substrate with P-tap ties.
3. Route interconnects using `Poly`, `Metal1`, and `Metal2` layers while adhering to 90 nm design rules.
4. Run **Assura $\rightarrow$ Run DRC** and confirm zero design rule violations (`No DRC errors found`).
5. Run **Assura $\rightarrow$ Run LVS** and verify `Schematic and Layout Match` with zero net, device, pin, or parameter mismatches.
6. Run **Assura $\rightarrow$ Run QRC (RCX)** to generate the `av_extracted` view with coupled parasitic resistances and capacitances.

---

## Overall Performance Summary

| Analysis / Verification | Main Result |
| --- | --- |
| **Transient analysis (`0–100 ns`)** | Sustained periodic oscillation (`0.07 V` – `0.92 V`) |
| **Oscillation frequency ($f_{osc}$)** | **$13.2021\text{ GHz}$** ($13.2\text{ GHz}$) |
| **Average power ($P_{avg}$)** | **$115.353\ \mu\text{W}$** ($115.4\ \mu\text{W}$) |
| **DRC (Assura)** | **0 reported violations (`No DRC errors found`)** |
| **LVS (Assura)** | **0 reported mismatches (`Schematic and Layout Match`)** |
| **Parasitic extraction (QRC/RCX)** | **`av_extracted` view generated** |

---

## Applications of Current-Starved Ring Oscillators

* Voltage-Controlled Oscillators (VCOs)
* Phase-Locked Loops (PLLs) and Delay-Locked Loops (DLLs)
* On-chip clock generation circuits and frequency synthesizers
* Clock and Data Recovery (CDR) circuits
* Low-power integrated timing and control circuits

---

## Tools and Technologies

| Tool / Technology | Purpose |
| --- | --- |
| **Cadence Virtuoso 6.1.8** | Schematic and physical layout design |
| **ADE Explorer / Maestro** | Simulation setup, initial conditions, and analysis |
| **Spectre** | Circuit transient simulation |
| **ViVA XL & Calculator** | Waveform visualization, frequency, and power measurements |
| **90 nm CMOS PDK (`gpdk090`)** | Transistor models (`pmos1v`, `nmos1v`) and physical design rules |
| **Assura DRC** | Physical design-rule verification |
| **Assura LVS** | Layout-to-schematic verification |
| **Assura QRC / RCX** | Parasitic resistance and capacitance extraction (`av_extracted`) |
| **GitHub** | Project documentation and version control |

---

## Project Structure

```text
Current_Starved_Ring_Oscillator/
│
├── README.md
├── 226EC1002 DIC LAB PROJECT REPORT.pdf
│
├── CSRO_PVG/
│   ├── csro/
│   ├── csro.drc
│   ├── csro.lvsdb/
│   └── cds.lib
│
└── Screenshots/
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

A 3-stage Current-Starved Ring Oscillator was successfully designed, simulated, and physically verified at the 90 nm CMOS technology node (`gpdk090`) in Cadence Virtuoso. Pre-layout transient analysis with an initial condition of `/vosc = 0 V` confirmed stable oscillation at **$13.2021\text{ GHz}$** with an average power dissipation of **$115.353\ \mu\text{W}$** for $V_{DD} = 1.0\text{ V}$ and $V_{ctrl} = 1.8\text{ V}$, while the physical layout passed Assura DRC (`0` violations) and Assura LVS (`Schematic and Layout Match`) cleanly prior to QRC/RCX parasitic extraction.

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
