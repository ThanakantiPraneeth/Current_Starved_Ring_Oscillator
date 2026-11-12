
---

## Design Specifications

The current-starved ring oscillator was designed using a 90 nm CMOS process in Cadence Virtuoso. The design uses PMOS current-source transistors to regulate the current supplied to the inverter-based delay stages.

| Parameter | Specification |
|---|---|
| Circuit type | PMOS-source Current-Starved Ring Oscillator |
| Technology | 90 nm CMOS |
| Number of delay stages | 3 (as documented in the technical report) |
| Supply voltage (`VDD`) | 1.8 V |
| Control voltage (`Vctrl`) | 1 V (nominal condition) |
| Output node | `Vosc` |
| Simulation tool | Cadence Virtuoso |
| Simulator | Spectre |
| Main analysis | Transient |
| Physical verification | DRC and LVS |
| Post-layout analysis | Parasitic-extracted simulation |

## Mathematical Background

### 1. Ring Oscillator Frequency

The oscillation frequency of an N-stage ring oscillator is approximately determined by:

\[
f_{osc}=\frac{1}{2Nt_d}
\]

Where:

- \(f_{osc}\) = Oscillation frequency.
- \(N\) = Number of inverter delay stages.
- \(t_d\) = Average propagation delay of each stage.

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

The frequency measurement in the report is approximately 8.076 GHz.

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

## Cadence Virtuoso Simulation Methodology

The design and simulation process was carried out through the following steps:

1. Created the current-starved inverter delay cell using PMOS and NMOS transistors.
2. Connected multiple delay stages in a closed feedback loop.
3. Added PMOS current-source devices to control the current supplied to the inverter stages.
4. Applied the supply voltage of 1.8 V.
5. Applied the control voltage to the PMOS current-source gates.
6. Configured the transient analysis in ADE Explorer.
7. Executed the Spectre simulation.
8. Observed the output waveform at the `Vosc` node.
9. Measured the oscillation frequency using the ViVA XL Calculator.
10. Evaluated average power using the supply-power waveform.
11. Performed parametric analysis to study the effect of the control voltage.
12. Performed PVT and Monte Carlo analyses to evaluate circuit robustness.
13. Designed the physical layout and performed DRC and LVS verification.
14. Extracted layout parasitics and evaluated post-layout performance.

## ADE Explorer and Calculator Measurements

The Cadence ViVA XL Calculator was used to obtain the oscillator frequency from the output waveform.

The frequency measurement expression used in the calculator was:

```skill
frequency(VT("/vosc"))
```

The average power expression used in ADE Explorer was:

```skill
average(getData(":pwr" ?result "tran"))
```

The measurement process involves:

- Selecting the output waveform.
- Identifying successive oscillation cycles.
- Calculating the time period.
- Determining the oscillation frequency.
- Measuring the average power consumption over the transient simulation interval.

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

## Frequency and Power Analysis

A parametric analysis was performed by varying the control voltage while maintaining the supply voltage at 1.8 V.

The control voltage was investigated over the range of 0 V to 1 V.

The analysis demonstrates the relationship between the control voltage, oscillation frequency, and average power consumption.

### Frequency Variation

As the control voltage increases, the available current through the PMOS current-source devices increases.

This reduces the charging and discharging time of the internal nodes, resulting in lower propagation delay and higher oscillation frequency.

### Power Variation

The average power consumption increases with the control voltage because more current is available to the inverter stages.

This establishes a trade-off between operating frequency and power consumption.

| Control voltage | Frequency trend | Power trend |
|---|---|---|
| 0 V | Low | Low |
| 0.2 V | Increasing | Increasing |
| 0.4 V | Increasing | Increasing |
| 0.6 V | Increasing | Increasing |
| 0.8 V | Increasing | Increasing |
| 1.0 V | Approximately 8.1 GHz in the nominal report | Approximately 260 µW in the nominal report |

## PVT Analysis

Process, Voltage, and Temperature analysis was performed to investigate the sensitivity of the oscillator to manufacturing and operating-condition variations.

The analysis considered five process corners:

| Process corner | Description |
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

| Performance parameter | Observed result |
|---|---:|
| Maximum frequency | 13.87 GHz |
| Minimum frequency | 1.238 GHz |
| Maximum average power | 469.9 µW |
| Minimum average power | 19.24 µW |

### PVT Observations

- The FF process corner generally provides higher oscillation frequency.
- The SS process corner generally provides lower oscillation frequency.
- Increasing temperature generally reduces the oscillation frequency under the investigated conditions.
- Higher control voltage increases the available current and generally increases power consumption.
- The oscillator maintains oscillation over the investigated temperature and process conditions.

