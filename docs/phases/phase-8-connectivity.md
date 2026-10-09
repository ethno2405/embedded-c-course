# Phase 8: Connectivity and Beyond

**Time:** Open-ended. Select the modules that interest you. Each module takes 2–4 weeks.
**Prerequisites:** Phase 7
**Hardware:** ESP32-C3 or ESP32-S3 DevKit, Raspberry Pi Pico 2 and Debug Probe, 2 × SN65HVD230 CAN modules, KiCad (software)

## Goal

This phase has independent modules. Each module adds one skill. Module 8.1 completes the main data logger project.

## Module 8.1: Wi-Fi and MQTT with the ESP32

### Reading

- Espressif ESP-IDF Programming Guide: "Get Started", "Build System", "Wi-Fi Driver", "ESP-MQTT"
- MQTT basics: mqtt.org, HiveMQ "MQTT Essentials" series

### Topics

- **ESP-IDF:** the Espressif framework. It uses FreeRTOS, CMake, and components. The tool `idf.py` builds, flashes, and monitors. `menuconfig` sets the configuration.
- **Wi-Fi station mode:** connect to an access point. Handle the events "connected", "got IP", and "disconnected". Connect again after a disconnect.
- **MQTT:** publish/subscribe protocol over TCP. A broker (for example Mosquitto) forwards messages. Topics, QoS levels 0/1/2, retained messages, last will.
- **Security:** use TLS for real deployments. Do not put Wi-Fi passwords in git. Use NVS (non-volatile storage) or a configuration file that git ignores.
- **Two-chip design:** the STM32 measures and logs. The ESP32 is a network coprocessor. They communicate over UART with a simple framed protocol (reuse the frame format from the bootloader).

### Exercises

1. Build and flash the ESP-IDF "hello_world" and "station" examples.
2. Install Mosquitto on your PC (or in Docker). Publish a message from the ESP32. Read it with `mosquitto_sub`.
3. Connect the ESP32 to the STM32 over UART. Forward each measurement to MQTT.
4. Show the data in Grafana (with Telegraf or Node-RED and InfluxDB). Optional.
5. Disconnect the Wi-Fi access point. The data logger must keep the data on the SD card and send it later.

## Module 8.2: USB device with TinyUSB (Pico 2)

The Nucleo-64 board has no USB connector for the STM32F411. Use the Pico 2 for USB.

### Reading

- Pico 2 / RP2350 datasheet, and "Getting started with Raspberry Pi Pico-series"
- "USB in a NutShell" (beyondlogic.org)
- TinyUSB documentation and examples

### Topics

- USB basics: host and device, enumeration, descriptors (device, configuration, interface, endpoint), endpoint types (control, bulk, interrupt, isochronous).
- Device classes: CDC (virtual serial port), HID (keyboard, mouse, gamepad), MSC (mass storage), MIDI.
- The Pico SDK: CMake, `pico_sdk_import.cmake`, `picotool`, debugging with the Debug Probe and OpenOCD.

### Exercises

1. Set up the Pico SDK. Blink the LED. Debug with the Debug Probe.
2. Make a USB CDC device. Port your command line to it.
3. Make a USB MIDI controller with 4 buttons and a potentiometer.
4. Make a USB HID device: a macro keypad.

## Module 8.3: PIO on the Pico 2

### Topics

- PIO (programmable I/O) state machines run small programs that control pins with exact timing. They free the CPU from bit-banging.

### Exercises

1. Run the WS2812 PIO example. Compare it with your timer and DMA solution from phase 5.
2. Write a PIO program for a protocol that the chip does not have in hardware.

## Module 8.4: CAN bus

The STM32F411 has no CAN peripheral. Use the ESP32 TWAI controller (a CAN 2.0 controller) with an SN65HVD230 transceiver. Another option is an STM32 board with a CAN peripheral (for example a Nucleo-F446RE).

### Reading

