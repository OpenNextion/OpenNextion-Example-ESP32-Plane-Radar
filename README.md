# Plane Radar

<img width="800" height="450" alt="plane-radar" src="https://github.com/user-attachments/assets/716d0992-dab8-47ba-8f1a-2aec7f607419" />

**3D printed case (STL + assembly):** [MakerWorld](https://makerworld.com/en/models/2872376-esp32-plane-radar-live-ads-b-on-a-round-display#profileId-3207083) · **Firmware:** [Releases](https://github.com/MatixYo/ESP32-Plane-Radar/releases)

Firmware for an **ESP32-C3 Super Mini** and a **1.28″ round GC9A01** display (240×240). Shows a circular **ADS-B radar** around your configured location, with **WiFiManager** for first-time setup.

This fork also includes first-pass board targets for OpenNextion ESP32-S3 display boards while keeping the original circular radar UI intact.

## What it does

1. **Wi‑Fi setup** (if needed) — captive portal on AP **`PlaneRadar-Setup`**
2. **Radar** — live aircraft from [adsb.fi](https://opendata.adsb.fi/) on a sonar-style grid

After Wi‑Fi is saved, the device reconnects automatically; the radar runs in the main loop with periodic ADS-B updates (~5 s).

## Supported boards

| PlatformIO env | Board | Display | Resolution | Notes |
|----------------|-------|---------|------------|-------|
| `supermini` | ESP32-C3 Super Mini | GC9A01 round SPI TFT | 240×240 | Original target |
| `onx2432g028` | OpenNextion ONX2432G028 | ST7789 SPI TFT | 240×320 | 240×240 radar view centered vertically |
| `onx3248g035` | OpenNextion ONX3248G035 | ST7796U SPI TFT | 320×480 | 240×240 radar view centered; frame sprite uses PSRAM |

The OpenNextion targets use ESP32-S3R8 modules with 16 MB flash and 8 MB OPI PSRAM. Touch and SD card hardware are present on the boards but are not used by this firmware yet.

## Controls (BOOT, active LOW)

| Action | Effect |
|--------|--------|
| **Short tap** | Cycle range preset (5 → 10 → 15 → 25 km); saved to flash |
| **Hold 3 s** | Clear Wi‑Fi, location, and units; reboot into setup portal |

BOOT is GPIO **9** on the original ESP32-C3 Super Mini target and GPIO **0** on the OpenNextion ESP32-S3 targets. During setup you can also hold BOOT at power-on to force a credential reset (same as the long press).

## Wi‑Fi setup portal

**First-time setup** (no saved Wi‑Fi):

1. Connect to **`PlaneRadar-Setup`**
2. Open **`http://plane-radar.local`** (preferred) or **`http://192.168.4.1`** — both are shown on the yellow setup screen; captive portal may open automatically
3. Set home Wi‑Fi, then save

**Reconfigure anytime** (after the device is on your network):

1. Open **`http://plane-radar.local`** or **`http://<device-ip>`** (e.g. from your router or serial log at boot)
2. Change Wi‑Fi, location, units, or runway overlay; save

The same portal runs on the setup AP and on the device’s LAN IP while connected to Wi‑Fi. mDNS hostname is `plane-radar` → **plane-radar.local** (`kPortalHostname` in `config.h`). Some clients resolve `.local` slowly; use the IP if needed.

**Custom fields** (stored in NVS):

| Field | Purpose |
|-------|---------|
| **Latitude / Longitude** | Radar center and ADS-B query position (defaults in `config.h` until set) |
| **Display distances in miles** | Ring scale label in **mi** instead of **km** (e.g. `6mi` vs `10km`) |
| **Show airport runways** | Major-airport runway overlay on the radar (off to hide) |

After a reset, the device reboots and shows the setup screen immediately (no “Connecting” loop on stale credentials).

## Radar display

### Grid

- Dark blue background, subdued green rings and crosshairs
- White **N / S / E / W** at the bezel; range label on the **east** spoke (ring 3 = ¾ of outer radius)
- White center dot

Layout and colors: `include/ui/radar_theme.h`.

### Range presets

| Ring 3 label | Outer radius (aircraft scale) |
|------------|-------------------------------|
| 5 km / 3 mi | ~6.7 km |
| 10 km / 6 mi | ~13.3 km (default) |
| 15 km / 9 mi | ~20 km |
| 25 km / 16 mi | ~33.3 km |

Preset and miles/km choice persist across reboot (`planeradar` NVS namespace).

### Runways

- Major airports from OurAirports (`large_airport`); all open runway strips in range (helipads excluded)
- Teal runway lines with one ICAO label per airport (e.g. `KJFK`); toggle in the Wi‑Fi setup portal
- Update the embedded list: `python3 scripts/build_large_airports.py`

### Aircraft

- **Inside the outer ring** — red heading triangle, magenta speed vector (clipped at the ring), callsign / type / altitude tags
- **Outside the ring** (still within ADS-B fetch) — small **red dot on the screen rim** at the correct bearing (direction cue; not distance-accurate past the ring)
- **Tags** — placed toward the **center**: west (left) → tag on the **right** of the symbol; east (right) → tag on the **left**

As range decreases (or aircraft approach), targets move inward; beyond-ring dots become full symbols when they cross the outer ring.

### ADS-B

- Source: `https://opendata.adsb.fi/api/v3/`
- Fetch radius: `ui::radar::fetchRadiusKm()` — scales with the active preset to roughly the screen edge (so rim dots have data)
- Poll interval: `kAdsbFetchIntervalMs` (5 s) in `config.h`
- Ground aircraft hidden by default (`kAdsbShowGroundAircraft`)

## Configuration

Edit **`include/config.h`** for hardware and behavior:

| Area | Keys / notes |
|------|----------------|
| Portal | `kPortalApName`, `kPortalIp`, `kPortalHostname` / `kPortalHostUrl` (mDNS; needs `-DWM_MDNS` in `platformio.ini`) |
| Wi‑Fi timing | connect attempts, reconnect grace, portal timeout (`0` = no timeout) |
| BOOT | `kBootPin`, `kBootResetHoldMs`, `kBootTapMinMs` |
| Display SPI | pins, `kDisplayInvert`, `kDisplayRgbOrder`, `kDisplaySpiWriteHz` |
| Default location | `kDefaultRadarLat`, `kDefaultRadarLon` (until portal overrides) |
| ADS-B | `kAdsbFetchIntervalMs`, `kAdsbShowGroundAircraft` |

Range presets: `include/ui/radar_range.h` (`kRangePresets`).

## Project layout

```
include/
  config.h
  hardware/
    board_support.h
    lgfx_config.hpp
    display.h
    display_font.h
  data/
    large_airports.h
  ui/
    radar_theme.h
    radar_range.h
    radar_display.h
    runway_overlay.h
    status_screens.h
  services/
    wifi_setup.h
    radar_location.h
    adsb_client.h
boards/
  onx2432g028.json
  onx3248g035.json
data/
  ui_font.vlw              — embedded smooth UI font (Noto Sans Bold)
scripts/
  build_large_airports.py
src/
  main.cpp
  data/
    large_airports_data.cpp
  hardware/
  ui/
  services/
```

## Wiring (GC9A01 ↔ ESP32-C3 Super Mini)

| Display | ESP32-C3 |
|---------|----------|
| VCC | 3V3 |
| GND | GND |
| RST | GPIO **0** |
| CS | GPIO **1** |
| DC | GPIO **10** |
| SDA (MOSI) | GPIO **3** |
| SCL (SCLK) | GPIO **4** |
| BOOT (user) | GPIO **9** |

## Wiring (OpenNextion ONX boards)

| Function | ONX2432G028 / ONX3248G035 |
|----------|---------------------------|
| I2C SDA / SCL | GPIO **8** / GPIO **7** |
| LCD SCLK / MOSI | GPIO **5** / GPIO **1** |
| LCD CS / DC | GPIO **2** / GPIO **3** |
| LCD BL | GPIO **6** |
| LCD RST | PCF8574 EXIO6 |
| SDCS release | PCF8574 EXIO7 |
| BOOT (user) | GPIO **0** |

## Build

Build a specific target with its PlatformIO environment:

```bash
pio run -e supermini
pio run -e onx2432g028
pio run -e onx3248g035
```

Flash a target:

```bash
pio run -e <env> -t upload --upload-port <PORT>
```

Monitor serial output:

```bash
pio device monitor -e <env> --port <PORT> --baud 115200
```

- Available PlatformIO envs: **`supermini`**, **`onx2432g028`**, **`onx3248g035`**
- Serial: **115200** baud
- USB CDC on boot is enabled in `platformio.ini` for the Super Mini; the OpenNextion ESP32-S3 boards use their USB-UART serial port

### Web-flashable release image

Single `.bin` for [esptool-js](https://espressif.github.io/esptool-js/) and similar tools (flash at **0x0**):

```bash
pio run -e <env> -t merge
```

Outputs:

```text
.pio/build/<env>/firmware-merged.bin
```

The helper script can also build and copy a merged image into `release/`:

```bash
chmod +x scripts/merge-firmware.sh   # once
./scripts/merge-firmware.sh --env supermini -o release/plane-radar-supermini-merged.bin
./scripts/merge-firmware.sh --env onx2432g028 -o release/plane-radar-onx2432g028-merged.bin
./scripts/merge-firmware.sh --env onx3248g035 -o release/plane-radar-onx3248g035-merged.bin
```

Skip rebuild if firmware is already built:

```bash
./scripts/merge-firmware.sh --env onx3248g035 --no-build -o release/plane-radar-onx3248g035-merged.bin
```

Put the board in download mode (hold **BOOT**, tap **RESET**), then flash with Chrome/Edge over USB, or use `esptool`:

```bash
python -m esptool --chip esp32c3 -p <PORT> -b 921600 write_flash 0x0 .pio/build/supermini/firmware-merged.bin
python -m esptool --chip esp32s3 -p <PORT> -b 921600 write_flash 0x0 .pio/build/onx2432g028/firmware-merged.bin
python -m esptool --chip esp32s3 -p <PORT> -b 921600 write_flash 0x0 .pio/build/onx3248g035/firmware-merged.bin
```

### CI and releases (GitHub Actions)

| Workflow | When | Output |
|----------|------|--------|
| [Build](.github/workflows/build.yml) | Push / PR to `main` | Artifact `plane-radar-supermini` (merged + split `.bin` files, ~90 days) |
| [Release](.github/workflows/release.yml) | Git tag `v*` (e.g. `v1.0.0`) | GitHub Release asset `plane-radar-v1.0.0.bin` + `.sha256` |

To ship a version users can download:

```bash
git tag v1.0.0
git push origin v1.0.0
```

The release workflow builds firmware in CI and attaches the merged image to the release. Download from **Releases** on GitHub, then flash at **0x0** (ESP32-C3, 4 MB).

## Dependencies

- [LovyanGFX](https://github.com/lovyan03/LovyanGFX)
- [WiFiManager](https://github.com/tzapu/WiFiManager)
- [ArduinoJson](https://github.com/bblanchon/ArduinoJson)
