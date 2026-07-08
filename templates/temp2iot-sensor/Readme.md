# Temp2IoT Sensor

- [Overview](#overview)
- [Requirements](#requirements)
- [Setup](#setup)
- [Template Configuration](#template-configuration)
  - [Host-Level Macros](#host-level-macros)
  - [Items](#items)
  - [Discovery Rules](#discovery-rules)
  - [Triggers](#triggers)
- [References](#references)

## Overview

Zabbix template for monitoring [Temp2IoT](https://github.com/100prznt/Temp2IoT)
ESP-based temperature sensors via HTTP API.

The template auto-discovers all sensors connected to a Temp2IoT device
using Low-Level Discovery (LLD) against the device's `/api` endpoint and
creates one temperature item plus four triggers (warn/crit high, warn/crit
low) per detected sensor.

Firmware version and system name are captured into the host inventory
automatically.

### :chart: Zabbix Monitoring

- One master HTTP item polls the device every minute
- All sensor items derive from the master via dependent-item preprocessing
  (no additional API calls)
- LLD picks up new sensors automatically after they appear in the API response
- Per-sensor thresholds via context macros — no template modifications needed

### Supported checks

- Temperature per sensor (°C, 5-minute rolling average for triggers)
- Firmware version tracking (host inventory)
- System name tracking (host inventory)

## Requirements

- Zabbix Server or Proxy version 7.0 or later
- Temp2IoT device reachable via HTTP from the Zabbix Server or Proxy
- Device `/api` endpoint returns JSON of the form:

```json
  {
    "systemname": "hau-env01",
    "firmware": "1.2.3",
    "sensors": [
      { "name": "rack-intake", "value": 24.5 },
      { "name": "rack-exhaust", "value": 31.2 }
    ]
  }
```

## Setup

1. Import [`template_temp2iot_sensor.yaml`](template_temp2iot_sensor.yaml)
   into Zabbix under *Data collection → Templates → Import*.
2. Create a host in Zabbix for your Temp2IoT device.
3. Set the host's interface to the device's IP or FQDN (agent interface;
   the template uses `{HOST.CONN}` and does not connect via Zabbix agent).
4. Link the template `Temp2IoT Sensor` to the host.
5. Optional: override per-sensor thresholds via host-level context macros
   (see below).
6. Wait for the first poll cycle (~1 minute) or trigger *"Discover Temp2IoT
   Sensors"* → *Execute now* to populate items and triggers immediately.

## Template Configuration

### Host-Level Macros

Global fallback thresholds (defined at template level, adjust globally
per host if desired):

| Macro                | Default | Description                     |
| -------------------- | ------- | ------------------------------- |
| `{$TEMP.HIGH.CRIT}`  | `35`    | Critical high threshold (°C)    |
| `{$TEMP.HIGH.WARN}`  | `30`    | Warning high threshold (°C)     |
| `{$TEMP.LOW.CRIT}`   | `5`     | Critical low threshold (°C)     |
| `{$TEMP.LOW.WARN}`   | `10`    | Warning low threshold (°C)      |

**Per-sensor overrides** use context macros with the sensor name in quotes:

| Macro                              | Example value | Description                          |
| ---------------------------------- | ------------- | ------------------------------------ |
| `{$TEMP.HIGH.CRIT:"sensorname"}`   | `32`          | Critical high, this sensor only      |
| `{$TEMP.HIGH.WARN:"sensorname"}`   | `28`          | Warning high, this sensor only       |
| `{$TEMP.LOW.CRIT:"sensorname"}`    | `4`           | Critical low, this sensor only       |
| `{$TEMP.LOW.WARN:"sensorname"}`    | `8`           | Warning low, this sensor only        |

Sensors without an explicit override use the global fallbacks.

**Example**: rack air-intake tighter than default, exhaust more tolerant:
```json
{$TEMP.HIGH.WARN:"rack-intake"}   = 28
{$TEMP.HIGH.CRIT:"rack-intake"}   = 32
{$TEMP.HIGH.WARN:"rack-exhaust"}  = 45
{$TEMP.HIGH.CRIT:"rack-exhaust"}  = 50
```
### Items

Items collected once per device:

| Name                          | Type        | Description                                    |
| ----------------------------- | ----------- | ---------------------------------------------- |
| API: Get Raw Data             | HTTP agent  | Master item, polls `/api` every 60 seconds     |
| Temp2IoT: Firmware Version    | Dependent   | Firmware version → host inventory (`OS`)       |
| Temp2IoT: System Name         | Dependent   | System name → host inventory (`Name`)          |

Items collected per sensor (via LLD):

| Name                         | Type        | Description                             |
| ---------------------------- | ----------- | --------------------------------------- |
| `{#SENSORNAME}` temperature  | Dependent   | Temperature reading in °C (float)       |

### Discovery Rules

| Name                          | Description                                              |
| ----------------------------- | -------------------------------------------------------- |
| Discover Temp2IoT Sensors     | Discovers all entries in the `sensors[]` array by name.  |

### Triggers

Each trigger is created per discovered sensor. All triggers use a 5-minute
rolling average to avoid flapping on brief spikes. Warning-level triggers
depend on the corresponding critical trigger, so only the higher-severity
alert fires when both thresholds are exceeded.

| Name                                                     | Severity  |
| -------------------------------------------------------- | --------- |
| `{#SENSORNAME}: temperature critical high (>X°C)`        | High      |
| `{#SENSORNAME}: temperature warning high (>X°C)`         | Warning   |
| `{#SENSORNAME}: temperature critical low (<X°C)`         | High      |
| `{#SENSORNAME}: temperature warning low (<X°C)`          | Warning   |

## References

- [Temp2IoT hardware & firmware project](https://github.com/100prznt/Temp2IoT)
- [Zabbix Documentation: HTTP agent](https://www.zabbix.com/documentation/current/en/manual/config/items/itemtypes/http)
- [Zabbix Documentation: LLD](https://www.zabbix.com/documentation/current/en/manual/discovery/low_level_discovery)
- [Zabbix Documentation: User macros with context](https://www.zabbix.com/documentation/current/en/manual/config/macros/user_macros_context)