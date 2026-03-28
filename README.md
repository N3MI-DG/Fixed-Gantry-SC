
# Fixed Gantry StealthChanger Mod

## Overview

This mod adds StealthChanger capability to fixed gantry printers. It uses a custom dock design that mounts directly to the printer frame and uses a ramping system to mount the toolhead. 

The ramping is broken into two actions: 

1. The tool runs into the dock in the Y direction, lifting it partially and aligning the screws that are on the tool to the dock.
2. The tool then gets shoved in the X direction to fully lift the tool from the shuttle and engage with the dock.

While a little crude, it works really well and is very reliable.

## Printing

Print with Typical Voron Recommended settings

* `Layer Height` - 0.2mm
* `Extrusion Width` - 0.4mm, forced
* `Infill Percentage` - 40%
* `Infill Type` - Grid, Gyroid, honeycomb, Triangle or Cubic
* `Wall Count` - 4
* `Solid Top/Bottom Layers` - 5
* `Layer Height` - 0.2mm
* `Supports` - Not required, models have built in supports.
