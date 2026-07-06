---
name: opendss-powerflow-analysis
description: Use when running a steady state load flow in OpenDSS, when bus voltages, losses, equipment loading, or power balance must be evaluated against limits, or when a voltage drop or undervoltage complaint must be quantified.
---

# OpenDSS Power Flow (Snapshot) Analysis

## Overview
A snapshot solve gives the operating point for one loading condition. The analysis deliverable is not the solve itself but the screening: voltages against ANSI C84.1, loading against ratings, and losses against expectation.

## Workflow
```
Redirect feeder.dss        ! model with voltagebases + CalcVoltageBases already set
Set mode=snapshot
Solve
Export Summary             ! convergence, total MW/Mvar, losses
Export Voltages            ! per node magnitude, angle, pu
Export Powers kva elem     ! per element terminal flows
Export Losses              ! per element + total
Export Overloads           ! elements above normal rating
Export Capacity            ! remaining headroom (needs EnergyMeter)
```

## Screening Criteria
| Quantity | Limit | Source |
|---|---|---|
| Service voltage | 0.95 to 1.05 pu (Range A) | ANSI C84.1 |
| Utilization voltage | 0.917 to 1.05 pu (Range B, short duration) | ANSI C84.1 |
| Transformer loading | <= 100% nameplate continuous (planning studies often 80%) | project criteria |
| Feeder losses | Typically 1% to 5% of load; >5% or <0.1% warrants a model check | rule of thumb |
| DER interconnection voltage | Per IEEE 1547-2018 Category ranges | IEEE 1547 |

## Reading the Exports
- `_EXP_VOLTAGES.csv`: columns `pu1, pu2, pu3` per bus. Minimum pu across all nodes is the headline number. A pu of exactly 0 is an unenergized island, not a low voltage.
- `_EXP_POWERS.csv`: terminal 1 positive = power flowing INTO the element. Source row shows total generation; algebraic sum against total load + losses must balance within solver tolerance.
- `_EXP_LOSSES.csv`: final row is total circuit losses (kW, kvar). Loss percent = total loss kW / total load kW.
- Report voltages with the measured basis stated (line to neutral vs line to line) and to 4 significant figures at most; more implies false precision.

## Quick Python Post Processing
```python
from dss import DSS
DSS.Text.Command = 'compile "feeder.dss"'
DSS.ActiveCircuit.Solution.Solve()
names, pu = DSS.ActiveCircuit.AllNodeNames, DSS.ActiveCircuit.AllBusVmagPu
worst = min(zip(pu, names))
print(f"min voltage {worst[0]:.4f} pu at {worst[1]}; losses {DSS.ActiveCircuit.Losses[0]/1000:.1f} kW")
```

## Common Mistakes
| Mistake | Fix |
|---|---|
| Reporting the solved case as "the" answer | State loading assumption (peak, demand factor, dispatch) with the result |
| Screening only three phase buses | Single phase laterals usually hold the minimum voltage |
| Ignoring convergence status because numbers appeared | Check Export Summary first; a non converged case still exports numbers |
| Comparing LN magnitudes to LL limits | Normalize in pu before screening |
