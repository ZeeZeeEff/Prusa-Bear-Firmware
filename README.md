# Prusa Bear Firmware 3.14.1

Three MK3S/Einsy 1.0a build variants based on Prusa firmware 3.14.1.

## Status

All three variants compiled successfully in English and multilingual configurations on 14 November 2026. Physical printer validation is pending.
The local environment cannot resolve github.com. Sources were retrieved and published through the GitHub connector. The included Actions workflow provides the build environment.
Check the workflow result before using any generated firmware.

## Firmware ZIP downloads

Each hardware ZIP contains English and multilingual HEX firmware, the README, license and a build manifest identifying the source commit.

- [Original Prusa extruder](firmware-zips/Prusa-Bear-3.14.1-STOCK.zip)
- [BearExxa V1](firmware-zips/Prusa-Bear-3.14.1-EXXA_V1.zip)
- [BearExxa V2 beta.6 hardware](firmware-zips/Prusa-Bear-3.14.1-EXXA_V2.zip)
- [Full firmware source](firmware-zips/Prusa-Bear-3.14.1-source.zip)
- [SHA-256 checksums](firmware-zips/SHA256SUMS)

All six builds and ZIP publication passed in [build 43599315541](https://github.com/ZeeZeeEff/Prusa-Bear-Firmware/actions/runs/43599315541). Physical hardware testing is pending. These ZIPs are committed to this branch and are independent of Actions artifact retention. The source archive excludes previously generated ZIPs.

## Compatibility

| CMake variant | Hardware |
| --- | --- |
| MK3S_BEAR_STOCK | Standard-height Bear frame with original MK3S extruder, X axis and E3D V6 |
| MK3S_BEAR_EXXA_V1 | Standard-height Bear frame with BearExxa V1 and E3D V6 |
| MK3S_BEAR_EXXA_V2 | BearExxa V2 hardware 2.0.0-beta.6, Revo Micro and its specified geared motor |

These configurations assume the standard MK3S electronics, bed, motors and sensors except for the listed extruder changes. Extended Z frames and other extruders require separate configuration. V2 remains experimental and specific to the beta.6 hardware.

## Source provenance

- Prusa base commit `74a577bc0e5902767b072ee85f499dc1361bf6ba`, release `v3.14.1`.
- V1 calibration reference commit `8a174c9f6119e84403720078023c2dfb849f33cf` in soapee01/Prusa-Bear-Firmware.
- V2 variant copied from Bear Lab release `3.14.1-BEX201B1`, file `Firmware/variants/MK3S.h`, Git blob `44afa9951b7ec3a8a80987e3cca645ef4d8c75aa` (see source link for authoritative content).

[Prusa source](https://github.com/prusa3d/Prusa-Firmware/tree/74a577bc0e5902767b072ee85f499dc1361bf6ba)
[V1 calibration patch](https://github.com/soapee01/Prusa-Bear-Firmware/commit/8a174c9f6119e84403720078023c2dfb849f33cf)
[V2 source](https://github.com/bear-lab-3d/BearExxaV2-Prusa-Firmware/tree/3.14.1-BEX201B1)

## Implementation process

1. Inspect the fork and resolve the official release commit. The fork's MK3 branch includes later development commits, so create a separate branch at the exact release commit.
2. Compare the V1 community patch against Prusa 3.14.0 and port its two geometry changes to the 3.14.1 implementation.
3. Compare the V2 release variant against Prusa 3.14.1 and copy that hardware configuration into its own named variant.
4. Scope runtime V1/V2 changes using variant macros.
5. Build each variant through Prusa's existing CMake targets. Produce English and multilingual binaries separately.
6. Inspect build reports and validate on matching physical hardware.

This preparation used the GitHub connector to retrieve sources. It did not complete a local Git clone. The upstream license and history must accompany the resulting branch.

## Modifications

### Original extruder

`MK3S_BEAR_STOCK.h` includes upstream `MK3S.h`. This provides an explicitly named output with the original hardware settings.

### BearExxa V1

`MK3S_BEAR_EXXA_V1.h` includes upstream `MK3S.h`, sets the display name to `Bear MK3S BEX1`, and changes `Z_MAX_POS_XYZ_CALIBRATION_CORRECTION` from 9 to 2 mm.

Prusa 3.14.1 uses that configuration constant in automatic Z calibration. The older community patch hard-coded the equivalent value in Marlin_main.cpp.

The manual end-stop calibration in `ultralcd.cpp` uses `Z_MAX_POS - 3` for V1, replacing the standard MK3S assumption of `Z_MAX_POS + 4`. Z travel remains 210 mm. Other settings come from 3.14.1, including its 80 °C ambient limit.

### BearExxa V2

The variant imports the Bear Lab beta configuration with these substantive differences from stock MK3S.

| Setting | Stock | V2 |
| --- | --- | --- |
| Display name | Prusa i3 MK3S | Bear MK3S BEX2 |
| Hotend label | E3Dv6full | E3DRevoMicro |
| E steps/mm | 280 | 415 |
| E microsteps | 32 | 16 |
| Z maximum | 210 mm | 207 mm |
| Z calibration correction | 9 mm | 3 mm |
| Normal X/Y maximum feedrate | 200 mm/s | 300 mm/s |
| E holding/running current register | 30 | 21 |
| E homing current register | 18 | 15 |
| Hotend PID P/I/D | 16.13 / 1.1625 / 56.23 | 29.37 / 5.43 / 39.71 |
| Alternative hotend fan detection | Enabled | Disabled |
| Fan tachometer checking | Enabled | Disabled |
| Model-based temperature monitoring | Enabled | Disabled |

The two-wire Revo Micro fan configuration uses full-speed cooling above the auto-fan temperature threshold. Traditional temperature limits and thermal-runaway checks remain configured. Disabling model-based monitoring and tachometer checks removes those diagnostic capabilities.

`Marlin_main.cpp` defaults crash detection to off for V2 when the EEPROM setting is first initialized. Existing EEPROM settings persist. `ultralcd.cpp` hides fan-check and extruder-info menus for V2, following the reference port.

The original startup splash and numeric Prusa build version are retained. The variant display name and output filename identify the hardware. The Bear Lab release-specific build-number and repository overrides are omitted.

## Exact firmware code changes

This audit compares Prusa release commit `74a577bc0e5902767b072ee85f499dc1361bf6ba` with firmware implementation commit `e1bd7152b212a834d3444c97d56e4ca80e7b7ed8`. Subsequent commits through `2db08ad61514d42089821ed9f22e06bdcf616951` change documentation, packaging and generated ZIPs. The following diffs cover every firmware source change. Hunk headers identify original and new line numbers. Plus signs indicate additions and minus signs indicate removals.

[Full implementation comparison](https://github.com/ZeeZeeEff/Prusa-Bear-Firmware/compare/74a577bc0e5902767b072ee85f499dc1361bf6ba...e1bd7152b212a834d3444c97d56e4ca80e7b7ed8)

| File | Actual repository change | Purpose |
| --- | --- | --- |
| Firmware/Marlin_main.cpp | 4 added lines | V2 EEPROM crash-detection default |
| Firmware/ultralcd.cpp | 8 added lines | V1 manual calibration and two V2 menu guards |
| Firmware/variants/MK3S_BEAR_STOCK.h | 1 added line | Include stock configuration |
| Firmware/variants/MK3S_BEAR_EXXA_V1.h | 8 added lines | Include stock configuration and apply V1 overrides |
| Firmware/variants/MK3S_BEAR_EXXA_V2.h | 716 added lines | Copy of Bear Lab configuration with variant marker |

The two existing runtime files therefore have 12 added lines and zero deleted lines. The copied V2 configuration accounts for most added lines. Its exact differences from stock are shown below, including comments, so reviewers can inspect its behavior without reviewing hundreds of identical lines. A small line count alone cannot establish safety.

### Safety assessment and validation limits

Compilation and language-generation checks passed for all six targets. Physical hardware tests are pending. Firmware safety has not been guaranteed or certified.

| Variant | Scope of behavior changes | Review implications |
| --- | --- | --- |
| Stock | Uses upstream MK3S configuration | Bear-specific runtime guards are inactive for this build. Correct hardware and calibration still matter. |
| V1 | Automatic Z correction changes from 9 to 2 mm, manual calibration position changes from Z_MAX_POS + 4 to Z_MAX_POS - 3, display name changes | Calibration geometry needs verification on the actual assembled printer. Stock heater and fan protection configuration is inherited. |
| V2 | Geared motor, hotend, motion and calibration settings plus protection changes | Requires hardware-specific validation and review of the disabled monitoring features. |

For V2, `THERMAL_MODEL` is disabled, `FANCHECK` is disabled, and crash detection defaults to off for a newly initialized EEPROM setting. Normal X/Y maximum feedrate increases from 200 to 300 mm/s. These settings were imported from the Bear Lab beta reference. The higher feedrate and crash-detection default are additional upstream choices beyond the basic motor and geometry adaptation.

The V2 variant still configures traditional thermal-runaway thresholds, minimum/maximum temperatures, the watchdog and the safety timer. Their presence in the source does not establish that the assembled printer has passed fault-response tests. Existing EEPROM settings and slicer start G-code can override relevant defaults.

A physical validation record should cover correct temperature readings, heater/fan behavior, fault responses using the upstream test procedure, motor direction/current, travel limits, self-test, Z/XYZ calibration and first-layer operation on each supported hardware configuration.

[Bear Lab validation procedure](https://github.com/bear-lab-3d/BearExxaV2-Prusa-Firmware/blob/3.14.1-BEX201B1/README.md#test)

### Complete runtime and small-header diffs

#### Firmware/Marlin_main.cpp

```diff
@@ -1269,7 +1269,11 @@ void setup()
 	if (silentMode == 0xff) silentMode = 0;
 	tmc2130_mode = TMC2130_MODE_NORMAL;
 
+#ifdef BEAR_EXXA_V2
+  tmc2130_sg_stop_on_crash = eeprom_init_default_byte((uint8_t*)EEPROM_CRASH_DET, false);
+#else
   tmc2130_sg_stop_on_crash = eeprom_init_default_byte((uint8_t*)EEPROM_CRASH_DET, farm_mode ? false : true);
+#endif
 
 	if (tmc2130_sg_stop_on_crash) {
     puts_P(_N("CrashDetect ENABLED!"));
```

#### Firmware/ultralcd.cpp

```diff
@@ -1707,7 +1707,9 @@ static void lcd_support_menu()
 
   MENU_ITEM_BACK_P(STR_SEPARATOR);
   MENU_ITEM_SUBMENU_P(_T(MSG_XYZ_DETAILS), lcd_menu_xyz_y_min);
+#ifndef BEAR_EXXA_V2
   MENU_ITEM_SUBMENU_P(_T(MSG_INFO_EXTRUDER), lcd_menu_extruder_info);
+#endif
   MENU_ITEM_SUBMENU_P(_T(MSG_INFO_SENSORS), lcd_menu_show_sensors_state);
 
 #ifdef TMC2130
@@ -2873,12 +2875,16 @@ bool lcd_calibrate_z_end_stop_manual(bool only_z)
 calibrated:
     // Let the machine think the Z axis is a bit higher than it is, so it will not home into the bed
     // during the search for the induction points.
+#ifdef BEAR_EXXA_V1
+    current_position[Z_AXIS] = Z_MAX_POS - 3.f;
+#else
 	if ((PRINTER_TYPE == PRINTER_MK25) || (PRINTER_TYPE == PRINTER_MK2) || (PRINTER_TYPE == PRINTER_MK2_SNMM)) {
 		current_position[Z_AXIS] = Z_MAX_POS-3.f;
 	}
 	else {
 		current_position[Z_AXIS] = Z_MAX_POS+4.f;
 	}
+#endif
     plan_set_position_curposXYZE();
     return true;
 
@@ -4200,7 +4206,9 @@ static void menuitems_temperature_common() {
 }
 
 void SETTINGS_FANS_CHECK() {
+#ifndef BEAR_EXXA_V2
     MENU_ITEM_TOGGLE_P(_T(MSG_FANS_CHECK), fans_check_enabled ? _T(MSG_ON) : _T(MSG_OFF), lcd_set_fan_check);
+#endif
 }
 
 #define SETTINGS_SOUND \
```

#### Firmware/variants/MK3S_BEAR_EXXA_V1.h

```diff
@@ -0,0 +1,8 @@
+#include "MK3S.h"
+
+// BearExxa V1 geometry, ported from soapee01 8a174c9.
+#define BEAR_EXXA_V1
+#undef CUSTOM_MENDEL_NAME
+#define CUSTOM_MENDEL_NAME "Bear MK3S BEX1"
+#undef Z_MAX_POS_XYZ_CALIBRATION_CORRECTION
+#define Z_MAX_POS_XYZ_CALIBRATION_CORRECTION 2
```

#### Firmware/variants/MK3S_BEAR_STOCK.h

```diff
@@ -0,0 +1 @@
+#include "MK3S.h"
```

### Exact V2 configuration differences from stock

The repository adds this entire 716-line file. The comparison below uses the original Prusa MK3S configuration as the baseline and includes every difference. The additional commented feature switches have no compiled effect.

```diff
--- Prusa-v3.14.1/Firmware/variants/MK3S.h
+++ Firmware/variants/MK3S_BEAR_EXXA_V2.h
@@ -1,4 +1,5 @@
 #ifndef CONFIGURATION_PRUSA_H
 #define CONFIGURATION_PRUSA_H
+#define BEAR_EXXA_V2
 
 #include <limits.h>
@@ -15,12 +16,13 @@
 #define PRINTER_MMU_NAME PRINTER_MK3S_MMU3_NAME
 #define FILAMENT_SIZE "1_75mm_MK3S"
-#define NOZZLE_TYPE "E3Dv6full"
+#define NOZZLE_TYPE "E3DRevoMicro"
 
 // Printer name
-#define CUSTOM_MENDEL_NAME "Prusa i3 MK3S"
+#define CUSTOM_MENDEL_NAME "Bear MK3S BEX2"
 
 // Electronics
 #define MOTHERBOARD BOARD_EINSY_1_0a
 #define STEEL_SHEET
+//#define NEW_FIRST_LAYER_CAL //from front to back
 #define HAS_SECOND_SERIAL_PORT
 
@@ -41,5 +43,5 @@
 
 // Steps per unit {X,Y,Z,E}
-#define DEFAULT_AXIS_STEPS_PER_UNIT   {100,100,3200/8,280}
+#define DEFAULT_AXIS_STEPS_PER_UNIT   {100,100,3200/8,415}
 
 // Endstop inverting
@@ -64,9 +66,9 @@
 #define Y_MAX_POS 212.5
 #define Y_MIN_POS -4 //orig -4
-#define Z_MAX_POS 210
+#define Z_MAX_POS 207
 #define Z_MIN_POS 0.15
 
 // Z height correction value
-#define Z_MAX_POS_XYZ_CALIBRATION_CORRECTION 9
+#define Z_MAX_POS_XYZ_CALIBRATION_CORRECTION 3
 
 // Canceled home position
@@ -93,5 +95,5 @@
 #define SHEET_PRINT_ZERO_REF_Y -2.f
 
-#define DEFAULT_MAX_FEEDRATE                {200, 200, 12, 120}      // (mm/sec)   max feedrate (M203)
+#define DEFAULT_MAX_FEEDRATE                {300, 300, 12, 120}      // (mm/sec)   max feedrate (M203)
 #define DEFAULT_MAX_FEEDRATE_SILENT         {100, 100, 12, 120}      // (mm/sec)   max feedrate (M203), silent mode
 
@@ -133,5 +135,5 @@
 
 // Fan check
-#define FANCHECK
+//#define FANCHECK
 
 // Safety timer
@@ -221,5 +223,5 @@
 #define TMC2130_USTEPS_XY   16        // microstep resolution for XY axes
 #define TMC2130_USTEPS_Z    16        // microstep resolution for Z axis
-#define TMC2130_USTEPS_E    32        // microstep resolution for E axis
+#define TMC2130_USTEPS_E    16        // microstep resolution for E axis
 #define TMC2130_INTPOL_XY   1         // extrapolate 256 for XY axes
 #define TMC2130_INTPOL_Z    1         // extrapolate 256 for Z axis
@@ -283,8 +285,8 @@
 
 //new settings is possible for vsense = 1, running current value > 31 set vsense to zero and shift both currents by 1 bit right (Z axis only)
-#define TMC2130_CURRENTS_H {16, 20, 35, 30}  // default holding currents for all axes
+#define TMC2130_CURRENTS_H {16, 20, 35, 21}  // default holding currents for all axes
 #define TMC2130_CURRENTS_FARM 36             // E 805 mA peak for ECool/farm mode
-#define TMC2130_CURRENTS_R {16, 20, 35, 30}  // default running currents for all axes
-#define TMC2130_CURRENTS_R_HOME {8, 10, 20, 18}  // homing running currents for all axes
+#define TMC2130_CURRENTS_R {16, 20, 35, 21}  // default running currents for all axes
+#define TMC2130_CURRENTS_R_HOME {8, 10, 20, 15}  // homing running currents for all axes
 
 #define TMC2130_STEALTH_Z
@@ -337,7 +339,7 @@
 //#define  DEFAULT_Ki 4.875
 //#define  DEFAULT_Kd 86.085
-#define  DEFAULT_Kp 16.13
-#define  DEFAULT_Ki 1.1625
-#define  DEFAULT_Kd 56.23
+#define  DEFAULT_Kp 29.37
+#define  DEFAULT_Ki 5.43
+#define  DEFAULT_Kd 39.71
 #endif
 
@@ -352,5 +354,5 @@
 #define EXTRUDER_AUTO_FAN_TEMPERATURE 50
 #define EXTRUDER_AUTO_FAN_SPEED   255  // == full speed
-#define EXTRUDER_ALTFAN_DETECT
+//#define EXTRUDER_ALTFAN_DETECT
 #define EXTRUDER_ALTFAN_SPEED_SILENT 128
 
@@ -406,24 +408,24 @@
 
 // model-based temperature check
-#define THERMAL_MODEL 1              // enable model-based temperature checks
-#define THERMAL_MODEL_DEBUG 1        // extended runtime logging
-
-#define THERMAL_MODEL_CAL_C_low 5    // C estimation lower limit
-#define THERMAL_MODEL_CAL_C_high 20  // C estimation upper limit
-#define THERMAL_MODEL_CAL_C_thr 0.01 // C estimation iteration threshold
-#define THERMAL_MODEL_CAL_C_itr 30   // C estimation iteration limit
-
-#define THERMAL_MODEL_CAL_R_low 5    // R estimation lower limit
-#define THERMAL_MODEL_CAL_R_high 50  // R estimation upper limit
-#define THERMAL_MODEL_CAL_R_thr 0.01 // R estimation iteration threshold
-#define THERMAL_MODEL_CAL_R_itr 30   // R estimation iteration limit
-
-#define THERMAL_MODEL_CAL_T_low 50   // Default calibration cooling temperature (C)
-#define THERMAL_MODEL_CAL_T_high 230 // Default calibration working temperature (C)
-
-#define THERMAL_MODEL_Ta_corr -7     // Default ambient temperature correction
-
-#include "thermal_model/e3d_v6.h"
-#define THERMAL_MODEL_DEFAULT E3D_V6 // Default model parameters
+//#define THERMAL_MODEL 1              // enable model-based temperature checks
+//#define THERMAL_MODEL_DEBUG 1        // extended runtime logging
+
+//#define THERMAL_MODEL_CAL_C_low 5    // C estimation lower limit
+//#define THERMAL_MODEL_CAL_C_high 20  // C estimation upper limit
+//#define THERMAL_MODEL_CAL_C_thr 0.01 // C estimation iteration threshold
+//#define THERMAL_MODEL_CAL_C_itr 30   // C estimation iteration limit
+
+//#define THERMAL_MODEL_CAL_R_low 5    // R estimation lower limit
+//#define THERMAL_MODEL_CAL_R_high 50  // R estimation upper limit
+//#define THERMAL_MODEL_CAL_R_thr 0.01 // R estimation iteration threshold
+//#define THERMAL_MODEL_CAL_R_itr 30   // R estimation iteration limit
+
+//#define THERMAL_MODEL_CAL_T_low 50   // Default calibration cooling temperature (C)
+//#define THERMAL_MODEL_CAL_T_high 230 // Default calibration working temperature (C)
+
+//#define THERMAL_MODEL_Ta_corr -7     // Default ambient temperature correction
+
+//#include "thermal_model/e3d_v6.h"
+//#define THERMAL_MODEL_DEFAULT E3D_V6 // Default model parameters
 
 /*------------------------------------
@@ -702,3 +704,13 @@
 #define DEFAULT_ARC_SEGMENTS_PER_SEC 0 // Use feedrate to choose segment length. Set to 0 to disable
 
+/*------------------------------------
+ COMMUNITY FEATURES
+ *------------------------------------*/
+
+//Show filename instead of print time after SD card print finished
+//#define SHOW_FILENAME_AFTER_FINISH
+
+//Remove the "AutoLoad filament" LCD menu entry if autoload is enabled.
+//#define REMOVE_AUTOLOAD_FILAMENT_MENU_ENTRY
+
 #endif //__CONFIGURATION_PRUSA_H
```

## Build process

Use a Linux x86-64 environment with Git, Python 3.12, CMake, Ninja and gettext. Prusa bootstrap downloads AVR GCC 7.3.0 and prusa3dboards 1.0.6.

To build the published source, clone the firmware branch.

```bash
git clone --branch bear-3.14.1 https://github.com/ZeeZeeEff/Prusa-Bear-Firmware.git
cd Prusa-Bear-Firmware
```

The branch already includes the three configurations, runtime adaptations and build workflow. Its parent is the exact Prusa 3.14.1 release commit.

```bash
python3 -m venv .venv
. .venv/bin/activate
python -m pip install pyelftools polib regex
python utils/bootstrap.py
cmake -S . -B build-bear -G Ninja \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_TOOLCHAIN_FILE=cmake/AvrGcc.cmake \
  '-DFW_VARIANTS=MK3S_BEAR_STOCK;MK3S_BEAR_EXXA_V1;MK3S_BEAR_EXXA_V2'
cmake --build build-bear --target \
  MK3S_BEAR_STOCK_ENGLISH MK3S_BEAR_STOCK_MULTILANG \
  MK3S_BEAR_EXXA_V1_ENGLISH MK3S_BEAR_EXXA_V1_MULTILANG \
  MK3S_BEAR_EXXA_V2_ENGLISH MK3S_BEAR_EXXA_V2_MULTILANG
```

Use these explicit targets. The upstream ALL_MULTILANG packaging step assumes Revo filenames outside this selected set. HEX outputs are under build-bear, with multilingual release copies in build-bear/release. Prusa's build enforces the flash region limit and runs language checks.

The included GitHub Actions workflow runs the same six targets on pushes to bear-3.14.1 and uploads the HEX files. Fork Actions may need to be enabled in GitHub. A successful workflow confirms compilation only.

## Installation and hardware verification

Select the exact hardware variant when flashing through PrusaSlicer. Record existing calibration and EEPROM settings first.

For a V2 conversion, Bear Lab specifies an All Data factory reset followed by the wizard, so the motor and motion defaults are loaded. This clears stored calibration. Recalibrate Z, XYZ, first layer and PID for the actual hardware. Inspect M503 for steps, feedrates and PID. Check extrusion distance, filament sensing, heating, cooling and print behavior.

For V2, use the Bear Lab slicer changes, including 0.4 mm starting retraction and removal of stock M907 extruder-current commands from start G-code. Tune extrusion multiplier and Linear Advance. Follow the full upstream hardware-specific procedure.

[Bear Lab V2 setup and validation](https://github.com/bear-lab-3d/BearExxaV2-Prusa-Firmware/blob/3.14.1-BEX201B1/README.md)

## Verification record

- Verified the release commit and source configuration through GitHub.
- Compared V1 calibration changes with the reference commit.
- Compared the V2 variant with the referenced release.
- Checked that each targeted runtime replacement matches exactly once.
- All six firmware targets passed AVR compilation and linking, including multilingual catalog generation and the build's flash-size checks.
- English builds use 230756 program bytes / 5665 static data bytes for stock and V1, and 217874 / 5462 bytes for V2.
- V2 emits unused-function warnings for the disabled fan-check code.
- Physical printer tests are pending.

[Successful build of commit e1bd715](https://github.com/ZeeZeeEff/Prusa-Bear-Firmware/actions/runs/43598836992)
[Download firmware artifact](https://github.com/ZeeZeeEff/Prusa-Bear-Firmware/actions/runs/43598836992/artifacts/14115214771)

The ZIP includes versioned English and multilingual HEX files and additional multilingual filename aliases. Select the filename containing your exact variant and language mode. The artifact ZIP SHA-256 reported by GitHub is `5a154551615f25cfc49eb0102216c0f9094b93d448d3b99f38632114416e0549`. GitHub Actions artifacts have limited retention.


## License

GNU GPL v3 under the existing LICENSE. Credit remains with Prusa Research, the Marlin contributors, Bear Lab and the referenced community contributors.
