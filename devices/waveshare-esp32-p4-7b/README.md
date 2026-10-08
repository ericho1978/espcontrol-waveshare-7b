# Waveshare ESP32-P4-WIFI6-Touch-LCD-7B

EspControl hardware profile for the Waveshare 7B 1024x600 MIPI-DSI panel.
The panel uses an ESP32-P4 main SoC and an ESP32-C6 hosted WiFi/BLE
co-processor connected over SDIO.

## Hardware profile

- ESP32-P4, 360 MHz, 32 MB Flash.
- 1024x600 MIPI-DSI display, mounted with 180-degree rotation.
- GT911 touch controller: SDA GPIO7, SCL GPIO8, reset GPIO22, interrupt GPIO21.
- Backlight PWM: GPIO32, active-low, limited to 80% maximum power.
- ESP32-C6 SDIO: CMD/CLK/D0-D3 = GPIO19/18/14/15/16/17.
- ESP32-C6 reset: GPIO54, active-high, SDIO clock 20 MHz.

## Development

Use `dev.yaml` for local builds. The current phase is offline-only: no COM3
device operation and no automatic firmware update endpoint.

The similarly sized Guition JC1060P470 profile is used as the software/UI
reference only. Its 16 MB flash setting, display model, and backlight pin are
not valid for this Waveshare board.
