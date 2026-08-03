# Wiring diagram

The project uses a **DFRobot A02YYUW** ultrasonic distance sensor. The sensor is connected through UART and its supply is switched off during deep sleep to reduce battery drain.

## Complete wiring overview

```text
                    +-------------------+
                    |   LOLIN32 Lite    |
                    |                   |
              3.3 V o------------------------------+
                    |                              |
               GND  o--------------------+         |
                    |                    |         |
            GPIO16  o<-------------------|---------|----- A02YYUW TX
                    |                    |         |
            GPIO17  o--------------------|---- ON  |  Pololu 2810 LV
                    |                    |         |
            GPIO13  o----o/ o------------+         |
                    |   Wartungsschalter           |
                    |        |                     |
                    |        +---------------------+
                    |                              |
                    +-------------------+          |
                                                   |
                                                   v
                                             +-----------+
                                             | Pololu    |
                                             | 2810 LV   |
                                             |           |
LOLIN32 3.3 V ------------------------------> VIN       |
LOLIN32 GND  -------------------------------> GND       |
GPIO17 -------------------------------------> ON        |
                                             |           |
                                             | VOUT -----+------> A02YYUW VCC
                                             +-----------+

LOLIN32 GND ----------------------------------------------> A02YYUW GND
A02YYUW TX -----------------------------------------------> GPIO16
A02YYUW RX ------------------------------------------------ not connected

Wartungsschalter:
GPIO13 ---- Schalter ---- GND
```

## Connection table

| Component / pin | Connect to |
|---|---|
| A02YYUW VCC | Pololu 2810 `VOUT` |
| A02YYUW GND | LOLIN32 Lite `GND` |
| A02YYUW TX | LOLIN32 Lite `GPIO16` |
| A02YYUW RX | Leave unconnected / floating |
| Pololu 2810 VIN | LOLIN32 Lite `3.3 V` |
| Pololu 2810 GND | LOLIN32 Lite `GND` |
| Pololu 2810 ON | LOLIN32 Lite `GPIO17` |
| Maintenance switch terminal 1 | LOLIN32 Lite `GPIO13` |
| Maintenance switch terminal 2 | LOLIN32 Lite `GND` |

## A02YYUW power switch

Use the **Pololu Mini MOSFET Slide Switch with Reverse Voltage Protection, LV (#2810)**.

For automatic operation, leave the physical slide switch on the Pololu module in the **OFF** position. The LOLIN32 Lite then controls the module through its `ON` input.

- **GPIO17 HIGH:** A02YYUW powered on
- **GPIO17 LOW:** A02YYUW powered off
- while the ESP32 is in deep sleep, the A02YYUW remains unpowered

Product reference: [Pololu #2810](https://www.pololu.com/product/2810)

## Maintenance switch

The maintenance switch is independent of the Pololu module and is connected directly between **GPIO13 and GND**.

- **Switch open:** normal operation / deep sleep enabled
- **Switch closed:** maintenance mode / ESP32 stays awake

GPIO13 uses the ESP32 internal pull-up resistor, so no additional resistor is required for the maintenance switch.

## Important

All ground connections must be common. The LOLIN32 Lite GND, Pololu GND, A02YYUW GND and maintenance-switch GND all belong to the same electrical ground.
