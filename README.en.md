# EspControl Waveshare 7B

[中文文档](README.md) | English

This repository provides a Waveshare ESP32-P4-WIFI6-Touch-LCD-7B hardware profile and reproducible ESPHome examples based on [EspControl](https://github.com/jtenniswood/espcontrol).

The goal is to preserve EspControl's web configuration, Home Assistant integration, and card system while maintaining a separate hardware profile for the Waveshare 7B and gradually adding verified home-automation use cases.

## Status

Completed:

- Added an independent Waveshare 7B hardware profile.
- Added support for the ESP32-P4, 32 MB flash, and 1024x600 MIPI-DSI display.
- Integrated the GT911 touchscreen, GPIO32 backlight, and the ESP32-C6 SDIO Wi-Fi/BLE coprocessor.
- Completed offline configuration checks and ESP-IDF builds based on upstream v2.11.2.
- Ported the cover-art button, icon, and text compensation required for the rotated 7-inch display.
- Preserved reproducible build records and a separate hardware-support branch.
- Created an isolated upstream v2.12 update branch for review.

Planned:

- Review and retain the Waveshare-specific changes while upgrading to upstream v2.12.0.
- Add Chinese firmware strings and font support.
- Document Home Assistant camera, doorbell snapshot, and TTS examples.
- Document ESPHome Device Builder as a build and management option.
- Define separate home 7-inch and commercial 4.3-inch product profiles.

Not yet considered stable features in this branch:

- Voice wake word, STT, and two-way intercom from the current product line.
- H264/MJPEG playback and video transport logic.
- Real-device flashing and long-duration hardware acceptance testing.

## Hardware Profile

| Item | Specification |
| --- | --- |
| Main MCU | ESP32-P4, 360 MHz |
| Flash | 32 MB |
| Display | 1024x600 MIPI-DSI |
| Touch | GT911, SDA GPIO7, SCL GPIO8, RESET GPIO22, INT GPIO21 |
| Backlight | GPIO32, active-low, maximum power limited to 80% |
| Wi-Fi/BLE | ESP32-C6 over SDIO |
| SDIO | CMD/CLK/D0-D3 = GPIO19/18/14/15/16/17 |
| C6 reset | GPIO54, active-high |
| SDIO clock | 20 MHz |

## Repository Layout

```text
devices/waveshare-esp32-p4-7b/   Waveshare 7B ESPHome profile
dev-docs/stages/                  Stage records and build evidence
components/                       EspControl and hardware components
common/                           Shared device configuration
product/v2/translations/          Firmware translation sources
```

## Local Build

The local development entry point is:

```text
devices/waveshare-esp32-p4-7b/dev.yaml
```

Create an untracked `secrets.yaml` in the profile directory:

```yaml
wifi_ssid: "your-wifi-name"
wifi_password: "your-wifi-password"
```

Example commands:

```powershell
esphome config devices/waveshare-esp32-p4-7b/dev.yaml
$env:NINJAFLAGS = "-j3"
esphome compile devices/waveshare-esp32-p4-7b/dev.yaml
```

The project uses ESPHome `2026.9.1` for the validated build environment. Check `.github/esphome.env` before building. The current phase is offline-build focused; do not run `esphome run` until the hardware preflight and rollback gates have passed.

## ESPHome Device Builder

ESPHome Device Builder can be used as a build and device-management entry point, while this Git repository remains the source of truth.

```text
GitHub repository -> Device Builder -> compile/download/OTA -> Home Assistant
```

Never commit real Wi-Fi, RTSP, Home Assistant, or SSH credentials.

## Branches and Upstream

- `main`: personal project stable line.
- `feature/waveshare-p4-7b-support`: Waveshare 7B hardware support line.
- `feature/upstream-v2.12`: isolated upstream update and compatibility review.
- `feature/chinese-firmware`: planned Chinese firmware translation line.

Remotes:

```text
origin   https://github.com/ericho1978/espcontrol-waveshare-7b.git
upstream https://github.com/jtenniswood/espcontrol.git
```

Upstream changes must be reviewed and validated on an isolated branch. Do not directly overwrite the Waveshare 7B profile or the current product features.

## Chinese Localization

Firmware strings are sourced from:

```text
product/v2/translations/strings.en.txt
```

The planned Chinese translation will use a separate `strings.zh-cn.txt` source and then be generated through the project build script:

```powershell
python scripts/build.py i18n
```

Do not edit the generated `components/espcontrol/i18n_generated.h` directly. Chinese font resources also require separate flash, memory, and display verification.

## License and Attribution

This repository contains code and resources from EspControl. Keep and comply with the repository `LICENSE` and `NOTICE` files. The upstream project uses the PolyForm Noncommercial License 1.0.0; commercial deployment or sale requires the appropriate permission.

Upstream project:

- https://github.com/jtenniswood/espcontrol
- https://jtenniswood.github.io/espcontrol/

## Security Boundary

Do not commit:

- `secrets.yaml`
- Real Wi-Fi passwords
- Camera RTSP usernames or passwords
- Home Assistant, SSH, or Docker credentials
- Device backups containing real configuration
- Unsanitized runtime logs or screenshots

This repository is a reproducible hardware-adaptation and productization case study. It does not claim that every feature has passed real-device acceptance testing.
