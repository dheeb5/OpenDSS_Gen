---
name: opendss-harmonics-analysis
description: Use when running harmonic distortion studies in OpenDSS — THD at buses, individual harmonic magnitudes, IEEE 519 compliance screening, resonance checks with capacitors or cable charging — or when harmonic results look implausibly amplified.
---

# OpenDSS Harmonics Analysis

## Overview
Harmonics mode replays the converged fundamental solution at each harmonic frequency, with nonlinear elements (loads, PV, generators) injecting per their assigned `Spectrum`. Monitors record one row per harmonic; THD is computed in post processing.

## Workflow (validated)
```
Redirect feeder.dss
New Spectrum.sixpulse numharm=5 harmonic=[1 5 7 11 13] %mag=[100 20 14 9 7] angle=[0 0 0 0 0]
Edit Load.Plant spectrum=sixpulse
New Monitor.mon_poi element=Transformer.T1 terminal=2 mode=0
Solve                      ! fundamental power flow MUST converge first
Set mode=harmonics         ! solves every defined spectrum frequency
Solve
Export Monitors mon_poi    ! rows: one per harmonic; columns: V, VAngle, I, IAngle per phase
```
`%mag` is percent of the element's FUNDAMENTAL current. Default: solves all frequencies present in any spectrum; restrict via `Set harmonics=[5 7 11 13]`.

## THD Post Processing
```python
import csv
rows = list(csv.DictReader(open("Ckt_Mon_mon_poi_1.csv")))
rows = [{k.strip(): float(v) for k, v in r.items()} for r in rows]
fund = next(r for r in rows if abs(r["Harmonic"] - 1.0) < 0.01)
thd_v = (sum(r["V1"]**2 for r in rows if r["Harmonic"] > 1.05) ** 0.5) / fund["V1"] * 100
```
Same formula per phase and for current (TDD uses maximum demand current as the base, not the instantaneous fundamental).

## IEEE 519-2022 Voltage Limits (at the PCC)
| Bus voltage | Individual harmonic | THD |
|---|---|---|
| V <= 1.0 kV | 5% | 8% |
| 1 kV < V <= 69 kV | 3% | 5% |
| 69 kV < V <= 161 kV | 1.5% | 2.5% |

## Modeling Cautions
- Keep capacitors and cable capacitance in the model — parallel resonance near a characteristic harmonic is the phenomenon of interest. If a single harmonic is amplified far above its injected percentage, suspect (and report) resonance; sweep with `Set mode=harmonics` plus a frequency scan rather than deleting the capacitor.
- Vendor emission data is usually a CURRENT spectrum. Applying it as a Vsource voltage spectrum behind a small impedance is an approximation that can wildly overstate harmonic currents; prefer spectrum on the Load/PVSystem/Generator element itself (current injection), and state the method used.
- Loads participate as harmonic impedances; set `%SeriesRL` if the default parallel R‖L representation is inappropriate.

## Common Mistakes
| Mistake | Consequence |
|---|---|
| Skipping the fundamental Solve | Injections scale from a wrong operating point |
| Reading THD off a single Export Voltages | That export is per frequency; THD needs all rows from a monitor |
| Comparing current THD to IEEE 519 tables without TDD basis | 519 current limits are in percent of maximum demand current and depend on Isc/IL |
| Spectrum angles all zero for diverse sources | Overstates summation; note it as a conservative assumption |
