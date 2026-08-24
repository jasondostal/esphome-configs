# irrigation-controller

8-zone lawn irrigation controller. ESP32 dev board + 8-channel optoisolated
relay module, driving standard 24VAC solenoid valves off the transformer that
came with the Orbit B-hyve this replaced.

## Hardware

- ESP32 dev board (`esp32dev`)
- 8-channel 5V optoisolated relay module — **active-low** trigger
- 24VAC wall-wart irrigation transformer
- 24V→5V 5A AC-DC converter module (powers the ESP32 from the same transformer)
- 24VAC solenoid valves (existing in-ground)

## Wiring

```
24VAC wall wart -> Wago split:
  branch 1 -> 24V→5V AC-DC converter -> ESP32 5V
  branch 2 -> relay COM terminals (single wire daisy-chained across the back)

ESP32 5V   -> relay board VCC
ESP32 GND  -> relay board GND
ESP32 GPIO -> relay IN1–IN8 (direct; optoisolation handles the level)

relay NO 1–4  -> individual valve wires
24VAC common  -> all valve commons
```

Zone map (4 wired, 4 reserved for expansion):

| Zone | Area | GPIO |
|---|---|---|
| 1 | NW — Driveway | GPIO32 |
| 2 | NE — Front Yard | GPIO33 |
| 3 | SE — Swamp/Firewood | GPIO25 |
| 4 | SW — Backyard | GPIO26 |
| 5–8 | Reserved | GPIO27 / 14 / 12 / 13 |

## Design notes

- Built on ESPHome's native [`sprinkler`](https://esphome.io/components/sprinkler.html)
  component — zone sequencing, auto-advance, per-zone durations, and repeat
  cycles all run **on the device**. Home Assistant only decides *when* a run
  starts; if HA is down, watering still happens.
- Relay GPIOs are `inverted: true` (active-low board) with
  `restore_mode: ALWAYS_OFF` — a power cycle can never boot up with a valve open.
- `api: reboot_timeout: 0s` so the device doesn't reboot itself when no HA is
  connected — it's designed to run standalone.
- `web_server` on port 80 gives full local control from any browser on the LAN.
- Per-zone run durations are exposed as minute-denominated `number` entities
  wrapping `sprinkler.set_valve_run_duration`.
