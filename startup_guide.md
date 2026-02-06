# Setup Guide for Modular Klipper Config

This guide walks you through setting up this modular Klipper configuration on your Raspberry Pi 3 with Mainsail.

## Step 1: Install Base System

If you haven't already:

1. Flash MainsailOS to your SD card
2. Boot your Raspberry Pi
3. Connect to Mainsail web interface (usually http://mainsailos.local)

## Step 2: Flash Klipper Firmware to SKR Mini E3 v3.0

1. SSH into your Raspberry Pi:
   ```bash
   ssh pi@mainsailos.local
   ```

2. Navigate to Klipper directory and configure firmware:
   ```bash
   cd ~/klipper
   make menuconfig
   ```

3. Configure with these settings:
   - Micro-controller Architecture: `STMicroelectronics STM32`
   - Processor model: `STM32G0B1`
   - Bootloader offset: `8KiB bootloader`
   - Communication interface: `USB (on PA11/PA12)`
   - Press `Q` to quit and `Y` to save

4. Compile firmware:
   ```bash
   make clean
   make
   ```

5. Copy firmware to SD card:
   - The compiled firmware is at `~/klipper/out/klipper.bin`
   - Copy it to a microSD card and rename it to `firmware.bin`
   - Insert SD card into SKR Mini E3 v3.0
   - Power cycle the board (it will flash automatically)
   - Remove SD card after ~10 seconds

6. Verify the board is connected:
   ```bash
   ls /dev/serial/by-id/*
   ```
   You should see something like: `usb-Klipper_stm32g0b1xx_XXXXX-if00`

## Step 3: Deploy Your Modular Config

### Option A: Using Git (Recommended - Method #1)

1. Navigate to config directory:
   ```bash
   cd ~/printer_data/config
   ```

2. Initialize git repository:
   ```bash
   git init
   git remote add origin https://github.com/yourusername/your-repo.git
   ```

3. Upload your config files to this directory (via Mainsail web interface or SCP)

4. Update the MCU serial in `hardware/mcu.cfg`:
   - Get your actual serial ID: `ls /dev/serial/by-id/*`
   - Edit `hardware/mcu.cfg` and replace the placeholder

5. Commit to git:
   ```bash
   git add .
   git commit -m "Initial Klipper configuration"
   git push -u origin main
   ```

### Option B: Manual Upload

1. Using Mainsail web interface:
   - Go to "Machine" tab
   - Navigate to "config" folder
   - Upload all your config files maintaining the directory structure:
     - `printer.cfg` at root
     - Create `hardware/`, `macros/`, `tuning/` folders
     - Upload respective files to each folder

2. Update MCU serial in `hardware/mcu.cfg` using Mainsail's built-in editor

## Step 4: Update MCU Serial ID

**CRITICAL**: You must update the MCU serial ID or Klipper won't start!

1. Find your serial ID:
   ```bash
   ls /dev/serial/by-id/*
   ```

2. Edit `hardware/mcu.cfg` in Mainsail
   - Replace the placeholder serial with your actual ID
   - Save the file

## Step 5: Verify Configuration

1. In Mainsail, click "Restart Firmware" (top right)
2. Check for errors in the console
3. If you see errors, check:
   - MCU serial is correct
   - All `[include ...]` files exist
   - No typos in config files

## Step 6: Initial Calibration

Once Klipper connects successfully:

### 6.1 Home the Printer
```gcode
G28
```
If this works, your steppers and endstops are configured correctly!

### 6.2 PID Tune the Hotend
```gcode
PID_EXTRUDER TEMP=210
```
Wait for it to complete, then save:
```gcode
SAVE_CONFIG
```

### 6.3 PID Tune the Bed
```gcode
PID_BED TEMP=60
```
Wait for it to complete, then save:
```gcode
SAVE_CONFIG
```

### 6.4 Calibrate Z-Offset
1. Home all axes: `G28`
2. Disable steppers: `M84`
3. Move Z to 0: `G1 Z0`
4. Manually adjust Z using paper test
5. Get current Z position: `GET_POSITION`
6. Update `position_endstop` in `hardware/steppers.cfg`
7. Save and restart

### 6.5 Calibrate E-steps
1. Mark filament 120mm from extruder entry
2. Heat hotend to printing temp
3. Extrude 100mm: `G91` then `G1 E100 F100`
4. Measure remaining distance to mark
5. If not 20mm, calculate:
   ```
   new_rotation_distance = old_rotation_distance * (100 / actual_extruded)
   ```
6. Update `rotation_distance` in `hardware/extruder.cfg`
7. Save and restart

## Step 7: Configure Your Slicer

Add these to your slicer (see README.md for details):

**Start G-code:**
```gcode
START_PRINT EXTRUDER_TEMP=[first_layer_temperature] BED_TEMP=[first_layer_bed_temperature]
```

**End G-code:**
```gcode
END_PRINT
```

## Step 8: First Test Print

1. Slice a simple test print (calibration cube, benchy)
2. Upload to Mainsail
3. Start the print
4. Watch the first layer carefully
5. Adjust Z-offset if needed using baby stepping

## Optional: Future Enhancements

When you're ready:

- **Pressure Advance**: Follow guide in `tuning/pressure_advance.cfg`
- **Input Shaper**: Requires ADXL345 accelerometer
- **Klackender Probe**: Uncomment `hardware/probe.cfg` when installed

## Troubleshooting

**"MCU unable to connect"**
- Check MCU serial ID in `hardware/mcu.cfg`
- Verify USB cable is connected
- Try different USB port on Pi

**"Unknown command"**
- Check for typos in macros
- Verify all include files exist

**"Option 'position_endstop' is not valid"**
- Some options can't be in included files when already defined
- Move conflicting options to main `printer.cfg`

**Noctua fan too quiet?**
- Adjust `max_power` in `hardware/fans.cfg`
- Some Noctua fans need higher minimum PWM

## Getting Help

- [Klipper Discourse](https://www.klipper3d.org/Contact.html)
- [Klipper Discord](https://discord.klipper3d.org)
- [r/klippers subreddit](https://reddit.com/r/klippers)