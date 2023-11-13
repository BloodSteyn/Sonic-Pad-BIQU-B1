# Sonic-Pad-BIQU-B1
Customisation of the Creality Printer.cfg for the BIQU B1 running Klipper via the Creality Sonic Pad

This printer.cfg file contains the default configuration as provided by Creality on the Sonic Pad Software page here: Creality Sonic Pad printer.cfg Configuration Tutorial - https://www.creality.com/pages/download-creality-sonic-pad

* All relevant configuration has been set up for my B1 with BLTouch ABL, please see BLTouch section.
* My Printer is connected to USB 3, but you may be using a different port, please see USB Configuration section.
* I am using a Dual Gear Extruder, so my E-Steps (Rotation Distance) may differ from yours, please see Rotation Distance section.
* Input Shaping is preset as per the accelerometer calibration I ran, so you will have to run your own to overwrite it, please see Input Shaping section.
* I have a few custom Macros, please see the MACROS section.
* The config is already set up to use **StealthChop** mode on the Steppers/Drivers otherwise the movements make quite a loud noise. Achieved by setting `stealthchop_threshold: 999999`


## BLTouch
I have my Probe fitted to this model - https://www.printables.com/model/183545
Probe Offsets are: X=40, Y=0

You may need to edit this section in the printer.cfg file to accomodate your own setup.
```
[bltouch]			# enable for BLTouch - fast-mode
sensor_pin: ^P0.10		# Probe
control_pin: P2.0		# SERVOS
x_offset: 40			# modify as needed for bltouch location
y_offset: 0			# modify as needed for bltouch location
#z_offset: 0			# modify as needed for bltouch or run PROBE_CALIBRATE
```

## USB Configuration
During the Firmware Setup steps of the Sonic Pad, you will be asked check the Printer Connection.
I found I could only do this step if my B1 was connected to USB 1, `/dev/serial/by-id/usb_serial_1`
Afterwards, I moved the printer to USB 3, and had to manually change the MCU section to this:
```
[mcu]
# As per Sonic Pad
serial: /dev/serial/by-id/usb_serial_3
restart_method: command
```

## Rotation Distance
I installed the Dual Gear Extruder on my B1, so I had to calibrate my E-Steps (Rotation Distance in Klipper).
You will have to do your E-Step calibration, but I have included the "default" value
The max_extrude_only_distance config limits how long a move your extruder can push in a single gcode move. 
This little bastard can cause you headaches and errors in Klipper if for example you have a large print, and you
have set a Skirt. The extrustion may be more than this lenght, causing the print to bomb out. Set it higher here.
```
[extruder]
max_extrude_only_distance: 1600.0
rotation_distance: 23.51388051
# rotation_distance: 32.682276 Old Value
```

## Input Shaping
Delete or run using Sonic Pad Accelerometer.
```
[input_shaper]
#shaper_freq_x: 38.76
#shaper_freq_y: 57.69
#shaper_type: mzv
```

## MACROS
I have included a few custom macros that help me out with filament loading / unloading as well as a decent START_PRINT
Add the following to your Machine Start G-Code section in OrcaSlicer:
```
M140 S0
M104 S0 
START_PRINT EXTRUDER_TEMP=[nozzle_temperature_initial_layer] BED_TEMP=[bed_temperature_initial_layer_single]
```
This will parse the Nozzle and Bed temperature into the beginning of the START_PRINT Macro
```
[gcode_macro START_PRINT]
gcode:
	{% set BED_TEMP = params.BED_TEMP|default(60)|float %}
	{% set EXTRUDER_TEMP = params.EXTRUDER_TEMP|default(210)|float %}
```
The Unload, Load & Purge Macros are handy for swapping filament, although I haven't figured out how to use them when the 
Sonic Pad has Paused the Print using either the PAUSE gcode, or the touch screen.

