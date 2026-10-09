# Phase 4: Serial Protocols

**Time:** 4–5 weeks
**Prerequisites:** Phase 3
**Hardware:** Your board (reference: Nucleo-F411RE), logic analyzer, USB-to-UART adapter, BME280 module, SSD1306 OLED, ST7789 or ILI9341 TFT
**Board:** The board details in this phase (pins, registers, manual sections, tool commands) are for the reference board. On a different board, translate them with your board documents, or ask the AI mentor. See [../boards.md](../boards.md).

## Goal

At the end of this phase, you can:

- Write interrupt-driven UART, I2C, and SPI drivers with registers.
- Read a device datasheet and write a driver for the device.
- Decode each protocol with the logic analyzer and compare it with the datasheet.
- Separate a bus driver from a device driver, and test the device driver on your PC.

## Reading

| Source | Part |
|---|---|
| SparkFun tutorials | "Serial Communication", "I2C", "Serial Peripheral Interface (SPI)" |
| RM0383 | Sections: "Universal synchronous asynchronous receiver transmitter (USART)", "Inter-integrated circuit (I2C) interface", "Serial peripheral interface (SPI)" |
| Noviello, *Mastering STM32* | The chapters about UART, I2C, and SPI |
| Bosch BME280 datasheet | All sections. Special attention: memory map, compensation formulas, measurement modes |
| SSD1306 datasheet | Command table, addressing modes |
| ST7789 or ILI9341 datasheet | Command list, the 4-line serial interface, memory write |
| NXP UM10204 "I2C-bus specification and user manual" | Reference for I2C timing |

## Topics

### 4.1 Use the logic analyzer

- Connect the analyzer GND to the board GND first.
- Set the sample rate to at least 4 times (better 10 times) the signal frequency.
- In PulseView, add a protocol decoder (UART, I2C, SPI). Set the decoder parameters to match your configuration.
- Use a trigger on the first edge to capture a short event.
- Toggle a spare GPIO pin in your code to mark events in the capture ("debug pin").

### 4.2 UART

- Two wires: TX and RX, plus GND. **Connect TX of one device to RX of the other device.**
- No clock line. Both sides must use the same baud rate (for example 115200), the same data bits (8), parity (none), and stop bits (1): "8N1".
- Frame: start bit (low), 8 data bits (LSB first), stop bit (high). The line is high when idle.
- On the Nucleo board, USART2 (PA2, PA3, AF7) connects to the ST-LINK virtual COM port. You do not need the USB-to-UART adapter for USART2.
- Baud rate register: with oversampling by 16, `USARTDIV = f_PCLK / (16 × baud)`. `BRR` holds the integer part and a 4-bit fraction. USART2 uses the APB1 clock.
- Flags: `TXE` (transmit data register empty), `TC` (transmission complete), `RXNE` (receive data register not empty), `ORE` (overrun: a byte was lost).
- **Interrupt-driven design:**
  - Transmit: `uart_write()` puts bytes in a TX ring buffer and enables the `TXE` interrupt. The handler takes bytes from the buffer and disables the interrupt when the buffer is empty.
  - Receive: the `RXNE` handler puts each byte in an RX ring buffer. The main loop reads the buffer.
- **`printf` over UART:** newlib calls `_write(int fd, const char *buf, int len)`. Implement it with your UART driver. Add `-u _printf_float` to the linker flags if you print `float` values. Note: `printf` uses much flash and stack. Look at the map file.

### 4.3 I2C

- Two wires: SCL (clock) and SDA (data). Both are **open drain**. A **pull-up resistor** on each line pulls it high (typically 4.7 kΩ). Many modules have pull-ups on the board.
- Each device has a 7-bit address. The BME280 uses `0x76` or `0x77`. The SSD1306 uses `0x3C` or `0x3D`.
- Transaction:
  1. **START:** SDA goes low while SCL is high.
  2. Address byte: 7-bit address + R/W bit.
  3. The receiver sends **ACK** (pulls SDA low) or **NACK**.
  4. Data bytes, each followed by ACK or NACK.
  5. **STOP:** SDA goes high while SCL is high.
