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
