# Phase 2: How a Microcontroller Starts

**Time:** 2–3 weeks
**Prerequisites:** Phase 0, phase 1, and the [software setup](../software-setup.md)
**Hardware:** Your board (reference: Nucleo-F411RE), USB cable, logic analyzer (exercise 2.7)
**Board:** The board details in this phase (pins, registers, manual sections, tool commands) are for the reference board. On a different board, translate them with your board documents, or ask the AI mentor. See [../boards.md](../boards.md).

## Goal

At the end of this phase, you can:

- Explain each step between reset and the first line of `main()`.
- Write a linker script and startup code for the STM32F411.
- Configure the system clock with the PLL.
- Control GPIO pins with registers only.
- Build, flash, and debug firmware without an IDE.

## Reading

| Source | Part |
|---|---|
| github.com/cpq/bare-metal-programming-guide | Read all. Do the steps on your board. |
| Oualline, *Bare Metal C* | The chapters about startup code, the linker, and the linker script |
| Yiu, *Definitive Guide* | The chapters about the programmer's model, the memory map, and the reset sequence |
| RM0383 | Sections: "Memory and bus architecture", "Reset and clock control (RCC)", "General-purpose I/Os (GPIO)", "Embedded Flash memory interface" |
| PM0214 | Section about the processor core registers |
| Interrupt blog | "From Zero to main(): Bare metal C" and "From Zero to main(): Demystifying Firmware Linker Scripts" |

## Topics

### 2.1 The Cortex-M4 core

- 32-bit RISC core. It executes the **Thumb-2** instruction set (16-bit and 32-bit instructions).
- Registers: `R0`–`R12` (general purpose), `R13` = `SP` (stack pointer), `R14` = `LR` (link register, return address), `R15` = `PC` (program counter), `xPSR` (status).
- Two stack pointers: `MSP` (main) and `PSP` (process). After reset, the core uses `MSP`. An RTOS uses `PSP` for tasks (phase 6).
- The STM32F411 has a single-precision FPU. The FPU is off after reset. The startup code must enable it in the `CPACR` register before you use `float`.

### 2.2 The memory map

| Address | Region |
|---|---|
| `0x0000 0000` | Alias. Shows flash, system memory, or SRAM. The `BOOT0` pin selects which one. |
| `0x0800 0000` | Flash (512 KB on the STM32F411RE) |
| `0x1FFF 0000` | System memory (ST built-in bootloader) |
| `0x2000 0000` | SRAM (128 KB) |
| `0x4000 0000` | Peripherals (APB1, APB2, AHB1, AHB2) |
| `0xE000 0000` | Cortex-M private peripherals: NVIC, SysTick, SCB, debug |

Find the base address of each peripheral in the "Memory map" table of RM0383.

### 2.3 The reset sequence

1. The core reads the 32-bit word at address `0x0000 0000`. This value is the initial `MSP`.
2. The core reads the word at address `0x0000 0004`. This value is the address of the reset handler.
3. The core jumps to the reset handler.

The first part of flash is therefore the **vector table**: the initial stack pointer, then the addresses of the reset handler and all exception and interrupt handlers.

### 2.4 Startup code

The reset handler does these steps before it calls `main()`:

1. Copy the `.data` section from flash (its LMA) to SRAM (its VMA).
2. Fill the `.bss` section with zeros.
3. Enable the FPU (set bits 20–23 of `SCB->CPACR`).
4. Optional: configure the system clock.
5. Optional: call the C library initialization (`__libc_init_array`) for constructors.
6. Call `main()`. If `main()` returns, stay in an endless loop.

You can write the startup code in C. You do not need assembly.

### 2.5 The linker script

A linker script tells the linker:

- Which memory regions exist (`MEMORY` block): `FLASH` at `0x08000000`, length 512K; `RAM` at `0x20000000`, length 128K.
- Where each section goes (`SECTIONS` block).
- Which symbols the startup code uses: `_sidata` (load address of `.data`), `_sdata`, `_edata`, `_sbss`, `_ebss`, `_estack`.

Important details:

- Put the vector table first in flash. Use `KEEP(*(.isr_vector))`. Without `KEEP`, the option `--gc-sections` removes it because no code refers to it.
- Use `AT> FLASH` for `.data`: the section runs in `RAM` but is stored in `FLASH`.
- Set `_estack` to the end of `RAM`. The stack grows down.

