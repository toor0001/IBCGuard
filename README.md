# IBC Rainwater Tank Level Sensor

🌐 **Language:** **English** | [Deutsch](README_DE.md)

[![ESPHome](https://img.shields.io/badge/ESPHome-ESP32-blue)](https://esphome.io/)
[![Home Assistant](https://img.shields.io/badge/Home%20Assistant-compatible-blue)](https://www.home-assistant.io/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

A battery-powered, contactless level sensor for a **1,000 L IBC rainwater tank**, based on a **LOLIN32 Lite (ESP32)**, a waterproof **DFRobot A02YYUW ultrasonic sensor**, ESPHome and Home Assistant.

> **Project status:** Work in progress. The project has been migrated from the VL53L0X to the A02YYUW. The ESPHome configuration and wiring documentation have already been updated; the enclosure/sensor mount may still need mechanical changes for the A02YYUW.

<p align="center">
  <img src="images/ibc1.jpeg" alt="IBCLEVEL enclosure" width="48%">
  <img src="images/ibc2.jpeg" alt="IBCLEVEL enclosure interior" width="48%">
</p>

## Quick links

- [ESPHome configuration](esphome/ibc-fuellstand.yaml)
- [Example secrets file](esphome/secrets.example.yaml)
- [3D-print files](3d/)
- [Current wiring](fritzing/README.md)
- [License](LICENSE)

## Why this project?

It is difficult to tell from the outside how much rainwater remains in an IBC tank. The A02YYUW is mounted at the top and measures the distance to the water surface without touching the water. ESPHome converts this distance into a fill level in percent and an estimated volume in liters, then publishes the values to Home Assistant.

The enclosure with the sensor is simply placed over the existing lid opening instead of the original IBC lid. This means no permanent modifications to the IBC itself are required.

The ESP32 is battery powered. During normal operation it wakes up, powers the A02YYUW, collects multiple UART measurements, publishes the median result and returns to deep sleep. The ultrasonic sensor is fully powered down through a ready-made **Pololu Mini MOSFET Slide Switch LV #2810** while the ESP32 sleeps so it does not waste battery power.

## Features

- Waterproof contactless distance measurement with **DFRobot A02YYUW**
- LOLIN32 Lite / ESP32
- UART communication at 9600 baud
- Median filtering of multiple valid measurements
- Fill level in **%** and estimated water volume in **liters**
- Native ESPHome + Home Assistant integration
- Battery-oriented deep-sleep operation
- Hardware power switching for the A02YYUW using **Pololu 2810**
- Physical maintenance switch
- OTA updates in maintenance mode
- Wi-Fi and diagnostic entities

## Hardware

| Component | Purpose | Link |
|---|---|---|
| LOLIN32 Lite | ESP32 controller with battery support | [Amazon.de](https://link.amazon/B08XCjKmW) |
| DFRobot A02YYUW | Waterproof ultrasonic distance measurement | [Amazon.de](https://link.amazon/B0dWRfbC4) |
| **Pololu Mini MOSFET Slide Switch with Reverse Voltage Protection, LV #2810** | Fully powers down the A02YYUW during deep sleep | [Pololu](https://www.pololu.com/product/2810) |
| 18650 battery holder | Holder for a replaceable 3.7 V 18650 Li-Ion cell | [Amazon.de](https://link.amazon/B01DdEQ1R) |
| JST-PH 2-pin connector / cable | Connection between battery holder and LOLIN32 Lite; **check polarity before connecting** | [Amazon.de](https://link.amazon/B0gWDZhb8) |
| Slide switch | Physical maintenance / no-sleep mode | [Amazon.de](https://link.amazon/B07UrAODV) |
| Micro-USB extension / panel adapter | External charging, programming and USB access | [Amazon.de](https://link.amazon/B010nKBUo) |
| Heat-set threaded inserts | Threaded mounting points in the 3D-printed enclosure | [Amazon.de](https://link.amazon/B04F2Wwbw) |
| Screws | Mounting electronics and enclosure parts | [Amazon.de](https://link.amazon/B0b1rHkyN) |
| 3D printed enclosure | Mounting on the IBC | [3D files](3d/) |

### Affiliate disclosure

Some links above are Amazon affiliate links. As an Amazon Associate I earn from qualifying purchases, at no additional cost to you.

## A02YYUW

The A02YYUW operates from **3.3–5 V**, outputs measurements through UART at **9600 baud**, and draws only a few milliamps. Even so, leaving it permanently powered would waste a significant amount of energy in a device that spends most of its time asleep.

The **A02YYUW RX pin is left unconnected**. This selects the processed/stable output mode. The sensor's **TX pin** is connected to the ESP32 RX input.

## Wiring

### A02YYUW and Pololu 2810 → LOLIN32 Lite

For automatic operation, leave the physical slide switch on the Pololu module in the **OFF** position. The ESP32 then controls the sensor using the module's `ON` input.

```text
LOLIN32 3.3 V -------- VIN   Pololu 2810
LOLIN32 GND   -------- GND   Pololu 2810
GPIO17        -------- ON    Pololu 2810
Pololu VOUT   -------- VCC   A02YYUW
A02YYUW GND   -------- GND
A02YYUW TX    -------- GPIO16
A02YYUW RX    -------- leave unconnected
```

This gives the following behavior:

- **GPIO17 HIGH:** A02YYUW powered on
- **GPIO17 LOW:** A02YYUW powered off
- during deep sleep the A02YYUW remains unpowered

The current wiring is also documented in [`fritzing/README.md`](fritzing/README.md).

### Maintenance switch

```text
GPIO13 ---- switch ---- GND
```

GPIO13 uses the ESP32 internal pull-up resistor.

- **Switch open:** normal operation / deep sleep enabled
- **Switch closed:** maintenance mode / ESP32 stays awake

All grounds must be connected together.

## ESPHome

The current configuration is available here:

**[esphome/ibc-fuellstand.yaml](esphome/ibc-fuellstand.yaml)**

Important pins:

```yaml
substitutions:
  a02_rx_pin: GPIO16
  a02_power_pin: GPIO17

# Maintenance switch:
# GPIO13 -> switch -> GND
```

### Measurement strategy

Normal operation roughly follows this cycle:

```text
Wake up
→ power A02YYUW on
→ short settling time
→ collect multiple valid UART measurements
→ calculate median
→ publish distance, percent and liters
→ power A02YYUW off
→ deep sleep
```

In maintenance mode the ESP32 remains awake and continuously generates fresh measurements. The default sleep interval is currently **15 minutes**.

## Setup

1. Install **ESPHome Device Builder** in Home Assistant.
2. Create a device named `ibc-fuellstand`.
3. Use [`esphome/secrets.example.yaml`](esphome/secrets.example.yaml) as a template for Wi-Fi, API and OTA credentials.
4. Replace the generated YAML with [`esphome/ibc-fuellstand.yaml`](esphome/ibc-fuellstand.yaml).
5. For first setup, close the maintenance switch so GPIO13 is connected to GND and the ESP32 stays awake.
6. Flash the firmware over USB first.
7. Add the ESPHome device to Home Assistant and verify the raw distance values.

## Calibration

Adjust the example values to your actual installation:

```yaml
substitutions:
  tank_capacity_l: "1000"
  distance_empty_cm: "100"
  distance_full_cm: "10"
  sleep_time: "15min"
```

Mount the sensor in its final position and note the measured distance for a known full and empty tank. Values outside the calibrated range are clamped to 0–100%.

## Home Assistant

Typical entities include:

- **IBC Abstand** — distance to the water surface
- **IBC Füllstand** — calculated fill level
- **IBC Wassermenge** — estimated water volume
- **IBC WLAN-Signal** — Wi-Fi signal strength
- **IBC Uptime aktiv** — active awake time
- **IBC Wartungsmodus** — maintenance-switch state
- **Letzte Entfernungsmessung gültig** — measurement diagnostic
- **ESPHome-Version** — installed ESPHome version

While the ESP32 is in deep sleep, the ESPHome device will temporarily appear offline. Home Assistant keeps the last successfully reported values.

### Example device view

![IBC level sensor in Home Assistant](images/home-assistant-device.png)

## 3D printed enclosure

The existing enclosure was originally designed for the VL53L0X. After changing to the A02YYUW, the sensor opening/mount in the lid must be checked and may need to be redesigned.

Current files:

- **Base:** [`3d/IBCLEVEL_BASE_v38.stl`](3d/IBCLEVEL_BASE_v38.stl)
- **Lid:** [`3d/IBCLEVEL_LID_v26.zip`](3d/IBCLEVEL_LID_v26.zip)

These files should therefore currently be treated as **prototype files**.

## Battery and safety

A suitable 3.7 V 18650 Li-Ion cell can be used as long as it is in good condition. Do not use damaged, swollen or overheated cells.

**Always verify JST connector polarity before connecting the battery.** The LOLIN32 Lite battery connector may be wired opposite to common pre-wired JST-PH leads. Never rely only on wire colors or connector orientation.

Protect the electronics and battery from rain, condensation and other environmental exposure. A 3D-printed enclosure is not automatically waterproof.

## Support

[![Buy me a coffee via PayPal](https://img.shields.io/badge/☕_Buy_me_a_coffee-via_PayPal-0070BA?logo=paypal&logoColor=white)](https://paypal.me/toor0001)

## Contributing

Issues, suggestions and improvements are welcome. Feedback from builds using other IBC tanks or mounting arrangements is especially useful.

## License

Source code and documentation are released under the [MIT License](LICENSE) unless otherwise noted. A separate license may be added for the 3D models later.