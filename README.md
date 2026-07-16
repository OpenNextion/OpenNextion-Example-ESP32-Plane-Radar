# OpenNextion ESP32 Plane Radar

[![English](https://img.shields.io/badge/lang-English-blue)](./README.md)
[![中文](https://img.shields.io/badge/lang-中文-red)](./README.zh-CN.md)

<p align="center">
  <img src="docs/images/opennextion-esp32-plane-radar-demo.jpg" alt="OpenNextion ESP32 Plane Radar demo on OpenNextion display" width="820">
</p>

OpenNextion ESP32 Plane Radar is an OpenNextion board support fork of
[ESP32 Plane Radar](https://github.com/MatixYo/ESP32-Plane-Radar). It adds
ready-to-build PlatformIO firmware targets for OpenNextion ESP32-S3 rectangular
display boards while keeping the original circular ADS-B radar UI and first-time
Wi-Fi setup flow.

This repository is intended to make ESP32 Plane Radar easier to build, flash,
and validate on supported OpenNextion development boards while the upstream
board support pull requests are under review.

## Supported Displays

The current public branch focuses on two OpenNextion portrait displays:

| PlatformIO env | Display model | Size | Resolution | Orientation | Status |
| --- | --- | --- | --- | --- | --- |
| `onx3248g035` | [ONX3248G035][onx3248g035] | 3.5 inch | 320 x 480 | Portrait | Verified |
| `onx2432g028` | [ONX2432G028][onx2432g028] | 2.8 inch | 240 x 320 | Portrait | Verified |

This fork is documented primarily for the OpenNextion boards. Upstream hardware
targets are retained in the source tree for compatibility.

Build-time board selection is explicit:

```sh
pio run -e onx3248g035
pio run -e onx2432g028
```

Do not flash firmware built for one display model onto another display model.

## Controls

Plane Radar uses a single active-low BOOT button for device actions.

| Action | Effect |
| --- | --- |
| Short tap | Cycle range preset: 5 -> 10 -> 15 -> 25 km; saved to flash |
| Hold 3 s | Clear Wi-Fi, location, and units; reboot into setup portal |
| Hold during setup / boot | Force credential reset and setup portal |

BOOT is GPIO `0` on the OpenNextion ESP32-S3 targets.

## Wi-Fi Setup Portal

First-time setup starts a captive portal on the AP `PlaneRadar-Setup`.

1. Connect to `PlaneRadar-Setup`.
2. Open `http://plane-radar.local` or `http://192.168.4.1`.
3. Set home Wi-Fi and save.

After the device joins your LAN, the same portal remains available at
`http://plane-radar.local` or the device IP address shown in the serial log. Use
it to update Wi-Fi, radar location, units, and runway overlay settings.

Custom fields stored in NVS:

| Field | Purpose |
| --- | --- |
| Latitude / Longitude | Radar center and ADS-B query position |
| Display distances in miles | Shows ring labels in miles instead of kilometers |
| Show airport runways | Toggles the major-airport runway overlay |

After a reset, the device reboots and shows the setup screen immediately instead
of looping on stale credentials.

## Background

I found an excellent plane radar project on GitHub called
[ESP32 Plane Radar](https://github.com/MatixYo/ESP32-Plane-Radar). The original
project is mainly designed for a round display, but the displays I have on hand
are rectangular OpenNextion ESP32 displays, so I decided to port Plane Radar to
these rectangular screens.

The OpenNextion boards use ESP32-S3 as the main controller and provide useful
hardware resources for DIY projects, including schematics, board files, and
mechanical reference files for enclosure or stand design.

This fork keeps the original radar behavior and adds the OpenNextion-specific
hardware support needed by the supported boards.

## 3D Printed Enclosure

I also designed simple 3D printed enclosures for the supported OpenNextion
display sizes and will publish them on MakerWorld. Anyone who needs them can
download and print them for free.

Each enclosure is intended to be a simple desktop shell for the matching
OpenNextion display. Peel off the tape around the edge of the matching display,
then press the display into the printed enclosure and use the adhesive edge to
hold it in place.

MakerWorld project links:

- 3.5 inch ONX3248G035 enclosure: link to be added
- 2.8 inch ONX2432G028 enclosure: link to be added

## Current Porting Work

This version is based on ESP32 Plane Radar and adds OpenNextion multi-board
support. The main changes are:

### 1. OpenNextion Board Support

OpenNextion ESP32 Plane Radar includes dedicated PlatformIO environments for:

- [ONX3248G035][onx3248g035] 3.5 inch portrait display
- [ONX2432G028][onx2432g028] 2.8 inch portrait display

Each board has its own PlatformIO board JSON file and board macro:
`BOARD_ONX3248G035` or `BOARD_ONX2432G028`.

### 2. Display and Board Initialization

The port adds the OpenNextion display initialization required by the supported
boards:

- ST7796U SPI TFT panel support for ONX3248G035
- ST7789 SPI TFT panel support for ONX2432G028
- PCF8574 IO expander support for LCD reset and SDCS release
- Backlight GPIO setup
- Board-specific LovyanGFX panel selection

Board pin mapping is handled by the PlatformIO board target and `include/config.h`.

Touch and SD card hardware are present on the OpenNextion boards, but this
firmware does not use them yet.

### 3. Rectangular Display UI Layout

The original project uses a 240 x 240 circular radar. The OpenNextion boards use
portrait rectangular screens, so the radar is kept at the top and an information
panel is added below it.

Current layout behavior:

- ONX2432G028: 240 x 240 radar at the top, compact bottom info panel
- ONX3248G035: larger 320 x 320 radar at the top, larger bottom info panel
- Upstream compatibility target: keeps the original 240 x 240 radar UI

The information panel shows range, aircraft count, update age, and nearest
aircraft details where screen space allows.

### 4. PSRAM Frame Sprite for 3.5 inch Display

ONX3248G035 uses a 320 x 480 full-frame sprite. The frame sprite is allocated in
PSRAM for this board to avoid allocation failure and display flicker.

### 5. PlatformIO Build and Merge Targets

The project includes PlatformIO environments for all supported targets and uses
`pio run -t merge` or the helper script to generate a single merged binary for
full flashing from address `0x0`.

## Current Validation Status

### Display Validation

<p align="center">
  <img src="docs/images/opennextion-esp32-plane-radar-validation.jpg" alt="OpenNextion ESP32 Plane Radar validation on OpenNextion display" width="720">
</p>

- ONX3248G035 has been validated on real hardware
- ONX2432G028 has been validated on real hardware
- Wi-Fi setup flow has been validated on the OpenNextion targets
- Radar display and ADS-B refresh have been validated visually on hardware
- BOOT short tap range switching and long-press reset are supported
- Touch and SD card are not used by this firmware yet

### Firmware Validation Matrix

Legend: ✅ Verified / ⚠️ Partially verified or hardware-dependent / ⏳ Not used

| Board | Build | Boot | Display | Wi-Fi setup | Radar UI | Touch | SD card | Notes |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| ONX3248G035 | ✅ Verified | ✅ Verified | ✅ Verified | ✅ Verified | ✅ Verified | ⏳ Not used | ⏳ Not used | 3.5 inch ST7796U display |
| ONX2432G028 | ✅ Verified | ✅ Verified | ✅ Verified | ✅ Verified | ✅ Verified | ⏳ Not used | ⏳ Not used | 2.8 inch ST7789 display |

## Radar Features

### Grid and Range

- Dark blue background with green rings and crosshairs
- White N / S / E / W labels and an east-side range label
- Range presets: 5 km, 10 km, 15 km, and 25 km
- Miles or kilometers can be selected in the setup portal

Range presets are stored in NVS and configured in `include/ui/radar_range.h`.

### Aircraft

- Aircraft inside the outer ring use a red heading triangle
- Magenta speed vectors show track direction
- Callsign, type, and altitude tags are drawn near each aircraft
- Aircraft outside the outer ring are shown as red bearing dots on the screen rim

### Runways

- Major airports are generated from OurAirports `large_airport` data
- Open runway strips are shown in teal when enabled
- The setup portal can enable or disable the runway overlay
- Regenerate the embedded list with `python3 scripts/build_large_airports.py`

### ADS-B Data

- Data source: `https://opendata.adsb.fi/api/v3/`
- Fetch radius is derived from the active radar range
- Poll interval is controlled by `kAdsbFetchIntervalMs` in `include/config.h`
- Ground aircraft are hidden by default with `kAdsbShowGroundAircraft`

## Firmware Download and Flashing

Download firmware from the GitHub Release page when release binaries are
available. A merged binary is intended for full initial flashing from address
`0x0`.

Release files follow this naming pattern:

```text
opennextion-esp32-plane-radar-<version>-<target>.bin
```

| Target | Example firmware file | Flash address |
| --- | --- | --- |
| `onx3248g035` | `opennextion-esp32-plane-radar-v0.1.0-onx3248g035.bin` | `0x0` |
| `onx2432g028` | `opennextion-esp32-plane-radar-v0.1.0-onx2432g028.bin` | `0x0` |

Flash a merged binary with:

```sh
VERSION=v0.1.0
python -m esptool --chip esp32s3 -p /dev/cu.wchusbserial1110 -b 921600 write_flash \
  0x0 ./opennextion-esp32-plane-radar-${VERSION}-onx2432g028.bin
```

Replace `VERSION`, serial port, and firmware target name as needed for your
board.

For this project, full firmware flashing is recommended for first installation.
OTA firmware downloads are not provided unless the OTA flow is separately
validated.

## Local Build, Flash and Monitor

This project uses PlatformIO with the Arduino framework. The commands below are
for OpenNextion boards.

### Build

```bash
pio run -e onx2432g028
pio run -e onx3248g035
```

### Clean and Rebuild

```bash
pio run -e onx2432g028 -t clean
pio run -e onx3248g035 -t clean
```

For a deeper clean, remove the board-specific build and dependency directories:

```bash
rm -rf .pio/build/onx2432g028 .pio/libdeps/onx2432g028
rm -rf .pio/build/onx3248g035 .pio/libdeps/onx3248g035
```

### Upload

```bash
pio run -e onx2432g028 -t upload --upload-port <PORT>
pio run -e onx3248g035 -t upload --upload-port <PORT>
```

Example OpenNextion serial port on macOS:

```bash
pio run -e onx3248g035 -t upload --upload-port /dev/cu.wchusbserial1110
```

### Monitor Serial Log

```bash
pio device monitor -e <env> --port <PORT> --baud 115200
```

The firmware prints the LAN configuration URL after Wi-Fi connects, for example
`http://plane-radar.local` or `http://<device-ip>`.

### Generate a Single Merged Binary

PlatformIO merge target:

```bash
pio run -e onx2432g028 -t merge
pio run -e onx3248g035 -t merge
```

Outputs:

```text
.pio/build/<env>/firmware-merged.bin
```

Helper script:

```bash
VERSION=v0.1.0
chmod +x scripts/merge-firmware.sh
./scripts/merge-firmware.sh --env onx2432g028 -o release/opennextion-esp32-plane-radar-${VERSION}-onx2432g028.bin
./scripts/merge-firmware.sh --env onx3248g035 -o release/opennextion-esp32-plane-radar-${VERSION}-onx3248g035.bin
```

Skip rebuild if firmware is already built:

```bash
VERSION=v0.1.0
./scripts/merge-firmware.sh --env onx3248g035 --no-build -o release/opennextion-esp32-plane-radar-${VERSION}-onx3248g035.bin
```

## Dependencies

- [LovyanGFX](https://github.com/lovyan03/LovyanGFX)
- [WiFiManager](https://github.com/tzapu/WiFiManager)
- [ArduinoJson](https://github.com/bblanchon/ArduinoJson)
- [adsb.fi Open Data API](https://opendata.adsb.fi/)

## Roadmap

Planned next steps:

- Keep this OpenNextion fork aligned with upstream ESP32 Plane Radar where practical
- Publish convenient merged firmware binaries in GitHub Releases
- Add enclosure links when OpenNextion printable cases are ready
- Continue validating display, Wi-Fi, ADS-B, and UI behavior on supported hardware

## Credits

This project is based on ESP32 Plane Radar. Thanks to the original author and the
related open source projects.

- ESP32 Plane Radar: https://github.com/MatixYo/ESP32-Plane-Radar
- OpenNextion open source projects: https://github.com/OpenNextion
- OpenNextion board documentation: https://nextion.tech/wiki/

## License

This project preserves the upstream ESP32 Plane Radar license terms. See
`LICENSE` for details. Third-party libraries may have their own license notices.

## Disclaimer

This project is not the official upstream ESP32 Plane Radar project.

Plane Radar displays aircraft data from public ADS-B sources and depends on
network availability, DNS, Wi-Fi quality, local configuration, and third-party
service availability. Flashing and using third-party firmware involves risk.
Please use it only after understanding the risks. This project is not
responsible for device damage, data loss, network connection issues, inaccurate
or delayed aircraft data, legal consequences, or any other consequences of use.

[onx3248g035]: https://nextion.tech/wiki/onx3248g035/
[onx2432g028]: https://nextion.tech/wiki/onx2432g028/
