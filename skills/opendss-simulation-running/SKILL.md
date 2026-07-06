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

## Common Mistakes
| Mistake | Consequence |
|---|---|
| Using Show/Plot in CI or agent pipelines | Editor spawn, spurious exit code 2, apparent hang |
| Looking for outputs next to the caller | Files are written to the active data path (Compile changes it) |
| Solving harmonics/dynamics without a prior snapshot Solve | Wrong or non initialized operating point |
| Treating a nonzero exit code as solve failure | Verify via Export Summary; editor spawn also returns nonzero |
