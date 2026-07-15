---
name: opendss-simulation-running
description: Use when executing an OpenDSS solve, when a solution does not converge, when Show or Plot commands hang or spawn editor windows in headless automation, when output files cannot be located, or when simulation results must be captured programmatically.
---

# OpenDSS Simulation Running

## Overview
The run cycle is: compile the script, set the solution mode, `Solve`, then export results to files. In headless or agent driven automation, `Export` (CSV, silent) is the correct output channel; `Show` writes a .txt report and then tries to open a GUI editor.

## Run Commands
```bash
opendss study.dss                 # compile and run a script file
opendss -c "Redirect study.dss"   # same, keeping current directory semantics
```
Inside scripts: `Compile <file>` changes the working/data path to the file's directory; `Redirect <file>` returns to the caller's directory afterward. Output files land in the active data path, named `<CircuitName>_EXP_<REPORT>.csv` and `<CircuitName>_Mon_<monitor>_1.csv`.

## Solution Modes
| Mode | Command | Use |
|---|---|---|
| Snapshot | `Set mode=snapshot` + `Solve` | Single power flow (default) |
| Daily / Yearly | `Set mode=daily number=24 stepsize=1h` | QSTS time series (see opendss-timeseries-qsts) |
| Fault study | `Set mode=faultstudy` | All bus fault duties (see opendss-fault-study) |
| Harmonics | `Set mode=harmonics` | After a converged fundamental solve (see opendss-harmonics-analysis) |
| Dynamics | `Set mode=dynamics number=200 stepsize=0.0002` | Electromechanical transients (see opendss-dynamics-analysis) |

## Capturing Results (headless safe)
```
Export Voltages      ! bus voltages, magnitude/angle/pu
Export Powers kva elem
Export Losses
Export Currents
Export Overloads
Export Monitors <name>   ! or: Export Monitors ALL
Export Meters
Export Summary
```
Avoid `Show` and `Plot` in automation: `Show` spawns an editor (harmless GUI errors like "libEGL warning" and nonzero exit codes on headless boxes); `Plot` requires a GUI backend. If a legacy script uses Show, either read the generated `<Circuit>_*.txt` directly or issue `Set editor=/bin/true` first.

## Convergence Debugging Checklist (in order)
1. `Export Summary` — confirm "Solution Converged" and iteration count.
2. All buses energized? `Export Voltages` — pu near 0 means an island; check for missing lines or open switches.
3. Voltage bases correct? Re-check `Set voltagebases` list covers every kV level, then `CalcVoltageBases`.
4. Raise iteration limit: `Set maxiterations=100 maxcontroliter=50`.
5. Switch algorithm: `Set algorithm=Newton` (robust for heavily loaded or ill conditioned cases; default Normal/fixed point is faster).
6. Temporarily soften loads: `Set loadmodel=admittance` (linear) to test whether topology solves at all, then return to `powerflow`.
7. Look for micro impedance branches and zero length lines; replace with switch lines or reactors.
8. Isolate controls: `Set controlmode=off` to test whether regulator/cap control hunting is the cause.

## Driver-script pattern (keep the master Solve-free)
Do not put `Solve`/`Export` in the master model file. Keep one standalone driver per study (snapshot, harmonics, fault, QSTS) that `Redirect`s the master, then solves and exports. This lets the same model serve every study, keeps the model reproducible, and makes each run's intent explicit. Before running any driver, confirm it redirects the current, complete master — projects accumulate stale or parallel masters (for example an old `Master_Part1.dss` alongside a newer `Master.dss`), and redirecting the wrong one silently omits elements.

## Engine limitations seen in the field
- `Set mode=faultstudy` + `Solve` reproducibly SEGFAULTS on some builds (confirmed on dss-python 0.15.7 / DSS C-API 0.14.5); the prior snapshot solve is unaffected. Fall back to the explicit `Fault` element + snapshot workflow (see opendss-fault-study). Verify with `Export Summary` that the fault solve actually completed before trusting any number.
- On case-sensitive Linux filesystems, `Redirect LoadShape.DSS` fails against a file named `LoadShape.dss`; match case exactly.

## Common Mistakes
| Mistake | Consequence |
|---|---|
| Using Show/Plot in CI or agent pipelines | Editor spawn, spurious exit code 2, apparent hang |
| Looking for outputs next to the caller | Files are written to the active data path (Compile changes it) |
| Solving harmonics/dynamics without a prior snapshot Solve | Wrong or non initialized operating point |
| Treating a nonzero exit code as solve failure | Verify via Export Summary; editor spawn also returns nonzero |
| Running a second study without guarding export names | `<Circuit>_EXP_*.csv` overwrites the prior run silently; preserve/rename baselines, restore canonical names after |
| Putting Solve/Export in the master file | Cannot reuse the model across studies; use a driver per study that Redirects the master |
| Redirecting a stale or parallel master | Silently omits split-out elements; confirm the driver points at the current complete master |
