# Current_Starved_Ring_Oscillator
Designing and simulating a current-starved ring oscillator (CSRO)—often used as a Voltage-Controlled Oscillator (VCO)—in Cadence Virtuoso involves building the inverter chain with current-limiting transistors and running transient and parametric simulations.

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
