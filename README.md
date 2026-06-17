# home-assistant-blueprints

A collection of Home Assistant automation blueprints.

## Blueprints

### Cover: Open & Close on Sun Schedule + MQTT Remote

**File:** [`cover_open_close_schedule.yaml`](cover_open_close_schedule.yaml)

Automatically open covers at sunrise and/or close them at sunset (each independently toggleable), plus an MQTT remote control with customizable action subtypes.

**Inputs:**
- Covers to control (multi-select)
- Enable/disable sun triggers
- Sunrise & sunset offsets (seconds)
- MQTT remote device
- Open/close action subtypes

---

### Prevent Blinds from Closing if Window is Open

**File:** [`prevent_blind_closing_when_window_open.yaml`](prevent_blind_closing_when_window_open.yaml)

Stops and reopens a blind if it starts closing while the associated window is open.

---

### Aqara JY-GZ-01AQ – Multi-detector Emergency Bundle

**File:** [`aqara_jy_gz_01aq_emergency.yaml`](aqara_jy_gz_01aq_emergency.yaml)

Syncs ringing/muting across Aqara smoke detectors and runs emergency actions (HVAC off, lights on, TTS, notifications, etc.).

---

### Smart Fountain / Multi-Zone Schedule

**File:** [`smart_fountain_schedule.yaml`](smart_fountain_schedule.yaml)

A single blueprint with **two operating modes**. The original fountain
behaviour is fully preserved (and is the default), and a multi-zone
adaptive watering mode has been added on top — no second blueprint needed.

#### Mode 1 — Simple schedule (original fountain behaviour, default)

Turns the selected switch(es) ON at a start time and OFF at an end time
(supports schedules that cross midnight). Re-synchronizes state after a
Home Assistant restart. Optional notifications. **Backward compatible**:
pick a single switch for a classic fountain, or several switches that all
follow the same on/off window.

**Inputs (Mode 1):**
- Fountain / zone switches (one or several)
- Operating mode = *Simple schedule*
- Start time / End time
- Optional notifications

---

#### Mode 2 — Adaptive sequential watering (multi-zone)

Set **Operating mode = *Adaptive sequential watering*** on the same
blueprint to switch the selected switches into a full **adaptive
irrigation** system, in **sequential mode** (one valve open at a time —
safe for limited water pressure). Same intelligence as
`Smart Plant Watering v6` but applied across N valves. The **end time is
ignored** in this mode.

**Adaptive duration:**
The per-zone run-time is computed daily from a base value multiplied by
six weather coefficients, then bounded by safety min/max:

```
duration_per_zone = base_duration × coef_temp × coef_humidity
                                  × coef_wind × coef_dry_days
                                  × coef_rain × coef_forecast
```

| Coefficient | Rule |
|---|---|
| 🌡️ Temperature | +2 % per °C above 22 °C, −2 % per °C below |
| 💧 Humidity   | +1 % per % below 50 % (none above 50 %) |
| 🌬️ Wind       | +5 % per 10 km/h |
| ☀️ Dry days   | +8 % per consecutive day without ≥2 mm rain |
| 🌧️ Rain today | Cancel ≥ 5 mm, partial reduction ≥ 2 mm |
| 🔮 Forecast   | −50 % if rain probability > 70 %, −25 % if > 40 % |

**Features:**
- ⛔ **Auto-cancel** if it has rained enough today, or if computed duration is too short to be useful
- ☀️ Auto-incremented **consecutive dry-days counter** (reset on rainy days)
- 🌡️ **Heatwave label** in notifications (above configurable threshold)
- 💧 **Per-zone water tracking** via SONOFF-SWV-style sensors + grand total + optional **cumulative liters counter**
- 📊 Real-time **status** (running, completed, cancelled, safety-off) via `input_text`
- 📚 **Compact history** of the last 5 sessions via `input_text`
- 🔔 **Five rich notifications**: cycle started (with all weather details), per-zone completion (with measured liters), cycle complete (with full recap), cycle cancelled (with reason and skipped zones), safety-off after HA restart
- ♻️ **HA-restart safety**: all valves forced OFF if HA reboots mid-cycle

**Required Home Assistant helpers:**
| Helper | ID example | Purpose |
|---|---|---|
| `input_number` | `input_number.smart_watering_dry_days` (0–30, step 1) | Consecutive dry-days counter |

