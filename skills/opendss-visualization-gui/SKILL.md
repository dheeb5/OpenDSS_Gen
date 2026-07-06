---
name: opendss-visualization-gui
description: Use when a single line diagram, voltage profile, or plot of OpenDSS results is needed, when preparing a model for the OpenDSS Desktop GUI or OpenDSS-G, when Plot commands fail headlessly, or when a .dsp project file is requested.
---

# OpenDSS Visualization and GUI Handoff

## Overview
The scripting engine has no display. Visualization paths: (a) bus coordinates + Plot in the OpenDSS Desktop GUI, (b) import into OpenDSS-G for an interactive single line diagram, (c) headless plotting from exports with matplotlib, (d) dss-python's built in plot backend.

## Bus Coordinates (prerequisite for any circuit plot)
Create `<name>_BusCoords.dss` — one `busname, x, y` per line, schematic coordinates are fine:
```
SourceBus, 0, 50
MidBus,   20, 50
LoadBus,  40, 50
SecBus,   50, 50
```
Reference it after the model compiles: `Buscoords BusCoords.dss`. Then, in a GUI session:
```
Plot Circuit Power Max=2000 dots=y labels=y subs=y
Plot Profile Phases=All
```
Plot commands are ignored or fail on the headless engine — keep them at the end of the script so automation still completes the solve and exports.

## Headless Plotting Options
```python
import dss
dss.plot.enable()          # dss-python >= 0.15: Plot commands render via matplotlib
from dss import DSS
DSS.Text.Command = 'compile "feeder.dss"'
DSS.Text.Command = "Solve"
DSS.Text.Command = "Plot Profile Phases=All"   # saves/render figure via matplotlib backend
```
Or simply matplotlib over `_EXP_VOLTAGES.csv` / monitor CSVs — most robust for reports.

## OpenDSS-G (.dsp projects)
- `.dsp` is OpenDSS-G's proprietary project container, created only by the tool itself; it is not documented for hand authoring. Do NOT fabricate one.
- Correct path: OpenDSS-G > File > Import > select the `.dss` file. The importer builds the single line diagram (using Buscoords if present, auto layout otherwise) and saves the `.dsp` alongside.
- On Linux, OpenDSS-G runs under Wine (`opendss-g` wrapper if installed; see opendss-env-setup). First launch takes ~3 minutes.
- Keep the model GUI friendly: one file or a Master.dss with Redirects, relative paths only, no OS specific absolute paths.

## Common Mistakes
| Mistake | Consequence |
|---|---|
| Plot commands in a headless CI run without dss.plot enabled | Errors or silent no ops mistaken for failures |
| Hand writing a .dsp file | OpenDSS-G will not open it |
| GPS coordinates when only a schematic exists | Overlapping unreadable plot; schematic grid coordinates are acceptable |
| Buscoords before the circuit exists | Command needs an active circuit; place after compile |
