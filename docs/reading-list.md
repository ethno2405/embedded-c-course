# Reading List

Each phase document tells you what to read and when. This document lists all sources in one place.

## Core books

| Book | Use it for | Phase |
|---|---|---|
| Elecia White, *Making Embedded Systems*, 2nd edition (O'Reilly, 2024) | Overview of the field from a software developer's view | 0–6 |
| Steve Oualline, *Bare Metal C* (No Starch Press, 2022) | C on a microcontroller, startup code, linker scripts | 1–2 |
| Joseph Yiu, *The Definitive Guide to ARM Cortex-M3 and Cortex-M4 Processors*, 3rd edition | Reference for the core: registers, exceptions, NVIC, memory map | 2–3, 7 |
| Carmine Noviello, *Mastering STM32*, 2nd edition | STM32 peripherals and the ST HAL | 3–7 |
| James Grenning, *Test-Driven Development for Embedded C* | Unit tests for embedded code | 7 |
| Richard Barry, *Mastering the FreeRTOS Real Time Kernel* (free PDF, freertos.org) | FreeRTOS | 6 |

## Electronics books

| Book | Use it for |
|---|---|
| Charles Platt, *Make: Electronics*, 3rd edition | Electronics from zero, with experiments |
| Paul Scherz and Simon Monk, *Practical Electronics for Inventors* | Reference for components and circuits |
| Horowitz and Hill, *The Art of Electronics* | Deep reference. Use it later, not as a first book. |

## C language

| Source | Use it for |
|---|---|
| K. N. King, *C Programming: A Modern Approach*, 2nd edition | Learn C from the beginning. Uses C99. Learn the C23 changes from phase 1, topic 1.13. |
| Beej's Guide to C Programming (free, beej.us) | Short, practical introduction to C |
| Kernighan and Ritchie, *The C Programming Language*, 2nd edition | Classic, short. Old (C89). |
| Jens Gustedt, *Modern C*, 3rd edition (free PDF) | Modern C, including C23 |
| cppreference.com, "C reference" | Reference for the language and library, with the standard version of each feature |
| Michael Barr, *Embedded C Coding Standard* (free, Barr Group) | Coding rules for embedded C |
| Compiler Explorer (godbolt.org) | See the assembly that the compiler makes |

## Online courses and guides

| Source | Use it for |
|---|---|
| Quantum Leaps, "Modern Embedded Systems Programming" (Miro Samek, YouTube) | Free video course: from machine code to RTOS and active objects |
| github.com/cpq/bare-metal-programming-guide | Bare-metal STM32 with no IDE and no HAL |
| Interrupt blog by Memfault (interrupt.memfault.com) | Boot process, linker scripts, fault analysis, code size |
| Embedded Artistry (embeddedartistry.com) | Articles and reading lists |
| DigiKey / Shawn Hymel, "Introduction to RTOS" and "Introduction to Zephyr" (YouTube) | RTOS concepts, Zephyr |
| SparkFun tutorials (learn.sparkfun.com) | Pull-up resistors, logic levels, I2C, SPI, serial, schematics |
| Ben Eater (YouTube) | Digital logic, how a CPU works |
| Phil's Lab (YouTube) | STM32 hardware and PCB design |

## Vendor documents for the Nucleo-F411RE

Download these from st.com. Keep them in a local `datasheets/` directory. Do not add them to git.

| Document | Content |
|---|---|
| **RM0383** — STM32F411xC/E Reference Manual | Every peripheral and register |
| **STM32F411xC/E Datasheet** | Pinout, alternate functions, electrical data, absolute maximum ratings |
| **PM0214** — STM32 Cortex-M4 Programming Manual | Core registers, instructions, NVIC, SysTick, SCB |
| **UM1724** — STM32 Nucleo-64 boards User Manual | Board schematic, jumpers, solder bridges |
| **ES0287** — STM32F411xC/E Errata Sheet | Known silicon bugs |
| **ARMv7-M Architecture Reference Manual** (arm.com) | Full architecture reference |

## Digital logic and FPGAs (phase 8, module 8.9)

| Source | Use it for |
|---|---|
| Harris and Harris, *Digital Design and Computer Architecture, RISC-V Edition* | Digital logic, HDLs, and how a RISC-V CPU works |
| HDLBits (hdlbits.01xz.net) | Free online Verilog exercises with automatic checks |
| Russell Merrick, *Getting Started with FPGAs* (No Starch Press), nandland.com | A practical introduction to FPGAs |
| Lushay Labs (learn.lushaylabs.com) | Tutorials for the Tang Nano boards with open-source tools |
| ZipCPU blog (zipcpu.com) | Deeper topics: simulation, formal verification, bus design |
| YosysHQ documentation | Yosys, nextpnr, and the OSS CAD Suite |

## Component datasheets

| Component | Document |
|---|---|
| BME280 | Bosch BME280 datasheet |
| SSD1306 | Solomon Systech SSD1306 datasheet |
| ST7789 | Sitronix ST7789 datasheet |
| DS3231 | Analog Devices (Maxim) DS3231 datasheet |
| WS2812B | Worldsemi WS2812B datasheet |
| SD card | SD Association, "Physical Layer Simplified Specification" |
