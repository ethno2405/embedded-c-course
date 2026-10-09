# Phase 7: Professional Practices

**Time:** 3–4 weeks
**Prerequisites:** Phase 6
**Hardware:** Your board (reference: Nucleo-F411RE) with the stage 3 data logger hardware, USB-to-UART adapter
**Board:** The board details in this phase (pins, registers, manual sections, tool commands) are for the reference board. On a different board, translate them with your board documents, or ask the AI mentor. See [../boards.md](../boards.md).

## Goal

At the end of this phase, you can:

- Find the cause of a HardFault from the fault registers and the stack frame.
- Test embedded code on your PC with unit tests and test doubles.
- Use static analysis and a CI pipeline.
- Decide when to use your own drivers, the ST LL drivers, or the ST HAL.
- Write a bootloader that updates the firmware safely.
- Read a map file and reduce code size.

## Reading

| Source | Part |
|---|---|
| Grenning, *Test-Driven Development for Embedded C* | All chapters |
| Interrupt blog | "How to debug a HardFault on an ARM Cortex-M MCU", "How to write a bootloader from scratch", "Code size optimization" articles, "Unit testing basics" |
| Yiu, *Definitive Guide* | The chapters about fault exceptions and the MPU |
| RM0383 | Section "Embedded Flash memory interface": sectors, erase, program |
| ST application note AN2606 | "STM32 microcontroller system memory boot mode" (the built-in ST bootloader) |
| Noviello, *Mastering STM32* | The chapters about the HAL, the LL drivers, and the bootloader |

## Topics

### 7.1 Fault analysis

- A fault starts the `HardFault_Handler` (or `MemManage`, `BusFault`, `UsageFault` if you enable them in `SCB->SHCSR`).
- Fault status registers in the SCB:

  | Register | Content |
  |---|---|
  | `CFSR` | Configurable fault status: MemManage (`MMFSR`), BusFault (`BFSR`), UsageFault (`UFSR`) bits |
  | `HFSR` | HardFault status. `FORCED` means a configurable fault escalated. |
  | `MMFAR` | Address that caused a MemManage fault (if `MMARVALID`) |
  | `BFAR` | Address that caused a BusFault (if `BFARVALID`) |

- The stacked `PC` shows the instruction that failed. The stacked `LR` shows the caller. Find the source line with `arm-none-eabi-addr2line -e firmware.elf <address>`.
- The handler must find the correct stack: bit 2 of `EXC_RETURN` (in `LR`) selects `MSP` or `PSP`. This needs a few lines of assembly.
- **Note:** on the STM32, address `0x0000 0000` is an alias of flash. A read through a `NULL` pointer does **not** fault. Use the MPU to protect the first bytes of memory if you want to catch `NULL` reads.
- Enable traps in `SCB->CCR`: `DIV_0_TRP` (divide by zero) and `UNALIGN_TRP` (unaligned access).

### 7.2 Logging and tracing

- **UART logging:** simple, but slow and it uses a pin.
- **SEGGER RTT:** writes to a ring buffer in SRAM. The debugger reads it. Very fast. Works with J-Link, OpenOCD, and probe-rs.
- **ITM/SWO:** a trace output pin from the core (PB3). The Nucleo ST-LINK can receive it. Do a check of the solder bridge settings in UM1724.
- Use log levels (`ERROR`, `WARN`, `INFO`, `DEBUG`) and a compile-time switch to remove debug logs.
- **Post-mortem data:** store the fault registers in a RAM section that the startup code does not clear (`.noinit`). After the reset, print them.

### 7.3 Unit tests on the PC

- Run most tests on your PC. They are fast and easy to debug.
- Separate hardware access from logic:
  - **Logic modules:** parsers, state machines, ring buffers, sensor compensation. Test them directly.
  - **Hardware abstraction layer:** small functions that touch registers. Replace them with test doubles on the PC.
