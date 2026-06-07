# [Simple-Multi-Hotend]

[Simple-Multi-Hotend]This is an FDM 3D printing multi-material/multi-color solution. It achieves rapid multi-material printing by switching preheated hot ends loaded with different consumables during the printing process. Its goal is to achieve low-cost, simple, and stable multi-material printing. To achieve high compatibility, hot end switching only requires XY axis movement of the print head, and the switching structure is independent of the hot end itself. Compatibility with other hot ends can be achieved by modifying the hot end fixing components, etc.
<img width="3960" height="3060" alt="打印头" src="https://github.com/user-attachments/assets/66c70b3b-5862-4e04-8ac4-0b782e6da9f5" />

![QQ20260404-193802](https://github.com/user-attachments/assets/bb0c0aa7-5b82-490f-add7-e86a71225352)

---

## Core features
- **Automatic preheating**：Turn on orcaslicer. The Ooze prevention function inserts a preheating command before material switching based on the preheating time you set. When switching, it releases the old hot end and switches to the standby temperature, then grabs the preheated hot end. If the next switching time is shorter than the heating time, it keeps the temperature from dropping.
- **Underactuated switching mechanism**：The underactuated design eliminates the need for additional active components, utilizing the printhead's own XY axis movement and the docking station's mechanical interaction to achieve hot-end switching and filament clamping/unclamping
- **Magnetic Maxwell positioning coupling mechanism**：The hot-end module is locked using an N52 magnet with [Maxwell coupling].(https://en.wikipedia.org/wiki/Kinematic_coupling)The positioning mechanism features three slots formed by positioning pins at the printhead end and a ball head formed by round pins at the hot-end module end. Magnetic engagement allows for six-degree-of-freedom spatial positioning of the hot-end module, eliminating over-positioning. Therefore, even with machining and assembly errors, as well as wear and deformation, the hot-end module can adaptively eliminate gaps and achieve precise positioning.
- **Automatic nozzle offset calibration**:The offset of each head's XYZ axes relative to T0 is calibrated using a nozzle touch calibrator, with reference to [FoxChanger](https://github.com/Noisyfox/FoxChanger/tree/main/NozzleProbe)，The nozzles need to be thoroughly cleaned to obtain the correct nozzle offset. Calibration is not required before every print; it is typically necessary immediately after assembly or in cases of nozzle collision.

## Hardware and software requirements

### hardware
* **printer** In principle, models that support XY movement of the print head（Like voron2.4、voron trident），This open-source code only provides the printhead and basic dock design. You will need to customize the aluminum profile for the fixed dock and other compatible modifications based on your specific printer model.。
* **motherboard** Because it uses wired heating, each hot-end module has a heating element and a thermistor. A small number of hot-end modules can be added using spare interfaces on the motherboard. If more hot-end modules are needed, an expansion board must be made.( https://oshwhub.com/johnnybyzhang/f103-based-klipper-ext-can-usb) to add heating element connectors and thermal connectors to the motherboard.。

### software
* **firmware** Overall hot-end switching is achieved via klipper macro commands. If nozzle offset calibration is required, it needs to be downloaded.[toolchange](https://github.com/viesturz/klipper-toolchanger) The plugin works in conjunction with an automatic calibration macro.
* **Slicing software**：Use Orcaslicer，You can explore other slicing software on your own.

## Composition overview

### components
- Printhead: It has parts other than the hot end.
- Hot-end module: Includes a hot end, connected to a Teflon tube, heating element, and thermistor wire.
- Dock: Used to store docked hot ends. It can plug the openings of the docked hot ends to prevent material leakage, and has cooling fans to dissipate heat from the docked hot ends (for unsealed docks, a side centrifugal fan is used to dissipate heat from all hot ends; for sealed docks, an axial fan is used for each dock).

### Component
#### Print version
- 1.5×5 (diameter*length) cylindrical pins: used to form slots (×6) for Maxwell coupling positioning.
- 3×6 (diameter * length) round-head pins: used to form the balls for Maxwell coupling positioning (3 for one hot end).
- 5×20 (diameter * length) internal thread round head pin: Mounted on the dock to support the extruder swing arm (one pin per dock）
- 6×4 (diameter*thickness) N52 magnets: used to attach the hot-end module to the printhead and docking station (3 magnets per hot-end, 1 magnet per docking station, 3 magnets per printhead）
- M3×6 countersunk screws
- M3×25 screws
- M3×8 screws
- M3×10 screws
- M3×12 screws
- M3×14 screws
- M3×20 screws
- M2.5×16 screws
- M2×8 screws
- M3×4×4 hot melt nuts
- HGX extruder gear set
- 4020 worm gear fan
- 2510 axial fan
- Omron micro switch
- 10×2 (width×thickness) self-adhesive heat-resistant silicone strips
#### CNC Version
See BOM table

## Installation and Configuration Guide

### printer.cfg
   You can reuse your previous Printer.cfg file, referring to the format in my Printer.cfg file and the tips below to modify it, or you can directly use mine and then change it to your own motherboard's pinout.
   1. Reference toolchange.cfg and calibration.cfg.
   2.The hot end is added in accordance with the extruder1 format definition. The heating rod thermistor pins are wired according to their own motherboard, sharing the E0 extruder, while the others are virtual extruders.
   3. The probe uses a fixed micro-motion mechanism. At the start of printing, the hot end is completely held in the dock and the nozzle is plugged with a high-temperature resistant silicone pad. It's only removed after heating to the target temperature to address initial ink leakage. No nozzle wiping is required. During leveling, the hot end is not engaged, and the micro-motion is at the lowest point of the printhead. Other leveling methods are not yet supported; please refer to my probe documentation directly.相关配置包括网床和调平的配置（主要是增加了调平前释放抓取的热端）。
   4. Add a docking station cooling fan configuration, and add all extruders to the extruders associated with the docking station cooling fan and the throat cooling fan.
   5. Modify the print start gcode to automatically extrude and scribble lines onto the hot end to be used before printing.
   6. Modify the print end gcode and add the UNTOOL command to ensure that the hot end is automatically unloaded after printing, so that you don't have to manually remove the hot end for the next print. Add a command to turn off all hot end heating.
### toolchange.cfg
   1. Add toolchange.cfg. After successful addition, the dashboard interface will display T0, T1, T2, and UNTOOL commands. (Do not click on them before adjusting the coordinates, or you will crash.) Clicking T0 will capture T0. After capturing T0, clicking T1 will automatically unload T0 and then capture T1. Clicking UNTOOL will unload the current hot end.
   2. Fixing the dock and adjusting its coordinates: Manually attach the hot end to the print head, drag the print head to the far right of the X-axis, then drag it forward. Fix the first dock at a position where the hot end locking screw can just pass through the large hole of the dock (this position is the coordinate of the first dock). Then remove the hot end, click "Return to Position," and attach the hot end to the print head again. Slowly move the print head to the dock position you just found so that the locking screw passes through the large hole of the dock. Note the coordinates at this time and fill them into the dock position of T0 in the toolchange. One dock occupies 30mm of width. The coordinates of the other docks are obtained using an arithmetic sequence. If there is any error, you can fine-tune the coordinates of each dock yourself. 
   <img width="1351" height="935" alt="image" src="https://github.com/user-attachments/assets/122c4eaf-4a12-4bd4-9988-8d8c0eaeec44" />
   
   3. After correctly setting the coordinates, annotate the normal speed, unannotate the test speed, and switch to slow speed for testing (to make it easier to see if there is any misalignment and immediately click emergency stop).
   4. After the test speed is switched to normal, comment out the test speed and uncomment the normal speed (you can test a more suitable speed according to your own machine).
### calibration.cfg
   1. Download the [toolchange](https://github.com/viesturz/klipper-toolchanger) plugin.
   1. Add calibration.cfg
   2. After aligning xyz and leveling, fix the calibrator in place, click T0 to grab the first hot end, and move it until the nozzle tip is about 1mm above the center of the calibrator. Record the coordinates at this point and fill them into the calibrator coordinates in calibration.cfg. You also need to define a safe position outside the docking station range to prevent collisions with other hot ends in the docking station during calibration.
   3. After setting the coordinates, enter CALIBRATE_TOOL TOOL=_ to calibrate a specific hot end (T0 needs to be calibrated first), and enter CALIBRATE_ALL_TOOLS to calibrate all (you need to specify which hot ends are included). The offset will be automatically saved after calibration.
### orcaslicer
   1. After setting the coordinates, enter CALIBRATE_TOOL TOOL=_ to calibrate a specific hot end (T0 needs to be calibrated first), and enter CALIBRATE_ALL_TOOLS to calibrate all (you need to specify which hot ends are included). The offset will be automatically saved after calibration.
- Print the starting Gcode (modify according to your hot end number):
```Gcode
   ; OrcaSlicer retrieves the heated bed temperature, initial nozzles, a list of nozzles used, and their corresponding temperatures, and sends this information to the printer.
PRINT_START BED=[bed_temperature_initial_layer_single] INITIAL_TOOL=[initial_tool] TOOLS="{if is_extruder_used[0]}0,{endif}{if is_extruder_used[1]}1,{endif}{if is_extruder_used[2]}2,{endif}{if is_extruder_used[3]}3,{endif}{if is_extruder_used[4]}4,{endif}" TEMPS="{if is_extruder_used[0]}{nozzle_temperature_initial_layer[0]},{endif}{if is_extruder_used[1]}{nozzle_temperature_initial_layer[1]},{endif}{if is_extruder_used[2]}{nozzle_temperature_initial_layer[2]},{endif}{if is_extruder_used[3]}{nozzle_temperature_initial_layer[3]},{endif}{if is_extruder_used[4]}{nozzle_temperature_initial_layer[4]},{endif}"
```
-Gcode for consumable replacement:
```Gcode
M104 S{nozzle_temperature[next_extruder]} T{next_extruder}
T{next_extruder}
```
   2. In the process panel, enable the wipe tower (if the material and parameters are well adjusted, and the model does not involve rapid micro-extrusion, you can drag the model to the dock and turn off the wipe tower to try printing without the wipe tower) and Ooze automatic preheating (I set the softening temperature to -180 and the preheating time to 35 seconds, adjust according to your own hot end heating speed).


## Things to note
- The printout is made using ABS, with 4 layers of walls and 60% infill.
- The extruder spring should not be tightened too much. If it is too tight, the swing arm will deform when it rotates, causing the extrusion wheel to not open. The specific situation depends on the actual rigidity of the swing arm that you print out.
- To reduce heat transfer efficiency, the two screws securing the heating block to the TZ2.0 hot end need to be removed, and it is only secured by the side set screws.
- Magnet installation must strictly follow the diagram (some magnets should be pressed all the way down, others should be coplanar with a certain surface). Correct assembly should ensure that only the round pin at the hot-end component end contacts the cylindrical pin groove at the back plate end, with a slight gap between the magnets. This allows for the automatic gap-eliminating function of Maxwell coupling positioning. You can check for any play by gently shaking the hot end; if there is play, check the installation for correctness. If printing accuracy deteriorates, check the positioning assembly. Magnets may slip out and contact the magnet at the other end, causing Maxwell coupling failure.
