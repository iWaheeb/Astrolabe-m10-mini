
## Basic GPS port according to DS009 standard

| Pin | Signal                           | Connects to                                                        |
| --- | -------------------------------- | ------------------------------------------------------------------ |
| 1   | VCC + 5V                         | TVS Diode (to absorb spikes) -> AP2112 VIN pin (voltage regulator) |
| 2   | TX (OUT, from flight controller) | SAM-M10Q RXD                                                       |
| 3   | RX (IN, to flight controller)    | SAM-M10Q TXD                                                       |
| 4   | I2C SCL                          | IST8310 SCL                                                        |
| 5   | I2C SDA                          | IST8310 SDA                                                        |
| 6   | GND                              | GND                                                                |