If Creality allowes us to execure this macro from the Pad, instead of the Console on Fluidd/Mainsail please make use of the 
"Q Load / Q Unload" macros instead, as the main 3 are set up to move the head to the left side of the bed for the purge and
could interfere with the PAUSE command or crash into your print.

The "Q" Macros do not move the head at all, so it will do the purging wherever the Sonic Pad parks te head during PAUSE.
Non of the Macros will turn off the heaters, so please do it manually, or uncomment the  # M104 S0
```
[gcode_macro FILAMENT_UNLOAD]
gcode:
 # Heat Extruder to Chosen Temp (190 Default), wait for heating to complete
 {% set EXTRUDER_TEMP = params.EXTRUDER_TEMP|default(190)|float %}
 M109 S{EXTRUDER_TEMP}
 G90                   # Use Absolute Position to move head to middle left
 G1 X-4 Y117 Z40 F3000
 M83                   # Put the extruder into relative mode
 G92 E0.0              # Reset the extruder so that it thinks it is at position zero
 G1 E-50 F350          # Move the extruder backward 50mm at a speed of 6mm/s
 G1 E-400 F2700        # Move the extruder backward 400mm at a speed of 45mm/s
 G92 E0.0              # Reset the extruder again
 # M104 S0             # Turn off Extruder
 M82                   # Put the extruder back into absolute mode. 
```
```
[gcode_macro FILAMENT_LOAD]
gcode:
 # Heat Extruder to Chosen Temp (190 Default), wait for heating to complete
 {% set EXTRUDER_TEMP = params.EXTRUDER_TEMP|default(190)|float %}
 M109 S{EXTRUDER_TEMP}
 G90                   # Use Absolute Position to move head to middle left
 G1 X-4 Y117 Z40 F3000
 M83                   # Put the extruder into relative mode
 G92 E0.0              # Reset the extruder so that it thinks it is at position zero
 G1 E400 F2700         # Move the extruder forward 400mm at a speed of 45mm/s 
 G1 E50 F350           # Move the extruder forward 50mm at a speed of 6mm/s
 G92 E0.0              # Reset the extruder again
# M104 S0              # Turn off Extruder
 M82                   # Put the extruder back into absolute mode.
```
```
[gcode_macro FILAMENT_PURGE]
gcode:
 # Heat Extruder to Chosen Temp (190 Default), wait for heating to complete
 {% set EXTRUDER_TEMP = params.EXTRUDER_TEMP|default(190)|float %}
 M109 S{EXTRUDER_TEMP}
 G90                   # Use Absolute Position to move head to middle left
 G1 X-4 Y117 Z40 F3000
 M83                   # Put the extruder into relative mode
 G92 E0.0              # Reset the extruder so that it thinks it is at position zero
 G1 E100 F350          # Move the extruder forward 50mm at a speed of 6mm/s ro Purge Filament
 G1 E-3 F350           # Move the extruder backward 3mm at a speed of 6mm/s to prevent oozing
 G92 E0.0              # Reset the extruder again
# M104 S0              # Turn off Extruder
 M82                   # Put the extruder back into absolute mode.
```
```
[gcode_macro Q_UNLOAD]
gcode:
 M83                   # Put the extruder into relative mode
 G92 E0.0              # Reset the extruder so that it thinks it is at position zero
 G1 E-50 F350          # Move the extruder backward 50mm at a speed of 6mm/s
 G1 E-400 F2700        # Move the extruder backward 400mm at a speed of 45mm/s
 G92 E0.0              # Reset the extruder again
 M82                   # Put the extruder back into absolute mode.
```
```
[gcode_macro Q_LOAD]
gcode:
 M83                   # Put the extruder into relative mode
 G92 E0.0              # Reset the extruder so that it thinks it is at position zero
 G1 E400 F2700         # Move the extruder forward 400mm at a speed of 45mm/s 
 G1 E50 F350           # Move the extruder forward 50mm at a speed of 6mm/s
 G92 E0.0              # Reset the extruder again
 M82                   # Put the extruder back into absolute mode.
```