- **Test doubles:** a stub returns fixed values. A fake has simple working logic (for example the fake I2C bus). A mock records the calls and checks them. The **fff** (Fake Function Framework) library makes fakes for C functions.
- Frameworks: **Unity** with **Ceedling** (C, popular in embedded), **CppUTest** (C++ framework, tests C code, used in Grenning's book), **GoogleTest** (C++).
- Compile host tests with `-fsanitize=address,undefined`.

### 7.4 Static analysis and coding rules

- Compiler warnings: `-Wall -Wextra -Wconversion -Wshadow -Wdouble-promotion -Wformat=2 -Wundef`. Use `-Werror` in CI.
- Tools: `cppcheck`, `clang-tidy`, the GCC option `-fanalyzer`.
- **MISRA C:** a coding standard for safety-critical systems (automotive, medical). Learn what it is and why its rules exist. Full compliance needs a commercial tool.

### 7.5 Drivers: your own, LL, or HAL

| Option | Advantages | Disadvantages |
|---|---|---|
| Your own register drivers | Full control, small, you know every line | Takes time, you must maintain it |
| ST LL (low-layer) drivers | Thin `static inline` functions over registers, small | STM32 only |
| ST HAL | Fast to start, CubeMX generates the setup | Large, hides details, sometimes slow, uses callbacks with fixed patterns |
| Zephyr / vendor-neutral RTOS drivers | Portable to many vendors | Large learning curve, more layers |

- In practice, many teams use the HAL or LL for setup and write their own code for critical paths.
- Read the HAL source for the peripherals that you wrote yourself. Compare the code.

### 7.6 Bootloader and firmware update

- **Flash layout of the STM32F411 (512 KB):** sectors 0–3 are 16 KB, sector 4 is 64 KB, sectors 5–7 are 128 KB. You can erase only full sectors.
- Example layout:

  | Region | Sectors | Address |
  |---|---|---|
  | Bootloader | 0–1 (32 KB) | `0x0800 0000` |
  | Settings | 2 (16 KB) | `0x0800 8000` |
  | (unused) | 3 (16 KB) | `0x0800 C000` |
  | Application | 4–5 (192 KB) | `0x0801 0000` |
  | Download slot | 6–7 (256 KB) | `0x0804 0000` |

- **Image header:** magic number, version, size, CRC32. The bootloader starts the application only if the CRC is correct.
- **Jump to the application:**
  1. Disable interrupts and stop SysTick. Reset the peripherals that the bootloader used.
  2. Set `SCB->VTOR` to the application vector table address.
  3. Load `MSP` from the first word of the application.
  4. Jump to the address in the second word.
- The application uses its own linker script with a different `FLASH` origin.
- **Flash programming:** unlock with the keys in `FLASH->KEYR`, erase a sector (`SER`, `SNB`, `STRT`), program with the correct `PSIZE`, wait for `BSY` to clear, lock again. The CPU cannot read flash while it erases or programs. Code that must run during this time must be in SRAM.
- **Update protocol over UART:** send the image in frames. Each frame has a length, a sequence number, and a CRC. The receiver sends ACK or NACK. Write the PC side in Python.
- **Safe update:** write the new image to the download slot. Do a check of the CRC. Only then copy it to the application slot. If the power fails during the copy, the bootloader copies again at the next start.
- The STM32 also has a built-in ROM bootloader (AN2606). You start it with the `BOOT0` pin.

### 7.7 Code size and the map file

- The map file lists each section and symbol with its address and size.
- Tools: `arm-none-eabi-size`, `arm-none-eabi-nm --size-sort -S`, `puncover`, `bloaty`.
- Large items: `printf` with float, the HAL, FatFs options, font tables, debug strings.
- Options: `-Os`, `-flto` (link-time optimization), and removal of unused features.

### 7.8 Build information and CI

- Put the version, the git commit hash, and the build date in the firmware. Print them at startup and with a command `version`.
- CI pipeline (GitHub Actions): install the Arm toolchain, build all firmware, run the host tests, run static analysis, keep the `.elf` and `.map` files as artifacts.

## Exercises

### Exercise 7.1: Fault handler

1. Write a HardFault handler that prints `CFSR`, `HFSR`, `MMFAR`, `BFAR`, and the stacked `PC`, `LR`, and `xPSR`.
2. Make faults on purpose: write to an address that is not valid (for example `0xFFFFFFF0`), call a function pointer with an even address, divide by zero with `DIV_0_TRP` enabled, and overflow a stack.
3. For each fault, find the source line with `addr2line`.
4. Store the fault data in a `.noinit` section. Print it after the reset.

### Exercise 7.2: Unit tests

1. Set up Ceedling or CppUTest for the course code.
2. Add tests for the ring buffer, the command parser, the debounce state machine, and the BME280 compensation.
3. Write a test for the BME280 driver with fff fakes for the bus functions.
4. Run the tests with sanitizers.

### Exercise 7.3: Compare with the HAL

1. Make a project in STM32CubeMX with the same UART and I2C configuration as your drivers.
2. Compare the code size and the speed of a 100-byte transfer.
3. Read the HAL source for `HAL_I2C_Mem_Read`. List the differences from your driver.

### Exercise 7.4: Bootloader

1. Write the bootloader and change the application linker script.
2. Add the image header with CRC32. Add a post-build script that computes the CRC and writes it into the `.bin` file.
3. Write the UART update protocol and a Python script for the PC side.
4. Disconnect the power during an update. Show that the device recovers.

### Exercise 7.5: Code size

1. Print a code size report for the data logger. Find the 10 largest symbols.
2. Reduce the size by 20 %. Record each change and its effect.

### Exercise 7.6: CI

1. Put the course code on GitHub (or another git server).
2. Add a CI pipeline that builds the firmware and runs the host tests on every push.

## Project: Data logger, stage 4

See [../projects/data-logger.md](../projects/data-logger.md), stage 4.

## Done criteria

- [ ] I can find the source line of a HardFault from the fault data.
- [ ] My logic modules have host unit tests that run in CI.
- [ ] I can explain when to use my own drivers, the LL drivers, or the HAL.
- [ ] My bootloader updates the application over UART and survives a power loss.
- [ ] I can read a map file and find the largest symbols.

## Next

[Phase 8: Connectivity and beyond](phase-8-connectivity.md)
