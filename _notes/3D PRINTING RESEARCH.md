---
---

# Useful MARLIN code
* `G92E0` Reset printer
* `M503` Current settings
* `M83` Relative extrusion
* `M92 Exx.xx` Set steps per mm
* `M500` Store current settings
* `G1E100F100` Extrude 100mm, feed 100mm
* `M104 SXXX` Set extruder temp

New Steps per mm = (Current steps per mm) x [100 / (measured distance filament traveled)]
measured 101.3mm.
current steps/mm is 93.
So, 93 x (100 / 101.3) = 91.80

# Calibrating filament
 * Set the nozzle temp
 * Set the steps per mm you want
 * Reset extrusion state
 * Set relative extrusion
 * Extrude 100mm with feed 100mm(%?)
```
M104 S240
M92 E100
G92 E0
M83
G1 E100 F100
```

# Filament specific steps for CR10


| Filament          |Vendor    |Steps per/mm |MARLIN      |
|-------------------|----------|-------------|------------|
|DARK GRAY EASYPRINT|EASYPRINT |95           |M92 E95     |
|LIGHT GRAY 3kg     |EASYPRINT |96.5         |M92 E96.5|
|LIGHT GRAY         |PRIMAVALUE|97           |M92 E97|
|LIGHT GRAY         |PRIMA     |106.21       |M92 E106.21 |
|Silver             |PRIMA     |100          |M92 E100 |
|Clear              |DEVIL     |100          |M92 E100 |
|GOLD               |DEVIL     |99.8         |M92 E99.8 |
|BLACK              |DEVIL     |102          |M92 E102 |
|White              |DEVIL     |114.1        |M92 E114.1 |
|Light Gray         |DEVIL     |119.31       |M92 E119.31 |