### 2.6 The clock system (RCC)

- After reset, the STM32F411 uses **HSI** (internal RC oscillator, 16 MHz).
- **HSE** is the external clock. On the Nucleo board it comes from the ST-LINK (8 MHz) or from a crystal. Exercise 0.8 asked you to find the source on your board.
- The **PLL** multiplies a clock. The maximum system clock of the STM32F411 is 100 MHz.
- `SYSCLK` → AHB prescaler → `HCLK` (core, AHB) → APB1 prescaler (max 50 MHz) and APB2 prescaler (max 100 MHz).
- **Before you increase the clock, set the flash wait states** (`FLASH->ACR`, `LATENCY` field). Flash cannot run at 100 MHz. The table in RM0383 gives the number of wait states for each frequency and supply voltage. If the wait states are too low, the CPU reads wrong instructions and crashes.
- For frequencies near 100 MHz, the voltage regulator must be in scale 1 (`PWR->CR`, `VOS` field).
- **Each peripheral has a clock enable bit** in an RCC register (`RCC->AHB1ENR`, `RCC->APB1ENR`, `RCC->APB2ENR`). A peripheral with no clock ignores all writes. This is the most common beginner bug.
- Open STM32CubeMX, select the STM32F411RE, and look at the "Clock Configuration" tab. It shows the full clock tree.

### 2.7 GPIO registers

| Register | Function |
|---|---|
| `MODER` | 2 bits per pin: input, output, alternate function, analog |
| `OTYPER` | 1 bit per pin: push-pull or open-drain |
| `OSPEEDR` | 2 bits per pin: edge speed. Use low speed unless you need fast edges. |
| `PUPDR` | 2 bits per pin: no pull, pull-up, pull-down |
| `IDR` | Input data (read) |
| `ODR` | Output data (read and write) |
| `BSRR` | Write-only. Bits 0–15 set pins. Bits 16–31 reset pins. **Atomic.** No read-modify-write. |
| `AFR[0]`, `AFR[1]` | 4 bits per pin: alternate function number |

### 2.8 Build, flash, and debug

- **Compile and link:** see the flags in [software-setup.md](../software-setup.md).

  | Flag | Purpose |
  |---|---|
  | `-mcpu=cortex-m4 -mthumb` | Generate Thumb-2 code for the Cortex-M4 |
  | `-mfloat-abi=hard -mfpu=fpv4-sp-d16` | Use the FPU and pass `float` in FPU registers |
  | `-ffunction-sections -fdata-sections` | Put each function and variable in its own section |
  | `-Wl,--gc-sections` | Remove sections that nothing uses |
  | `-Wl,-Map=x.map` | Write a map file: the address and size of each symbol |
  | `--specs=nano.specs` | Use newlib-nano, a small C library |
  | `-nostdlib` | Use no C library and no default startup files (optional for the first blinky) |

- **Output files:** `.elf` (with debug data), `.bin` (raw image, `objcopy -O binary`), `.hex` (Intel HEX).
- **Flash:** `openocd -f board/st_nucleo_f4.cfg -c "program build/blinky.elf verify reset exit"`. The Nucleo board also shows a USB drive. You can copy a `.bin` file onto it.
- **Debug with GDB:**

  ```
  openocd -f board/st_nucleo_f4.cfg          # terminal 1
  arm-none-eabi-gdb build/blinky.elf          # terminal 2
  (gdb) target extended-remote :3333
  (gdb) monitor reset halt
  (gdb) load
  (gdb) break main
  (gdb) continue
  (gdb) x/10wx 0x40020000                     # read GPIOA registers
  (gdb) info registers
  ```

- **Debug in VS Code:** Cortex-Debug with `"servertype": "openocd"` and the SVD file. You can see and change peripheral registers in the "Peripherals" view.

## Exercises

Put the code for this phase in `labs/phase-2-boot/`.

### Exercise 2.1: First build

1. Write the smallest firmware possible: a vector table with two entries (stack pointer and reset handler) and a reset handler that loops forever.
2. Write the linker script.
3. Build it. Run `arm-none-eabi-objdump -h` and `arm-none-eabi-objdump -d`. Find the vector table at `0x08000000`.
4. Look at the first 8 bytes of the `.bin` file with a hex viewer. Explain each byte.

