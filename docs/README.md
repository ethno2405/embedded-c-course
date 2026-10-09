# Embedded C Course

This course teaches embedded programming in C. It starts at zero for hardware, electronics, and low-level protocols.

## Who this course is for

- **Required:** you can write programs in at least one programming language. You know variables, functions, loops, and data structures, and you can use git and a terminal.
- **Not required:** knowledge of C, electronics, or computer architecture. The course teaches these topics.
- **Your background changes the time for phase 1:**

  | Your background | Phase 1 time |
  |---|---|
  | You know C or C++ well | 1 week |
  | You know Rust, Go, or another compiled language with pointers or references | 2 weeks |
  | You know only managed or scripting languages (C#, Java, Python, JavaScript, and similar) | 4–6 weeks |

The AI mentor assesses your background in the first session and saves it in `learner/profile.md`. See the [main README](../README.md) for how to start. Without an AI mentor, use the table above to plan phase 1.

The course has nine phases. Each phase has topics, reading, exercises, and a project. One main project, the **environmental data logger**, grows from phase 2 to phase 8. Side projects give more practice.

## How to use this course

You can use the course with an AI mentor or alone. With an AI mentor, the mentor follows [mentor/](mentor/) and tracks your progress in `learner/progress.md`. Alone, follow these steps:

1. Read the phase document from top to bottom before you start the phase.
2. Do the reading and the exercises in the given order.
3. Do the project for the phase.
4. Do a check of the "Done criteria" list at the end of the phase. Go to the next phase only when all items are complete.
5. Write your notes, measurements, and problems in your lab notebook.
6. Record your progress in `learner/progress.md`. Copy it from [templates/progress-template.md](templates/progress-template.md).

The times assume 6–8 hours of work per week.

## Roadmap

| Phase | Document | Time | Main result |
|---|---|---|---|
| 0 | [Electronics basics](phases/phase-0-electronics.md) | 2–3 weeks | You can build and measure simple circuits and read a datasheet. |
| 1 | [C for embedded systems](phases/phase-1-c-for-embedded.md) | 1–6 weeks, depends on your background (parallel to phase 0) | You can write C that controls hardware registers correctly. |
| 2 | [How a microcontroller starts](phases/phase-2-boot.md) | 2–3 weeks | Bare-metal blinky with your own startup code and linker script. |
| 3 | [Interrupts and timers](phases/phase-3-interrupts-timers.md) | 3 weeks | Interrupt-driven input, PWM output, and time measurement. |
| 4 | [Serial protocols](phases/phase-4-serial-protocols.md) | 4–5 weeks | UART command line, I2C sensor, OLED, and SPI display drivers. |
| 5 | [ADC, DMA, and low power](phases/phase-5-adc-dma-low-power.md) | 3–4 weeks | Battery-friendly data logger that writes to an SD card. |
| 6 | [Software architecture and RTOS](phases/phase-6-architecture-rtos.md) | 3–4 weeks | Data logger on FreeRTOS with tasks and queues. |
| 7 | [Professional practices](phases/phase-7-professional-practices.md) | 3–4 weeks | Fault analysis, unit tests, CI, and a bootloader. |
| 8 | [Connectivity and beyond](phases/phase-8-connectivity.md) | Open-ended | Wi-Fi, USB, CAN, Zephyr, PCB design, Rust, and an introduction to FPGAs. |

## Main project: environmental data logger

The full specification is in [projects/data-logger.md](projects/data-logger.md).

| Stage | Phase | Features |
|---|---|---|
| 0 | 2 | Bare-metal blinky. Your own startup code. |
| 1 | 4 | UART command line, BME280 sensor, OLED display. |
| 2 | 5 | SD card log, real-time clock, low-power sleep. |
| 3 | 6 | FreeRTOS tasks and queues. |
| 4 | 7 | Bootloader, firmware update over UART, unit tests. |
| 5 | 8 | Wi-Fi and MQTT through an ESP32. Custom PCB. |

## Documents

| Document | Content |
|---|---|
| [boards.md](boards.md) | Board comparison and how to select your board |
| [hardware.md](hardware.md) | Tools and components to buy |
| [software-setup.md](software-setup.md) | Toolchain, debugger, and editor setup on Windows |
| [reading-list.md](reading-list.md) | Books, courses, and vendor documents |
| [glossary.md](glossary.md) | Terms and abbreviations |
| [phases/](phases/) | One document for each phase |
| [projects/](projects/) | Project specifications and task documents |
| [topics/](topics/) | Detailed explanations of specific topics |
| [mentor/](mentor/) | Instructions for the AI mentor: assessment and teaching |
| [templates/](templates/) | Templates for new documents, the learner profile, and progress |

## Platform

You select your board. See [boards.md](boards.md) for a comparison, or ask the AI mentor for a suggestion.

- **Reference board:** STM32 Nucleo-F411RE (Cortex-M4F, 100 MHz, 512 KB flash, 128 KB SRAM, built-in ST-LINK debugger). All pins, registers, manual sections, and tool commands in the phase documents are for this board. On a different board, you (or the AI mentor) translate these details.
- **Phase 8 boards:** Raspberry Pi Pico 2 (RP2350) and Raspberry Pi Debug Probe, and an ESP32-C3 or ESP32-S3 DevKit (Wi-Fi and Bluetooth). These are suggestions for the phase 8 modules.

## General advice

1. **Write a driver before you use a library driver.** Write your own GPIO, UART, and I2C code with registers first. Then read the ST HAL. You then understand what the HAL hides.
2. **Use the logic analyzer from the first day of phase 4.** When a protocol "does not work", you usually cannot see the signal. The analyzer shows you the signal.
3. **Keep a lab notebook.** Write down register values, pin assignments, measurements, and errata problems.
4. **Expect hardware faults.** Many problems are loose wires, missing pull-up resistors, wrong logic levels, or a missing common ground. Do a check of the hardware before you debug the code.
5. **Learn the C patterns first.** Most vendor code, application notes, and jobs use C. Learn opaque handles, function-pointer tables, and static allocation before you use C++ or Rust on a microcontroller.
