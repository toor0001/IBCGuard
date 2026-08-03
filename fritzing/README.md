# Wiring diagram

The project uses a **DFRobot A02YYUW** ultrasonic distance sensor. The sensor is connected through UART and its supply is switched off during deep sleep to reduce battery drain.

## Current connections

| Function | LOLIN32 Lite / connection |
|---|---|
| A02YYUW TX | GPIO16 (ESP32 RX) |
| A02YYUW RX | Leave unconnected / floating for processed output mode |
| A02YYUW GND | GND |
| A02YYUW VCC | Pololu 2810 VOUT |
| Pololu 2810 VIN | 3.3 V |
| Pololu 2810 GND | GND |
| Pololu 2810 ON | GPIO17 |
| Maintenance switch | GPIO13 ↔ GND |

## A02YYUW power switch

Use the **Pololu Mini MOSFET Slide Switch with Reverse Voltage Protection, LV (#2810)**.

For automatic operation, leave the physical slide switch on the Pololu module in the **OFF** position. The LOLIN32 Lite then controls the module through its `ON` input.

```text
LOLIN32 3.3 V -------- VIN   Pololu 2810
LOLIN32 GND   -------- GND   Pololu 2810
GPIO17        -------- ON    Pololu 2810
Pololu VOUT   -------- VCC   A02YYUW
A02YYUW GND   -------- GND
A02YYUW TX    -------- GPIO16
A02YYUW RX    -------- not connected
```

With this wiring:

- **GPIO17 HIGH:** A02YYUW powered on
- **GPIO17 LOW:** A02YYUW powered off
- while the ESP32 is in deep sleep, the A02YYUW remains unpowered

Product reference: [Pololu #2810](https://www.pololu.com/product/2810)

## Maintenance switch

```text
GPIO13 ---- switch ---- GND
```

GPIO13 uses the ESP32 internal pull-up resistor.

- **Switch open:** normal operation / deep sleep enabled
- **Switch closed:** maintenance mode / ESP32 stays awake

All grounds must be connected together.
