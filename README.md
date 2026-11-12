
# Current-Starved Ring Oscillator (CSRO)

Designing and simulating a **Current-Starved Ring Oscillator (CSRO)**—often used as a Voltage-Controlled Oscillator (VCO)—in Cadence Virtuoso involves building the inverter chain with current-limiting transistors and running transient and parametric simulations.

This project presents the design, schematic implementation, simulation, performance evaluation, physical layout, and verification of a Current-Starved Ring Oscillator using **90 nm CMOS technology** in Cadence Virtuoso.

For a comprehensive breakdown of the design methodology, performance metrics, Monte Carlo analysis, and PVT (Process, Voltage, Temperature) considerations, please refer to the detailed technical document: **CSRO_REPORT.pdf**.

---

## Table of Contents

1. [Theory of Operation](#theory-of-operation)
2. [Design Specifications](#design-specifications)
3. [Mathematical Background](#mathematical-background)
4. [Schematic Design](#schematic-design)
5. [Transient Simulation](#transient-simulation)
6. [Frequency Analysis](#frequency-analysis)
7. [Power and Performance](#power-and-performance)
8. [ADE Explorer and Calculator Measurements](#ade-explorer-and-calculator-measurements)
9. [Frequency and Power Analysis](#frequency-and-power-analysis)
10. [PVT Analysis](#pvt-analysis)
11. [Layout](#layout)
12. [Design Rule Check (DRC)](#design-rule-check-drc)
13. [Layout Versus Schematic (LVS)](#layout-versus-schematic-lvs)
14. [Overall Performance Summary](#overall-performance-summary)
15. [Applications](#applications-of-current-starved-ring-oscillators)
16. [Tools and Technologies](#tools-and-technologies)
17. [Project Structure](#project-structure)
18. [Conclusion](#conclusion)

---

## Theory of Operation

A conventional ring oscillator generates a periodic output waveform by connecting an odd number of inverter stages in a closed loop, but it inherently has limited control over its oscillation frequency.

A **Current-Starved Ring Oscillator (CSRO)** improves upon this by limiting the current available to each inverter stage using PMOS source transistors.

The approximate oscillation frequency for an N-stage ring oscillator is governed by:

\[
F_{osc}=\frac{1}{2\cdot N\cdot t_d}
\]

Where:

- \(F_{osc}\) = Oscillation frequency.
- \(N\) = Number of delay stages.
- \(t_d\) = Average propagation delay of each stage.

In a current-starved configuration, modifying the control voltage (\(V_{ctrl}\)) changes the available charging current, which in turn alters the charging/discharging time of the internal nodes and directly tunes the oscillation frequency.

The core components of this 90 nm CMOS architecture operate as follows:

* **Delay stages:** Each CMOS inverter stage (comprising one middle PMOS and one bottom NMOS) introduces a finite propagation delay to the signal.
* **PMOS current source:** An upper PMOS transistor acts as a current source, limiting the charging current available to the inverter.
* **Frequency control:** The control voltage (\(V_{ctrl}\)) is applied to the gate of the upper PMOS; varying this voltage determines how quickly the output node charges, dynamically controlling the delay.
* **Feedback connection:** The output of the final inverter is routed back to the input of the first inverter, ensuring continuous switching and sustained oscillation under a 1.8 V supply (\(V_{DD}\)).

---

## Design Specifications

The current-starved ring oscillator was designed using a 90 nm CMOS process in Cadence Virtuoso.

| Parameter | Specification |
|---|---|
| Circuit type | Current-Starved Ring Oscillator |
| Technology | 90 nm CMOS |
| Number of delay stages | 3 (as documented in the technical report) |
| Supply voltage (\(V_{DD}\)) | 1.8 V |
| Control voltage (\(V_{ctrl}\)) | 1 V (nominal condition) |
| Output node | `Vosc` |
| Simulation tool | Cadence Virtuoso |
| Simulator | Spectre |
| Main analysis | Transient |
| Physical verification | DRC and LVS |
| Post-layout analysis | Parasitic-extracted simulation |

### Main Design Components

1. CMOS inverter delay stages.
2. PMOS current-source transistors.
3. NMOS pull-down transistors.
4. Supply voltage source.
5. Control voltage source.
6. Feedback connections.
7. Oscillator output node.

---

## Mathematical Background

### 1. Ring Oscillator Frequency

The oscillation frequency of an N-stage ring oscillator is approximately determined by:

\[
f_{osc}=\frac{1}{2Nt_d}
\]

For a three-stage ring oscillator:

\[
f_{osc}=\frac{1}{6t_d}
\]

The total oscillation period is:

\[
T=2Nt_d
\]

Therefore, the propagation delay can be calculated as:

\[
t_d=\frac{T}{2N}
\]

### 2. Frequency Measurement

The oscillation frequency is measured from the output waveform using:

\[
f_{osc}=\frac{1}{T}
\]

The time period is obtained by measuring the time difference between two successive equivalent waveform transitions:

\[
T=t_2-t_1
\]

For the nominal simulation documented in the report:

\[
t_1=1.014\ ns
\]

\[
t_2=1.138\ ns
\]

\[
T=1.138-1.014=0.124\ ns
\]

Hence:

\[
f_{osc}=\frac{1}{0.124\ ns}
\]

\[
\boxed{f_{osc}\approx 8.065\ GHz}
\]

The frequency measurement documented in the report is approximately 8.076 GHz.

### 3. Current-Starving Principle

The PMOS current-source transistor controls the current supplied to each inverter stage.

The control voltage \(V_{ctrl}\) regulates the available charging current. This affects the time required to charge and discharge the internal capacitances.

The relationship can be summarized as:

- Lower available current → Higher propagation delay → Lower frequency.
- Higher available current → Lower propagation delay → Higher frequency.

Thus, the control voltage provides a mechanism for tuning the oscillator frequency.

### 4. Power Consumption

The average power consumption is evaluated from the supply voltage and average supply current:

\[
P_{avg}=V_{DD}\times I_{avg}
\]

For the dynamic switching component, power is also commonly related to:

\[
P_{dynamic}=\alpha C_L V_{DD}^{2}f_{osc}
\]

Where:

- \(\alpha\) = Switching activity factor.
- \(C_L\) = Effective load capacitance.
- \(V_{DD}\) = Supply voltage.
- \(f_{osc}\) = Oscillation frequency.

The total simulated average power is measured using the supply-power waveform in Cadence Virtuoso.

---

## Schematic Design

The fundamental schematic of the CSRO was built in Cadence Virtuoso using 90 nm CMOS technology.

The circuit consists of current-starved inverter stages connected in a feedback configuration. The PMOS current-source devices control the current supplied to the inverter stages.

**Schematic of the Current-Starved Ring Oscillator:**

![Schematic](Screenshot%20from%202026-11-03%2011-31-04.png)

---

## Transient Simulation

Transient analysis was performed using Cadence ADE Explorer and the Spectre simulator.

The transient response was observed at the oscillator output node (`Vosc`) to verify the generation of a periodic oscillating waveform.

The simulation confirms that the circuit produces sustained oscillations.

**Initial transient response of the oscillator:**

![Transient Simulation](Screenshot%20from%202026-11-03%2011-49-01.png)

---

## Frequency Analysis

The oscillation frequency was calculated from the output waveform using the ViVA XL Calculator.

The frequency measurement expression used was:

```skill
frequency(VT("/vosc"))
```

The calculator determines the frequency from the time-domain output waveform.

**Frequency analysis showing the oscillation frequency:**

![Frequency Analysis](Screenshot%20from%202026-11-03%2011-54-35.png)

An additional ADE Explorer screenshot shows an oscillation frequency of approximately **13.2 GHz** under a separate simulation configuration.

---

## Power and Performance

The average power consumption of the oscillator was evaluated using the supply-power waveform.

The average power is calculated using:

\[
P_{avg}=V_{DD}\times I_{avg}
\]

The calculator expression used in ADE Explorer was:

```skill
average(getData(":pwr" ?result "tran"))
```

**Output waveform alongside power consumption:**

![Power Waveform 1](Screenshot%20from%202026-11-03%2011-59-22.jpg)

**Detailed view of average power calculations:**

![Power Waveform 2](Screenshot%20from%202026-11-03%2012-11-28.jpg)

The additional ADE Explorer result shows an average power of approximately **115.4 µW**.

This measurement belongs to the displayed ADE configuration and is separate from the nominal power result documented in the technical report.

---

## ADE Explorer and Calculator Measurements

The Cadence ViVA XL Calculator was used to obtain the oscillator frequency from the output waveform.

### Frequency Measurement Expression

```skill
frequency(VT("/vosc"))
```

### Average Power Measurement Expression

```skill
average(getData(":pwr" ?result "tran"))
```

The measurement process involves:

1. Selecting the output waveform.
2. Identifying successive oscillation cycles.
3. Calculating the time period.
4. Determining the oscillation frequency.
5. Measuring the average power over the transient simulation interval.

### Additional ADE Simulation Observation

An additional ADE Explorer result visible in the supplied screenshots shows:

| Measurement | Observed value |
|---|---:|
| Oscillation frequency | Approximately 13.2 GHz |
| Average power | 115.4 µW |
| Frequency expression | `frequency(VT("/vosc"))` |
| Power expression | `average(getData(":pwr" ?result "tran"))` |

The screenshot also shows the ADE design variables:

- `vdd = 1`
- `vosc = 1.8`

These values belong to the displayed ADE configuration. They should be treated separately from the nominal 1.8 V supply and 1 V control-voltage results documented elsewhere in the report.

---

## Frequency and Power Analysis

A parametric analysis was performed by varying the control voltage while maintaining the supply voltage at 1.8 V.

The control voltage was investigated over the range of 0 V to 1 V.

The analysis demonstrates the relationship between control voltage, oscillation frequency, and average power consumption.

### Frequency Variation

As the control voltage increases, the available current through the PMOS current-source devices increases.

This reduces the charging and discharging time of the internal nodes, resulting in lower propagation delay and higher oscillation frequency.

### Power Variation

The average power consumption increases with the control voltage because more current is available to the inverter stages.

This establishes a trade-off between operating frequency and power consumption.

| Control Voltage | Frequency Trend | Power Trend |
|---|---|---|
| 0 V | Low | Low |
| 0.2 V | Increasing | Increasing |
| 0.4 V | Increasing | Increasing |
| 0.6 V | Increasing | Increasing |
| 0.8 V | Increasing | Increasing |
| 1.0 V | Approximately 8.1 GHz in the nominal report | Approximately 260 µW in the nominal report |

---

## PVT Analysis

Process, Voltage, and Temperature (PVT) analysis was performed to investigate the sensitivity of the oscillator to manufacturing and operating-condition variations.

The analysis considered five process corners.

### Process Corners

| Process Corner | Description |
|---|---|
| FF | Fast NMOS, Fast PMOS |
| FS | Fast NMOS, Slow PMOS |
| NN | Nominal NMOS, Nominal PMOS |
| SF | Slow NMOS, Fast PMOS |
| SS | Slow NMOS, Slow PMOS |

### Simulation Conditions

| Parameter | Values |
|---|---|
| Control voltage | 0.5 V, 0.625 V, 0.75 V, 0.875 V, 1 V |
| Temperature | -40°C, 27°C, 125°C |
| Process corners | FF, FS, NN, SF, SS |
| Supply voltage | 1.8 V |

### PVT Results

| Performance Parameter | Observed Result |
|---|---:|
| Maximum frequency | 13.87 GHz |
| Minimum frequency | 1.238 GHz |
| Maximum average power | 469.9 µW |
| Minimum average power | 19.24 µW |

### PVT Observations

- The FF process corner generally provides higher oscillation frequency.
- The SS process corner generally provides lower oscillation frequency.
- Increasing temperature generally reduces oscillation frequency under the investigated conditions.
- Higher control voltage increases the available current and generally increases power consumption.
- The oscillator maintains oscillation over the investigated temperature and process conditions.

---



## Layout

After completing the schematic-level design and simulations, the circuit was implemented in Cadence Virtuoso Layout Suite XL.

The physical layout was constructed using the 90 nm CMOS technology design rules.

The layout implementation includes:

- PMOS and NMOS transistor structures.
- Supply and ground routing.
- Control-voltage connections.
- Internal inverter-stage connections.
- Feedback routing.
- Output routing for the oscillator signal.
- Metal interconnections and contact structures.

**Physical layout of the Current-Starved Ring Oscillator:**

![Layout](Screenshot%20from%202026-11-06%2013-56-32.png)

---

## Design Rule Check (DRC)

Design Rule Check (DRC) was performed to verify that the physical layout satisfies the fabrication design rules.

The supplied DRC screenshot confirms:

```text
No DRC errors found.
Total DRC violations: 0
```

This indicates that the layout passed the reported design-rule checks.

**DRC verification result:**

![DRC Clean](Screenshot%20from%202026-11-10%2016-25-21.png)

<details>
<summary>DRC Verification Details</summary>

- Physical layout checked against the 90 nm CMOS design rules.
- No reported DRC errors.
- Total reported DRC violations: 0.

</details>

---

## Layout Versus Schematic (LVS)

Layout Versus Schematic (LVS) verification was performed to verify the correspondence between the physical layout and the original schematic.

The supplied LVS run summary reports:

```text
0 cells have 0 Net mismatches
0 cells have 0 Device mismatches
0 cells have 0 Pin mismatches
0 cells have 0 Parameter mismatches

Total DRC violations: 0
```

The reported comparison results confirm that the extracted layout connectivity and device information match the schematic.

**LVS run summary:**

![LVS Summary](Screenshot%20from%202026-11-10%2016-25-47.jpg)

**Final LVS verification:**

![LVS Match](Screenshot%20from%202026-11-10%2016-25-52.png)

---



## Overall Performance Summary

The following table summarizes the major performance results obtained during the design and evaluation of the current-starved ring oscillator.

| Analysis | Main Result |
|---|---|
| Transient analysis | Sustained periodic oscillation |
| Nominal frequency | Approximately 8.076 GHz |
| Additional ADE screenshot | Approximately 13.2 GHz |
| Nominal average power | Approximately 260 µW |
| Additional ADE screenshot power | 115.4 µW |
| Frequency tuning | Controlled using Vctrl |
| PVT frequency range | 1.238–13.87 GHz |
| PVT power range | 19.24–469.9 µW |
| Monte Carlo samples | 200 per condition |
| Monte Carlo yield | 100% for both conditions |
| DRC | 0 reported violations |
| LVS | 0 reported mismatches |
| Post-layout frequency | Approximately 8.088 GHz |

**Note:** The frequency and power values correspond to different simulation configurations and analysis conditions. They should not be interpreted as measurements from one common operating point.

---

## Applications of Current-Starved Ring Oscillators

Current-starved ring oscillators are useful in integrated circuit applications that require controllable frequency generation.

Some potential applications include:

- Voltage-Controlled Oscillators (VCOs).
- Phase-Locked Loops (PLLs).
- Clock generation circuits.
- On-chip timing references.
- Frequency synthesizers.
- Clock recovery circuits.
- Integrated communication systems.
- Low-power timing and control circuits.

---

## Tools and Technologies

The following tools and technologies were used in the design, simulation, and verification process.

| Tool / Technology | Purpose |
|---|---|
| Cadence Virtuoso 6.1.8 | Schematic and layout design |
| ADE Explorer | Simulation setup and analysis |
| ADE Assembler | Parametric and statistical analysis |
| Spectre | Circuit simulation |
| ViVA XL | Waveform visualization |
| ViVA XL Calculator | Frequency and power measurements |
| 90 nm CMOS PDK | Transistor models and physical design rules |
| DRC | Design-rule verification |
| LVS | Layout-to-schematic verification |
| GitHub | Project documentation and version control |

---

## Project Structure

A suggested organization for the GitHub repository is:

```text
Current-Starved-Ring-Oscillator/
│
├── README.md
│
├── Schematic/
│   └── CSRO_Schematic.png
│
├── Simulation/
│   ├── Transient_Response.png
│   ├── Frequency_Analysis.png
│   ├── Power_Analysis.png
│   └── ADE_Results.png
│
├── Layout/
│   ├── CSRO_Layout.png
│   └── DRC_Result.png
│
├── Verification/
│   ├── LVS_Summary.png
│   └── LVS_Match.png
│
└── Post_Layout/
    └── Post_Layout_Simulation.png
```

**Note:** The folder structure above is a suggested GitHub organization. The files should be placed in the corresponding folders before using these paths in Markdown image links.

---

## Conclusion

The Current-Starved Ring Oscillator was designed and evaluated using Cadence Virtuoso and a 90 nm CMOS technology.

The simulation results demonstrate sustained oscillation, controllable frequency operation, and the expected relationship between control voltage, propagation delay, and power consumption.

PVT analysis was used to investigate variations in process corners and temperature, while Monte Carlo analysis evaluated the statistical effects of random device variations.

The physical layout was verified using DRC and LVS, with zero reported DRC violations and zero reported schematic-to-layout mismatches.

Post-layout simulation further confirmed that the oscillator maintains its intended functionality after parasitic extraction.

Overall, this project demonstrates the complete design flow of a Current-Starved Ring Oscillator, covering:

- Circuit design and schematic implementation.
- Transient simulation.
- Frequency and power analysis.
- Parametric analysis.
- PVT analysis.
- Monte Carlo statistical analysis.
- DRC and LVS verification.
- Post-layout simulation.
