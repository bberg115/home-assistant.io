---
layout: page
title: "One wire Sensor"
description: "Instructions on how to integrate One wire (1-wire) sensors into Home Assistant."
date: 2017-09-15 10:10
sidebar: true
comments: false
sharing: true
footer: true
logo: onewire.png
ha_category: DIY
ha_release: 0.12
ha_iot_class: "Local Polling"
---

The `onewire` platform supports sensors which are using the One wire (1-wire) bus for communication.

Supported devices:

- [DS18B20](https://datasheets.maximintegrated.com/en/ds/DS18B20.pdf)
- [DS18S20](https://www.maximintegrated.com/en/products/analog/sensors-and-sensor-interface/DS18S20.html)
- [DS1822](https://datasheets.maximintegrated.com/en/ds/DS1822.pdf)
- [DS1825](https://datasheets.maximintegrated.com/en/ds/DS1825.pdf)
- [DS28EA00](https://datasheets.maximintegrated.com/en/ds/DS28EA00.pdf) temperature sensors
- [DS2406/TAI-8570](https://datasheets.maximintegrated.com/en/ds/DS2406.pdf) Temperature and pressure sensor made by AAG
- [DS2438/B1-R1-A](https://datasheets.maximintegrated.com/en/ds/DS2438.pdf) Temperature, pressure and humidity sensor by AAG

The 1-Wire bus can be connected directly to the IO pins of Raspberry Pi or using dedicated interface adapter (e.g [DS9490R](https://datasheets.maximintegrated.com/en/ds/DS9490-DS9490R.pdf)).

## {% linkable_title Raspberry Pi setup %}

In order to setup 1-Wire support on Raspberry Pi, you'll need to edit `/boot/config.txt` following [this documentation](https://www.waveshare.com/wiki/Raspberry_Pi_Tutorial_Series:_1-Wire_DS18B20_Sensor#Enable_1-Wire).
To edit `/boot/config.txt` on Hass.io use [this documentation](https://developers.home-assistant.io/docs/en/hassio_debugging.html) to enable SSH and edit `/mnt/boot/config.txt` via `vi`.

## {% linkable_title Interface adapter setup %}

### {% linkable_title owfs %}

When an interface adapter is used, sensors can be accessed on Linux hosts via [owfs 1-Wire file system](http://owfs.org/). When using an interface adapter and the owfs, the `mount_dir` option must be configured to correspond a directory, where owfs device tree has been mounted.

### {% linkable_title Units with multiple sensors %}

This platform works with devices with multiple sensors which will cause a discontinuity in recorded values. Existing devices will receive a new ID and therefore show up as new devices.
If you wish to maintain continuity it can be resolved in the database by renaming the old devices to the new names.

Connect to your database using the instructions from [Database section](/docs/backend/database/). Check the names of sensors:

```sql
SELECT entity_id, COUNT(*) as count FROM states GROUP BY entity_id ORDER BY count DESC LIMIT 10;
```
Alter the names of sensors using the following examples:

```sql
UPDATE states SET entity_id='sensor.<sensor_name>_temperature' WHERE entity_id LIKE 'sensor.<sensor_name>%' AND attributes LIKE '%\u00b0C%';
UPDATE states SET entity_id='sensor.<sensor_name>_pressure' WHERE entity_id LIKE 'sensor.<sensor_name>%' AND attributes LIKE '%mb%';
UPDATE states SET entity_id='sensor.<sensor_name>_humidity' WHERE entity_id LIKE 'sensor.<sensor_name>%' AND attributes LIKE '%%%' ESCAPE '';
```

Remember to replace `<sensor_name>` with the actual name of the sensor as seen in the `SELECT` query.

## {% linkable_title Configuration %}

To enable One wire sensors in your installation, add the following to your `configuration.yaml` file:

```yaml
# Example configuration.yaml entry
sensor:
  - platform: onewire
    names:
      some_id: your name
```

{% configuration %}
names:
  description: ID and friendly name of your sensors.
  required: false
  type: string
mount_dir:
  description: Location of device tree if owfs driver used.
  required: false
  type: string
{% endconfiguration %}

