# Solar Assistant / Victron GX MQTT Driver

Hubitat device driver for integrating Solar Assistant and Victron GX devices over MQTT.

This repository contains a single Groovy driver file intended to be imported into Hubitat Elevation. It subscribes to MQTT topics published by your energy system, emits Hubitat events for common measurements, and can publish control messages back to MQTT when configured.

## Features

- Connects Hubitat to Solar Assistant / Victron GX devices using MQTT
- Supports a primary MQTT broker plus an optional fallback broker address
- Exposes power, energy, voltage, battery, temperature, and cell voltage readings in Hubitat
- Supports switch-style control through configurable MQTT topics and values
- Can parse either raw payloads or JSON payloads containing a `value` field
- Averages incoming readings for power and voltage-related metrics

## Requirements

- A Hubitat Elevation hub
- A reachable MQTT broker
- Solar Assistant or a Victron GX device publishing the MQTT topics you want to consume
- The MQTT topic names and payload formats used by your installation

## Installation

1. Open your Hubitat web UI.
2. Go to **Drivers Code**.
3. Create a new driver.
4. Paste in the contents of `user_driver_jeremy_akers_Solar_Assistant_Victron_MQTT_Driver_621.groovy`.
5. Save the driver.
6. Create a device in Hubitat and assign this driver to it.
7. Open the device page and fill in the MQTT settings and topic preferences.
8. Save preferences to trigger the driver update/initialize flow.

## Hubitat capabilities

The driver declares these Hubitat capabilities:

- Initialize
- Notification
- Sensor
- Switch
- PowerMeter
- EnergyMeter
- VoltageMeasurement
- Battery
- TemperatureMeasurement

## Configuration

### MQTT connection

- `MQTTBroker`: primary MQTT broker address
- `MQTTBrokerFallback`: optional fallback MQTT broker address
- `username`: MQTT username, if required
- `password`: MQTT password, if required
- `polltime`: keepalive interval in minutes

### Inbound topics

Configure only the topics your installation actually publishes.

- `keepalive_topic`
- `switch_topic`
- `voltage_topic`
- `min_cell_voltage_topic`
- `max_cell_voltage_topic`
- `power_topic`
- `load_topic`
- `energy_topic`
- `soc_topic`
- `temp_topic`
- `charge_limit`

### Switch control settings

- `switch_on_val`: payload to publish for the on state
- `switch_off_val`: payload to publish for the off state

### Publish / output behavior

- `topicPub`: outbound MQTT publish topic
- `QOS`: MQTT QoS value
- `retained`: whether outbound messages are retained
- `json_payload`: whether inbound payloads are JSON objects with a `value` property

### Averaging and logging

- `count_to_avg`: number of messages to accumulate before averaging
- `logEnable`: enables debug logging

## MQTT topic mapping

These mappings come from the driver's current `parse(...)` logic.

| Preference | Hubitat event / behavior |
|---|---|
| `switch_topic` | updates `switch`; may also reset `power` to `0 W` when disabled |
| `energy_topic` | updates `energy` in `kwh` |
| `power_topic` | updates averaged `power` in `W`; may also drive outbound publish logic |
| `load_topic` | used for internal load-based publish logic |
| `voltage_topic` | updates averaged `voltage` in `V` |
| `min_cell_voltage_topic` | updates averaged `min_cell_voltage` in `V` |
| `max_cell_voltage_topic` | updates averaged `max_cell_voltage` in `V` |
| `soc_topic` | updates `battery` in `%` |
| `temp_topic` | updates `temperature` in `C` |
| `charge_limit` | updates `charge_limit` in `A` |

## Commands and attributes

### Commands

- `poll`
- `disconnect`
- `publishMsg(String)`
- `setMode(number)`
- `set_monitored_battery_soc(number)`
- `updateVersion`
- `logsOff`

The driver also implements `on()` and `off()` through the `Switch` capability.

### Key attributes

- `status`
- `power`
- `energy`
- `voltage`
- `battery`
- `temperature`
- `charge_limit`
- `min_cell_voltage`
- `max_cell_voltage`
- `dwDriverInfo`

## Behavior notes

- The driver uses `initialize()` to connect to MQTT and subscribe to configured topics.
- If the primary broker cannot be connected and `MQTTBrokerFallback` is configured, the driver will try the fallback broker.
- `poll()` publishes an empty keepalive payload when connected, and starts a reconnect cycle when disconnected.
- If `json_payload` is enabled, incoming payloads are expected to be parseable JSON with a `value` field.
- Debug logging auto-disables after a delay via `logsOff()`.

## Limitations

- This repository has no automated tests, linting, or build system.
- The driver is intended for the Hubitat runtime, not standalone Groovy execution.
- Validation is currently manual on a Hubitat hub with a live MQTT broker.

## Repository contents

- `user_driver_jeremy_akers_Solar_Assistant_Victron_MQTT_Driver_621.groovy` — main Hubitat driver
- `AGENTS.md` — repository guidance for coding agents
- `LICENSE` — Apache 2.0 license

## License

Licensed under the Apache License, Version 2.0. See [LICENSE](LICENSE).
