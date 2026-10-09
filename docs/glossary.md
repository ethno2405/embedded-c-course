# Glossary

Add a term to this list when you find it for the first time.

| Term | Meaning |
|---|---|
| ADC | Analog-to-digital converter. Converts a voltage to a number. |
| AF | Alternate function. A GPIO pin mode that connects the pin to a peripheral (for example UART TX). |
| AHB, APB | Advanced High-performance Bus, Advanced Peripheral Bus. Internal buses that connect the core to memory and peripherals. |
| BSRR | GPIO bit set/reset register. Sets or clears pins in one atomic write. |
| CMSIS | Cortex Microcontroller Software Interface Standard. Arm headers and APIs for Cortex-M devices. |
| CPOL, CPHA | SPI clock polarity and clock phase. Together they give the SPI mode (0–3). |
| Debounce | Removal of the false transitions that a mechanical switch makes when it opens or closes. |
| DMA | Direct memory access. Hardware that moves data without the CPU. |
| EXTI | External interrupt and event controller (STM32). Makes interrupts from GPIO edges. |
| Flash | Non-volatile memory. Holds the program. |
| GPIO | General-purpose input/output pin. |
| HAL | Hardware abstraction layer. A library that hides the registers. |
| HSE, HSI | High-speed external clock, high-speed internal clock (STM32). |
| I2C | Inter-Integrated Circuit. Two-wire bus (SCL, SDA) with addresses. |
| ISR | Interrupt service routine. The function that runs when an interrupt occurs. |
| LMA, VMA | Load memory address, virtual memory address. Where a section is stored, and where it is used at run time. |
| Linker script | A file that tells the linker where to put code and data in memory. |
| LSE, LSI | Low-speed external clock (32.768 kHz crystal), low-speed internal clock. |
| MCU | Microcontroller unit. CPU, memory, and peripherals on one chip. |
| MSP, PSP | Main stack pointer, process stack pointer (Cortex-M). |
| NVIC | Nested vectored interrupt controller. Part of the Cortex-M core that manages interrupts. |
| Open drain | An output that can only pull a line low. A resistor pulls the line high. |
| PLL | Phase-locked loop. Multiplies a clock frequency. |
| Pull-up, pull-down | A resistor that sets a defined level on a line when nothing drives it. |
| PWM | Pulse-width modulation. A square wave with a variable on-time. |
| RCC | Reset and clock control (STM32). Enables clocks for peripherals. |
| RTC | Real-time clock. Keeps the date and time. |
| RTOS | Real-time operating system. |
| SPI | Serial Peripheral Interface. Four-wire bus (SCK, MOSI, MISO, CS). |
| SRAM | Static RAM. Volatile working memory. |
| SVD | System View Description. XML file that describes the registers of a device. Debuggers use it. |
| SWD | Serial Wire Debug. Two-wire debug interface (SWDIO, SWCLK). |
| UART | Universal asynchronous receiver-transmitter. Serial port with no clock line. |
| Vector table | A table of addresses at the start of flash. The core uses it to find the reset handler and the interrupt handlers. |
| `volatile` | C qualifier. Tells the compiler that a value can change outside the program flow. |
| Watchdog | A timer that resets the MCU if the software does not refresh it in time. |
