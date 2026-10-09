# Project: Environmental Data Logger

The data logger is the main project of the course. It starts as a blinky in phase 2. It ends as a connected, low-power device with a bootloader in phase 8. Each stage adds features to the code from the previous stage. Do not start again from zero.

## Final system

```
Sensors and I/O            Your board (FreeRTOS)                    Outputs
---------------            ---------------------                    -------
BME280  --I2C-->           sensor task --queue--> logger task   --SPI-->   SD card (FatFs)
DS3231  --I2C-->                       --queue--> display task  --I2C-->   SSD1306 OLED
Button  --EXTI->           command line task      <--USART2-->  PC (ST-LINK COM port)
Battery --ADC-->           supervisor task (watchdog)
                           network task           --USART1-->   ESP32 --Wi-Fi--> MQTT broker --> dashboard

Flash (reference board): bootloader (sectors 0-1), settings (sector 2), application (sectors 4-5), download slot (sectors 6-7)
```

The peripheral names (USART1, USART2), the ST-LINK COM port, and the flash sectors are for the reference board. On a different board, use the matching peripherals and the flash layout of your MCU.

## Stage 0: Template (phase 2)

**Features**

- CMake project with your own linker script and startup code.
- System clock at 100 MHz.
- LD2 blinks at 1 Hz. B1 changes the blink rate.

**Acceptance criteria**

- [ ] The project builds from the command line with no warnings.
- [ ] The map file shows the vector table at `0x08000000`.
- [ ] The MCO output shows the expected clock frequency.

## Stage 1: Measure and show (phase 4)

**Features**

- Interrupt-driven UART driver on USART2. `printf` works.
- Command line with these commands: `help`, `read` (print one measurement), `interval <s>` (set the measurement interval), `status` (uptime, interval, sensor state), `i2cscan`.
- BME280 driver (I2C) with integer compensation.
- SSD1306 display shows temperature, humidity, pressure, and uptime.
- Measurement every `interval` seconds (default 5 s). Non-blocking main loop.

**Acceptance criteria**

- [ ] The values match a reference thermometer within the BME280 accuracy.
- [ ] The command line answers while the logger measures.
- [ ] The firmware detects a missing sensor and shows an error. It does not hang.
- [ ] The BME280 compensation has host unit tests.

## Stage 2: Log and save power (phase 5)

**Features**

- DS3231 RTC. Commands `time` and `settime`.
- FatFs on the SD card. One CSV file per day: `YYYYMMDD.CSV`, with the columns `timestamp,temperature_c,humidity_pct,pressure_hpa`.
- Battery voltage measurement with the ADC (use a voltage divider if the battery voltage is higher than 3.3 V).
- Low-power mode: the MCU sleeps in Stop mode between measurements. The RTC wakes it.
- The button wakes the display for 10 seconds.
- IWDG watchdog. The reset cause is logged in the CSV file as a comment line.

**Acceptance criteria**

- [ ] The logger runs for 24 hours without errors. The CSV files open in a spreadsheet program.
- [ ] Removal of the SD card does not crash the firmware. The logger continues when the card is back.
- [ ] The average current is measured and documented. The battery life is calculated.

## Stage 3: RTOS (phase 6)

**Features**

- FreeRTOS with these tasks: sensor, logger (SD card), display, command line, and a supervisor that refreshes the watchdog when all tasks report health.
- Communication only with queues, stream buffers, and notifications. No shared global variables.
- Command `tasks`: stack high water mark and CPU time of each task.
- Tickless idle for low power.

**Acceptance criteria**

- [ ] The supervisor detects a stuck task. The watchdog resets the device.
- [ ] Each task has a stack margin of at least 25 %.
- [ ] The current consumption is not higher than in stage 2 by more than 10 %.

## Stage 4: Robust and updatable (phase 7)

**Features**

- Bootloader with an image header and CRC32.
- Firmware update over UART with a Python script.
- HardFault handler that saves fault data in `.noinit` RAM and logs it after the reset.
- Command `version`: version, git hash, build date.
- Host unit tests and a CI pipeline.

**Acceptance criteria**

- [ ] A power loss during an update does not make the device unusable.
- [ ] A corrupt image (wrong CRC) is rejected.
- [ ] CI builds the bootloader and the application and runs all host tests.

## Stage 5: Connected (phase 8)

**Features**

- ESP32 network coprocessor connected over USART1 (or USART6).
- MQTT topics: `logger/<id>/measurement`, `logger/<id>/status`. Last will message for "offline".
- Store-and-forward: when the network is down, the data stays on the SD card and goes to the broker later.
- Optional: custom PCB (module 8.6).

**Acceptance criteria**

- [ ] The measurements appear in the MQTT broker within 10 seconds.
- [ ] After a network outage of 1 hour, no measurement is lost.
- [ ] No Wi-Fi password is in git.
