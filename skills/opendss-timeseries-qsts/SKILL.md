---
name: opendss-timeseries-qsts
description: Use when a study needs time series simulation in OpenDSS — daily or yearly load profiles, PV output variation, storage cycling, energy calculations, regulator or capacitor switching counts — rather than a single snapshot.
---

# OpenDSS Time Series (QSTS) Analysis

## Overview
Quasi Static Time Series (QSTS) runs a sequence of power flows, one per time step, with loads and DER following LoadShapes. Deliverables are energy (EnergyMeter), time traces (Monitor), and exceedance statistics.

## Workflow (validated)
```
Redirect feeder.dss
New Loadshape.day24 npts=24 interval=1 mult=[0.4 0.35 0.32 0.3 0.32 0.4 0.55 0.7 0.82 0.9 0.95 1.0 0.98 0.95 0.92 0.9 0.94 1.0 0.98 0.9 0.8 0.65 0.55 0.45]
Edit Load.Plant daily=day24
New EnergyMeter.M1 element=Line.L1 terminal=1
New Monitor.mon_load element=Load.Plant terminal=1 mode=1 ppolar=no
Set mode=daily number=24 stepsize=1h
Solve
Export Monitors mon_load
Export Meters
```
`number` = steps to run, `stepsize` = interval (`1h`, `15m`, `1s`). `mode=yearly` uses the `yearly` shape property and typically `number=8760`.

## Monitor Modes
| mode | Records |
|---|---|
| 0 | V and I phasors at the terminal |
| 1 | P and Q per phase (`ppolar=no` gives rectangular kW/kvar columns) |
| 3 | Element state variables (machines, inverters, storage) |
| 5 | Taps (attach to RegControl transformer winding) |
- Loadshape from file: `New Loadshape.y npts=8760 interval=1 mult=(file=profile.csv)` — one multiplier per line.
- PV: assign `daily=<shape>` to `PVSystem` irradiance via `Tshape`/`daily` properties; Storage dispatch follows `dispmode` or a shape.

## Reading Results
- Monitor CSV: `<Ckt>_Mon_<name>_1.csv`, one row per step, columns per channel. First two columns are hour and t(sec).
- `Export Meters`: kWh, kvarh, max kW, losses kWh, overload duration registers per EnergyMeter zone.
- Voltage exceedance: use a mode=0 monitor at critical buses, post process rows below 0.95 pu; report count and duration, not just the minimum.

## Common Mistakes
| Mistake | Symptom | Fix |
|---|---|---|
| Loadshape npts vs number mismatch | Shape wraps or truncates silently | Align `npts`, `interval`, `number`, `stepsize` |
| Forgetting `daily=` assignment on loads | Flat profile, identical steps | Assign the shape to every time varying element |
| EnergyMeter added after Solve | Empty registers | Define instrumentation before solving |
| Using snapshot loads at peak kW with a peaky shape | Double counting peak | Set kW at base and let mult carry the profile, or normalize the shape |
| 1 s steps for an energy study | Hours of runtime, no benefit | Match stepsize to the phenomenon |
