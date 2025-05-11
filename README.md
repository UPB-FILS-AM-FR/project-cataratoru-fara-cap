# Doom Port for ESP32S3

| | |
|-|-|
|`Vaireanu Cristian Gabriel` |
## Description
The scope of this project is to provide me and other makers who might want to reproduce this with a learning oportunity
regarding microcontrolers, ESP32, and emulating/porting retro games on microcontrolers.

## Motivation
One of my favourit games franchises of all time is DOOM, I've known for a long time about the "Can it run doom" hacking
trend. This project was the perfect oportunity for me to hop on

## Architecture
The Nano ESP32 features the ESP32-S3 system on a chip (SoC) from Espressif, which is embedded in the NORA-W106 module. 
The ESP32-S3 has a dual-core microprocessor Xtensa® 32-bit LX7, and has support for the 2.4 GHz Wi-Fi® band as well 
as Bluetooth® 5. The operating voltage of this SoC is 3.3 V.


### Block diagram

![Block Diagram](schematics/block_diagram.png)

### Schematic
    +-------------------------------------------------------------------------------------------------+
    |                                   ASCII WIRING DIAGRAM                                          |
    |                           Arduino Nano ESP32 S3, Display, Joystick, Buttons                     |
    +-------------------------------------------------------------------------------------------------+

  BREADBOARD POWER RAILS:

                          +---------------------------------+
                          |     ARDUINO NANO ESP32 S3       |
                          |                                 |
                          |             [ESP32-S3]          |
                          |               [USB]             |
                          |                                 |
                          |  3V3  o-------------------------D (to 3.3V Rail)
                          |  GND  o-------------------------G (to GND Rail)
                          |                                 |
                          |   A0  o----(JY_VRx)             | -----> To Joystick VRx
                          |   A1  o----(JY_VRy)             | -----> To Joystick VRy
                          |                                 |
                          |   D4  o----(JY_SW)              | ----> To Joystick SW
                          |   D5  o----(B1_S)               | ----> To Button 1 Signal
                          |   D6  o----(B2_S)               | ----> To Button 2 Signal
                          |                                 |
                          |   D8  o----(LCD_RS)             | ----> To LCD RS/DC
                          |   D9  o----(LCD_RST)            | ----> To LCD RST
                          |  D10  o----(LCD_CS)             | ----> To LCD_CS (Display)
                          |  D11  o----(SPI_MOSI)           | ----> To LCD_SDA & SD_MOSI
                          |  D13  o----(SPI_SCK)            | ----> To LCD_CLK & SD_SCK
                          | (D12 o----(SPI_MISO)        )   | ----> (To SD_MISO, if using SD)
                          | (D7  o----(SD_CS_PIN)       )   | ----> (Example: To SD_CS, if using SD)
                          |                                 |
                          +---------------------------------+


    +-----------------------------+        +---------------------------+       +----------+                 +----------+
    |   SPI LCD (ST7735S based)   |        |        JOYSTICK           |       | BUTTON 1 |                 | BUTTON 2 |
    |                             |        |                           |       |  (BTN1)  |                 |  (BTN2)  |
    | VCC   o---------------------D        | VCC (lbl +5V) o-----------D       |          |                 |          |
    | GND   o---------------------G        | GND           o-----------G       | Sig o-----(B1_S from D5)
    | CLK   o----(SPI_SCK from D13)        |                           |       |          |                 | Sig o-----(B2_S from D6)
    | SDA   o----(SPI_MOSI from D11)       | VRx           o----(JY_VRx to A0) | Gnd o-----G                | Gnd o-----G
    | RS/DC o----(LCD_RS from D8)          | VRy           o----(JY_VRy to A1) +----------+                 +----------+
    | RST   o----(LCD_RST from D9)         | SW            o----(JY_SW to D4)
    | CS    o----(LCD_CS from D10)         +---------------------------+
    |                             |
    | -- SD Card Slot --
    | SD_CS   o--(To D7 or other free pin)  (Unique CS for SD Card)
    | SD_MOSI o--(To SPI_MOSI / D11)        (Shares MOSI with LCD)
    | SD_SCK  o--(To SPI_SCK / D13)         (Shares SCK with LCD)
    | SD_MISO o--(To SPI_MISO / D12)        (MISO line for SD Card data out)
    +-----------------------------+


    ----------------------------------------------------------------------------------------------------
