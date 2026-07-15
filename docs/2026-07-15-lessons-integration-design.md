# Design: integrate study-hardened lessons into dss-powers and the skill suite

Date: 2026-07-15

## Purpose
Fold the durable, generalizable lessons learned from completed OpenDSS studies (islanded 480 V grid-forming BESS microgrid; load flow, fault, and harmonics deliverables) into the agent orchestrator and the skill library, so future runs do not re-learn them. No client or project identifiers are included; all lessons are stated as generic OpenDSS practice.

## Changes

### agents/dss-powers.md
- New "Operating doctrine" section: segment-by-segment build with review pauses; model only what is drawn; document out-of-scope devices (arresters, NGR, bonding jumpers) rather than fabricating them; keep the master Solve/Export free and run studies from drivers; preserve superseded elements; tag and carry assumptions; guard export filenames; run sensitivity sweeps reversibly.
- Step 3 gates extended: pre-run hygiene gate (current/complete master, define-before-reference order, Linux case sensitivity) and the faultstudy-mode segfault fallback.
- Step 4 report: require separating definitive results from placeholder-conditioned estimates in the same table.

### skills/opendss-simulation-running
- Driver-script pattern (master stays Solve free); field engine limitations (faultstudy segfault on dss-python 0.15.7 / DSS C-API 0.14.5; Linux case sensitivity); Common Mistakes rows for export-name collision, Solve-in-master, stale master.

### skills/opendss-fault-study
- Fallback when faultstudy mode crashes (explicit Fault element + snapshot); reading the fault current from the correct terminal and verifying the far-side branch carries ~0 A; Common Mistakes rows for placeholder-source duties and wrong-branch reads.

### skills/opendss-harmonics-analysis
- Every converter/nonlinear element needs a Spectrum; define spectra before referencing elements; inverter Norton-equivalent amplification caution; idling-Storage THD artifact; TDD needs Isc/IL from a fault study; matching Common Mistakes rows.

### skills/opendss-model-building
- Field-tested conventions: switches/breakers/ATS/fuses as near-zero Switch=true lines; bonding jumpers documented; NEC ampacity/impedance basis and parallel-conductor division; single-phase line-to-line convention; grid-forming BESS as Storage; zig-zag as a 2-winding equivalent; PV kV must match the transformer secondary; RETIRED/SUPERSEDED convention; master stays Solve free. Matching Common Mistakes rows.

### AGENTS.md
- Two universal hard rules added: driver-script pattern with current-master check, and export-name collision discipline, plus placeholder-result labeling.

## Out of scope
Dynamics, timeseries-qsts, efficiency, env-setup, and visualization skills are unchanged; no new lessons for them arose from the source studies.
