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

### Smart Fountain Schedule (single switch)

**File:** [`smart_fountain_schedule.yaml`](smart_fountain_schedule.yaml)

Turns a single smart plug / valve ON at a start time and OFF at an end time
(supports schedules that cross midnight). Re-synchronizes state after a
Home Assistant restart. Optional notifications.

**Inputs:**
- Fountain smart plug (single switch)
- Start time / End time
- Optional notifications

> Need to control **several** valves/plugs (e.g. multi-zone irrigation)?
> See the multi-zone blueprint below, or simply create one automation per
> zone using this blueprint if each zone needs its own schedule.

---

### Smart Multi-Zone Adaptive Watering (Sequential)

**File:** [`smart_multi_zone_schedule.yaml`](smart_multi_zone_schedule.yaml)

Full-featured **adaptive irrigation** for **multi-zone** setups, in
**sequential mode** only (one valve open at a time — safe for limited
water pressure). Same intelligence as `Smart Plant Watering v6` but
applied across N valves.

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

**How to choose between the two blueprints:**

| Scenario                                                                                    | Use                                                                       |
| ------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------- |
| 1 single fountain / valve, simple ON/OFF schedule                                           | `smart_fountain_schedule.yaml`                                            |
| Multi-zone irrigation, **adaptive to weather**, **shared water pressure**                   | `smart_multi_zone_schedule.yaml` (always sequential)                      |
| Multi-zone with **different schedules** per zone (e.g. zone A 6h, zone B 18h)               | `smart_fountain_schedule.yaml` — create **one automation per zone**       |

**Quick example — 4 irrigation zones in Belgium (IRM-KMI weather):**

```yaml
Zones (in order):              switch.vanne_pelouse, switch.vanne_potager,
                               switch.vanne_haie, switch.vanne_fruitiers
Water sensors (same order):    sensor.vanne_pelouse_water_consumed, ...
Watering start time:           06:00:00
Weather entity:                weather.irm_kmi_home
Rain today sensor:             sensor.rain_today_mm
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

→ On a hot, dry day after 5 dry days: each zone runs ~16 min (base 10 × ~1.6).
Cycle ≈ 4 × 16 + 3 × 30 s = ~65 min, only one valve open at a time.

→ On a rainy day (6 mm today): cycle is automatically **cancelled**,
dry-days counter is **reset to 0**, and a "🚫 Watering cancelled" notification
is sent with the full weather context. Status helper is updated accordingly.

→ After each zone: a "💧 Zone X/N done" notification reports actual duration
and **liters used** (start sensor value → end sensor value).

→ At the end: a "✅ Watering complete" notification recaps total time,
**total liters**, **per-zone breakdown**, and **cumulative total**.