- **Read a register:** write the register address, then a **repeated START**, then read the data. The last read byte gets a NACK.
- Speeds: 100 kHz (standard mode), 400 kHz (fast mode).
- **Clock stretching:** a slow device can hold SCL low.
- **Bus recovery:** if a reset occurs during a transfer, a device can hold SDA low. Fix: configure SCL as a GPIO, toggle it up to 9 times until SDA goes high, then send a STOP.
- The STM32F4 I2C peripheral needs a strict sequence of events (`SB`, `ADDR`, `TXE`, `BTF`, `RXNE`). The read of `SR1` then `SR2` clears `ADDR`. Follow the sequence diagrams in RM0383 exactly.
- Nucleo pins: I2C1 on PB8 (SCL) and PB9 (SDA), AF4.

### 4.4 SPI

- Four wires: SCK (clock), MOSI (controller out), MISO (controller in), CS (chip select, active low). One CS line for each device.
- Full duplex: for each byte sent, one byte is received.
- **Mode** = clock polarity (CPOL) and clock phase (CPHA). Mode 0: CPOL=0, CPHA=0 (most common). The device datasheet gives the mode.
- Much faster than I2C (MHz range). No addresses, no ACK.
- Control CS with a GPIO pin in software. This is simpler than hardware CS.
- **Pin conflict:** SPI1 SCK uses PA5, the LD2 pin. Use SPI2 (PB13 SCK, PB14 MISO, PB15 MOSI) or remap SPI1 SCK to PB3.

### 4.5 Device drivers

- **Layer 1, bus driver:** `i2c_write()`, `i2c_read_reg()`, `spi_transfer()`. Knows the peripheral registers. Knows nothing about the device.
- **Layer 2, device driver:** `bme280_init()`, `bme280_read()`. Knows the device registers. Uses the bus through an ops table (exercise 1.8).
- Advantages: the device driver works on I2C and SPI, and you can test it on your PC with a fake bus.
- Every driver function returns an error code. Use timeouts in all wait loops. **Never write `while (!(I2C1->SR1 & I2C_SR1_SB));` without a timeout.**

### 4.6 BME280

- Read the chip ID (register `0xD0`). It must be `0x60`.
- Read the calibration data (trimming parameters) once at startup.
- Configure oversampling and mode (`ctrl_hum`, `ctrl_meas`, `config`). **Write `ctrl_hum` before `ctrl_meas`.** The change to `ctrl_hum` takes effect only after a write to `ctrl_meas`.
- Read the raw data in one burst from `0xF7` to `0xFE`.
- Use the integer compensation formulas from the datasheet. Test them on your PC with the example values.

### 4.7 SSD1306 OLED

- 128 × 64 pixels, 1 bit per pixel. A full frame buffer needs `128 × 64 / 8 = 1024` bytes.
- The display memory has 8 "pages". Each page is 8 rows high. One byte is a vertical column of 8 pixels.
- Each I2C transfer starts with a control byte: `0x00` for commands, `0x40` for data.
- Draw in a RAM frame buffer. Send the full buffer to the display when the frame is complete.
- Fonts: store a small bitmap font (for example 5 × 7) as a `const` array in flash.

### 4.8 ST7789 / ILI9341 TFT

- SPI with an extra **D/C** pin: low = command, high = data.
- Colors in RGB565: 16 bits per pixel.
- A 240 × 240 frame needs 115 200 bytes. This is almost all SRAM. Do not use a full frame buffer. Set a window (column and row address commands), then send the pixels for the window.
- Phase 5 sends pixels with DMA.

## Exercises

### Exercise 4.1: UART transmit with polling