WIRING NOTES:

1.  **Power:**
    * Connect Arduino `3V3` to the (+) 3.3V breadboard rail.
    * Connect Arduino `GND` to the (-) GND breadboard rail.
    * LCD `VCC` connects to the (+) 3.3V rail. LCD `GND` connects to the (-) GND rail.
    * Joystick `VCC` (often labeled +5V on module) should first be attempted with the (+) 3.3V rail.
        * If joystick performance is poor on 3.3V, use an external 5V supply for the joystick's
        *  VCC and implement voltage dividers (e.g., 1.8kΩ & 3.3kΩ) for the `VRx` and `VRy` signals
        *   before they connect to Arduino `A0`/`A1`.
    * Joystick `GND` connects to the (-) GND rail.

2.  **LCD (Display Specific):**
    * `CLK` (or `SCK`) to Arduino `D13`.
    * `SDA` (or `MOSI`) to Arduino `D11`.
    * `RS` (or `D/C`) to Arduino `D8`.
    * `RST` to Arduino `D9`.
    * `CS` (Chip Select for display) to Arduino `D10`.

3.  **Joystick:**
    * `VRx` (X-axis analog) to Arduino `A0`.
    * `VRy` (Y-axis analog) to Arduino `A1`.
    * `SW` (digital switch) to Arduino `D4`. (In code, use `pinMode(D4, INPUT_PULLUP);`).

4.  **Buttons:**
    * Button 1: One leg (`Sig`) to Arduino `D5`, the other leg (`Gnd`) to the (-) GND rail.
    (In code, use `pinMode(D5, INPUT_PULLUP);`).
    * Button 2: One leg (`Sig`) to Arduino `D6`, the other leg (`Gnd`) to the (-) GND rail.
    *  (In code, use `pinMode(D6, INPUT_PULLUP);`).

5.  **SD Card (If your LCD module has a slot and you intend to use it):**
    * The SD card and LCD can share the SPI bus lines `SCK` (D13) and `MOSI` (D11).
    * `SD_MISO` (Master In, Slave Out for SD card) connects to Arduino `D12` (this is the ESP32's CIPO/MISO line).
    * `SD_CS` (Chip Select for SD card) **must connect to a separate digital pin** on the Arduino
    * (e.g., `D7` as shown, or any other free digital pin). Each SPI device needs its own unique CS pin.

6.  **General:**
    * This is a conceptual layout. Use jumper wires for actual connections on your breadboard.
    * Double-check all connections against the Arduino Nano ESP32 S3 official pinout before powering on.
    * The pins `A0`, `A1`, `D4`, `D5`, `D6`, `D7`, `D8`, `D9`, `D10`, `D11`, `D12`, `D13` refer to the Arduino
    * pin mapping for the ESP32 S3.
----------------------------------------------------------------------------------------------------

### Components


<!-- This is just an example, fill in with your actual components -->

| Device | Usage | Price |
|--------|--------|-------|
| ESP-32-S3 NANO           |  Chip                 | [100 RON](roboRomania.ro) |
| Buttons                  | Buttons               | [6 RON](roboRomania.ro)  |
| Jumper Wires             | Connecting components | [15 RON](roboRomania.ro) |
| Breadboard               | Project board         | [26 RON](roboRomania.ro) |
| Joystick                 | Joystick              | [18 RON](roboRomania.ro) |
| LCD SPI TFT 128X160      | Display               | [120 RON](roboRomania.ro) |

### Libraries
TBD

## Log

<!-- write every week your progress here -->
### Week 29 Apr - 5 May
### Week 6 - 12 May

### Week 7 - 19 May

### Week 20 - 26 May

## Reference links

[Tutorial 1](https://www.hackster.io/naveenbskumar/yes-arduino-nano-esp32-can-play-doom-ccfde1)

[Article 1](https://www.explainthatstuff.com/induction-motors.html)

[Link title]((https://docs.espressif.com))