### Exercise 2.2: Blinky with registers

1. Enable the clock for GPIOA in `RCC->AHB1ENR`.
2. Set PA5 as an output in `GPIOA->MODER`.
3. Toggle PA5 in a loop with a delay loop. Use `volatile` for the loop counter. Explain why.
4. Flash the board. LD2 blinks.

### Exercise 2.3: Debug the startup

1. Add an initialized global variable (`uint32_t g_counter = 42;`) and a zero global variable.
2. Set a breakpoint on the reset handler. Step through the `.data` copy and the `.bss` loop.
3. Read the values in SRAM with `x/` before and after the loops.
4. Comment out the `.data` copy. What is the value of `g_counter` in `main()`? Why?

### Exercise 2.4: Use BSRR

Change the blinky to use `BSRR` instead of `ODR`. Explain why `BSRR` is safe when an interrupt also changes other pins of the same port.

### Exercise 2.5: Read the button

1. Configure PC13 as an input.
2. Turn on LD2 while B1 is pressed. Use polling.
3. Do you need the internal pull-up? Use your answer to exercise 0.8.

### Exercise 2.6: Configure the clock

1. Calculate the PLL values for a 100 MHz system clock from HSI or HSE. Keep the PLL input frequency between 1 and 2 MHz. Do a check of your values in STM32CubeMX.
2. Set the voltage scale, the flash wait states, and the bus prescalers. Then start the PLL and switch `SYSCLK` to the PLL.
3. The blink rate of your delay loop increases. Calculate the expected change. Measure it.
4. Make an error on purpose: set zero wait states at 100 MHz. What happens?

### Exercise 2.7: Measure the clock

1. Configure MCO1 (PA8) or MCO2 (PC9) to output a clock (use a prescaler to keep the frequency low enough for your logic analyzer).
2. Measure the frequency with the logic analyzer in PulseView.

### Exercise 2.8: Use CMSIS

1. Download the CMSIS core headers and the STM32F4 device headers (`stm32f4xx.h`, `stm32f411xe.h`, `system_stm32f4xx.c`).
2. Replace your own register definitions with the CMSIS definitions.
3. Compare the code size in the map file before and after the change.

### Exercise 2.9: Build system

1. Write a CMake project with a toolchain file (`cmake/arm-none-eabi.cmake`).
2. Add targets `flash` and `size`.
3. Generate `compile_commands.json` for the editor.

## Project: Data logger, stage 0

See [../projects/data-logger.md](../projects/data-logger.md), stage 0.

Make a template project that all later phases copy:

```
labs/template/
├── CMakeLists.txt
├── cmake/arm-none-eabi.cmake
├── linker/stm32f411re.ld
├── startup/startup_stm32f411.c
├── src/main.c
├── src/clock.c, clock.h        # 100 MHz system clock
├── src/gpio.c, gpio.h          # your GPIO driver
├── third_party/cmsis/
└── .vscode/launch.json          # Cortex-Debug configuration
```

The firmware blinks LD2 at 1 Hz at a 100 MHz system clock. B1 changes the blink rate.

## Done criteria

- [ ] I can explain each step from reset to `main()`.
- [ ] I wrote a linker script and startup code that work.
- [ ] I can explain each line of the map file for my blinky.
- [ ] The system clock runs at 100 MHz. I measured it.
- [ ] I can flash and debug from the command line and from VS Code.
- [ ] I can read and change peripheral registers in the debugger.
- [ ] The template project builds with CMake.

## Common problems

| Problem | Cause |
|---|---|
| Nothing happens. The debugger shows the PC at an address that is not valid. | The vector table is not at `0x08000000`. Do a check of `KEEP` and the section name. |
| The GPIO does not change. | The peripheral clock is not enabled. |
| The program crashes after the clock change. | The flash wait states are too low, or the voltage scale is wrong. |
| A `float` operation causes a HardFault. | The FPU is not enabled in `CPACR`. |
| OpenOCD cannot connect. | The firmware puts the SWD pins (PA13, PA14) in a different mode, or the MCU is in a low-power mode. Hold the reset button, start the connection, then release the button ("connect under reset"). |

## Next

[Phase 3: Interrupts and timers](phase-3-interrupts-timers.md)
