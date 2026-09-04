# bme680-air

Indoor air node — temperature, humidity, barometric pressure and VOC gas
resistance from a Bosch BME680, plus dew point, sea-level pressure and a rough
IAQ index computed on-device.

Two configs live here, for two different boards:

| Config | Board | Status |
|---|---|---|
| [`bme680-air-s3.yaml`](bme680-air-s3.yaml) | ESP32-S3-DevKitC-1 style, N16R8 | **In use** |
| [`bme680-air.yaml`](bme680-air.yaml) | Seeed XIAO ESP32C3 | Shelved — see below |

## Hardware

- ESP32-S3 module, N16R8 (16MB flash, 8MB **octal** PSRAM, CH343 USB-UART bridge)
- BME680 breakout — **I²C address `0x77`** on this one; ESPHome's `bme680`
  platform defaults to `0x76`, so check yours

## Wiring

```
BME680 VCC -> 3V3      (NOT 5V)
BME680 GND -> GND
BME680 SDA -> GPIO8
BME680 SCL -> GPIO9
BME680 SDO/CS -> leave floating (I²C mode; SDO sets the address)
```

These boards silkscreen the real GPIO number next to every pin, so there's no
position-counting.

**Pins to avoid on an R8 board:**

| Pins | Why |
|---|---|
| GPIO33–37 | Octal PSRAM. Using them looks like random crashes, not a wiring fault |
| GPIO26–32 | SPI flash |
| GPIO19/20 | Native USB D−/D+ |
| GPIO43/44 | UART0 → the CH343 (your serial console) |
| GPIO0/45/46 | Strapping pins; GPIO46 is input-only |
| GPIO48 | Onboard RGB LED on most of these boards |

## Design notes

- **`hardware_uart: UART0` is set explicitly, and it matters.** ESPHome's
  *default* on the ESP32-S3 is `USB_SERIAL_JTAG` — the board's *native* USB jack.
  These boards have two USB-C ports; if you're driving the CH343 one (as here)
  and leave the default alone, the node runs perfectly while the serial console
  sits dead silent after the ROM bootloader messages. Confirm what you actually
  got with `esphome config <yaml> | grep -A8 '^logger:'`.
- **`flash_size: 16MB`** under `esp32:` — without it ESPHome assumes 4MB and you
  lose most of the partition space.
- `i2c: scan: true` prints found addresses in the boot log. That is how the
  `0x77`/`0x76` mismatch gets caught; the failure mode is otherwise a silent
  `bme680.sensor marked FAILED: unspecified` with no readings at all.
- `web_server` runs with `local: true`, embedding its JS/CSS in flash rather
  than pulling from a CDN — the UI works with the WAN down, same rule as
  everything else in this repo.
- Dew point (Magnus), sea-level pressure and IAQ are all `template` sensors with
  on-device lambdas. Nothing here needs Home Assistant to compute a value.

## Reading the gas sensor

The BME680's gas element is a **metal-oxide (MOX)** film on a micro-hotplate held
at 320 °C. Reducing gases adsorb onto the film and lower its resistance, so:

**Higher resistance = cleaner air. Lower = more VOCs.** It's inverted.

Three things this sensor is not:

1. **It is not a CO₂ sensor.** Nothing in a BME680 measures CO₂. "eCO₂" numbers
   from MOX sensors are *inferred* from VOCs and correlate only because humans
   exhale both. For real CO₂ you need an NDIR sensor (SCD40/SCD41).
2. **It is not selective.** One number covers all reducing gases — cooking,
   solvents, alcohol, perfume, off-gassing plastics, breath. It cannot tell you
   *which*.
3. **It is not comparable between sensors.** Every die has its own baseline.
   Only the *change* against this unit's own history means anything.

New sensors also need burn-in — Bosch says ~48h powered. This one read 1.0 kΩ at
first power-up and was still climbing through 20 kΩ four hours later, against a
settled indoor baseline that should land in the tens-to-hundreds of kΩ.

The `IAQ` sensor here is the community gas+humidity heuristic (0 = clean,
500 = foul), **not** Bosch BSEC — that needs the closed-source blob via
`bme68x_bsec2_i2c` and days of its own calibration. Until burn-in finishes the
reading sits pinned near 500 because gas resistance is below the formula's floor.
**The `gas_floor` / `gas_ceiling` constants in the lambda must be recalibrated to
whatever this unit actually settles at in clean air**, or the number stays
useless. There's a `TODO` on them.

## The XIAO ESP32C3 config, and why it's shelved

`bme680-air.yaml` was the original target and works fine as a sensor — but the
**XIAO ESP32C3 has no onboard antenna at all.** Unlike the XIAO C6 and S3, Seeed
routes the radio only to the u.FL/IPEX-1 connector; the bundled external antenna
is mandatory, not an upgrade. Without it the node *sees* SSIDs at around −82 dB
from stray PCB pickup but never completes an association:

```
Connecting to 'SSID' (…) … attempt 2/2
Connecting to network failed (callback)
Retry phase: SCAN_CONNECTING -> RESTARTING
```

That looks exactly like wrong credentials or a MAC allowlist. It is neither.
Swapping to a board with an antenna, same SSID and same credentials, associated
on the first try.

Replacements must be **IPEX-1 / u.FL**, not IPEX-4/MHF4 (smaller, incompatible).

Two other XIAO C3 notes, both the *inverse* of the S3 board:

- Its USB-C port **is** the chip's USB-Serial/JTAG peripheral — there's no bridge
  — so the logger there needs `hardware_uart: USB_SERIAL_JTAG` explicitly.
- I²C on the XIAO is D4 = GPIO6 (SDA) and D5 = GPIO7 (SCL).
