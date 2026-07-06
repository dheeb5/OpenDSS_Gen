# OpenDSS_Gen — Agent Entry Point

Instructions for ANY coding agent (Claude Code, Codex, Gemini CLI, Cursor, etc.) working on OpenDSS tasks with this library.

## What this repo is
A skill library for EPRI OpenDSS distribution system simulation: environment setup, model authoring, simulation execution, efficiency, and one skill per analysis type. Every embedded DSS example has been executed and verified against dss-python 0.15.7 (DSS C-API 0.14.5).

## How to use it
1. **Match your task to a skill** using the table below.
2. **Read the full SKILL.md** before acting — the Common Mistakes tables are checklists, not trivia.
3. For multi step objectives, adopt the plan-first workflow in `agents/dss-powers.md`: write a plan naming the skills per step, then execute.

| Task | Skill file |
|---|---|
| Detect or install OpenDSS / OpenDSS-G | `skills/opendss-env-setup/SKILL.md` |
| Create or edit a circuit model | `skills/opendss-model-building/SKILL.md` |
| Solve, capture outputs, fix convergence | `skills/opendss-simulation-running/SKILL.md` |
| Speed up models or scenario loops | `skills/opendss-model-efficiency/SKILL.md` |
| Load flow / voltage / losses analysis | `skills/opendss-powerflow-analysis/SKILL.md` |
| Daily/yearly time series (QSTS) | `skills/opendss-timeseries-qsts/SKILL.md` |
| Short circuit / fault duties | `skills/opendss-fault-study/SKILL.md` |
| Harmonics / THD / IEEE 519 | `skills/opendss-harmonics-analysis/SKILL.md` |
| Dynamics / transients / GFM | `skills/opendss-dynamics-analysis/SKILL.md` |
| Plots, SLD, OpenDSS-G, .dsp | `skills/opendss-visualization-gui/SKILL.md` |

## Hard rules (all agents)
- Prefer `Export` over `Show`/`Plot` in any headless run; `Show` spawns an editor and returns nonzero exit codes.
- Never report a result from a solve whose convergence you did not verify.
- Never hand author an OpenDSS-G `.dsp` file; use the tool's Import feature.
- Validate every DSS snippet by running it (`opendss <file>`) before delivering it.
- State assumptions for any missing engineering data (source strength, %Z, spectra, profiles).

## Smoke test
```bash
cd examples && opendss run_pf.dss && cat DemoFeeder_EXP_VOLTAGES.csv
```
Expect four buses with pu between 0.95 and 1.0.
