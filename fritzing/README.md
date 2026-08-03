# Wiring diagram

The project now uses a **DFRobot A02YYUW** ultrasonic distance sensor instead of the VL53L0X.

The A02YYUW is connected through UART and its supply is switched off during deep sleep to reduce battery drain.

## Current connections

| Function | LOLIN32 Lite / connection |
|---|---|
| A02YYUW TX | GPIO16 (ESP32 RX) |
| A02YYUW RX | Leave unconnected / floating for processed output mode |
| A02YYUW GND | GND |
| A02YYUW VCC | Switched 3.3 V |
| Sensor power control | GPIO17 |
| Maintenance switch | GPIO13 ↔ GND |

## Recommended sensor power switch

Use a **P-channel MOSFET high-side switch**, for example an **AO3401A**, with a pull-up resistor on the gate so the sensor remains off while the ESP32 is in deep sleep.

```text
LOLIN32 Lite 3.3 V ----+--------- Source  AO3401A
                       |
                      100k
                       |
GPIO17 ----------------+--------- Gate
                                  Drain -------- A02YYUW VCC

LOLIN32 Lite GND ------------------------------ A02YYUW GND
A02YYUW TX ------------------------------------ GPIO16
A02YYUW RX ------------------------------------ not connected
```

With this circuit:

- **GPIO17 LOW:** A02YYUW powered on
- **GPIO17 HIGH:** A02YYUW powered off
- **GPIO17 floating during deep sleep:** the 100 kΩ pull-up keeps the MOSFET off

A ready-made low-voltage high-side MOSFET/load-switch PCB can also be used instead of the discrete AO3401A circuit. It must work from approximately 3.3 V and accept a 3.3 V logic control input.

## Maintenance switch

```text
GPIO13 ---- switch ---- GND
```

GPIO13 uses the ESP32 internal pull-up resistor.

- **Switch open:** normal operation / deep sleep enabled
- **Switch closed:** maintenance mode / ESP32 stays awake

All grounds must be connected together.
