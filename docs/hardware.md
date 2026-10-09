# Hardware

This document lists the tools and components for the course. Prices are approximate, in USD.

**Select your board first.** See [boards.md](boards.md), or ask the AI mentor: `Help me choose a board.` The lists below show the reference board (STM32 Nucleo-F411RE). Replace it with your board and, if your board has no debugger on it, add a debug probe. The modules and tools work with all 3.3 V boards.

## Where to buy

- **Original parts:** DigiKey, Mouser, Farnell, LCSC. Use these suppliers for boards and chips.
- **Cheap modules:** AliExpress. Delivery is slow. Some parts are copies or have wrong labels. Always do a check of the chip ID in software.

## Buy for phase 0 to phase 3

| Item | Purpose | Price |
|---|---|---|
| Your board (reference: STM32 Nucleo-F411RE) | Main board. The reference board has a built-in ST-LINK debugger. | $15 for the reference board |
| Debug probe (only if your board has no debugger on it) | Flash and debug | $5–20 |
| Breadboard (830 points), 2 pieces | Circuits without soldering | $10 |
| Jumper wires (male-male, male-female, female-female) | Connections | $5 |
| Battery holder for 2 × AA, and batteries | Safe 3 V supply for phase 0 | $3 |
| Component kit: resistors (E12 values), LEDs, push buttons, capacitors (ceramic and electrolytic), potentiometers, NPN transistors (2N2222 or BC547), diodes (1N4148, 1N4007) | Basic circuits | $20 |
| Passive buzzer | Sound output with PWM | $1 |
| Digital multimeter (auto-range, with continuity buzzer) | Measure voltage, current, resistance | $25–50 |
| Logic analyzer, 8 channels, 24 MHz (Saleae-compatible clone) | Look at digital signals. **Very important tool.** | $12 |
| Soldering iron (for example Pinecil), solder (lead-free or 63/37), flux, tip cleaner | Solder headers onto modules | $40 |

## Buy for phase 4 and phase 5

| Item | Purpose | Price |
|---|---|---|
| USB-to-UART adapter (CP2102 or FT232, 3.3 V logic) | Second serial port, debug other boards | $5 |
| BME280 module with 6 pins (VCC, GND, SCL, SDA, CSB, SDO) | Temperature, humidity, pressure. The 6-pin type supports I2C and SPI. | $8 |
| SSD1306 0.96" OLED, I2C | Small monochrome display | $5 |
| ST7789 (240 × 240) or ILI9341 (320 × 240) TFT, SPI | Color display | $8 |
| MicroSD card module (SPI) and a microSD card (8–32 GB) | Data logging | $5 |
| DS3231 RTC module with coin cell | Real-time clock | $4 |
| WS2812B LED strip, 8–30 LEDs | DMA timing exercise | $5 |
| SG90 micro servo | PWM position control | $3 |
| Small DC motor and TB6612FNG driver module | Motor control | $7 |
| IMU module (for example MPU-6050) | Accelerometer and gyroscope | $4 |

**Caution:** Many sellers ship a BMP280 with a "BME280" label. The BMP280 has no humidity sensor. The BME280 chip ID is `0x60`. The BMP280 chip ID is `0x58`.

## Buy for phase 8

These items are suggestions for the phase 8 modules. If your main board already has a feature (for example USB on the board, Bluetooth on an nRF52840, or CAN on an STM32F446), you do not need the matching item. Ask the mentor.

| Item | Purpose | Price |
|---|---|---|
| Raspberry Pi Pico 2 (with headers) | Second platform, USB device projects, PIO | $6 |
| Raspberry Pi Debug Probe | SWD debugger and UART for the Pico 2 | $12 |
| ESP32-C3 or ESP32-S3 DevKit | Wi-Fi, Bluetooth LE, CAN (TWAI) | $8–15 |
| 2 × SN65HVD230 CAN transceiver modules (3.3 V) | CAN bus experiments | $6 |

## Optional tools for later

| Item | Purpose | Price |
|---|---|---|
| Oscilloscope, 4 channels (for example Rigol DHO804 or Siglent SDS1104X-E) | Analog signals, noise, rise times | $350–450 |
| Bench power supply with current limit | Safe supply for motors and new boards | $50–100 |
| USB current meter or Nordic Power Profiler Kit II | Measure low-power current | $20–100 |
| Helping hands with magnifier | Soldering | $15 |

## Reference board facts: Nucleo-F411RE

These facts apply to the reference board only. For a different board, the AI mentor records the matching facts in your profile. Find each fact in the board manual (UM1724) and the chip datasheet. Do not trust this table without a check.

| Item | Value |
|---|---|
| MCU | STM32F411RET6, Cortex-M4F, max 100 MHz |
| Flash / SRAM | 512 KB / 128 KB |
| User LED LD2 | PA5 (also Arduino D13 and SPI1 SCK) |
| User button B1 | PC13, low when pressed |
| ST-LINK virtual COM port | USART2: PA2 (TX), PA3 (RX) |
| Arduino I2C pins | PB8 (SCL, D15), PB9 (SDA, D14) |
| MCU current measurement | Jumper JP6 (IDD) |
