# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a KiCad hardware design repository for Thundergeddon, a tank robot project. It contains PCB designs for two main boards:

- **TankHull/** - Main controller board for the tank chassis (motors, power distribution)
- **TankTurret/** - Secondary board for the turret assembly

## Repository Structure

```
KiCad/
├── TankHull/           # Hull PCB project
│   ├── TankHull.kicad_sch    # Main schematic
│   ├── TankHull.kicad_pcb    # PCB layout
│   ├── motors.kicad_sch      # Motor control sub-schematic
│   ├── libs/                 # Project-specific symbols and footprints
│   ├── footprints/           # Additional footprints (CYA0630 inductor)
│   ├── Gerbers/              # Manufacturing output files
│   └── production/           # Fabrication files (BOM, positions, netlist)
├── TankTurret/         # Turret PCB project
│   ├── TankTurret.kicad_sch
│   ├── TankTurret.kicad_pcb
│   └── sym-lib-table         # References shared Thundergeddon_Power library
├── Libraries/          # Shared component libraries
│   ├── Thundergeddon_Power.kicad_sym  # Shared power symbols
│   ├── Everlight/      # IR receiver (IRM-V838M3)
│   ├── Vishay/         # IR sensor (TSSP77038)
│   ├── Jushuo/         # FPC connector (AFC07-S24ECA-00)
│   └── XKB/            # Tactile switch (TS-1187A)
└── footprints.pretty/  # Repository-level footprint library (currently empty)
```

## Library References

- Each project has a `sym-lib-table` defining symbol library paths
- TankHull uses project-local symbols in `libs/TankHull.kicad_sym`
- TankTurret references the shared `Libraries/Thundergeddon_Power.kicad_sym`
- Custom footprints include power regulators (SY8113BADC, TS2596SCS), IR emitter (VSMB10940)

## Manufacturing Outputs

The TankHull project uses a fabrication toolkit with options in `fabrication-toolkit-options.json`:
- Gerbers exported to `Gerbers/` directory
- Production files (BOM, pick-and-place positions) in `production/`
- STEP/STL 3D models exported for mechanical integration
