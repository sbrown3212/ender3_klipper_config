# Ender 3 Klipper Configuration

This repository contains the Klipper configuration for my Ender 3 3D printer.

## Hardware

- Printer: Creality Ender 3
- Mainboard: BigTreeTech SKR Mini E3 v3.0
- MCU: STM32G0B1 (8KiB bootloader, USB communication)
- Stepper Drivers: TMC2209 (all axes)
- Hotend Fan: Noctua (quieter than stock)
- Bed Leveling: Manual (Klackender probe planned for future)

## Modifications

- BTT SKR Mini E3 v3.0 mainboard
- Upgraded part cooling fan duct
- Noctua hotend cooling fan

## Planned Upgrades

 - Klackender probe for automatic bed leveling
 - Input shaper calibration (requires ADXL345 accelerometer)
 - Pressure advance tuning

## Configuration Structure

```
The configuration is split into logical modules:
├── printer.cfg              # Main config (includes only)
├── hardware/                # Hardware-specific configs
│   ├── mcu.cfg              # MCU and board pins
│   ├── steppers.cfg         # Stepper motors
│   ├── tmc_drivers.cfg      # TMC2209 drivers
│   ├── extruder.cfg         # Extruder/hotend
│   ├── bed.cfg              # Heated bed
│   ├── fans.cfg             # All fans
│   └── probe.cfg            # Future probe config
├── macros/                  # G-code macros
│   ├── start_print.cfg      # Print start routine
│   ├── end_print.cfg        # Print end routine
│   ├── pause_resume.cfg     # Pause/resume/cancel
│   └── utilities.cfg        # Utility macros
└── tuning/                  # Tuning parameters
    ├── pressure_advance.cfg
    └── input_shaper.cfg
```

## Initial Setup Checklist

After flashing Klipper firmware to the SKR Mini E3 v3.0:

- [ ] Update MCU serial ID in hardware/mcu.cfg
- [ ] Run PID tuning for extruder: PID_EXTRUDER TEMP=210
- [ ] Run PID tuning for bed: PID_BED TEMP=60
- [ ] Calibrate E-steps (extruder rotation_distance)
- [ ] Configure slicer start/end g-code to use macros
- [ ] Tune pressure advance (see tuning/pressure_advance.cfg)

## Slicer Configuration

### OrcaSlicer

Start G-code:

```gcode
START_PRINT EXTRUDER_TEMP=[nozzle_temperature_initial_layer] BED_TEMP=[first_layer_bed_temperature]
```

End G-code:

```gcode
END_PRINT
```

### PrusaSlicer / SuperSlicer

Start G-code:

```gcode
START_PRINT EXTRUDER_TEMP=[first_layer_temperature] BED_TEMP=[first_layer_bed_temperature]
```

End G-code:

```gcode
END_PRINT
```

### Cura

Start G-code:

```gcode
START_PRINT EXTRUDER_TEMP={material_print_temperature_layer_0} BED_TEMP={material_bed_temperature_layer_0}
```

End G-code:

```gcode
END_PRINT
```

## Useful Commands

- PID_EXTRUDER TEMP=210 - Tune hotend PID
- PID_BED TEMP=60 - Tune bed PID
- LOAD_FILAMENT - Load filament helper
- UNLOAD_FILAMENT - Unload filament helper
- M600 - Filament change (pause and unload)

## Resources

- Klipper Documentation
- Klipper Config Reference
- BTT SKR Mini E3 v3.0 Pinout

## Notes

- Remember to run SAVE_CONFIG after PID tuning
- Klipper appends config to the bottom of printer.cfg - that's normal
- Make sure to update firmware when Klipper updates