- Texas Instruments application report "Introduction to the Controller Area Network (CAN)" (SLOA101)
- ESP-IDF Programming Guide: "Two-Wire Automotive Interface (TWAI)"

### Topics

- Differential bus (CAN_H, CAN_L). 120 Ω termination at each end of the bus.
- Frames: 11-bit or 29-bit identifier, up to 8 data bytes. The identifier is also the priority. Arbitration is bit by bit, and no data is lost.
- Error handling: error counters, error passive, bus off.
- Higher protocols: CANopen, J1939, ISO-TP. OBD-II in cars.

### Exercises

1. Connect two ESP32 boards (or one ESP32 and another CAN node) with transceivers and termination.
2. Send and receive frames. Capture CAN_RX with the logic analyzer. Use the PulseView CAN decoder.
3. Write a CAN sniffer that prints all frames over UART.
4. Remove the termination. Observe the errors.

## Module 8.5: Zephyr RTOS

### Reading

- docs.zephyrproject.org: "Getting Started Guide", "Devicetree", "Kconfig"
- DigiKey / Shawn Hymel "Introduction to Zephyr" series

### Topics

- `west` (the meta-tool), the build system (CMake and Kconfig), devicetree (hardware description), the device driver model, logging, the shell.
- The board `nucleo_f411re` is supported.

### Exercises

1. Build and flash the "blinky" sample for `nucleo_f411re`.
2. Add the BME280 with a devicetree overlay. Use the Zephyr sensor driver.
3. Port the data logger to Zephyr. Compare the code size and the effort with your FreeRTOS version.

## Module 8.6: PCB design with KiCad

### Reading

- KiCad "Getting Started" documentation
- Phil's Lab (YouTube): KiCad and STM32 hardware design videos
- ST application note AN2867 "Guidelines for oscillator design on STM8AF/AL/S and STM32 MCUs/MPUs"
- ST application note AN4488 "Getting started with STM32F4xxxx MCU hardware development"

### Topics

- Schematic, symbols, footprints, layout, design rule check (DRC), Gerber files.
- Decoupling, ground planes, crystal layout, the SWD connector, USB routing.
- Manufacturing and assembly services: JLCPCB, PCBWay.

### Exercises

1. Design an Arduino-header shield for the Nucleo board with the BME280, the SD card slot, the RTC, and the OLED connector. Order it.
2. Advanced: design a full data logger board with an STM32 chip, a regulator, a battery connector, and an SWD connector.

## Module 8.7: Embedded Rust

### Reading

- *The Rust Programming Language* (doc.rust-lang.org/book). Read it first if you do not know Rust.
- *The Embedded Rust Book* (docs.rust-embedded.org)
- Embassy documentation (embassy.dev)

### Topics

- `no_std`, the PAC (peripheral access crate), the HAL crates, `embedded-hal` traits.
- Embassy: async/await executor for embedded. Tasks without an RTOS.
- probe-rs and `defmt` for logging.

### Exercises

1. Port the blinky and the BME280 reading to Rust with `embassy-stm32`.
2. Compare the code size, the safety guarantees, and the development speed with your C version.

## Module 8.8: Advanced project: self-balancing robot

### Topics

- IMU: accelerometer and gyroscope. Sensor fusion with a complementary filter (or a Kalman filter).
- PID control. Tune the gains.
- Motor control with PWM and an H-bridge driver (TB6612FNG).
- Encoders (optional) with the timer encoder mode.
- Battery power and safety.

### Exercises

1. Read the IMU at 500 Hz. Calculate the tilt angle with a complementary filter. Plot it over UART.
2. Implement a PID controller. Test it on your PC with a simple simulation.
3. Build the robot. Tune the PID gains. Add a remote control over BLE (ESP32) or UART.

## Done criteria

Each module is complete when all its exercises work. Write a short summary of each module in [../topics/](../topics/): what you learned, what was difficult, and what you want to do next.
