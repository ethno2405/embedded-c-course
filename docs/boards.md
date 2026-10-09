# Select Your Board

You select the microcontroller board for the course. The AI mentor helps you select it: ask `Help me choose a board.`

## The reference board

The course documents use one **reference board**: the **STM32 Nucleo-F411RE**. All pin names, register names, manual sections, and tool commands in the phase documents are for this board.

You can use a different board. The concepts are the same on all modern 32-bit microcontrollers. The AI mentor adapts the board-specific details to your board: pins, registers, manual sections, and tools. The more a board is different from the reference board, the more adaptation you need, and the more you must read the vendor documents yourself.

## Comparison

Prices are approximate, in USD, for the board only. "Fit" tells you how well the phase documents match the board without changes.

| Board | MCU and core | Debugger on the board | Price | Fit | Good for |
|---|---|---|---|---|---|
| **STM32 Nucleo-F411RE** (reference) | STM32F411RE, Cortex-M4F, 100 MHz, 512 KB flash, 128 KB SRAM | Yes (ST-LINK), with a virtual COM port | $15 | **Exact** | All learners. The easiest path through the course. |
| STM32 Nucleo-F446RE | STM32F446RE, Cortex-M4F, 180 MHz, 512 KB flash, 128 KB SRAM, CAN | Yes (ST-LINK) | $20–25 | **High.** Same board layout and LED, button, and console pins. Different reference manual (RM0390). | Learners who want CAN on the main board |
| WeAct "Black Pill" STM32F411CEU6 | Same MCU family as the reference board | No. Needs an ST-LINK or a CMSIS-DAP probe ($5–12). | $5–8 | **High.** Same reference manual (RM0383). Different pins (LED, button), no console through the debugger (needs a USB-to-UART adapter). | Low budget. Note: some sellers ship copies of the chip. |
| STM32 Nucleo-L476RG or Nucleo-G431RB | STM32L4 (low power) or STM32G4 (motor control, FDCAN), Cortex-M4F | Yes (ST-LINK) | $15–20 | **Medium.** Same board layout and tools. Newer peripheral versions: some registers (for example I2C and USART) are different. | Low-power or motor-control goals |
| Raspberry Pi Pico 2 + Debug Probe | RP2350, dual Cortex-M33, 150 MHz, 520 KB SRAM, external flash | No. Use the Raspberry Pi Debug Probe ($12). | $5 + $12 | **Medium–low.** Same concepts, but all peripherals, the boot process, and the tools are different. Excellent datasheet and C SDK. | Low budget, USB device projects, PIO |
| Nordic nRF52840 DK | nRF52840, Cortex-M4F, 64 MHz, 1 MB flash, 256 KB RAM, Bluetooth LE | Yes (SEGGER J-Link) | $50 | **Medium–low.** All peripherals and tools are different. The vendor SDK is based on Zephyr. | Bluetooth LE goals |
| ESP32-C3 or ESP32-S3 DevKit | RISC-V (C3) or Xtensa (S3), Wi-Fi and Bluetooth LE | USB JTAG on the chip (C3, S3) | $8–15 | **Low.** Register-level work is less common. The vendor framework (ESP-IDF) includes FreeRTOS. | Wi-Fi projects. Better as a second board (phase 8). |
| Arduino Uno R3 | ATmega328P, 8-bit AVR, 16 MHz, 32 KB flash, 2 KB RAM, 5 V logic | No | $25 | **Not suitable.** No DMA, very small RAM, 5 V logic, a different architecture. | Not recommended for this course |

Do a check of the current specifications and prices at the vendor before you buy.

## How to decide

1. **No preference:** select the reference board. The documents match it exactly, and it costs little.
2. **Low budget:** the reference board is already cheap. The Black Pill is cheaper, but you also need a debug probe and a USB-to-UART adapter, and you must adapt the pins.
3. **A specific goal** (Bluetooth, low power, CAN, motor control): select a board that supports the goal. Expect more adaptation in phases 2–7.
4. **You already have a board:** tell the mentor. It tells you how well the board fits and what you must change.
5. **Two boards:** many learners use the reference board for phases 2–7 and a second board (Pico 2, ESP32) for phase 8.

## What changes on a different board

| Item | What you must do |
|---|---|
| Pins (LED, button, console UART, I2C, SPI) | Find the pins in your board manual. The mentor records them in your profile. |
| Reference manual and datasheet | Download the documents for your MCU. Replace RM0383, PM0214, and UM1724 with them. |
| Register names and bits | Find the matching peripheral in your reference manual. Peripheral versions can be different, even in the same vendor family. |
| Clock tree | Your MCU has different clock sources, limits, and flash wait states. |
| Startup code and linker script | Use the memory sizes and the vector table of your MCU. |
| Compiler flags | The core can be different (for example Cortex-M33 or RISC-V). |
| Debug and flash tools | Use the OpenOCD or probe-rs configuration for your board, or the vendor tool. |

The modules and tools in [hardware.md](hardware.md) (sensors, displays, logic analyzer, multimeter) work with all 3.3 V boards.

## FPGA boards (phase 8, module 8.9)

An FPGA board is **not** a main board for this course. Phases 2–7 need a microcontroller. Buy an FPGA board only for module 8.9.

| Board | FPGA | Tools | Price | Notes |
|---|---|---|---|---|
| Sipeed Tang Nano 9K | Gowin GW1NR-9 | Open-source (Yosys, nextpnr, Apicula) or Gowin EDA | $15–20 | Cheapest. USB programmer and UART on the board. Some I/O banks are not 3.3 V: do a check of the schematic. |
| Sipeed Tang Nano 20K | Gowin GW2AR-18 | Open-source or Gowin EDA | $30 | More logic and memory. Room for a larger soft-core system. Do a check of the open-source tool support before you buy. |
| iCEBreaker | Lattice iCE40UP5K | Fully open-source (Yosys, nextpnr, IceStorm) | $70–80 | The best support from the open-source tools. Good documentation. |
| Digilent Basys 3 or Arty A7 | AMD Artix-7 | AMD Vivado (vendor tool, large download) | $150–300 | Large FPGA, industry-standard tools. Good if you want an FPGA job. |

If you do not know which one to select: the Tang Nano 9K for a low budget, the iCEBreaker for the easiest open-source path.
