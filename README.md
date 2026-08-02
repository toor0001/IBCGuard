# IBC Rainwater Tank Level Sensor

[![ESPHome](https://img.shields.io/badge/ESPHome-ESP32-blue)](https://esphome.io/)
[![Home Assistant](https://img.shields.io/badge/Home%20Assistant-compatible-blue)](https://www.home-assistant.io/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

A battery-powered, contactless level sensor for a **1,000 L IBC rainwater tank**, based on a **LOLIN32 Lite (ESP32)**, **VL53L0X Time-of-Flight sensor**, ESPHome and Home Assistant.

> **Project status:** Work in progress. The prototype is currently being tested and calibrated on the actual IBC tank. Dimensions, firmware settings and 3D-print files may still change.

## Why this project?

I use a 1,000-liter IBC container in my garden to collect rainwater. The problem is simple: from the outside it is difficult to tell how much water is still available.

This project measures the distance from the top of the IBC to the water surface without touching the water. ESPHome converts that distance into a tank level in percent and an estimated volume in liters and sends the values to Home Assistant.

Because the unit is intended to run from a battery, the ESP32 normally wakes up only for a measurement cycle, publishes the result and returns to deep sleep. A physical maintenance switch keeps it awake when calibration, logs or OTA updates are required.

## Features

- Contactless water-level measurement using a VL53L0X ToF sensor
- ESP32 / LOLIN32 Lite
- Native ESPHome integration with Home Assistant
- Tank level in **%**
- Estimated water volume in **liters**
- Raw distance value for calibration and diagnostics
- Median filtering to reduce measurement outliers
- Battery-oriented deep-sleep operation
- Physical maintenance switch to disable deep sleep
- OTA firmware updates while in maintenance mode
- Wi-Fi signal and diagnostic entities
- Custom 3D-printable enclosure for IBC installation

## How it works

The VL53L0X is mounted above the water surface and measures the distance downwards. Eleven valid measurements are collected and a median value is published. This makes the result less sensitive to individual bad readings.

The configured empty and full distances are then used to calculate the fill percentage:

```text
0 %   = distance measured when the tank is empty
100 % = distance measured when the tank is full
```

The current firmware uses a linear conversion from percentage to liters. This is a useful approximation for an IBC, but a future version may use a calibration curve if required.

## Hardware

| Component | Purpose | Link |
|---|---|---|
| LOLIN32 Lite | ESP32 controller with LiPo support | Amazon affiliate link – coming soon |
| VL53L0X ToF module | Contactless distance measurement | Amazon affiliate link – coming soon |
| LiPo battery | Power supply | Amazon affiliate link – coming soon |
| Slide switch | Physical maintenance / no-sleep mode | Amazon affiliate link – coming soon |
| USB extension / panel adapter | External charging and USB access | Amazon affiliate link – coming soon |
| Heat-set inserts + screws | Internal PCB mounting | Amazon affiliate link – coming soon |
| 3D printed enclosure | Weather-protected mounting on the IBC | See project files – coming soon |

### Affiliate disclosure

Some hardware links in this project may be Amazon affiliate links. If you purchase something through one of these links, I may receive a small commission at no additional cost to you. This helps support further development and documentation of the project.

**Als Amazon-Partner verdiene ich an qualifizierten Verkäufen.**

## Wiring

### VL53L0X → LOLIN32 Lite

| VL53L0X | LOLIN32 Lite |
|---|---|
| VCC | 3.3 V |
| GND | GND |
| SDA | GPIO16 |
| SCL | GPIO17 |

### Maintenance switch

The maintenance switch simply connects **GPIO13 to GND** when enabled.

```text
GPIO13 ---- switch ---- GND
```

The GPIO uses the ESP32 internal pull-up resistor, so no additional external pull-up is required.

- Switch open → normal operation / deep sleep enabled
- Switch closed → maintenance mode / ESP32 remains awake

The sensor and switch may share the same GND connection.

## ESPHome configuration

The current important pin assignments are:

```yaml
i2c:
  sda: GPIO16
  scl: GPIO17

binary_sensor:
  - platform: gpio
    name: "IBC Maintenance Mode"
    id: maintenance_mode
    pin:
      number: GPIO13
      mode:
        input: true
        pullup: true
      inverted: true
```

The complete ESPHome configuration will be provided in the repository after final calibration and testing.

### Current measurement strategy

The VL53L0X currently measures every **500 ms** while the ESP32 is awake.

```yaml
update_interval: 500ms
```

Eleven valid samples are collected:

```yaml
median:
  window_size: 11
  send_every: 11
  send_first_at: 11
```

This results in a new filtered distance value approximately every **5.5 seconds** while running continuously in maintenance mode.

## Deep sleep

The intended normal operating cycle is:

```text
Wake up
   ↓
Start sensor
   ↓
Take multiple distance measurements
   ↓
Calculate median
   ↓
Calculate % and liters
   ↓
Connect / publish to Home Assistant
   ↓
Deep sleep
```

The current planned sleep interval is **15 minutes**.

If the physical maintenance switch is enabled, the deep-sleep command is skipped and the device remains online. This is particularly useful for:

- initial setup
- calibration on the IBC
- live ESPHome logs
- troubleshooting
- OTA firmware updates

## Calibration

Do not blindly use the example distances for another tank. The installation height of the sensor and the geometry of the IBC affect the values.

For accurate results:

1. Install the sensor in its final position on the IBC.
2. Enable maintenance mode.
3. Observe the raw **IBC Distance** value in Home Assistant or ESPHome logs.
4. Determine the distance corresponding to an empty tank.
5. Determine the distance corresponding to a full tank.
6. Enter these values in the ESPHome substitutions.

Example:

```yaml
substitutions:
  tank_capacity_l: "1000"
  distance_empty_cm: "100"
  distance_full_cm: "10"
  sleep_time: "15min"
```

Values outside the calibrated range are clamped to 0–100 %.

## Home Assistant

The device uses the native ESPHome API. Once the ESP32 is running on the same network, Home Assistant should normally discover it automatically.

Typical entities include:

- **IBC Distance**
- **IBC Fill Level**
- **IBC Water Volume**
- **IBC Wi-Fi Signal**
- **IBC Active Uptime**
- **IBC Maintenance Mode**
- **Last Distance Measurement Valid**
- **ESPHome Version**

For a sleeping device, entities remain available in Home Assistant with their last reported values while the ESP32 is offline between measurement cycles.

## First commissioning

For initial testing it is recommended to keep the device awake and enable I²C scanning temporarily:

```yaml
i2c:
  sda: GPIO16
  scl: GPIO17
  scan: true
```

The VL53L0X should normally be detected at address **0x29**.

Before enabling normal deep-sleep operation, verify that:

- the ToF sensor is detected
- distance readings are plausible and stable
- GPIO13 correctly activates maintenance mode
- Home Assistant receives all entities
- OTA updates work
- the final sensor position provides reliable readings of the water surface

After testing, I²C scanning can be disabled again.

## 3D printed enclosure

A custom enclosure is being developed specifically for this project. It accommodates the LOLIN32 Lite, battery, external USB connection, maintenance switch and the sensor installation required for the IBC.

Current design goals include:

- flat printable base
- removable lid
- internal PCB mounting bosses for heat-set inserts
- battery holder/frame
- external USB access
- physical maintenance switch
- sensor mounted in the lid
- geometry suitable for installation on the IBC

STL files will be added after the dimensions have been verified with the final hardware.

## Power consumption

Low power consumption is one of the main goals of this project. Deep sleep dramatically reduces the ESP32's average consumption compared with continuous Wi-Fi operation.

Actual battery life depends on several factors, including battery capacity, Wi-Fi connection time, signal quality, measurement duration, wake interval and the quiescent current of the complete hardware. Real-world battery-life measurements will be added after longer-term testing.

## Support the project

If this project is useful to you and you would like to support further development, documentation and testing, a PayPal support link will be added here.

**PayPal:** coming soon

Support is completely optional. The project itself remains freely available.

## Roadmap

- [x] ESP32 prototype
- [x] VL53L0X distance measurement
- [x] Median filtering
- [x] Home Assistant integration
- [x] Physical maintenance mode
- [x] Deep-sleep logic
- [ ] Final IBC calibration
- [ ] Final enclosure verification
- [ ] Publish ESPHome YAML
- [ ] Publish STL files
- [ ] Battery runtime testing
- [ ] Add build photographs
- [ ] Add wiring diagram
- [ ] Add optional non-linear IBC calibration if useful

## Safety and outdoor use

This is a DIY project. The enclosure and electronics must be protected appropriately against rain, condensation and other environmental conditions. Do not assume that a 3D-printed enclosure is automatically waterproof. Use suitable seals and installation methods for your environment.

LiPo batteries require appropriate charging, handling and temperature precautions. Do not use damaged batteries or expose them to unsuitable environmental conditions.

## Contributing

Issues, suggestions and improvements are welcome. If you build your own version, feedback about different IBC tanks, sensor mounting positions and real-world battery runtime is particularly useful.

## License

Unless otherwise noted, the source code and documentation in this repository are released under the MIT License. See `LICENSE` for details.

3D models may receive a separate license when the final files are published.

---

Built as a practical DIY solution for monitoring the rainwater available in a garden IBC tank using ESPHome and Home Assistant.