**Optional helpers (recommended):**
| Helper | ID example | Purpose |
|---|---|---|
| `input_number` | `input_number.smart_watering_total_liters` (0–99999, step 0.1) | Cumulative liters across all cycles |
| `input_text`   | `input_text.smart_watering_status` (max 255) | Real-time status string |
| `input_text`   | `input_text.smart_watering_history` (max 255) | Last 5 cycles, compact format |

**🌧️ Rain sensor flexibility (since v2):**

The blueprint reads the value of the `rain_sensor` input **as-is** — it
does NOT compute a time window itself. The window is decided by the
sensor you pick. Pair it with `rain_period_label` (a cosmetic string)
so notifications match reality.

| You want…                                  | Pick a sensor that…                                            | `rain_period_label` |
|--------------------------------------------|----------------------------------------------------------------|---------------------|
| Pluie depuis minuit                        | exposes "rain today" (resets at midnight)                      | `today`             |
| **Pluie 12h glissantes** (Smart Plant Watering v6 style) | a Statistics sensor with `max_age: 12h` (recipe below)         | `last 12h`          |
| Pluie 24h glissantes                       | same recipe, `max_age: 24h`                                    | `last 24h`          |
| Désactiver la logique pluie                | leave the field empty (rain coefficient is forced to 1)        | (unused)            |

**12 h sliding window — ready-to-paste recipe** (in `configuration.yaml`):

```yaml
sensor:
  - platform: statistics
    name: "Rain last 12h"
    unique_id: rain_last_12h
    entity_id: sensor.your_cumulative_rain_in_mm   # any cumulative mm sensor
    state_characteristic: change
    max_age:
      hours: 12
```

Then in the blueprint: `rain_sensor = sensor.rain_last_12h`,
`rain_period_label = "last 12h"`.

**Which mode should I use?**

| Scenario                                                                                    | Mode                                                                |
| ------------------------------------------------------------------------------------------- | ------------------------------------------------------------------- |
| 1 single fountain / valve, simple ON/OFF schedule                                           | **Simple schedule** (default)                                       |
| Several switches sharing the same ON/OFF window                                             | **Simple schedule** with multiple switches                          |
| Multi-zone irrigation, **adaptive to weather**, **shared water pressure**                   | **Adaptive sequential watering**                                    |
| Multi-zone with **different schedules** per zone (e.g. zone A 6h, zone B 18h)               | Create **one automation per zone** (Simple schedule or Adaptive)    |

**Quick example — 4 irrigation zones in Belgium (IRM-KMI weather, 12h rain window):**

```yaml
Zones (in order):              switch.vanne_pelouse, switch.vanne_potager,
                               switch.vanne_haie, switch.vanne_fruitiers
Water sensors (same order):    sensor.vanne_pelouse_water_consumed, ...
Watering start time:           06:00:00
Weather entity:                weather.irm_kmi_home
Rain sensor:                   sensor.rain_last_12h     # Statistics 12h
Rain period label:             last 12h
Base duration per zone:        10 min
Max total cycle:               120 min
Min per zone:                  3 min
Pause between zones:           30 s
Dry-days counter:              input_number.smart_watering_dry_days
Cumulative liters:             input_number.smart_watering_total_liters
Current status:                input_text.smart_watering_status
Watering history:              input_text.smart_watering_history
Notifications:                 enabled, notify.mobile_app_my_phone
```

→ On a hot, dry stretch (5 dry days, no rain in last 12h): each zone
runs ~16 min (base 10 × ~1.6). Cycle ≈ 4 × 16 + 3 × 30 s = ~65 min,
only one valve open at a time.

→ If 6 mm of rain has fallen in the **last 12 h** (≥ 5 mm threshold):
the cycle is automatically **cancelled**, the dry-days counter is
**reset to 0**, and a `🚫 Watering cancelled` notification is sent
with full weather context. The status helper is updated accordingly.

→ After each zone: a `💧 Zone X/N done` notification reports actual
duration and **liters used** (start sensor value → end sensor value).

→ At the end: a `✅ Watering complete` notification recaps total time,
**total liters**, **per-zone breakdown**, and **cumulative total**.

