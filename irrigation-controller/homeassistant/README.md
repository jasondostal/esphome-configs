# The Home Assistant layer

The device sequences its own zones — HA only decides **when** a run starts and
**whether** to skip it. This directory is that layer, lifted from my running
instance with only the personal entity IDs genericized (`weather.home`,
`notify.mobile_app_your_phone` — substitute your own).

| File | What it is |
|---|---|
| `automations.yaml` | The three automations: a 1 AM forecast check that arms rain-skip when predicted precipitation (today + tomorrow) crosses a threshold, the scheduled run (per-day toggles + start time, honors rain-skip and a manual one-shot skip, notifies either way), and a run-complete notifier that also restores zone-enable state after a partial quick run. |
| `scripts.yaml` | Quick-run scripts (all / front / back / one zone) and stop. The partial runs snapshot the zone-enable switches with `scene.create` first, so your normal zone selection comes back when the run finishes. |
| `helpers.yaml` | YAML equivalents of the helpers everything depends on (day-of-week toggles, rain threshold in inches, start time, skip flags, last-run text). |
| `template_sensors.yaml` | Friendly state text, a computed "next run" preview that walks the day toggles, and a skip-status line — the glue for the dashboard's status hero. |
| `dashboard.yaml` | The Lovelace view: status hero and quick actions on top of a phone screen, schedule and rain status below, controller internals and device info folded away at the bottom. Needs `custom:fold-entity-row` (HACS); everything else is stock cards. |

Two design notes worth stealing:

- **Skip semantics are split.** `irrigation_rain_skip` is armed and cleared
  automatically by the forecast check; `irrigation_skip_next_run` is the
  human's one-shot lever and clears itself after the skipped run. The
  scheduled-run automation honors either and tells you which one fired.
- **Fail toward watering.** If the forecast call errors, the automation stops
  and the skip flag stays untouched — a broken weather integration means the
  lawn gets watered, not the other way around.