## Monte Carlo Analysis

Monte Carlo analysis was conducted to evaluate the statistical behavior of the oscillator under random device parameter variations.

A total of 200 simulation samples were used for each control-voltage condition.

The analysis considered:

1. \(V_{ctrl}=1\ V\)
2. \(V_{ctrl}=0.1\ V\)

### Monte Carlo Results

| Parameter | Vctrl = 1 V | Vctrl = 0.1 V |
|---|---:|---:|
| Number of samples | 200 | 200 |
| Yield | 100% | 100% |
| Mean frequency | 7.96888 GHz | 420.938 MHz |
| Frequency standard deviation | 636.128 MHz | 132.900 MHz |
| Minimum frequency | 6.528 GHz | 154.8 MHz |
| Maximum frequency | 9.715 GHz | 903.7 MHz |
| Mean power | 261.445 µW | 3.49059 µW |
| Power standard deviation | 28.6619 µW | 1.28092 µW |
| Minimum power | 199.8 µW | 1.062 µW |
| Maximum power | 338.7 µW | 8.839 µW |

### Monte Carlo Observations

At a control voltage of 1 V, the oscillator provides a mean frequency of approximately 7.97 GHz and mean power consumption of 261.445 µW.

At a control voltage of 0.1 V, the mean frequency decreases to approximately 420.938 MHz, while mean power decreases to approximately 3.49059 µW.

All 200 samples produced valid results under both simulation conditions, corresponding to a 100% simulated yield.

The statistical results demonstrate the influence of random device variations on oscillator frequency and power consumption.

## Physical Design and Verification

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

### Design Rule Check (DRC)

DRC was performed to verify that the physical layout satisfies the fabrication design rules.

The supplied DRC screenshot confirms:

```text
No DRC errors found.
Total DRC violations: 0
```

This indicates that the layout passed the reported design-rule checks.

### Layout Versus Schematic (LVS)

LVS was performed to verify the correspondence between the physical layout and the original schematic.

The supplied LVS run summary reports:

```text
0 cells have 0 net mismatches
0 cells have 0 device mismatches
0 cells have 0 pin mismatches
0 cells have 0 parameter mismatches
Total DRC violations: 0
```

The reported comparison results confirm that the extracted layout connectivity and device information match the schematic.

### Parasitic Extraction (PEX)

Parasitic extraction was performed to account for the additional resistance and capacitance introduced by the physical layout and interconnects.

The extracted view was used for post-layout transient simulation to evaluate the oscillator after including layout parasitic effects.

### Post-Layout Simulation

The technical report documents the following comparison:

| Parameter | Frequency |
|---|---:|
| Pre-layout simulation | 8.076 GHz |
| Post-layout simulation | 8.088 GHz |

The post-layout result demonstrates that the oscillator maintains its intended operation after parasitic extraction.

The small frequency difference indicates that the extracted layout preserves the intended oscillation behavior under the reported simulation conditions.

## Overall Performance Summary

| Analysis | Main result |
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

## Applications of Current-Starved Ring Oscillators

Current-starved ring oscillators are useful in integrated circuit applications that require controllable frequency generation.

Some potential applications include:

- Voltage-controlled oscillators (VCOs).
- Phase-locked loops (PLLs).
- Clock generation circuits.
- On-chip timing references.
- Frequency synthesizers.
- Clock recovery circuits.
- Integrated communication systems.
- Low-power timing and control circuits.

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

*Note: The folder structure above is a suggested GitHub organization. The files should be placed in the corresponding folders before using these paths in Markdown image links.*

## Tools and Technologies

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
| PEX | Parasitic extraction |
| GitHub | Project documentation and version control |

## Conclusion

The current-starved ring oscillator was designed and evaluated using Cadence Virtuoso and a 90 nm CMOS technology.

The simulation results demonstrate sustained oscillation, controllable frequency operation, and the expected relationship between control voltage, propagation delay, and power consumption.

PVT analysis was used to investigate variations in process corners and temperature, while Monte Carlo analysis evaluated the statistical effects of random device variations.

The physical layout was verified using DRC and LVS, with zero reported DRC violations and zero reported schematic-to-layout mismatches. Post-layout simulation further confirmed that the oscillator maintains its intended functionality after parasitic extraction.

Overall, this project demonstrates the complete design flow of a current-starved ring oscillator, covering circuit design, transient simulation, performance analysis, statistical evaluation, physical layout, verification, and post-layout analysis.

---

