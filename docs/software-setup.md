# Software Setup (Windows)

This document tells you which tools to install and how to do a check of each tool. All tools work natively on Windows. WSL2 also works, but USB access from WSL2 needs `usbipd-win`.

The tools are for the reference board (STM32 Nucleo-F411RE). The general tools (git, CMake, Ninja, VS Code, PulseView, a serial terminal, a host compiler) are the same for all boards. For a different board, replace the vendor tools, the debug configuration, the SVD file, and the compiler flags:

| Board family | Vendor tools | Debug server | Compiler |
|---|---|---|---|
| STM32 (all) | STM32CubeMX, STM32CubeProgrammer | OpenOCD or probe-rs, with the board or target file for your MCU | `arm-none-eabi-gcc` with the `-mcpu` of your core |
| Raspberry Pi Pico 2 | Pico SDK, `picotool` | OpenOCD (Raspberry Pi version) or probe-rs, with the Debug Probe | `arm-none-eabi-gcc` (`-mcpu=cortex-m33`) |
| Nordic nRF52840 | nRF Connect SDK, nRF Command Line Tools | SEGGER J-Link or probe-rs | `arm-none-eabi-gcc` (`-mcpu=cortex-m4`) |
| ESP32-C3 / S3 | ESP-IDF (`idf.py`) | OpenOCD (Espressif version) through the built-in USB JTAG | The ESP-IDF toolchain |

Ask the AI mentor for the exact setup for your board.

## 1. Install the tools

Install these tools in this order. Add each `bin` directory to the `PATH` variable if the installer does not do it.

| Tool | Source | Purpose |
|---|---|---|
| Git | git-scm.com | Version control |
| Arm GNU Toolchain (`arm-none-eabi`) | developer.arm.com, "Arm GNU Toolchain Downloads" | Compiler, linker, GDB, objdump |
| CMake | cmake.org | Build system generator |
| Ninja | github.com/ninja-build/ninja | Fast build tool |
| OpenOCD | github.com/xpack-dev-tools/openocd-xpack | Flash and debug server |
| probe-rs (optional) | probe.rs | Flash and debug tool. Works with C firmware too. |
| STM32CubeMX | st.com | Clock tree and pin assignment viewer |
| STM32CubeProgrammer | st.com | Flash tool, ST-LINK firmware update, option bytes |
| STM32CubeIDE (fallback) | st.com | Eclipse IDE with all tools included |
| VS Code | code.visualstudio.com | Editor |
| PulseView | sigrok.org | Logic analyzer software |
| Serial terminal: Tera Term or PuTTY | teratermproject.github.io, putty.org | Serial console |
| Python 3 and `pyserial` | python.org | Scripts and `miniterm` |
| A host C compiler (MSYS2 GCC, or LLVM/clang) | msys2.org, llvm.org | Host exercises and unit tests |

### VS Code extensions

- **C/C++** (Microsoft) or **clangd**
- **CMake Tools**
- **Cortex-Debug** (marus25)
- **Serial Monitor** (Microsoft), optional

### ST-LINK driver

Windows needs the ST-LINK USB driver. STM32CubeProgrammer installs it. If you do not install CubeProgrammer, install `STSW-LINK009` from st.com.

### SVD file

Cortex-Debug shows peripheral registers when you give it an SVD file. Download `STM32F411.svd` from the STM32F4 "System View Description" package on st.com, or from the `cmsis-svd` repository on GitHub.

## 2. Do a check of the tools

Run these commands in a new terminal. Each command must print a version.

```sh
git --version
arm-none-eabi-gcc --version
arm-none-eabi-gdb --version
cmake --version
ninja --version
openocd --version
python -m serial.tools.list_ports
```

## 3. Do a check of the board connection (reference board)

1. Connect the Nucleo board with a USB cable. Use a cable that carries data, not only power.
2. Open STM32CubeProgrammer. Select "ST-LINK" and click "Connect". The tool must show the device "STM32F411xC/E".
3. In STM32CubeProgrammer, update the ST-LINK firmware ("Firmware upgrade").
4. Open Device Manager. Find "STMicroelectronics STLink Virtual COM Port (COMx)". Write down the COM number.
5. Start OpenOCD:

   ```sh
   openocd -f board/st_nucleo_f4.cfg
   ```

   OpenOCD must print a line that contains `Cortex-M4` and wait for connections on port 3333.

## 4. Compiler flags for the STM32F411 (reference board)

Use these flags for all firmware in this course:

```
-mcpu=cortex-m4 -mthumb -mfloat-abi=hard -mfpu=fpv4-sp-d16
-std=c11 -Wall -Wextra -Wshadow -Wconversion -g3 -Og
-ffunction-sections -fdata-sections
```

Linker flags:

```
-T<linker script> -Wl,--gc-sections -Wl,-Map=<name>.map --specs=nano.specs
```

Phase 2 explains each flag.
