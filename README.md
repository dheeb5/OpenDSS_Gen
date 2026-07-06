# OpenDSS_Gen

Agent portable skill suite for [OpenDSS](https://opendss.epri.com/) distribution system simulation. Ten skills covering the full study lifecycle plus a plan-first orchestrator agent (**DSS-powers**). Works with Claude Code, Codex, and any agent that can read markdown (see [AGENTS.md](AGENTS.md)).

Every DSS example embedded in these skills was executed and verified against **dss-python 0.15.7** (DSS C-API 0.14.5, OpenDSS SVN 3723 parity) on Linux.

## Contents

```
skills/
  opendss-env-setup/          Detect/install engine (dss-python) and OpenDSS-G GUI (Wine)
  opendss-model-building/     Author .dss circuits: skeleton order, kV rules, element syntax
  opendss-simulation-running/ Solve modes, headless output capture, convergence debugging
  opendss-model-efficiency/   Right sized models, fast scenario loops via the Python API
  opendss-powerflow-analysis/ Snapshot load flow screening (ANSI C84.1, losses, loading)
  opendss-timeseries-qsts/    Daily/yearly QSTS, loadshapes, monitors, energy meters
  opendss-fault-study/        Fault duties, X/R, grounding, inverter based resource caveats
  opendss-harmonics-analysis/ Spectra, THD post processing, IEEE 519 limits, resonance
  opendss-dynamics-analysis/  Electromechanical dynamics, disturbances, when to use EMT instead
  opendss-visualization-gui/  Bus coordinates, plots, OpenDSS-G import, headless plotting
agents/
  dss-powers.md               Orchestrator: plans first, then applies the skills above
examples/
  feeder.dss + run_*.dss      Validated fixtures for all five study modes (smoke tests)
```

## Install

**Claude Code (user level):**
```bash
git clone https://github.com/dheeb5/OpenDSS_Gen ~/OpenDSS_Gen
ln -s ~/OpenDSS_Gen/skills/opendss-* ~/.claude/skills/
cp ~/OpenDSS_Gen/agents/dss-powers.md ~/.claude/agents/
```
Then ask for the `dss-powers` agent or invoke any skill by name.

**Claude Code (project level):** copy `skills/` into `.claude/skills/` and `agents/dss-powers.md` into `.claude/agents/` of the project.

**Codex / other agents:** point the agent at this repo; `AGENTS.md` is the entry point and routing table. Skills follow the [agentskills.io](https://agentskills.io/specification) frontmatter convention.

## Quick start

```bash
cd examples
opendss run_pf.dss      # snapshot power flow  -> DemoFeeder_EXP_VOLTAGES.csv
opendss run_fault.dss   # fault study          -> DemoFeeder_EXP_FAULTS.csv
opendss run_harm.dss    # harmonics            -> DemoFeeder_Mon_mon_sec_1.csv
opendss run_daily.dss   # 24 h QSTS            -> DemoFeeder_EXP_METERS.csv
opendss run_dyn.dss     # dynamics             -> DemoFeeder_Mon_mon_gen_1.csv
```
No `opendss` command? Start with `skills/opendss-env-setup/SKILL.md`.

## Design notes

- Skill descriptions carry only triggering conditions (when to load), never workflow summaries, per skill authoring best practice.
- DSS-powers is deliberately plan-first: it must name the skills each step will follow before running any command.
- Standards referenced across skills: ANSI C84.1, IEEE 519-2022, IEEE 1547-2018, IEEE 2800, IEEE C37.010/C37.13.

## License

MIT
