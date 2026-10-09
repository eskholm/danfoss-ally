# Danfoss Ally gateway replacement for Home Assistant

This repository provides a Home Assistant automation blueprint and scripts for
Danfoss Ally TRVs connected through Zigbee2MQTT. Create one automation per room
and select that room's thermostats and external temperature sensors.

## Files

- `blueprint/danfoss-ally.yaml`: room automation blueprint.
- `script/danfoss-ally-room-assistant.yaml`: **two** scripts, named
  `danfoss_ally_room_assistant` and `danfoss_ally_time_sync`.

## Installation

1. Copy `blueprint/danfoss-ally.yaml` into
   `config/blueprints/automation/eskholm/danfoss-ally.yaml`.
2. Merge both entries from `script/danfoss-ally-room-assistant.yaml` into
   Home Assistant's `scripts.yaml`. They are already formatted as
   `scripts.yaml` entries: do **not** add another `script:` wrapper there.
   If you manage scripts through the UI, create **both** scripts with those IDs.
3. Reload scripts and automations (or restart Home Assistant).
4. Create or update an automation from the blueprint for each room.
   Select the optional supply temperature sensor if you use `heat_available`
   (for example `sensor.temperature_01` with a threshold of 50 °C).

The blueprint calls `script.danfoss_ally_room_assistant`; the room assistant
starts `script.danfoss_ally_time_sync` independently for each TRV.

## Operation and Zigbee traffic

- Every 10 minutes, each room gets a random 0–119 second offset. Its
  thermostats then receive their external room temperature and average load,
  staggered by another 2–5 seconds per thermostat.
- Once per hour, each thermostat also receives an independent time
  synchronization job delayed by a random 0–3599 seconds. The job does not
  block setpoint or temperature updates. `time` is measured from the UTC
  Zigbee epoch (2000-01-01).
- A setpoint change is synchronized without waiting for the periodic update.
  Thermostats already at that target are skipped; needed writes are separated
  by 1–3 seconds.
- Window-entity changes cause an immediate room-data refresh. The blueprint
  does **not** change `window_open_external` or a window switch.
- Unavailable external temperature sensors are excluded from the mean. If
  none are valid, external temperatures are not published. Missing/invalid
  load estimates are excluded, and an absent room load is sent as `-8000`.
- The optional supply temperature sensor controls each existing
  `switch.*_heat_available` entity only when its state needs changing.

This is per-room pacing, **not** a global Zigbee rate limiter. With many
rooms, traffic can still overlap. Time synchronization jobs are capped at
250 concurrent runs; increase the helper script's `max` if needed.

The scripts retain the original topic lookup convention:
`input_text.<climate_entity_object_id>_base_topic` (default:
`zigbee2mqtt`) and `climate` entity `friendly_name` as the Zigbee2MQTT
device name. Verify these match your Zigbee2MQTT configuration.

Danfoss Ally in radiator-covered / external-room-sensor mode needs a fresh
external temperature within 30 minutes. The periodic 10-minute refresh
provides margin, assuming Home Assistant and Zigbee2MQTT are functioning.

## Migration from the earlier version

Both scripts must be installed together. Previous automation instances should
be reloaded after updating the blueprint. The original repository did not
include `heat_available`; if you use that behavior locally, select the new
optional sensor in the blueprint.

## License

GPL-3.0; see [LICENSE](LICENSE).
