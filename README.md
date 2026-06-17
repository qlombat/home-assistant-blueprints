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

### Smart Plant Watering (Multi-Zone)

**File:** [`smart_plant_watering.yaml`](smart_plant_watering.yaml)
**Guide:** [`GUIDE_COMPLET.md`](GUIDE_COMPLET.md)

Adaptive, weather-based irrigation — the multi-zone evolution of *Smart
Plant Watering v6*. Drives **several valves sequentially** (one zone at a
time, in the order selected), so only one valve is ever open — safe when
your water supply can't feed multiple zones at once.

**Adaptive per-zone duration:**

```
duration_per_zone = base_duration_minutes
                  × coef_temp × coef_humidity × coef_wind
                  × coef_dry_days × coef_rain × coef_forecast
```

| Coefficient | Rule |
|---|---|
| 🌡️ Temperature | +2 % per °C above normal, −2 % below |
| 💧 Humidity   | +1 % per % below 50 % |
| 🌬️ Wind       | +5 % per 10 km/h |
| ☀️ Dry days   | +8 % per consecutive dry day |
| 🌧️ Rain       | Cancel ≥ `rain_cancel_threshold`, partial reduction ≥ `rain_reduction_threshold` |
| 🔮 Forecast   | −50 % if rain probability > 70 %, −25 % if > 40 % |

**Features:**
- ⛔ Auto-cancel if it rained enough or computed time is too short (counter & status still updated)
- ☀️ Consecutive dry-days counter (reset on rainy days, +1 otherwise)
- 💧 Per-zone + total water tracking (SONOFF SWV style) + optional cumulative liters
- 📊 Real-time status + 📚 compact 5-session history (via `input_text`)
- 🔔 Rich notifications: started (full weather breakdown), per-zone (measured liters), complete (per-zone recap + total), cancelled (reason), restart safety-off
- ♻️ HA-restart safety: all valves forced OFF (a sequence can't be resumed)

**Migrating from v6 (single-zone):** input names are kept compatible. The
key differences:

| v6 (single)            | Multi-zone                                   |
| ---------------------- | -------------------------------------------- |
| `watering_switch`      | `watering_switches` (select one **or** more) |
| `water_volume_sensor`  | `water_volume_sensors` (one per zone, same order) |
| `base_duration_minutes`| now **per zone**                             |
| `max_duration_minutes` | now the **total** cap (all zones summed)     |
| `min_duration_minutes` | per-zone minimum (below → whole cycle cancelled) |

Existing helpers (`input_number.smart_watering_dry_days`,
`input_number.smart_watering_total_liters`, `input_text.smart_watering_status`,
`input_text.smart_watering_history`) are reused as-is.

**Rain window:** `rain_sensor_today` is read **as-is** — pick a "rain today"
sensor, or a Statistics sensor with `max_age: 12h` for a sliding window, and
set `rain_period_label` accordingly (cosmetic, shown in notifications).

