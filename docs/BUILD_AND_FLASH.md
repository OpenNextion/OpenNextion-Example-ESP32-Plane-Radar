# Build, Flash, and Merge Firmware

This project uses PlatformIO with the Arduino framework. The commands below are
for the supported OpenNextion ESP32-S3 boards.

> **Note for ONX2424G013 users:** The round display uses the ESP32-S3 built-in
> USB-Serial-JTAG (no external UART bridge chip). Uploads work with a single
> command — esptool resets the chip into download mode automatically, no BOOT
> button required. See the [Upload](#upload-from-platformio) section for details.

## Supported PlatformIO Environments

| Environment | Display model | Resolution | Orientation |
| --- | --- | --- | --- |
| `onx3248g035` | ONX3248G035 | 320 x 480 | Portrait |
| `onx2432g028` | ONX2432G028 | 240 x 320 | Portrait |
| `onx2424g013` | ONX2424G013 | 240 x 240 | Round |

## Build

```bash
pio run -e onx2432g028
pio run -e onx3248g035
pio run -e onx2424g013
```

## Clean Build Artifacts

The `clean` target only removes build artifacts. It does not rebuild firmware.

```bash
pio run -e onx2432g028 -t clean
pio run -e onx3248g035 -t clean
pio run -e onx2424g013 -t clean
```

To clean and then rebuild, run the build command after `clean`:

```bash
pio run -e onx2432g028 -t clean
pio run -e onx2432g028

pio run -e onx3248g035 -t clean
pio run -e onx3248g035

pio run -e onx2424g013 -t clean
pio run -e onx2424g013
```

For a deeper clean, remove the board-specific build and dependency directories:

```bash
rm -rf .pio/build/onx2432g028 .pio/libdeps/onx2432g028
rm -rf .pio/build/onx3248g035 .pio/libdeps/onx3248g035
rm -rf .pio/build/onx2424g013 .pio/libdeps/onx2424g013
```

## Upload from PlatformIO

### Rectangular Displays (One-Click Upload)

ONX3248G035 and ONX2432G028 use external USB-UART bridge chips (CH340/CP2102).
Upload is a single command — esptool handles the reset automatically.

```bash
pio run -e onx2432g028 -t upload --upload-port <PORT>
pio run -e onx3248g035 -t upload --upload-port <PORT>
```

Example OpenNextion serial port on macOS:

```bash
pio run -e onx3248g035 -t upload --upload-port /dev/cu.wchusbserial1110
```

### ONX2424G013 — One-Click Upload

The ONX2424G013 uses the ESP32-S3 built-in USB-Serial-JTAG with no external UART
bridge chip. Upload is a single command — esptool resets the chip into download
mode automatically:

```bash
pio run -e onx2424g013 -t upload --upload-port <PORT>
```

**Upgrading from an older firmware?** If the device still runs a firmware build
that holds the USB port as a CDC device (older builds without the
`-DARDUINO_USB_MODE=1` fix), enter download mode manually **once** to upgrade:

1. **Hold BOOT** (GPIO0 button on the back of the board)
2. **Press and release RST (EN)** once
3. **Release BOOT**
4. **Immediately run:**

```bash
pio run -e onx2424g013 -t upload --upload-port <PORT>
```

If you see a `Failed to connect` error, the chip was not in download mode.
Repeat the procedure and make sure you run the command immediately after step 3.

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
pio run -e onx2424g013 -t merge
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
./scripts/merge-firmware.sh --env onx2424g013 -o release/opennextion-esp32-plane-radar-${VERSION}-onx2424g013.bin
```

Skip rebuild if firmware is already built:

```bash
VERSION=v0.1.0
./scripts/merge-firmware.sh --env onx3248g035 --no-build -o release/opennextion-esp32-plane-radar-${VERSION}-onx3248g035.bin
./scripts/merge-firmware.sh --env onx2424g013 --no-build -o release/opennextion-esp32-plane-radar-${VERSION}-onx2424g013.bin
```

## Flash a Merged Release Binary

Merged release binaries should be flashed at address `0x0`.

### Rectangular Displays

```bash
python -m esptool --chip esp32s3 -p <PORT> -b 921600 write_flash \
  0x0 ./opennextion-esp32-plane-radar-v0.1.0-onx3248g035.bin
```

### ONX2424G013

Same as PlatformIO upload — esptool resets the chip automatically, no BOOT
button required. (Only when upgrading from an older firmware that holds the USB
port as a CDC device: enter download mode manually once — **hold BOOT, press
RST, release BOOT**.) Then flash:

```bash
python -m esptool --chip esp32s3 -p <PORT> -b 921600 write_flash \
  0x0 ./opennextion-esp32-plane-radar-v0.1.0-onx2424g013.bin
```

Replace `<PORT>` and the firmware filename for your board.