1. Configure USART2 at 115200 8N1.
2. Send "Hello" every second. Read it in Tera Term on the ST-LINK COM port.
3. Capture the TX line with the logic analyzer. Decode it. Measure the bit time. Compare it with `1 / 115200`.

### Exercise 4.2: Interrupt-driven UART

1. Use the ring buffer from exercise 1.6 for TX and RX.
2. Implement `_write()` so that `printf` works.
3. Echo every received byte.
4. Paste a long text into the terminal. Do you lose bytes? Count overrun errors.

### Exercise 4.3: Command line

1. Read characters into a line buffer until Enter.
2. Split the line into words. Look up the first word in a command table (`name`, `help text`, function pointer).
3. Commands: `help`, `led on|off`, `blink <ms>`, `uptime`, `reg <address>` (read a 32-bit register and print it in hex).
4. Support Backspace.

### Exercise 4.4: I2C scanner

1. Write a polling I2C driver for I2C1 at 100 kHz.
2. Send the address byte to each address from `0x08` to `0x77`. Print each address that sends ACK.
3. Connect the BME280 and the SSD1306. The scanner must find both.
4. Capture one transaction. Find the START, the address, the R/W bit, the ACK, and the STOP.
5. Disconnect the pull-up resistors (if the module allows it) or use a module with no pull-ups. What do you see?

### Exercise 4.5: BME280 driver

1. Write the device driver against the ops table.
2. Write host unit tests with a fake bus. Use the datasheet example values for the compensation.
3. Print the temperature, humidity, and pressure every second over UART.
4. Change to 400 kHz. Capture again.

### Exercise 4.6: SSD1306 driver

1. Initialize the display with the command sequence from the datasheet.
2. Write `fb_clear()`, `fb_pixel()`, `fb_char()`, `fb_text()`, `fb_flush()`.
3. Show the sensor values on the display.

### Exercise 4.7: SPI

1. Write a polling SPI driver for SPI2.
2. Connect the BME280 over SPI (CSB to a GPIO, SDO to MISO). Read the chip ID. Reuse the BME280 device driver with an SPI bus ops table.
3. Capture the SPI transfer. Find the mode and the clock frequency.

### Exercise 4.8: TFT driver

1. Initialize the ST7789 or ILI9341.
2. Fill the screen with colors. Measure how long a full-screen fill takes. Calculate the theoretical time from the SPI clock.
3. Draw a graph of the last 100 temperature values.

### Exercise 4.9 (optional): I2C bus recovery

1. Reset the MCU in the middle of a read (press the reset button many times while it reads).
2. Find a state where SDA stays low. Implement and test the bus recovery from topic 4.3.

## Project: Data logger, stage 1

See [../projects/data-logger.md](../projects/data-logger.md), stage 1.

## Done criteria

- [ ] My UART driver uses interrupts and ring buffers. `printf` works.
- [ ] My command line has a command table.
- [ ] I can read an I2C capture and an SPI capture without the decoder.
- [ ] My BME280 driver works on I2C and SPI and has host unit tests.
- [ ] The OLED shows the sensor values.
- [ ] All wait loops in my drivers have timeouts.

## Common problems

| Problem | Cause |
|---|---|
| The terminal shows wrong characters. | Wrong baud rate. `BRR` uses the wrong clock frequency. |
| No UART data. | TX and RX are swapped, the GPIO alternate function is wrong, or the grounds are not connected. |
| The I2C scanner finds no devices. | No pull-ups, wrong pins or alternate function, or the module has no power. |
| I2C stops after a reset. | A device holds SDA low. Do a bus recovery. |
| The chip ID is `0x58`. | The module has a BMP280, not a BME280. |
| SPI reads only `0xFF` or `0x00`. | Wrong SPI mode, CS is not low, or MISO is not connected. |
| LD2 behaves strangely when SPI1 runs. | PA5 is SPI1 SCK and the LD2 pin. |

## Next

[Phase 5: ADC, DMA, and low power](phase-5-adc-dma-low-power.md)
