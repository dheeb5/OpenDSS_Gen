---
name: opendss-model-building
description: Use when creating or editing an OpenDSS circuit model (.dss files) — defining sources, lines, transformers, loads, PV, storage, generators — or when a compile fails with element definition errors, wrong voltages appear at first solve, or a single line diagram must be translated into DSS script.
---

# OpenDSS Model Building

## Overview
An OpenDSS model is a script of `New <Class>.<Name> param=value ...` statements. Definition order matters: an element may reference only objects already defined, and voltage bases must be declared after topology is complete.

## Required Skeleton (always this order)
```
Clear
New Circuit.Name basekv=12.47 pu=1.0 phases=3 bus1=SourceBus MVAsc3=200 MVAsc1=210
! 1. Support objects: LineCode, LoadShape, Spectrum, TCC curves
! 2. Topology: Line, Transformer, Reactor, Capacitor
! 3. Consumption/injection: Load, Generator, PVSystem, Storage
! 4. Instrumentation: Monitor, EnergyMeter
Set voltagebases=[12.47, 0.48]
CalcVoltageBases
```
`New Circuit` creates the slack Vsource; MVAsc3/MVAsc1 (or R1/X1) set its short circuit strength.

## Element Quick Reference (validated syntax)
```
New Linecode.336ACSR nphases=3 r1=0.058 x1=0.1206 r0=0.1784 x0=0.4047 c1=3.4 c0=1.6 units=kft
New Line.L1 bus1=SourceBus bus2=MidBus linecode=336ACSR length=6 units=kft
New Transformer.T1 phases=3 windings=2 buses=[HVbus, LVbus] conns=[delta, wye] kvs=[12.47, 0.48] kvas=[1500, 1500] xhl=5.75 %loadloss=1.0
New Load.Plant phases=3 bus1=LVbus kv=0.48 kw=1200 pf=0.92 model=1
New Generator.G1 bus1=MidBus phases=3 kv=12.47 kw=500 pf=1.0 model=1
New PVSystem.PV1 phases=3 bus1=LVbus kv=0.48 kVA=500 Pmpp=450 irradiance=1.0 %cutin=0.1 %cutout=0.1
New Storage.B1 phases=3 bus1=LVbus kv=0.48 kWrated=250 kWhrated=500 %stored=50
```

## kV Rules (top compile time trap)
| Element configuration | kv means |
|---|---|
| 3 phase (wye or delta) | Line to line kV |
| 1 phase, line to neutral | Line to neutral kV |
| 1 phase, line to line (2 hot terminals) | Line to line kV |
Bus node syntax: `bus1=BusA.1.2.3` (phases), `.0` grounds a terminal (e.g. wye neutral solidly grounded is default; `bus1=B.1.2.3.4` exposes the neutral node).

## Field-tested modeling conventions
- **Switches, breakers, ATS, fuses:** model as `Switch=true enabled=true` Line elements with a near-zero length (`length=0.001` kft) on the SAME LineCode as the adjacent conductor. Status can be toggled without re-deriving impedance, and it avoids the conditioning problems of a separate zero-Z linecode.
- **Bonding/grounding jumpers (SBJ):** document in a comment at each occurrence; do not model as separate branches.
- **Ampacity and impedance basis:** NEC 310.16 (75 C column) for `normamps`, `emergamps = 1.25 * normamps`; NEC Chapter 9 Table 9 per-conductor R/X divided by the number of parallel conductors per phase for R1/X1. Take R0/X0 conservatively from the single (un-paralleled) conductor values as a planning approximation, and flag it for revision once as-built geometry exists.
- **Single-phase line-to-line branch:** `phases=1` with explicit `.1.2` node suffixes tapping two phases of the upstream bus, and `kv=` entered as the line-to-line value (the two-hot-terminal convention in the kV table above).
- **Grid-forming BESS:** use `Storage` (charge/discharge, grid-forming), not `Generator`. Its stiff control impedance is for load flow only; substitute a current-limited impedance for fault studies (see opendss-fault-study).
- **Zig-zag grounding bank:** model per EPRI's simplified equivalent as a 3-phase Wye-Delta 2-winding Transformer on its own bus (grounding only, no external load), rather than three single-phase units with node tricks.
- **PV kV must match the transformer secondary it ties to** (e.g., 0.208 vs 0.48). A wrong `kv` here is a silent voltage error that only shows up as an off-by-sqrt(3) pu at solve.
- **Preserve history:** header-comment superseded elements RETIRED or SUPERSEDED and keep them in the file; never delete.
- **Keep the master Solve-free:** the model file defines the circuit; run studies from separate drivers that Redirect it (see opendss-simulation-running).

## Validation Before Handing Off
1. Compile the file; fix parser errors top down (first error often cascades).
2. `Solve`, then check convergence and iteration count (`Export Summary` / `Summary`).
3. `Export Voltages` — every bus should have a plausible pu (0.9 to 1.1). A pu near 0.577 or 1.732 means a kv/connection mismatch; pu near 0 means an isolated island.
4. Sum load kW vs source injection (`Export Powers`) for an energy balance sanity check.

## Common Mistakes
| Mistake | Symptom | Fix |
|---|---|---|
| Missing `CalcVoltageBases` | All pu = 0 or nonsense | Add after topology, before Solve |
| Wrong kv for connection | pu off by sqrt(3) | Apply the kV rules table |
| Referencing a linecode before defining it | "LineCode not found" | Reorder: support objects first |
| Duplicate element names | Silent overwrite via implicit edit | Unique names; use `Edit` intentionally |
| Zero length or micro impedance lines for busbars | Convergence and conditioning issues | Model as `switch=y` line or a small Reactor |
| Assuming default load model | Unexpected voltage sensitivity | `model=1` constant PQ is default; set explicitly |
| PV/inverter kv not matching its transformer secondary | pu off by sqrt(3), silent voltage error | Set PVSystem/Storage kv to the actual secondary (0.208 vs 0.48) |
| Modeling a grid-forming BESS as a Generator | Wrong charge/discharge and fault behavior | Use `Storage`; substitute a current-limited impedance for fault duty |
| Inventing undrawn topology to "complete" a network | Non-auditable model | Model only what the drawing shows; document the rest in comments |
