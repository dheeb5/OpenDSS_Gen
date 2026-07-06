---
name: opendss-model-efficiency
description: Use when an OpenDSS model solves slowly, when QSTS or Monte Carlo loops take too long, when a model has grown too detailed for its study purpose, or when repeated recompilation of scripts dominates runtime.
---

# OpenDSS Model and Solve Efficiency

## Overview
Two levers: make the MODEL no more detailed than the study requires, and make the SOLVE LOOP avoid repeated parsing and file I/O. The biggest single win in scripted studies is driving the in memory engine through the Python API instead of recompiling text scripts per scenario.

## Model Right Sizing
| Study | Sufficient detail |
|---|---|
| Feeder level power flow | Aggregate secondary loads at the service transformer; skip service drops |
| Fault duty at a switchboard | Model impedances from source to that bus; lump everything downstream |
| Harmonics | Detail only the harmonic sources and the path to the point of evaluation; capacitors and cable capacitance MUST stay (resonance) |
| Dynamics | Keep machines/inverters and major impedances; static loads can lump |
- Replace an upstream network with the Vsource equivalent: `MVAsc3`/`MVAsc1` or `R1 X1 R0 X0` at the boundary bus.
- Merge series line sections with identical linecode into one Line of summed length.
- Avoid micro impedance "busbar" lines; use `switch=y` (they also destroy conditioning).
- OpenDSS can auto reduce: place an `EnergyMeter` at the feeder head, then `Reduce` (see Help) to eliminate short lines and merge nodes for planning class runs.

## Solve Loop Efficiency
```python
# Scenario sweeps: compile ONCE, then mutate and re-solve in memory
from dss import DSS
DSS.Text.Command = 'compile "feeder.dss"'
solution = DSS.ActiveCircuit.Solution
for kw in range(500, 3001, 100):
    DSS.Text.Command = f"Edit Load.Plant kw={kw}"
    solution.Solve()
    v = DSS.ActiveCircuit.AllBusVmagPu   # read results via API, no CSV round trip
```
- Never `Clear` + recompile inside a loop; `Edit` the changed element only.
- Read results from API properties (`AllBusVmagPu`, `CktElement.Powers`, monitor channels) instead of `Export` + file parse per iteration.
- Attach Monitors/EnergyMeters only where data is needed; every monitor samples every step in QSTS.
- QSTS: choose the coarsest `stepsize` the phenomenon allows (1 h for energy, 15 min for voltage regulation studies, 1 s only for control interaction).
- `Set controlmode=off` when regulator/capacitor actions are irrelevant to the question.
- Algorithm: default (`Normal`) fixed point is fastest when it converges; switch to `Newton` only for stressed cases.
- Parallelize scenarios across processes (each with its own DSS instance), not threads (the engine instance is stateful).

## Common Mistakes
| Mistake | Cost |
|---|---|
| Recompiling the whole model per scenario | 10x to 100x slower sweeps |
| Export CSV + parse inside loops | File I/O dominates runtime |
| Monitors on every element "just in case" | Memory and per step overhead |
| Modeling every service drop for a feeder head study | Slower solves, no accuracy gain at the point of interest |
| Deleting shunt capacitance to "simplify" a harmonics model | Misses resonance — wrong answers, not just slower ones |
