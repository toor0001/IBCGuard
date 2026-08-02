# IBC Rainwater Tank Level Sensor

🌐 **Language:** **English** | [Deutsch](README_DE.md)

[![ESPHome](https://img.shields.io/badge/ESPHome-ESP32-blue)](https://esphome.io/)
[![Home Assistant](https://img.shields.io/badge/Home%20Assistant-compatible-blue)](https://www.home-assistant.io/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

A battery-powered, contactless level sensor for a **1,000 L IBC rainwater tank**, based on a **LOLIN32 Lite (ESP32)**, **VL53L0X Time-of-Flight sensor**, ESPHome and Home Assistant.

> **Project status:** Work in progress. The electronics, ESPHome configuration and current enclosure prototype are available; final IBC calibration and long-term testing are still in progress.

<p align="center">
  <img src="images/ibc1.jpg" alt="Finished IBCLEVEL enclosure" width="48%">
  <img src="images/ibc2.jpg" alt="IBCLEVEL enclosure interior" width="48%">
</p>

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
| LOLIN32 Lite | ESP32 controller with battery support | [Amazon.de](https://link.amazon/B08XCjKmW) |
| VL53L0X Time-of-Flight sensor | Contactless distance measurement | [Amazon.de](https://link.amazon/B0c7CzOGX) |
| 18650 battery holder | Holder for a replaceable 3.7 V 18650 Li-Ion cell | [Amazon.de](https://link.amazon/B01DdEQ1R) |
| JST-PH 2-pin connector / cable | Connection between the battery holder and LOLIN32 Lite; **polarity must be checked before connecting** | [Amazon.de](https://link.amazon/B0gWDZhb8) |
| Slide switch | Physical maintenance / no-sleep mode | [Amazon.de](https://link.amazon/B07UrAODV) |
| Micro-USB extension / panel adapter | External charging, programming and USB access | [Amazon.de](https://link.amazon/B010nKBUo) |
| Heat-set threaded inserts | Threaded mounting points in the 3D-printed enclosure | [Amazon.de](https://link.amazon/B04F2Wwbw) |
| Screws | Mounting the electronics and enclosure components | [Amazon.de](https://link.amazon/B0b1rHkyN) |
| 3D printed enclosure | Mounting on the IBC | [3D files](3d/) |

### Heat-set inserts

The current enclosure uses two insert sizes from the pictured assortment:

- **Electronics / small mounting points:** M2 × 3 × 3.2 mm heat-set inserts.
- **Lid-to-base mounting posts:** M3 × 5 × 5 mm heat-set inserts.

As a practical starting point for the printed holes, use approximately **Ø 3.0 mm × 3.5 mm deep for M2** and **Ø 4.6–4.7 mm × 5.5 mm deep for M3**. Exact fit depends on filament, printer calibration and the actual insert geometry, so test on a small printed sample first.

For PLA/PETG, a temperature around **200–220 °C** is a useful starting range; **210 °C** worked as a sensible starting point for this build. Press the insert in slowly and straight with a soldering iron/heat-set tip, without forcing it. Allow it to cool completely before installing the screw.

### Battery option

A suitable **3.7 V 18650 Li-Ion cell can also be reused from an old or inexpensive mini power bank**, provided it is a standard single-cell 18650 and is in good condition. This can be a convenient source for a cell you may already have available. Only reuse cells that are undamaged and do not show swelling, corrosion, overheating or other signs of deterioration. Verify the cell voltage and condition before use.

### ⚠️ Important: battery connector polarity

**Do not connect the battery until you have verified the polarity.** The 2-pin battery connector on the LOLIN32 Lite may be wired opposite to commonly sold pre-wired JST-PH leads. On the board used for this project, the **`+` terminal is marked directly on the PCB next to the battery socket**; with the standard pre-wired connector used here, that position would otherwise be connected to the **black/negative wire**.

Before plugging the connector into the LOLIN32 Lite, compare the wire positions with the `+` marking on your actual board and, if necessary, swap the contacts in the connector housing or wire the battery holder accordingly. **Never rely on wire color or connector orientation alone. Reversed battery polarity can damage the board.**

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

### Shared GND connection

The LOLIN32 Lite used for this project provides only **one convenient GND pin**, while both the VL53L0X sensor and the maintenance switch need a ground connection. Both must therefore share the same GND.

A simple solution is to make a small **Y-splitter cable** from the LOLIN32 Lite GND pin to the two ground wires. Alternatively, the GND wire from the sensor and the GND wire from the switch can be joined/soldered together and connected to the single GND pin. Insulate the joint properly, for example with heat-shrink tubing, and make sure the connection is mechanically secure.

```text
                 +---- VL53L0X GND
LOLIN32 Lite GND-+
                 +---- maintenance switch
```

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

## Home Assistant & ESPHome setup

The steps below assume a Home Assistant installation that supports add-ons, such as Home Assistant OS or Home Assistant Supervised.

### 1. Install the ESPHome add-on

In Home Assistant, open:

**Settings → Add-ons → Add-on Store**

Search for **ESPHome Device Builder**, install it, then start the add-on. Enabling **Start on boot** is recommended. You can also enable **Show in sidebar** for easier access.

### 2. Create a new ESPHome device

Open ESPHome Device Builder and select **New device**. Create a device for the sensor, for example:

```text
ibc-fuellstand
```

The exact automatically generated YAML is not important because it will be replaced with the project configuration.

### 3. Add your secrets

Open the ESPHome `secrets.yaml` file and add your Wi-Fi credentials plus an API encryption key and OTA password.

Use [`esphome/secrets.example.yaml`](esphome/secrets.example.yaml) as a template:

```yaml
wifi_ssid: "YOUR_WIFI_NAME"
wifi_password: "YOUR_WIFI_PASSWORD"
api_encryption_key: "YOUR_API_ENCRYPTION_KEY"
ota_password: "YOUR_OTA_PASSWORD"
```

Do **not** copy your real `secrets.yaml` into a public GitHub repository.

### 4. Replace the generated YAML

Open the configuration of the newly created ESPHome device and replace its contents with:

**[esphome/ibc-fuellstand.yaml](esphome/ibc-fuellstand.yaml)**

Before installing, check the substitutions at the top of the file and adjust them if necessary:

```yaml
substitutions:
  device_name: ibc-fuellstand
  friendly_name: "IBC Füllstand"
  tank_capacity_l: "1000"
  distance_empty_cm: "100"
  distance_full_cm: "10"
  sleep_time: "15min"
```

The full/empty distances are calibration values and should later be adjusted to your actual tank and installation height.

### 5. Keep the device awake for first setup

For the first installation and testing, put the physical maintenance switch in the **ON** position so that **GPIO13 is connected to GND**. This prevents the ESP32 from entering deep sleep while you are flashing, viewing logs or performing OTA updates.

For the very first sensor check, you can also temporarily change:

```yaml
scan: false
```

to:

```yaml
scan: true
```

under the `i2c:` section. The VL53L0X should normally be detected at address **0x29**.

### 6. Install the firmware on the LOLIN32 Lite

For the first flash, connect the LOLIN32 Lite to the computer/Home Assistant host via USB. In ESPHome Device Builder select **Install** and choose the appropriate USB/serial installation method available in your setup.

After the first successful flash, the device should connect to your Wi-Fi network. Future updates can normally be installed wirelessly using **OTA**, as long as the maintenance switch keeps the device awake.

### 7. Add the ESPHome device to Home Assistant

Once the ESP32 is online, Home Assistant will usually discover it automatically.

Open:

**Settings → Devices & services**

Look for a newly discovered **ESPHome** device named **IBC Füllstand** and select **Configure / Add**.

If it is not discovered automatically, add the **ESPHome** integration manually and enter the device hostname or IP address, for example:

```text
ibc-fuellstand.local
```

or the ESP32's local IP address. The native ESPHome API uses port **6053** by default.

If Home Assistant asks for an encryption key, use the same `api_encryption_key` that you stored in `secrets.yaml`.

### 8. Check the entities in Home Assistant

After the integration has been added, Home Assistant should create the project entities automatically, including:

- **IBC Abstand** — measured distance to the water surface
- **IBC Füllstand** — calculated fill level in percent
- **IBC Wassermenge** — estimated water volume in liters
- **IBC WLAN-Signal** — Wi-Fi signal strength
- **IBC Uptime aktiv** — awake time for the current measurement cycle
- **IBC Wartungsmodus** — physical maintenance-switch state
- **Letzte Entfernungsmessung gültig** — measurement diagnostic
- **ESPHome-Version** — installed ESPHome firmware version

You can now add **IBC Füllstand** and/or **IBC Wassermenge** to any Home Assistant dashboard like normal sensor entities.

### 9. Return to normal deep-sleep operation

Once everything is working:

1. Set `scan: false` again if you enabled I²C scanning.
2. Install the final YAML once more if needed.
3. Open the maintenance switch so GPIO13 is no longer connected to GND.

The ESP32 will then wake up, take its measurements, send the values to Home Assistant and return to deep sleep for the configured interval.

While the ESP32 is sleeping, it is **normal for the ESPHome device to appear temporarily offline**. Home Assistant keeps displaying the last successfully received sensor values.

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

The enclosure has been prepared and test-printed using a **Bambu Lab A1 mini**.

![Current enclosure / slicer preview](images/enclosure-preview.png.png)

### Available files

- **Base:** [`3d/IBCLEVEL_BASE_v38.stl`](3d/IBCLEVEL_BASE_v38.stl)
- **Lid:** [`3d/IBCLEVEL_LID_v26.zip`](3d/IBCLEVEL_LID_v26.zip) — extract the STL before importing it into your slicer.

The lid is provided as a ZIP archive because the original high-resolution STL exceeds GitHub's normal browser upload size limit. The archive contains the unmodified STL geometry.

The enclosure is still considered a prototype until final fit and outdoor testing have been completed. Check dimensions and fit before committing to a final print.

## Power consumption

Low power consumption is a primary goal. Actual battery runtime depends on battery capacity, Wi-Fi signal, connection time, measurement duration, sleep interval and the standby current of the complete electronics. Real-world runtime data will be added after long-term testing.

## Support the project

IBCLEVEL is an open-source hobby project. If you find it useful and would like to support further development, testing and documentation, you can send a voluntary contribution via PayPal.

<a href="https://paypal.me/toor0001">
  <img src="images/support.png" alt="Support this project" width="300">
</a>

Support is completely optional. This is a voluntary contribution and is not presented as a tax-deductible donation.

## Safety and outdoor use

This is a DIY project. Protect the electronics appropriately against rain, condensation and other environmental conditions. A 3D-printed enclosure is not automatically waterproof.

Li-Ion batteries require suitable charging, handling and temperature precautions. Do not use damaged batteries, and always verify battery polarity before connecting them to the board.

## Contributing

Issues, suggestions and improvements are welcome. Feedback from builds using different IBC tanks or mounting arrangements is especially useful.

## License

Source code and documentation are released under the [MIT License](LICENSE), unless otherwise noted. 3D models may receive a separate license in a future revision.

---

Built as a practical DIY solution for monitoring the rainwater available in a garden IBC tank using ESPHome and Home Assistant.