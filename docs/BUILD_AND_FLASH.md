# Build, Flash, and Merge Firmware

This project uses PlatformIO with the Arduino framework. The commands below are
for the supported OpenNextion ESP32-S3 boards.

## Supported PlatformIO Environments

| Environment | Display model | Resolution | Orientation |
| --- | --- | --- | --- |
| `onx3248g035` | ONX3248G035 | 320 x 480 | Portrait |
| `onx2432g028` | ONX2432G028 | 240 x 320 | Portrait |

## Build

```bash
pio run -e onx2432g028
pio run -e onx3248g035
```

## Clean Build Artifacts

The `clean` target only removes build artifacts. It does not rebuild firmware.

```bash
pio run -e onx2432g028 -t clean
pio run -e onx3248g035 -t clean
```

To clean and then rebuild, run the build command after `clean`:

```bash
pio run -e onx2432g028 -t clean
pio run -e onx2432g028

pio run -e onx3248g035 -t clean
pio run -e onx3248g035
```

For a deeper clean, remove the board-specific build and dependency directories:

```bash
rm -rf .pio/build/onx2432g028 .pio/libdeps/onx2432g028
rm -rf .pio/build/onx3248g035 .pio/libdeps/onx3248g035
```

## Upload from PlatformIO

```bash
pio run -e onx2432g028 -t upload --upload-port <PORT>
pio run -e onx3248g035 -t upload --upload-port <PORT>
```

Example OpenNextion serial port on macOS:

```bash
pio run -e onx3248g035 -t upload --upload-port /dev/cu.wchusbserial1110
```

## Monitor Serial Log

```bash
pio device monitor -e <env> --port <PORT> --baud 115200
```

The firmware prints the LAN configuration URL after Wi-Fi connects, for example
`http://plane-radar.local` or `http://<device-ip>`.

## Generate a Single Merged Binary

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

## Flash a Merged Release Binary

Merged release binaries should be flashed at address `0x0`:

```bash
python -m esptool --chip esp32s3 -p <PORT> -b 921600 write_flash \
  0x0 ./opennextion-esp32-plane-radar-v0.1.0-onx3248g035.bin
```

Replace `<PORT>` and the firmware filename for your board.
