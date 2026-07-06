---
name: opendss-dynamics-analysis
description: Use when simulating electromechanical transients in OpenDSS — generator swings after a fault, frequency response, motor starting, grid forming inverter behavior in islanded systems — or when deciding whether OpenDSS dynamics mode is even the right tool versus EMT software.
---

# OpenDSS Dynamics Analysis

## Overview
Dynamics mode time steps machine differential equations (swing equation) against the phasor network solution. It is positive sequence RMS class dynamics, not EMT: adequate for rotor angle swings, frequency excursions, and basic inverter dispatch response; NOT for switching transients, PWM detail, or sub cycle controls (use PSCAD/EMTP class tools there).

## Workflow (validated)
```
Redirect feeder.dss
New Generator.G1 bus1=MidBus phases=3 kv=12.47 kw=500 pf=1.0 model=1 H=4 D=0.1 Xdp=0.27
New Monitor.mon_gen element=Generator.G1 terminal=1 mode=3   ! mode=3 = state variables
Solve                                    ! initialize from a converged power flow
Set mode=dynamics number=200 stepsize=0.0002
Solve
Export Monitors mon_gen                  ! columns: Frequency, Theta, Vd, PShaft, dSpeed, dTheta
```
`stepsize` in seconds (0.0002 s = ~1/83 of a 60 Hz cycle; keep <= 1 ms). `number` = steps, so simulated time = number x stepsize.

## Applying a Disturbance
```
New Fault.FLT bus1=MidBus phases=3 r=0.0001 ontime=0.1 temporary=yes
Set mode=dynamics number=5000 stepsize=0.0002
Solve
```
`ontime` (seconds into the run) closes the fault; `temporary=yes` lets it clear when current drops. For breaker clearing at a set time, run dynamics in segments: `Solve number=N`, then `Open Line.L1 term=1`, then continue with another `Solve`.

## Key Machine Parameters
| Property | Meaning |
|---|---|
| H | Inertia constant, MW s/MVA — dominates swing frequency |
| D | Damping factor |
| Xdp | Transient reactance (pu on machine base) — sets fault contribution |
| model=3 | Constant kW, dynamics capable model |
Inverters: `PVSystem`/`Storage` follow their control models per step; genuine grid forming droop/VSM behavior needs the vendor DLL (`UserModel=`) or the built in `dynamicexp`/GFM mode in recent versions — verify availability with `opendss --version` and state the modeling assumption in results.

## Common Mistakes
| Mistake | Consequence |
|---|---|
| Skipping the snapshot Solve before dynamics | Machines initialize from a wrong or zero operating point |
| Step size too large (> 1 ms) | Numerical oscillation mistaken for instability |
| Reading rotor angle from a mode=0 monitor | State variables need monitor `mode=3` |
| Using dynamics for switching surge or harmonic transient questions | Wrong physics class — use EMT tools |
| No damping (D=0) with matched H values | Undamped oscillation that is a modeling artifact |
