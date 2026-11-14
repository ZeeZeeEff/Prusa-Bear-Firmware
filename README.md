# Prusa Bear Firmware 3.14.1

Three MK3S/Einsy 1.0a build variants based on Prusa firmware 3.14.1.

## Status

All three variants compiled successfully in English and multilingual configurations on 14 November 2026. Physical printer validation is pending.
The local environment cannot resolve github.com. Sources were retrieved and published through the GitHub connector. The included Actions workflow provides the build environment.
Check the workflow result before using any generated firmware.

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
