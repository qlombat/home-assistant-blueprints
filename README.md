# home-assistant-blueprints

A collection of Home Assistant automation blueprints.

## Blueprints

### Automatic Blinds Management (Sun + MQTT Remote)

**File:** [`auto_blinds_sun_mqtt.yaml`](auto_blinds_sun_mqtt.yaml)

Opens blinds at sunrise and closes them at sunset, with support for an MQTT remote control device. Sun-based triggers can be enabled/disabled. The remote action subtypes are customizable (defaults: `brightness_move_up` / `brightness_move_down`).

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
