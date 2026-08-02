# IBC Rainwater Tank Level Sensor

[![ESPHome](https://img.shields.io/badge/ESPHome-ESP32-blue)](https://esphome.io/)
[![Home Assistant](https://img.shields.io/badge/Home%20Assistant-compatible-blue)](https://www.home-assistant.io/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

A battery-powered, contactless level sensor for a **1,000 L IBC rainwater tank**, based on a **LOLIN32 Lite (ESP32)**, **VL53L0X Time-of-Flight sensor**, ESPHome and Home Assistant.

> **Project status:** Work in progress. The electronics, ESPHome configuration and current enclosure prototype are available; final IBC calibration and long-term testing are still in progress.

![Current enclosure prototype](images/enclosure-preview.png.png)

## Quick links

- [ESPHome configuration](esphome/ibc-fuellstand.yaml)
- [Example secrets file](esphome/secrets.example.yaml)
- [3D-print files](3d/)
- [Wiring / Fritzing section](fritzing/)
- [License](LICENSE)

## Why this project?

I use a 1,000-liter IBC container in my garden to collect rainwater. From the outside, however, it is difficult to tell how much water is still available.

The VL53L0X is mounted at the top of the tank and measures the distance to the water surface without touching the water. ESPHome converts that distance into a fill level in percent and an estimated volume in liters and publishes the values to Home Assistant.

The ESP32 is intended to run from a battery. In normal operation it wakes up, takes several measurements, publishes a filtered result and returns to deep sleep. A physical maintenance switch keeps it awake for calibration, logs and OTA updates.

## Features

- Contactless VL53L0X Time-of-Flight measurement
- LOLIN32 Lite / ESP32
- Native ESPHome + Home Assistant integration
- Fill level in **%** and estimated volume in **liters**
- Raw distance value for calibration
- 11-sample median filter
- Battery-oriented deep-sleep operation
- Physical maintenance switch
- OTA updates in maintenance mode
- Wi-Fi and diagnostic entities
- Custom 3D-printable enclosure

## Hardware

| Component | Purpose | Link |
|---|---|---|
| LOLIN32 Lite | ESP32 controller with LiPo support | [Amazon.de](https://link.amazon/B08XCjKmW) |
| VL53L0X Time-of-Flight sensor | Contactless distance measurement | [Amazon.de](https://link.amazon/B0c7CzOGX) |
| 3.7 V LiPo / Li-Ion battery | Power supply; select a suitable battery for your build and verify connector polarity before use | — |
| Slide switch | Physical maintenance / no-sleep mode | [Amazon.de](https://link.amazon/B07UrAODV) |
| Micro-USB extension / panel adapter | External charging, programming and USB access | [Amazon.de](https://link.amazon/B010nKBUo) |
| Heat-set threaded inserts | Threaded mounting points in the 3D-printed enclosure | [Amazon.de](https://link.amazon/B04F2Wwbw) |
| Screws | Mounting the electronics and enclosure components | [Amazon.de](https://link.amazon/B0b1rHkyN) |
| 3D printed enclosure | Mounting on the IBC | [3D files](3d/) |

### Affiliate disclosure

Some links above are Amazon affiliate links. As an Amazon Associate I earn from qualifying purchases, at no additional cost to you.

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

```text
GPIO13 ---- switch ---- GND
```

GPIO13 uses the internal pull-up resistor.

- **Switch open:** normal operation / deep sleep enabled
- **Switch closed:** maintenance mode / ESP32 stays awake

The sensor and maintenance switch can share the same GND connection.

A Fritzing project and exported wiring diagram are planned in the [`fritzing`](fritzing/) directory.

## ESPHome

The complete current configuration is available here:

**[esphome/ibc-fuellstand.yaml](esphome/ibc-fuellstand.yaml)**

Create your own `secrets.yaml` using [esphome/secrets.example.yaml](esphome/secrets.example.yaml) as a template. Never commit your real Wi-Fi password, API encryption key or OTA password to a public repository.

Important pin assignments:

```yaml
i2c:
  sda: GPIO16
  scl: GPIO17

# Physical maintenance switch
# GPIO13 -> switch -> GND
```

### Measurement strategy

The VL53L0X measures every **500 ms** while the ESP32 is awake. Eleven valid values are collected and the median is published, so in continuous maintenance mode a new filtered value is produced approximately every **5.5 seconds**.

## Deep sleep

Normal operation:

```text
Wake up → measure → median filter → calculate % / liters
        → publish to Home Assistant → deep sleep
```

The default sleep interval is currently **15 minutes**.

With the physical maintenance switch enabled, deep sleep is skipped. This keeps Wi-Fi, ESPHome API and OTA available for calibration and troubleshooting.

## Calibration

The example values in the YAML must be adjusted to the actual installation:

```yaml
substitutions:
  tank_capacity_l: "1000"
  distance_empty_cm: "100"
  distance_full_cm: "10"
  sleep_time: "15min"
```

For best results, install the sensor in its final position, enable maintenance mode and record the raw distance for a known full and empty state. Values outside the calibrated range are clamped to 0–100%.

## Home Assistant

Home Assistant should normally discover the device automatically through the ESPHome integration. Typical entities include distance, fill level, water volume, Wi-Fi signal, active uptime, maintenance mode and measurement-valid diagnostics.

Because the device sleeps between measurement cycles, it is normal for the ESPHome API to be unavailable while the ESP32 is asleep. Home Assistant retains the last reported sensor values.

### Example device view

The screenshot below shows the current ESPHome device in Home Assistant, including the calculated fill level and water volume as well as diagnostic values.

![IBC level sensor in Home Assistant](images/home-assistant-device.png)

## First commissioning

For the first test, close the maintenance switch so GPIO13 is connected to GND. The ESP32 will remain awake. Temporarily enable I²C scanning:

```yaml
i2c:
  sda: GPIO16
  scl: GPIO17
  scan: true
```

The VL53L0X should normally appear at address **0x29**. Once the sensor, Home Assistant integration and OTA updates have been verified, set `scan: false` again.

## 3D printed enclosure

A custom enclosure was designed for the project and is being test-fitted with the real hardware. The current design provides space for the LOLIN32 Lite, battery, external USB connection and physical maintenance switch. The VL53L0X is mounted in the lid and measures downward into the IBC.

The enclosure has been prepared and test-printed using a **Bambu Lab A1 mini**. The current slicer preview is shown at the top of this README.

### Available files

- **Base:** [`3d/IBCLEVEL_BASE_v38.stl`](3d/IBCLEVEL_BASE_v38.stl)
- **Lid:** [`3d/IBCLEVEL_LID_v26.zip`](3d/IBCLEVEL_LID_v26.zip) — extract the STL before importing it into your slicer.

The lid is provided as a ZIP archive because the original high-resolution STL exceeds GitHub's normal browser upload size limit. The archive contains the unmodified STL geometry.

The enclosure is still considered a prototype until final fit and outdoor testing have been completed. Check dimensions and fit before committing to a final print.

## Power consumption

Low power consumption is a primary goal. Actual battery runtime depends on battery capacity, Wi-Fi signal, connection time, measurement duration, sleep interval and the standby current of the complete electronics. Real-world runtime data will be added after long-term testing.

## Support the project

If this project is useful to you and you would like to support further development and documentation, a PayPal support link will be added here.

**PayPal:** coming soon

## Roadmap

- [x] ESP32 prototype
- [x] VL53L0X distance measurement
- [x] Median filtering
- [x] Home Assistant integration
- [x] Physical maintenance mode
- [x] Deep-sleep logic
- [x] Publish ESPHome YAML
- [x] Publish example secrets file
- [x] Publish current STL files
- [x] Add enclosure preview
- [x] Add Home Assistant screenshot
- [ ] Final IBC calibration
- [ ] Final enclosure verification
- [ ] Battery runtime testing
- [ ] Add build photographs
- [ ] Add Fritzing wiring diagram

## Safety and outdoor use

This is a DIY project. Protect the electronics appropriately against rain, condensation and other environmental conditions. A 3D-printed enclosure is not automatically waterproof.

LiPo batteries require suitable charging, handling and temperature precautions. Do not use damaged batteries.

## Contributing

Issues, suggestions and improvements are welcome. Feedback from builds using different IBC tanks or mounting arrangements is especially useful.

## License

Source code and documentation are released under the [MIT License](LICENSE), unless otherwise noted. 3D models may receive a separate license in a future revision.

---

Built as a practical DIY solution for monitoring the rainwater available in a garden IBC tank using ESPHome and Home Assistant.
