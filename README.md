# ESPHome KinCony M30

ESPHome configuration for the KinCony M30 30-channel energy monitor board.

This repository currently contains a Wi-Fi based main configuration with Home Assistant friendly names, current-driven LED indication profiles, and a small bootstrap config for first-time flashing.

## Repository Contents

- `kincony-m30.yaml`: main ESPHome configuration for daily use
- `m30-bootstrap.yaml`: minimal bootstrap configuration
- `secrets.yaml.example`: template for local secrets

## Current State

The main configuration is tuned for Home Assistant use and exposes:

- `CT 01` through `CT 30` current, power, and energy sensors
- `L1`, `L2`, and `L3` voltage, frequency, total energy, temperature 1, and temperature 2 sensors
- `CT LED Indication` switch
- `Reboot device` button
- `Low Current LED Profile`, `Mid Current LED Profile`, `High Current LED Profile`, and `Alert Current LED Profile`

The 30 physical board LEDs are still used, but the per-channel LED entities are internal only. Home Assistant only sees the shared profile lights.

## LED Behavior

Each CT channel drives one physical indicator LED. The logic currently uses these thresholds:

| Current | Behavior |
| --- | --- |
| `<= 0A` | Off |
| `> 0A` and `<= 4A` | Low profile |
| `> 4A` and `<= 8A` | Mid profile |
| `> 8A` and `<= 12A` | High profile |
| `> 12A` and `<= 16A` | Alert profile, steady |
| `> 16A` | Alert profile, blinking every 500 ms |

Default profile colors and brightness:

- Low: green at `10%`
- Mid: yellow at `10%`
- High: orange at `10%`
- Alert: red at `10%`

Profile defaults are explicitly seeded on first boot and after OTA, which avoids the unreliable "all white at 100%" startup state that can happen when relying on `initial_state` alone.

## Wi-Fi and Secrets

Create a local `secrets.yaml` based on `secrets.yaml.example`:

```yaml
wifi_ssid: "your-wifi-name"
wifi_password: "your-wifi-password"
ota_password: "choose-a-long-ota-password"
ap_password: "choose-a-long-ap-password"
```

The actual `secrets.yaml` is ignored by git and should not be committed.

## Flashing

Important: the M30 needs external `12-24V DC` power. USB alone may not be enough for the board to enumerate correctly over serial.

Validate the main config:

```sh
esphome config kincony-m30.yaml
```

Flash the main config over USB:

```sh
esphome run kincony-m30.yaml --device /dev/tty.usbserial-XXX
```

If you want a minimal first-stage flash, use the bootstrap config:

```sh
esphome run m30-bootstrap.yaml --device /dev/tty.usbserial-XXX
```

## Home Assistant Notes

- Entity naming uses `CT 01` to `CT 30` for the current transformers.
- Shared three-phase values use `L1`, `L2`, and `L3`.
- The main config is Wi-Fi based, not Ethernet based.
- The board's red `RUN` LED is not controlled by this config.

## Known Notes

- ESPHome will warn that `GPIO12` is a strapping pin. That warning is expected because the WS2812 chain is connected there on this board.
- The `secrets.yaml` file is local-only by design.
