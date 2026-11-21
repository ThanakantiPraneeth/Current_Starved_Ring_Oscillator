# Current-Starved Ring Oscillator (CSRO)

Designing and simulating a **3-Stage Current-Starved Ring Oscillator (CSRO)**—often used as a Voltage-Controlled Oscillator (VCO)—in Cadence Virtuoso involves building a 12-transistor current-mirror biased inverter chain and running transient, frequency, power, and physical verification flows.

This project presents the complete design, schematic implementation, initial-condition transient simulation, performance evaluation, physical layout, verification (DRC, LVS, and QRC/RCX parasitic extraction), and post-layout (RCX) re-simulation of a **3-Stage Current-Starved Ring Oscillator** using **90 nm CMOS technology (`gpdk090`)** in Cadence Virtuoso IC 6.1.8.

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
11. [RC Extraction (QRC / Parasitic Extraction)](#rc-extraction-qrc--parasitic-extraction)
12. [Post-Layout (RCX) Simulation](#post-layout-rcx-simulation)
13. [Step-by-Step Implementation Workflow](#step-by-step-implementation-workflow)
14. [Overall Performance Summary](#overall-performance-summary)
15. [Power-Delay Product (PDP), Energy-Delay Product (EDP) and Effective Capacitance](#power-delay-product-pdp-energy-delay-product-edp-and-effective-capacitance)
16. [Applications of Current-Starved Ring Oscillators](#applications-of-current-starved-ring-oscillators)
17. [Tools and Technologies](#tools-and-technologies)
18. [Project Structure](#project-structure)
19. [Conclusion](#conclusion)
20. [Author & Acknowledgment](#author--acknowledgment)

---

## Theory of Operation

A conventional ring oscillator generates a periodic output waveform by connecting an odd number of inverter stages in a closed loop, but it inherently has limited control over its oscillation frequency.

A **Current-Starved Ring Oscillator (CSRO)** improves upon this by limiting the current available to each inverter stage via current-mirror biasing transistors controlled by an external control voltage ($V_{ctrl}$):

- **Biasing and Current Mirror Stage (`NM0`, `NM1`, `PM0`):** The external control voltage $V_{ctrl}$ drives the gate of bias input NMOS `NM0`, establishing a reference drain current $I_{ref}$ through diode-connected NMOS `NM1` and diode-connected PMOS `PM0`. This reference current is mirrored simultaneously across the upper PMOS current sources (`PM1`–`PM3`) and the lower NMOS current sinks (`NM2`–`NM4`).
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
| **Oscillation Frequency ($f_{osc}$)** | **$13.2021\text{ GHz}$** ($\approx 13.2\text{ GHz}$) | Pre-layout; period $T \approx 75.75\text{ ps}$ |
| **Average Power ($P_{avg}$)** | **$115.353\ \mu\text{W}$** ($\approx 115.4\ \mu\text{W}$) | Pre-layout; evaluated over transient window |
| **Physical Verification** | DRC, LVS, and QRC/RCX | 0 DRC errors, 0 LVS mismatches, `av_extracted` generated |
| **Post-Layout (RCX) Frequency** | **$5.487\text{ GHz}$** | `av_extracted` simulation |
| **Post-Layout (RCX) Power** | **$259.9\ \mu\text{W}$** | From `V1` supply current $\times$ $1.8\text{ V}$ |

### Main Design Components

1. **Bias & Current-Mirror PMOS (`PM0`–`PM3`, 4 instances, `gpdk090_pmos1v`):** `PM0` acts as the diode-connected reference load, while `PM1`–`PM3` mirror $I_{ref}$ as upper current sources ($W = 120\text{ nm}, L = 100\text{ nm}, m = 1$).
2. **Inverter Pull-Up PMOS (`PM4`–`PM6`, 3 instances, `gpdk090_pmos1v`):** Switching pull-up transistors for Stages 1, 2, and 3 ($W = 120\text{ nm}, L = 100\text{ nm}, m = 1$).
3. **Bias & Current-Mirror NMOS (`NM0`–`NM1`, 2 instances, `gpdk090_nmos1v`):** `NM0` receives the control voltage at its gate and `NM1` acts as the diode-connected NMOS current-mirror reference ($W = 120\text{ nm}, L = 100\text{ nm}, m = 1$).
4. **Current-Starving Pull-Down NMOS (`NM2`–`NM4`, 3 instances, `gpdk090_nmos1v`):** Current-sink transistors limiting discharge current for Stages 1, 2, and 3 ($W = 120\text{ nm}, L = 100\text{ nm}, m = 1$).
5. **Supply Voltage Source (`V0`):** DC source (`vdc = 1 V`).
6. **Control Voltage Source (`V1`):** DC source (`vdc = 1.8 V`).
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

From the simulated pre-layout oscillation frequency of $f_{osc} = 13.2021\text{ GHz}$:

$$T = \frac{1}{13.2021\text{ GHz}} \approx 75.75\text{ ps}$$

$$t_d = \frac{T}{6} \approx 12.62\text{ ps per stage}$$

### 3. Current-Starving Principle

The control voltage $V_{ctrl}$ regulates the available charging and discharging current $I_D$ supplied to each inverter stage:

- Lower available current ($V_{ctrl} \downarrow$) $\rightarrow$ Higher propagation delay ($t_d \uparrow$) $\rightarrow$ Lower frequency ($f_{osc} \downarrow$).
- Higher available current ($V_{ctrl} \uparrow$) $\rightarrow$ Lower propagation delay ($t_d \downarrow$) $\rightarrow$ Higher frequency ($f_{osc} \uparrow$).

### 4. Power Consumption

The average power consumption is evaluated from the supply voltage and average supply current:

$$P_{avg} = \frac{1}{T_{sim}} \int_{0}^{T_{sim}} P(t)\,dt = V_{supply} \times I_{avg}$$

For the dynamic switching component:

$$P_{dynamic} = \alpha C_L V_{DD}^{2} f_{osc}$$

where $\alpha$ is the switching activity factor, $C_L$ is the effective load capacitance, $V_{DD}$ is the supply voltage, and $f_{osc}$ is the oscillation frequency.

### 5. Delay, Power-Delay Product and Energy-Delay Product

The propagation delay per stage, $t_{pd}$, is the average of the low-to-high and high-to-low propagation delays:

$$t_{pd} = \frac{t_{PLH} + t_{PHL}}{2} = \frac{1}{2 N f_{osc}}$$

The Power-Delay Product (PDP) is the energy consumed per stage transition, and the Energy-Delay Product (EDP) weighs that energy against speed:

$$PDP = P_{avg} \times t_{pd} \qquad\qquad EDP = PDP \times t_{pd} = P_{avg} \times t_{pd}^{2}$$

### 6. Effective Switched (Short-Circuit) Capacitance

The total dynamic power including the short-circuit component is written in the same form as the dynamic power equation, with an equivalent lumped capacitance $C_{sc}$:

$$P_{dynamic\text{-}sc} = \alpha \, C_{sc} \, f_{osc} \, V_{DD}^{2} \quad\Longrightarrow\quad C_{sc} = \frac{P_{dynamic\text{-}sc}}{\alpha \, f_{osc} \, V_{DD}^{2}}$$

For the ring oscillator, every node completes one full charge/discharge cycle per oscillation period, so the activity factor is taken as $\alpha = 1$. Because the measured supply power also contains the short-circuit current of the inverters and the static bias branch, $C_{sc}$ is an effective lumped capacitance, not a purely physical one.

---

## Schematic Design

The fundamental schematic of the CSRO (`connectLib/csrosch/schematic`) was built in Cadence Virtuoso Schematic Editor L using 90 nm CMOS technology (`gpdk090_pmos1v` and `gpdk090_nmos1v`), uniform sizing ($W = 120\text{ nm}, L = 100\text{ nm}, m = 1$), and two DC sources `V0` ($1\text{ V}$) and `V1` ($1.8\text{ V}$).

**Schematic of the Current-Starved Ring Oscillator:**

![Schematic](Screenshot%20from%202026-11-03%2011-31-04.png)

---

## Transient Simulation

Transient analysis was performed using Cadence ADE Explorer and the Spectre simulator (`0` to `100 ns`, `moderate` accuracy). To break initial DC metastability and initiate oscillation, an initial condition of `/vosc = 0 V` was configured in ADE Explorer.

The transient response observed at the oscillator output node (`vosc`) confirms rapid startup and sustained periodic oscillations between $\approx 0.07\text{ V}$ and $\approx 0.92\text{ V}$.

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

The calculator determines a pre-layout oscillation frequency of **$13.2021\text{ GHz}$** ($13.2\text{ GHz}$ rounded) from the time-domain output waveform.

**Frequency analysis showing the oscillation frequency:**

![Frequency Analysis](Screenshot%20from%202026-11-03%2011-54-35.png)

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

### ADE Simulation Measurements (Pre-Layout)

| Measurement | Expression / Variable | Observed Value |
| --- | --- | --- |
| **Oscillation frequency** | `frequency(VT("/vosc"))` | **$13.2021\text{ GHz}$** ($13.2\text{ GHz}$) |
| **Average power** | `average(getData(":pwr" ?result "tran"))` | **$115.353\ \mu\text{W}$** ($115.4\ \mu\text{W}$) |
| **ADE design variables** | `vdd = 1`, `vosc = 1.8` | Values of sources `V0` and `V1` |

**Transient Output Voltage (`/vosc`) and Instantaneous Power (`:pwr`) Waveforms:**

![Transient and Power Waveforms](Screenshot%20from%202026-11-03%2011-59-22.png)

**ADE Explorer Scalar Outputs (`13.2 GHz` and `115.4 µW`):**

![ADE Scalar Outputs](Screenshot%20from%202026-11-03%2012-09-59.png)

---

## Layout

After completing the schematic-level design and simulations, the circuit (`csro/csro/layout`) was implemented in Cadence Virtuoso Layout Suite XL using 90 nm CMOS (`gpdk090`) design rules:

* All 7 PMOS devices grouped inside a shared `Nwell` connected to `vdd!` via N-tap contacts.
* All 5 NMOS devices placed in the P-substrate tied to `gnd!` via P-tap contacts.
* Interconnect routing using `Poly`, `Metal1`, and `Metal2` layers with `Via1` transitions for supply, ground, control voltage (`Vctrl`), internal inverter nodes, and feedback (`Vosc`).

**Physical layout of the Current-Starved Ring Oscillator:**

![Layout](Screenshot%20from%202026-11-06%2013-56-32.png)

---

## Design Rule Check (DRC)

Design Rule Check (DRC) was performed using **Assura DRC** to verify that the physical layout satisfies the 90 nm fabrication design rules.

The supplied DRC screenshot confirms:

```text
No DRC errors found.
Total DRC violations: 0
```

**DRC verification result:**

![DRC](Screenshot%20from%202026-11-10%2016-25-21.png)

* Physical layout checked against the 90 nm CMOS (`gpdk090`) design rules across `Nwell`, `Oxide`, `Poly`, `Pimp`/`Nimp`, `Cont`, `Metal1`, `Via1`, and `Metal2` layers.
* No reported DRC errors (`No DRC errors found`).
* Total reported DRC violations: 0.

---

## Layout Versus Schematic (LVS)

Layout Versus Schematic (LVS) verification was performed using **Assura LVS** to verify exact correspondence between the physical layout and the original schematic (`csrosch`).

**LVS run summary:**

![LVS](Screenshot%20from%202026-11-10%2016-25-47.png)

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

**Final LVS verification:**

![LAYOUT MATCH](Screenshot%20from%202026-11-10%2016-25-52.png)

---

## RC Extraction (QRC / Parasitic Extraction)

Following DRC and LVS, **Assura QRC (RCX)** was run on the layout to extract the parasitic resistance and capacitance of the interconnect. The extraction generates the `av_extracted` view of `csro`, which contains the original transistors together with parasitic capacitors (`pcapacitor`, values in the range of a few hundred attofarads) and parasitic resistors (`presistor`) added between nets.

**Schematic and AV Extraction Match:**

![Schematic and AV Extraction Match](Screenshot%202026-11-20%20203559.png)

**RC Extraction - Parasitic Extraction (`av_extracted` view):**

![RC Extraction](Screenshot%202026-11-20%20202940.png)

![RC Extraction](Screenshot%202026-11-20%20202917.png)

---

## Post-Layout (RCX) Simulation

To measure the effect of the layout parasitics, the extracted view was simulated with the same analysis settings as the pre-layout run.

### Testbench

1. A symbol (`csro/csro/symbol`) with pins `Vctrl` and `Vosc` was generated from the `csro` cell. The supply and ground are the global nets `vdd!` and `gnd!`.
2. A testbench `csro/csro_tb_new_o/schematic` was built with the `csro` symbol, a DC source `V1` (`vdc = 1.8`) on the `vdd` rail, a DC source `V0` (`vdc = 1`) on `Vctrl`, ground, and the `Vosc` output pin. These are the same voltages that are wired in the pre-layout schematic.

**Symbol of the `csro` cell (pins `Vctrl` and `Vosc`):**

![CSRO Symbol](Screenshot%202026-11-21%20082904.png)

**Post-layout testbench (`csro/csro_tb_new_o`):**

![Post-Layout Testbench](Screenshot%202026-11-21%20084053.png)

> **Note:** The source labelling in the Design Specifications table (`V0` = $V_{DD}$, `V1` = $V_{ctrl}$) is the reverse of how the sources are connected in the schematic and in this testbench. In both, `V1` ($1.8\text{ V}$) feeds the supply rail and `V0` ($1\text{ V}$) drives the control input. All simulation results in this README use the voltages exactly as wired.

### ADE Explorer Setup

| Setting | Value |
| :--- | :--- |
| **Simulator** | `spectre` |
| **Model library** | `gpdk090.scs`, section `NN` (the 45 nm model file was disabled) |
| **Analysis** | `tran`, `0` to `100n`, `moderate` accuracy |
| **Initial condition** | `0 V` on `/Vosc` |
| **Switch View List** | `spectre av_extracted cmos_sch cmos.sch schematic veriloga` |
| **Stop View List** | `spectre` |

Placing `av_extracted` before `schematic` in the Switch View List makes the simulator use the extracted view, so the parasitic R and C elements are included in the netlist.

### Post-Layout Transient Response

The extracted circuit continues to oscillate, with the output swinging between about $0.07\text{ V}$ and $0.85\text{ V}$, but at a visibly lower frequency than before layout.

**Post-layout output waveform, outputs table and supply current waveform (ADE Explorer):**

![Post-Layout ADE Results](Screenshot%202026-11-21%20091738.png)

### Post-Layout Measurement Expressions

The `:pwr` expression used before layout is not available from the testbench hierarchy (it returns `eval err`), so the power is computed from the supply current of `V1` multiplied by its voltage ($1.8\text{ V}$).

| Quantity | Calculator expression |
| :--- | :--- |
| **Frequency** | `frequency(VT("/Vosc"))` |
| **Average supply current** | `average(-IT("/V1/PLUS"))` |
| **Average power** | `average(-IT("/V1/PLUS") * 1.8)` |
| **Propagation delay** $t_{pd}$ | `1 / (6 * frequency(VT("/Vosc")))` |
| **PDP (per volt of supply)** | `average(-IT("/V1/PLUS") * 1) / (6 * frequency(VT("/Vosc")))` |
| **EDP (per volt of supply)** | `average(-IT("/V1/PLUS") * 1) / ((6 * frequency(VT("/Vosc"))) ** 2)` |

In the Calculator screenshots below, the supply-voltage factor in the expressions was entered as `1`, so the displayed current-based values are per volt of supply. Multiplying the PDP and EDP values by the actual supply voltage $V_1 = 1.8\text{ V}$ gives the final results listed in the Post-Layout Results table.

**Post-layout frequency and average supply current of `V1` (`5.487 GHz` and `144.4 µA`):**

![Post-Layout Frequency and Current](Screenshot%202026-11-21%20091213.png)

**Post-layout propagation delay $t_{pd}$ (`30.37 ps`):**

![Post-Layout tpd](Screenshot%202026-11-21%20091414.png)

**Post-layout PDP (Calculator output `4.385 fJ` with the voltage factor set to 1; $\times\,1.8\text{ V}$ gives `7.894 fJ`):**

![Post-Layout PDP](Screenshot%202026-11-21%20091452.png)

**Post-layout EDP (Calculator output `133.2e-27 J·s` with the voltage factor set to 1; $\times\,1.8\text{ V}$ gives `2.40e-25 J·s`):**

![Post-Layout EDP](Screenshot%202026-11-21%20091526.png)

**Post-layout Psc (Short Circuit Power of the Circuit is `173.2e-6 W`):**

![Post-Layout Psc](Screenshot%202026-11-21%20124304.png)

### Post-Layout Results

| Parameter | Value |
| :--- | :--- |
| **Oscillation frequency ($f_{osc}$)** | **$5.487\text{ GHz}$** |
| **Period ($T$)** | $182.25\text{ ps}$ |
| **Propagation delay ($t_{pd}$)** | **$30.37\text{ ps}$** |
| **High-to-low delay ($t_{PHL}$, estimated)** | $29.4\text{ ps}$ |
| **Low-to-high delay ($t_{PLH}$, estimated)** | $31.4\text{ ps}$ |
| **Average supply current ($I_{avg}$)** | $144.4\ \mu\text{A}$ |
| **Average power ($P_{avg} = 1.8\text{ V} \times I_{avg}$)** | **$259.9\ \mu\text{W}$** |
| **Power-Delay Product (PDP)** | **$7.894\text{ fJ}$** |
| **Energy-Delay Product (EDP)** | **$2.40 \times 10^{-25}\text{ J·s}$** |
| **Energy per cycle ($P_{avg}/f_{osc}$)** | $47.4\text{ fJ}$ |
| **Effective switched capacitance ($C_{sc}$)** | $14.62\text{ fF}$ |

### Propagation Delays $t_{PHL}$ and $t_{PLH}$ (Post-Layout)

The delay function of the ViVA XL Calculator was applied to the output `/Vosc` to measure the high time of the output pulse, i.e. the time from its rising crossing to its falling crossing of $0.6\text{ V}$:

```skill
delay(VT("/Vosc") 0.6 1 "rising" VT("/Vosc") 0.6 1 "falling" 0 0 nil nil)
```

The result is **$90.14\text{ ps}$**. Swapping the edges (`"falling"` first, `"rising"` second) returns the same magnitude with the opposite sign, **$-90.14\text{ ps}$**, because the first rising edge occurs before the first falling edge.

**Rising-to-falling delay on `/Vosc` (`90.14 ps`):**

![Delay Rising to Falling](Screenshot%202026-11-21%20131213.png)

**Falling-to-rising delay on `/Vosc` (`-90.14 ps`, same magnitude):**

![Delay Falling to Rising](Screenshot%202026-11-21%20131351.png)

**Extracting $t_{PHL}$ and $t_{PLH}$:** in a three-stage ring each node passes through the stages in turn, so for one node:

$$T_H = 2\,t_{PHL} + t_{PLH} \qquad T_L = 2\,t_{PLH} + t_{PHL} \qquad T = T_H + T_L = 3\,(t_{PLH} + t_{PHL})$$

Using the post-layout period $T = 182.25\text{ ps}$ ($f_{osc} = 5.487\text{ GHz}$) and the measured high time $T_H = 90.14\text{ ps}$:

$$T_L = T - T_H = 182.25 - 90.14 = 92.11\text{ ps}$$

$$t_{PLH} + t_{PHL} = \frac{T}{3} = 60.75\text{ ps} \qquad t_{PHL} - t_{PLH} = T_H - T_L = -1.97\text{ ps}$$

$$t_{PHL} = \frac{60.75 - 1.97}{2} \approx 29.4\text{ ps} \qquad t_{PLH} = \frac{60.75 + 1.97}{2} \approx 31.4\text{ ps}$$

$$t_{pd} = \frac{t_{PLH} + t_{PHL}}{2} = \frac{31.4 + 29.4}{2} \approx 30.4\text{ ps}$$

| Quantity | Value (Post-Layout) |
| :--- | :--- |
| **High time $T_H$** (measured at $0.6\text{ V}$) | $90.14\text{ ps}$ |
| **Low time $T_L = T - T_H$** | $92.11\text{ ps}$ |
| **$t_{PHL}$ (estimated)** | $29.4\text{ ps}$ |
| **$t_{PLH}$ (estimated)** | $31.4\text{ ps}$ |
| **$t_{pd} = (t_{PLH} + t_{PHL})/2$** | $30.4\text{ ps}$ (matches $1/(6 f_{osc}) = 30.37\text{ ps}$) |

> **Note:** $t_{PHL}$ and $t_{PLH}$ here are estimated from the high and low times of the output rather than measured directly between the input and output of one stage. The $0.6\text{ V}$ threshold is slightly above the midpoint of the output swing (about $0.46\text{ V}$ for a $0.07\text{ V}$ to $0.85\text{ V}$ swing), so the split between the two values is approximate, while their sum is fixed by the oscillation period. A direct measurement can be made at the $50\%$ level between two internal stage nodes of the extracted view.

---

## Step-by-Step Implementation Workflow

### 1. Schematic Design & Pre-Layout Simulation

1. Launch Cadence Virtuoso IC 6.1.8 and open library `connectLib` / `csro`. Create a new schematic cell view named `csrosch`.
2. Place 7 PMOS (`pmos1v`) and 5 NMOS (`nmos1v`) transistors from `gpdk090`, setting all dimensions to $W = 120\text{ nm}$, $L = 100\text{ nm}$, and $m = 1$.
3. Wire the current-mirror biasing network (`NM0`, `NM1`, `PM0`) to the three cascaded current-starved inverter stages (`PM1`–`PM6`, `NM2`–`NM4`), connecting output `vosc` back to the first inverter input.
4. Connect the DC sources `V0 = 1 V` and `V1 = 1.8 V`, add pins `vdd`, `gnd`, and `vosc`, and run **Check and Save**.
5. Open Virtuoso ADE Explorer / Maestro, select the `spectre` simulator, set an initial condition of `0 V` on `/vosc`, and configure a transient analysis (`tran`) from `0` to `100n` with `moderate` accuracy.
6. Add output expressions `frequency(VT("/vosc"))` and `average(getData(":pwr" ?result "tran"))`, run the simulation, and verify sustained oscillation at $13.2021\text{ GHz}$ with $115.353\ \mu\text{W}$ average power.

### 2. Physical Layout, DRC, LVS & Parasitic Extraction

1. Launch **Virtuoso Layout Suite XL** from the schematic view and generate all transistor instances along with `vdd!`, `gnd!`, `Vctrl`, and `Vosc` pins.
2. Group all PMOS devices inside a shared `Nwell` region with N-tap substrate ties, and place NMOS devices in the P-substrate with P-tap ties.
3. Route interconnects using `Poly`, `Metal1`, and `Metal2` layers while adhering to 90 nm design rules.
4. Run **Assura $\rightarrow$ Run DRC** and confirm zero design rule violations (`No DRC errors found`).
5. Run **Assura $\rightarrow$ Run LVS** and verify `Schematic and Layout Match` with zero net, device, pin, or parameter mismatches.
6. Run **Assura $\rightarrow$ Run QRC (RCX)** to generate the `av_extracted` view with coupled parasitic resistances and capacitances.

### 3. Post-Layout (RCX) Simulation

1. Generate a symbol for `csro/csro` and build the testbench `csro_tb_new_o` with the sources on `vdd!` and `Vctrl`.
2. Launch ADE Explorer from the testbench, select `spectre`, add the `gpdk090` model file, and set the `tran` analysis (`0`–`100n`, `moderate`).
3. Set the Switch View List to `spectre av_extracted cmos_sch cmos.sch schematic veriloga` and the initial condition `0 V` on `/Vosc`.
4. Run the simulation and measure frequency, supply current, $t_{pd}$, PDP and EDP in the ViVA XL Calculator.
5. Compare against the pre-layout results.

---

## Overall Performance Summary

| Analysis / Verification | Pre-Layout | Post-Layout (RCX) |
| --- | --- | --- |
| **Transient analysis (`0–100 ns`)** | Sustained oscillation (`0.07 V` – `0.92 V`) | Sustained oscillation (`0.07 V` – `0.85 V`) |
| **Oscillation frequency ($f_{osc}$)** | **$13.2021\text{ GHz}$** | **$5.487\text{ GHz}$** |
| **Propagation delay per stage ($t_{pd}$)** | **$12.62\text{ ps}$** ($= 1/(6 f_{osc})$) | **$30.37\text{ ps}$** |
| **$t_{PHL}$ / $t_{PLH}$ (estimated)** | – | **$29.4\text{ ps}$ / $31.4\text{ ps}$** |
| **Average power ($P_{avg}$)** | **$115.353\ \mu\text{W}$** | **$259.9\ \mu\text{W}$** |
| **Power-Delay Product (PDP)** | **$1.456\text{ fJ}$** | **$7.894\text{ fJ}$** |
| **Energy-Delay Product (EDP)** | **$1.84 \times 10^{-26}\text{ J·s}$** | **$2.40 \times 10^{-25}\text{ J·s}$** |
| **Effective switched capacitance ($C_{sc}$)** | **$2.70\text{ fF}$** | **$14.62\text{ fF}$** |
| **DRC (Assura)** | **0 reported violations (`No DRC errors found`)** | – |
| **LVS (Assura)** | **0 reported mismatches (`Schematic and Layout Match`)** | – |
| **Parasitic extraction (QRC/RCX)** | – | **`av_extracted` view generated** |

---

## Power-Delay Product (PDP), Energy-Delay Product (EDP) and Effective Capacitance

The PDP and EDP quantify the energy efficiency and speed trade-off of the oscillator. The effective switched (short-circuit) capacitance $C_{sc}$ expresses the total dynamic power as an equivalent lumped capacitance.

### Pre-Layout Calculation

Using $f_{osc} = 13.2021\text{ GHz}$ and $P_{avg} = 115.353\ \mu\text{W}$:

**Oscillation period:**

$$T = \frac{1}{f_{osc}} = \frac{1}{13.2021\text{ GHz}} \approx 75.75\text{ ps}$$

**Propagation delay per stage** ($N = 3$):

$$t_{pd} = \frac{T}{2N} = \frac{1}{6 \times 13.2021\text{ GHz}} \approx 12.62\text{ ps}$$

**Power-Delay Product:**

$$PDP = P_{avg} \times t_{pd} = 115.353\ \mu\text{W} \times 12.62\text{ ps} \approx 1.456\text{ fJ}$$

**Energy-Delay Product:**

$$EDP = PDP \times t_{pd} = 1.456\text{ fJ} \times 12.62\text{ ps} \approx 1.84 \times 10^{-26}\text{ J·s}$$

**Energy per oscillation cycle** (cross-check, equal to $2N \times PDP$):

$$E_{cycle} = \frac{P_{avg}}{f_{osc}} = \frac{115.353\ \mu\text{W}}{13.2021\text{ GHz}} \approx 8.74\text{ fJ}$$

### Post-Layout (RCX) Calculation

Using $f_{osc} = 5.487\text{ GHz}$ and $I_{avg} = 144.4\ \mu\text{A}$ drawn from the $1.8\text{ V}$ source `V1`:

$$P_{avg} = 1.8\text{ V} \times 144.4\ \mu\text{A} \approx 259.9\ \mu\text{W}$$

$$t_{pd} = \frac{1}{6 \times 5.487\text{ GHz}} \approx 30.37\text{ ps}$$

$$PDP = P_{avg} \times t_{pd} = 259.9\ \mu\text{W} \times 30.37\text{ ps} \approx 7.894\text{ fJ}$$

$$EDP = PDP \times t_{pd} = 7.894\text{ fJ} \times 30.37\text{ ps} \approx 2.40 \times 10^{-25}\text{ J·s}$$

$$E_{cycle} = \frac{P_{avg}}{f_{osc}} = \frac{259.9\ \mu\text{W}}{5.487\text{ GHz}} \approx 47.4\text{ fJ}$$

### Effective Switched (Short-Circuit) Capacitance

From the dynamic power relation $P_{dynamic\text{-}sc} = \alpha \, C_{sc} \, f_{osc} \, V_{DD}^{2}$, with $\alpha = 1$ (each node completes one charge/discharge cycle per oscillation period) and the supply voltage $V_{DD} = 1.8\text{ V}$ of source `V1`:

$$C_{sc} = \frac{P_{dynamic\text{-}sc}}{\alpha \, f_{osc} \, V_{DD}^{2}}$$

**Pre-layout:**

$$C_{sc,pre} = \frac{115.353\ \mu\text{W}}{1 \times 13.2021\text{ GHz} \times (1.8\text{ V})^{2}} = \frac{115.353 \times 10^{-6}}{4.277 \times 10^{10}} \approx 2.70\text{ fF}$$

**Post-layout (RCX):**

$$C_{sc,post} = \frac{259.9\ \mu\text{W}}{1 \times 5.487\text{ GHz} \times (1.8\text{ V})^{2}} = \frac{259.9 \times 10^{-6}}{1.778 \times 10^{10}} \approx 14.62\text{ fF}$$

**Per-stage values** (the effective capacitance is shared by the three stage nodes, $C_{sc}/N$):

| | Pre-Layout | Post-Layout (RCX) |
| :--- | :--- | :--- |
| **Total $C_{sc}$** | $2.70\text{ fF}$ | $14.62\text{ fF}$ |
| **Per stage ($C_{sc}/3$)** | $0.90\text{ fF}$ | $4.87\text{ fF}$ |

**Capacitance added by the layout parasitics:**

$$\Delta C_{sc} = C_{sc,post} - C_{sc,pre} = 14.62\text{ fF} - 2.70\text{ fF} \approx 11.9\text{ fF} \quad (\approx 4.0\text{ fF per stage},\ \approx 5.4\times\ \text{increase})$$

**Cross-check using the charge per cycle:**

$$C_{sc,post} = \frac{I_{avg}}{f_{osc} \, V_{DD}} = \frac{144.4\ \mu\text{A}}{5.487\text{ GHz} \times 1.8\text{ V}} \approx 14.62\text{ fF}$$

### Pre-Layout vs Post-Layout (RCX) Comparison

| Metric | Formula | Pre-Layout | Post-Layout (RCX) | 
| :--- | :--- | :--- | :--- | 
| **Oscillation frequency ($f_{osc}$)** | `frequency(VT("/Vosc"))` | $13.2021\text{ GHz}$ | $5.487\text{ GHz}$ | 
| **Period ($T$)** | $1/f_{osc}$ | $75.75\text{ ps}$ | $182.25\text{ ps}$ | 
| **Propagation delay ($t_{pd}$)** | $1/(2N f_{osc})$ | $12.62\text{ ps}$ | $30.37\text{ ps}$ | 
| **Average power ($P_{avg}$)** | see notes | $115.353\ \mu\text{W}$ | $259.9\ \mu\text{W}$ | 
| **PDP** | $P_{avg} \times t_{pd}$ | $1.456\text{ fJ}$ | $7.894\text{ fJ}$ | 
| **EDP** | $PDP \times t_{pd}$ | $1.84 \times 10^{-26}\text{ J·s}$ | $2.40 \times 10^{-25}\text{ J·s}$ | 
| **Energy per cycle** | $P_{avg}/f_{osc}$ | $8.74\text{ fJ}$ | $47.4\text{ fJ}$ | 
| **Effective capacitance ($C_{sc}$)** | $P/(\alpha f_{osc} V_{DD}^{2})$ | $2.70\text{ fF}$ | $14.62\text{ fF}$ | 

### Observations and Notes

- The extracted parasitic capacitance adds load to every stage output, and the extracted resistance slows the edges further, so the post-layout oscillation frequency drops by about 58% and the stage delay rises from $12.62\text{ ps}$ to $30.37\text{ ps}$.
- Because the same current now takes longer to charge a larger capacitance, the energy per cycle and the PDP rise sharply. EDP contains $t_{pd}^{2}$, so it degrades the most.
- The effective switched capacitance rises from about $2.70\text{ fF}$ to $14.62\text{ fF}$, an increase of about $11.9\text{ fF}$, which is the load added by the extracted layout parasitics.
- $C_{sc}$ is computed from the total supply power with $\alpha = 1$. It therefore includes the short-circuit current of the inverters and the static bias branch (`PM0`, `NM0`, `NM1`) and should be read as an effective capacitance. A different activity-factor convention, for example $\alpha = 0.5$, would scale the values accordingly.
- $t_{pd}$ is derived from the oscillation frequency and represents the average of $t_{PLH}$ and $t_{PHL}$. It can be cross-checked by measuring both across one stage at the 50% crossing points.
- $P_{avg}$ is the total circuit power and includes the static bias branch (`PM0`, `NM0`, `NM1`) as well as switching power.
- **Measurement method:** the pre-layout power was obtained with `average(getData(":pwr" ?result "tran"))`, while the post-layout power was obtained from the supply current of `V1` multiplied by $1.8\text{ V}$. For a strict like-for-like comparison, both runs can be measured with the supply-current expression.

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
| **ViVA XL & Calculator** | Waveform visualization, frequency, power, delay, PDP and EDP measurements |
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
    ├── Screenshot from 2026-11-10 16-25-52.png
    ├── Screenshot 2026-11-20 203559.png
    ├── Screenshot 2026-11-20 202940.png
    ├── Screenshot 2026-11-20 202917.png
    ├── Screenshot 2026-11-21 082904.png
    ├── Screenshot 2026-11-21 084053.png
    ├── Screenshot 2026-11-21 091738.png
    ├── Screenshot 2026-11-21 091213.png
    ├── Screenshot 2026-11-21 091414.png
    ├── Screenshot 2026-11-21 091452.png
    ├── Screenshot 2026-11-21 091526.png
    ├── Screenshot 2026-11-21 155842.png
    └── Screenshot 2026-11-21 155920.png
```

---

## Conclusion

A 3-stage Current-Starved Ring Oscillator was successfully designed, simulated, and physically verified at the 90 nm CMOS technology node (`gpdk090`) in Cadence Virtuoso. Pre-layout transient analysis with an initial condition of `/vosc = 0 V` confirmed stable oscillation at **$13.2021\text{ GHz}$** with an average power dissipation of **$115.353\ \mu\text{W}$**. This corresponds to a propagation delay of about $12.62\text{ ps}$ per stage, a power-delay product of about $1.456\text{ fJ}$, an energy-delay product of about $1.84 \times 10^{-26}\text{ J·s}$, and an effective switched capacitance of about $2.70\text{ fF}$. The physical layout passed Assura DRC (`0` violations) and Assura LVS (`Schematic and Layout Match`), and Assura QRC/RCX parasitic extraction generated the `av_extracted` view.

Post-layout simulation of the extracted view showed the effect of the interconnect parasitics: the oscillation frequency fell to **$5.487\text{ GHz}$**, the stage delay rose to **$30.37\text{ ps}$**, the average power rose to about **$259.9\ \mu\text{W}$**, the PDP rose to about **$7.894\text{ fJ}$**, the EDP rose to about **$2.40 \times 10^{-25}\text{ J·s}$**, and the effective switched capacitance rose to about **$14.62\text{ fF}$** (an increase of about $11.9\text{ fF}$ from the layout parasitics). These results show that parasitic R and C have a large impact on the speed and energy efficiency of this small, high-frequency oscillator and must be considered in the design.

---

## Author & Acknowledgment

* **Author:** Thanakanti Praneeth
* **Roll Number:** 226EC1002
* **Programme:** M.Tech in VLSI & Microelectronics
* **Department:** Electronics and Communication Engineering
* **Institution:** Indian Institute of Information Technology, Design and Manufacturing (IIITDM), Kurnool, Andhra Pradesh
* **Course:** Digital IC Design Laboratory

> **Acknowledgment:** Sincere thanks to **Dr. P. Ranga Babu** (Associate Professor, Department of Electronics and Communication Engineering, IIITDM Kurnool) for his valuable guidance, technical mentorship, and support throughout this Digital IC Laboratory project.
