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
  Zigbee epoch (2000-01-01). The time-zone attribute is the
  standard offset without DST; `dstShift` is sent separately.
- Setpoint changes are synchronized without waiting for the periodic update.
  When a thermostat's reported external sensor value differs from the latest
  room temperature by at least 0.1°C, or the sensor feedback state is older
  than 5 minutes, its MQTT request includes both external temperature and
  occupied heating setpoint. The source TRV may receive a temperature-only
  message because it already has the new target. The feedback is normally
  exposed as a `number.*_external_measured_room_sensor` entity associated
  with the same Home Assistant device. When that feedback is missing or
  unavailable, the temperature is sent as a conservative fallback.
- Setpoint events are queued per room and echoed updates that already agree
  with the other TRVs are ignored. Mirrored requests are separated by 1–3
  seconds. A five-second settling interval lets mirrored targets be
  reported before the next queued event is checked. A periodic update
  runs non-blockingly, preserving its 0–119 second jitter without
  blocking setpoints.
- A combined MQTT JSON payload may still produce multiple Zigbee writes in
  Zigbee2MQTT. This is traffic reduction, not a global Zigbee rate limiter.
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

## Following a run in Activity (Logbook)

Every Room Assistant run writes a START and END entry with the same
`[run_id]` (for example `[20261009-205500-123456]`). Entries include the
invocation reason, source/target setpoint, first TRV used as the room key,
mean temperature/load, number of TRVs, and elapsed time. Open **Activity**
(Logbook) in Home Assistant and filter on the first climate entity in
the room. Use the run ID to relate entries in a busy room. A missing END
entry suggests the run was interrupted or failed and should be checked in
the script trace and HA logs; START/END do not confirm physical Zigbee
delivery.

Enable **Detailed run logging** in an individual room's blueprint instance
to additionally record CALC, per-TRV MQTT sends, switch commands and clock
updates. Default is off to avoid unnecessarily growing Recorder. The delayed
clock updates use the originating room run ID, even when they finish much
later. Clock details are only logged when detailed logging was enabled at
the time the job was scheduled.

Room Assistant retains 100 traces, and the high-volume time-sync helper
retains 20. Trace retention is still bounded; Activity log entries follow
the Recorder retention configuration. The Activity logger is affected by
Logbook include/exclude filters. Changing logging does not create MQTT or
Zigbee traffic.

## Migration from the earlier version

Both scripts must be installed together. Previous automation instances should
be reloaded after updating the blueprint. The original repository did not
include `heat_available`; if you use that behavior locally, select the new
optional sensor in the blueprint.

## License

GPL-3.0; see [LICENSE](LICENSE).
