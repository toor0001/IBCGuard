# Wiring diagram

A Fritzing wiring diagram will be added here.

Current connections:

| Function | LOLIN32 Lite |
|---|---|
| VL53L0X VCC | 3.3 V |
| VL53L0X GND | GND |
| VL53L0X SDA | GPIO16 |
| VL53L0X SCL | GPIO17 |
| Maintenance switch | GPIO13 ↔ GND |

The maintenance switch uses the ESP32 internal pull-up resistor.
