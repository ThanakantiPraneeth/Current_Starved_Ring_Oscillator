# Current_Starved_Ring_Oscillator
Designing and simulating a current-starved ring oscillator (CSRO)—often used as a Voltage-Controlled Oscillator (VCO)—in Cadence Virtuoso involves building the inverter chain with current-limiting transistors and running transient and parametric simulations.

For a comprehensive breakdown of the design methodology, performance metrics, Monte Carlo analysis, and PVT (Process, Voltage, Temperature) considerations, please refer to the detailed technical document: **CSRO_REPORT.pdf**.

## Theory of Operation
A conventional ring oscillator generates a periodic output waveform by connecting an odd number of inverter stages in a closed loop, but it inherently has limited control over its oscillation frequency. A current-starved ring oscillator (CSRO) improves upon this by limiting the current available to each inverter stage using PMOS source transistors. 

The approximate oscillation frequency for an N-stage ring oscillator is governed by the equation $F_{osc}=\frac{1}{2 \cdot N \cdot t_{d}}$, where $N$ is the number of delay stages and $t_{d}$ is the average propagation delay of each stage. In a current-starved configuration, modifying the control voltage ($V_{ctrl}$) changes the available charging current, which in turn alters the charging/discharging time of the internal nodes and directly tunes the oscillation frequency.

The core components of this 90 nm CMOS architecture operate as follows:
* **Delay stages:** Each CMOS inverter stage (comprising one middle PMOS and one bottom NMOS) introduces a finite propagation delay to the signal.
* **PMOS current source:** An upper PMOS transistor acts as a current source, limiting the charging current available to the inverter.
* **Frequency control:** The control voltage ($V_{ctrl}$) is applied to the gate of the upper PMOS; varying this voltage determines how quickly the output node charges, dynamically controlling the delay.
* **Feedback connection:** The output of the final inverter is routed back to the input of the first inverter, ensuring continuous switching and sustained oscillation under a 1.8 V supply ($V_{DD}$).

## Schematic Design
The fundamental schematic of the CSRO built in Cadence Virtuoso.
![Schematic](Screenshot%20from%202026-11-03%2011-31-04.png)

## Transient Simulation
Initial transient response of the oscillator.
![Transient Simulation](Screenshot%20from%202026-11-03%2011-49-01.png)

## Frequency Analysis
Simulation showing the frequency calculation (e.g., ~13.2 GHz) based on the output waveform.
![Frequency Analysis](Screenshot%20from%202026-11-03%2011-54-35.png)

## Power and Performance
Visualization of the output waveform alongside power consumption metrics.
![Power Waveform 1](Screenshot%20from%202026-11-03%2011-59-22.jpg)

Detailed view of average power calculations.
![Power Waveform 2](Screenshot%20from%202026-11-03%2012-11-28.jpg)

## Layout
The physical layout of the current-starved ring oscillator.
![Layout](Screenshot%20from%202026-11-06%2013-56-32.png)

## Design Rule Check (DRC)
Verification showing that the layout passes all Design Rule Checks with 0 errors.
![DRC Clean](Screenshot%20from%202026-11-10%2016-25-21.png)

## Layout Versus Schematic (LVS)
LVS run summary and final verification showing that the schematic and layout match perfectly.
![LVS Summary](Screenshot%20from%202026-11-10%2016-25-47.jpg)
![LVS Match](Screenshot%20from%202026-11-10%2016-25-52.png)
