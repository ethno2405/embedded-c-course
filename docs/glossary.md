# Glossary

Add a term to this list when you find it for the first time.

| Term | Meaning |
|---|---|
| ADC | Analog-to-digital converter. Converts a voltage to a number. |
| AF | Alternate function. A GPIO pin mode that connects the pin to a peripheral (for example UART TX). |
| AHB, APB | Advanced High-performance Bus, Advanced Peripheral Bus. Internal buses that connect the core to memory and peripherals. |
| Bitstream | The configuration file for an FPGA. The tools generate it from the HDL design. |
| BSRR | GPIO bit set/reset register. Sets or clears pins in one atomic write. |
| CMSIS | Cortex Microcontroller Software Interface Standard. Arm headers and APIs for Cortex-M devices. |
| CPOL, CPHA | SPI clock polarity and clock phase. Together they give the SPI mode (0–3). |
| Debounce | Removal of the false transitions that a mechanical switch makes when it opens or closes. |
| DMA | Direct memory access. Hardware that moves data without the CPU. |
| EXTI | External interrupt and event controller (STM32). Makes interrupts from GPIO edges. |
| Flash | Non-volatile memory. Holds the program. |
| FPGA | Field-programmable gate array. A chip with configurable logic. You describe the hardware in an HDL. |
| GPIO | General-purpose input/output pin. |
| HAL | Hardware abstraction layer. A library that hides the registers. |
| HDL | Hardware description language, for example Verilog or VHDL. Describes hardware, not a sequence of instructions. |
| HSE, HSI | High-speed external clock, high-speed internal clock (STM32). |
| I2C | Inter-Integrated Circuit. Two-wire bus (SCL, SDA) with addresses. |
| ISR | Interrupt service routine. The function that runs when an interrupt occurs. |
| LMA, VMA | Load memory address, virtual memory address. Where a section is stored, and where it is used at run time. |
| Linker script | A file that tells the linker where to put code and data in memory. |
| LSE, LSI | Low-speed external clock (32.768 kHz crystal), low-speed internal clock. |
| LUT | Look-up table. The basic logic element of an FPGA. It implements any function of a few inputs. |
| MCU | Microcontroller unit. CPU, memory, and peripherals on one chip. |
| Metastability | An unstable state of a flip-flop when its input changes too near the clock edge. A synchronizer reduces the risk. |
| MSP, PSP | Main stack pointer, process stack pointer (Cortex-M). |
| NVIC | Nested vectored interrupt controller. Part of the Cortex-M core that manages interrupts. |
| Open drain | An output that can only pull a line low. A resistor pulls the line high. |
| Place and route | The FPGA tool step that puts the logic into the chip resources and connects them. |
| PLL | Phase-locked loop. Multiplies a clock frequency. |
| Pull-up, pull-down | A resistor that sets a defined level on a line when nothing drives it. |
| PWM | Pulse-width modulation. A square wave with a variable on-time. |
| RCC | Reset and clock control (STM32). Enables clocks for peripherals. |
| RTC | Real-time clock. Keeps the date and time. |
| RTOS | Real-time operating system. |
| Soft-core CPU | A CPU described in an HDL and built from FPGA logic, for example a small RISC-V core. |
| SPI | Serial Peripheral Interface. Four-wire bus (SCK, MOSI, MISO, CS). |
| SRAM | Static RAM. Volatile working memory. |
| SVD | System View Description. XML file that describes the registers of a device. Debuggers use it. |
| SWD | Serial Wire Debug. Two-wire debug interface (SWDIO, SWCLK). |
| Synchronizer | Two or more flip-flops in series that bring an asynchronous signal into a clock domain. |
| Synthesis | The FPGA tool step that converts HDL code into logic elements (LUTs, flip-flops). |
| Testbench | HDL code that drives inputs into a design and checks its outputs in simulation. |
| UART | Universal asynchronous receiver-transmitter. Serial port with no clock line. |
| Vector table | A table of addresses at the start of flash. The core uses it to find the reset handler and the interrupt handlers. |
| Verilog | A hardware description language. Used in phase 8, module 8.9. |
| `volatile` | C qualifier. Tells the compiler that a value can change outside the program flow. |
| Watchdog | A timer that resets the MCU if the software does not refresh it in time. |
