---
name: opendss-fault-study
description: Use when computing short circuit duties in OpenDSS — three phase, single line to ground, or line to line fault currents, X/R ratios, protective device duty checks — or when fault currents look implausibly high or low, especially with inverter based resources or impedance grounding.
---

# OpenDSS Fault Study

## Overview
`mode=faultstudy` computes bolted fault duties at every bus from the system admittance model. Accuracy is set entirely by the impedance data: source strength (MVAsc3/MVAsc1 or R/X), transformer %Z, conductor R0/X0, and grounding.

## Workflow (validated)
```
Redirect feeder.dss
Solve                    ! establish operating point first
Set mode=faultstudy
Solve
Export Faultstudy        ! <Ckt>_EXP_FAULTS.csv: per bus 3-Phase, 1-Phase(SLG), L-L amperes
```
For a specific fault with the full network response (voltages, branch flows during fault):
```
New Fault.F1 bus1=MidBus phases=3 r=0.0001      ! 3ph bolted
! SLG on phase A: New Fault.F1 bus1=MidBus.1 phases=1 r=0.0001
Set mode=snapshot
Solve
Export Currents
```
X/R at buses: `Show Faults` writes `<Ckt>_FaultStudy.Txt` including Thevenin impedances (headless note: Show spawns an editor; read the .txt file directly).

## Data Checklist Before Trusting Numbers
- Vsource `MVAsc1` as well as `MVAsc3` — SLG duty is wrong without zero sequence strength.
- Transformer winding connections: a delta wye transformer is a zero sequence source on the wye side and blocks it upstream.
- Zero sequence line data (`r0`, `x0`) — copying positive sequence values understates ground fault impedance error.
- Grounding: NGRs, zigzag transformers, and resistance grounding cap SLG current far below three phase duty; verify the exported SLG value against I = V_LN / (R_ground path) by hand.

## Inverter Based Resources (critical)
OpenDSS Vsource/Generator models deliver Thevenin limited current; real grid forming or grid following inverters limit to roughly 1.1 to 1.5 pu of rating (IEEE 2800 context). For islanded or IBR dominant systems:
- Represent the inverter for fault studies with an impedance sized to its current limit (V/Ilimit), NOT its stiff control impedance used for load flow.
- State this substitution explicitly in results; three phase duties from a stiff Vsource proxy are upper bounds.

## Common Mistakes
| Mistake | Consequence |
|---|---|
| Omitting MVAsc1 / zero sequence data | SLG duties meaningless |
| Reading duties while a Fault object is still active from a prior run | Contaminated results — `Clear` or remove Fault elements between studies |
| Using load flow source impedance for IBR fault duty | Orders of magnitude overstatement |
| Comparing first cycle duties to interrupting ratings directly | Apply X/R based multipliers per IEEE C37.010/C37.13 before device adequacy claims |
