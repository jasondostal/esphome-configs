# esphome-configs

The ESPHome YAML behind the devices running in my house. Each device gets its
own directory with the full, as-flashed config — not a trimmed-down demo. If a
device is in here, it's (or was) actually screwed to a wall or buried in a
rafter somewhere doing its job.

Write-ups for most of these live at [ducktyping.dev](https://ducktyping.dev).

## Devices

| Device | What it does | Write-up |
|---|---|---|
| [irrigation-controller](irrigation-controller/) | 8-zone sprinkler controller (ESP32 + relay board), replaced an Orbit B-hyve | coming soon |
| [bme680-air](bme680-air/) | Indoor air node (ESP32-S3 + BME680) — temp, humidity, pressure, VOC gas | coming soon |

More to come — a wall thermostat is on the bench right now.

## Design philosophy

One iron rule runs through every device in this repo: **it has to work without
Home Assistant.** HA enriches — schedules, dashboards, notifications — but it
never gates. The irrigation controller sequences its own zones; the thermostat
will run its own climate loop. If the server rack loses power, the house keeps
working.

## Secrets

Configs reference credentials via `!secret`. Copy `secrets.yaml.example` to
`secrets.yaml` next to the config you're building and fill in your own values.
`secrets.yaml` is gitignored.

## Fair warning

These configs switch real hardware — relay boards, 24VAC valve circuits,
eventually HVAC equipment. They work on **my** wiring with **my** components.
Check your own hardware (relay trigger polarity, transformer ratings, GPIO boot
states) before flashing anything that actuates the physical world.

## License

MIT — see [LICENSE](LICENSE).
