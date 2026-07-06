---
name: opendss-env-setup
description: Use when OpenDSS availability on the machine is unknown, when "import dss" raises ModuleNotFoundError, when the opendss command is not found, when OpenDSS must be installed on Linux or macOS, or when the OpenDSS-G graphical interface is needed.
---

# OpenDSS Environment Setup

## Overview
Detect an existing OpenDSS installation before installing anything. The EPRI engine ships three ways: the Windows COM build (OpenDSS.exe), the cross platform DSS C-API engine via the `dss-python` package, and the OpenDSS-G graphical front end (LabVIEW based, Windows only, runs under Wine on Linux).

## Detection (run in this order, stop at first hit)
```bash
command -v opendss && opendss --version        # CLI wrapper (common on preconfigured machines)
ls ~/.local/share/opendss-venv/bin/python 2>/dev/null   # dedicated venv convention
python3 -c "import dss; print(dss.__version__)"          # dss-python in active environment
python3 -c "import opendssdirect; print(opendssdirect.__version__)"
command -v opendss-g || ls ~/.wine/drive_c/ProgramData/OpenDSS-G/OpenDSSG.exe 2>/dev/null  # GUI
```
The system Python usually does NOT have `dss` — check dedicated venvs before concluding OpenDSS is absent.

## Install: engine (Linux/macOS, no admin rights needed)
```bash
python3 -m venv ~/.local/share/opendss-venv
~/.local/share/opendss-venv/bin/pip install dss-python opendssdirect.py
```
Then create a CLI wrapper at `~/.local/bin/opendss` (chmod +x). Shebang line must point at the venv python:
```python
#!/home/USER/.local/share/opendss-venv/bin/python
import sys, os
from dss import DSS
def run(cmd):
    DSS.Text.Command = cmd
    if DSS.Text.Result: print(DSS.Text.Result)
    if DSS.Error.Description: print("Error:", DSS.Error.Description, file=sys.stderr)
a = sys.argv[1:]
if a and a[0] == "-c": run(" ".join(a[1:]))
elif a: run(f'compile "{os.path.abspath(a[0])}"')
else:
    while True:
        try: line = input("DSS> ").strip()
        except (EOFError, KeyboardInterrupt): break
        if line.lower() in ("exit", "quit"): break
        if line: run(line)
```
Usage: `opendss file.dss`, `opendss -c "<command>"`, interactive `DSS>` prompt.

## Install: OpenDSS-G GUI on Linux (only if a GUI is explicitly required)
1. `sudo dpkg --add-architecture i386 && sudo apt install wine wine32:i386 winetricks cabextract`
2. If a 64 bit only Wine prefix exists, delete `~/.wine` and recreate (32 bit apps fail otherwise).
3. `winetricks -q dotnet48` (OpenDSS-G refuses to start without real .NET Framework).
4. Run the OpenDSS-G installer from sourceforge.net/projects/dssimpc/ under Wine.
5. First launch takes about 3 minutes (LabVIEW runtime initialization) — this is normal, not a hang.

## Verify
```bash
opendss -c "New Circuit.T basekv=12.47 bus1=B1 ! then Solve"   # or run examples/run_pf.dss
opendss examples/run_pf.dss && cat examples/DemoFeeder_EXP_VOLTAGES.csv
```
A converged snapshot with pu values near 1.0 confirms a working engine.

## Common Mistakes
| Mistake | Fix |
|---|---|
| `pip install opendss` | Package is `dss-python` (engine) or `opendssdirect.py` (API layer) |
| Concluding absent after `import dss` fails in system Python | Check venvs and `command -v opendss` first |
| Installing the Windows OpenDSS.exe on Linux for scripting | Use dss-python; reserve Wine for OpenDSS-G GUI only |
| Killing OpenDSS-G during first launch | LabVIEW init takes ~3 min; wait |
