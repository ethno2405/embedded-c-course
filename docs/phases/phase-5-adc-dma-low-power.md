# Phase 5: ADC, DMA, and Low Power

**Time:** 3–4 weeks
**Prerequisites:** Phase 4
**Hardware:** Your board (reference: Nucleo-F411RE), potentiometer, photoresistor, microSD module and card, DS3231 RTC, WS2812B strip, multimeter, logic analyzer
**Board:** The board details in this phase (pins, registers, manual sections, tool commands) are for the reference board. On a different board, translate them with your board documents, or ask the AI mentor. See [../boards.md](../boards.md).

## Goal

At the end of this phase, you can:

- Measure analog voltages with the ADC.
- Move data with DMA, with no CPU work for each byte.
- Use a file system on an SD card.
- Keep time with an RTC.
- Put the MCU in a low-power mode and measure the current.
- Use a watchdog.

## Reading

| Source | Part |
|---|---|
| RM0383 | Sections: "Analog-to-digital converter (ADC)", "DMA controller (DMA)", "Real-time clock (RTC)", "Power controller (PWR)", "Independent watchdog (IWDG)" |
| White, *Making Embedded Systems* | The chapters about peripherals, data handling, and power |
| ST application note AN4031 | "Using the STM32F2, STM32F4 and STM32F7 Series DMA controller" |
| ST application note AN4365 | "Using STM32F4 MCU power modes with best dynamic efficiency" |
| FatFs documentation (elm-chan.org) | "Application Interface" and "Device Control Interface" |
| SD Association, "Physical Layer Simplified Specification" | The section about SPI mode |
| WS2812B datasheet | Timing diagram |

## Topics

### 5.1 The ADC

- The STM32F411 has one 12-bit successive-approximation ADC (ADC1). It has 16 external channels and internal channels (temperature sensor, `VREFINT`, `VBAT`).
- Result: `value = Vin × 4095 / VREF`. On the Nucleo board, `VREF` is the 3.3 V supply. The supply is not exact. Measure `VREFINT` to calculate the real supply voltage.
- **Sample time:** a longer sample time gives a more accurate result with a high-impedance source. Find the formula for the maximum source impedance in the datasheet.
- **Modes:** single conversion, continuous, scan (many channels in a sequence), and injected channels.
- **Trigger:** software, or a timer event. A timer trigger gives an exact sample rate.
- The ADC clock comes from APB2 through a prescaler. Do a check of the maximum ADC clock in the datasheet.
- Pin: A0 on the Nucleo board is PA0, ADC1 channel 0. Set the pin to analog mode in `MODER`.

### 5.2 DMA

- The STM32F411 has DMA1 and DMA2. Each has 8 streams. Each stream has a channel selection for the request source. The request mapping table in RM0383 tells you which stream and channel connect to each peripheral.
- Directions: peripheral to memory, memory to peripheral, memory to memory (DMA2 only).
- Settings: source and destination address, number of items, item size, address increment, priority.
- **Circular mode:** the DMA starts again at the start of the buffer automatically.
- **Half-transfer and transfer-complete interrupts:** in circular mode, process the first half of the buffer while the DMA fills the second half (double buffering).
- The Cortex-M4 has no data cache. DMA data in SRAM is always coherent with the CPU. On a Cortex-M7 you must manage the cache. Remember this for later.
- Clear all DMA flags before you start a new transfer. Disable the stream and wait until it is disabled before you change its configuration.

### 5.3 SD card and FatFs

- An SD card supports an SPI mode. It is slower than the native SD mode but needs only SPI.
- Initialization: 74+ clock cycles with CS high at 100–400 kHz, `CMD0` (go to idle), `CMD8` (interface condition), `ACMD41` (loop until ready), `CMD58` (read OCR, find SDHC). Then increase the SPI clock.
- Data is in 512-byte blocks. Read: `CMD17`. Write: `CMD24`.
- **FatFs** is a small FAT file system library. You implement the disk interface in `diskio.c`: `disk_initialize`, `disk_status`, `disk_read`, `disk_write`, `disk_ioctl`. You also implement `get_fattime()`, which returns the time from your RTC.
- **Power loss:** call `f_sync()` after each important write. A power loss during a write can damage the file system.

### 5.4 Real-time clock

- The internal RTC runs from the LSE (32.768 kHz crystal) or the LSI (internal, less accurate). It keeps time in low-power modes. Do a check in UM1724 if your board has the LSE crystal.
- The RTC is in the backup domain. You must unlock it (`PWR->CR`, `DBP` bit) before you write to it.
- The RTC has alarms and a periodic wake-up timer. They can wake the MCU from Stop mode.
- The DS3231 is an accurate external RTC with a temperature-compensated crystal. It uses I2C and stores time in BCD format.

### 5.5 Low-power modes

| Mode | What stops | How to wake | Typical use |
|---|---|---|---|
| Sleep | CPU clock only | Any interrupt | Wait for the next event in the main loop (`__WFI()`) |
| Stop | All clocks in the 1.2 V domain. SRAM and registers keep their values. | EXTI line, RTC alarm or wake-up | Long waits with a fast restart |
| Standby | Almost everything. SRAM content is lost. | Wake-up pin, RTC, reset | Very long sleep. The MCU restarts from reset. |

- After a wake-up from Stop mode, the system clock is HSI. **Configure the PLL again.**
- Set unused pins to analog mode. This reduces current.
- On the Nucleo board, remove jumper JP6 (IDD) and connect a multimeter in current mode across the jumper pins. Then you measure the MCU current only. The ST-LINK and the other parts of the board use more current, but this current does not go through JP6.
- The debugger can keep clocks on in low-power modes (`DBGMCU->CR`). Disconnect the debugger for real measurements.

### 5.6 Watchdog

- The **independent watchdog (IWDG)** runs from the LSI. It resets the MCU if the software does not refresh it in time.
- Key values: `0xCCCC` (start), `0xAAAA` (refresh), `0x5555` (unlock `PR` and `RLR`).
- Once started, the IWDG cannot be stopped (only a reset stops it).
- Refresh the watchdog in one place only, when all parts of the system are healthy. Do not refresh it in a timer interrupt. That interrupt can run while the main loop is stuck.
- After a reset, read `RCC->CSR` to find the reset cause (watchdog, pin, power-on, software). Log it.
- `DBGMCU` can stop the watchdog while the core is halted in the debugger.

### 5.7 WS2812B LEDs

- One data wire. Each LED takes 24 bits (green, red, blue), then passes the rest of the data to the next LED.
- Bit period 1.25 µs (800 kHz). A "0" bit is high for approximately 0.4 µs. A "1" bit is high for approximately 0.8 µs. The datasheet gives the tolerances.
- A low time longer than the reset time (50 µs or more, depends on the version) latches the data.
- Method: a timer in PWM mode with a period of 1.25 µs. The DMA writes a new `CCR` value for each bit from a buffer in memory.
- The WS2812B needs 5 V supply. Its data input threshold at 5 V is approximately 3.5 V. A 3.3 V signal can work, but it is not guaranteed. Use a level shifter (for example a 74AHCT125) for a reliable design.

## Exercises

### Exercise 5.1: Single ADC reading

1. Read a potentiometer on PA0 with a software trigger. Print the value and the voltage over UART.
2. Read `VREFINT`. Calculate the real supply voltage. Correct the potentiometer voltage with it.
3. Read the internal temperature sensor. Compare it with the BME280. Explain the difference.

### Exercise 5.2: ADC with timer trigger and DMA

1. Configure TIM2 to trigger the ADC at 10 kHz.
2. Use DMA2 in circular mode to fill a buffer of 1000 samples.
3. In the half-transfer and transfer-complete interrupts, calculate the minimum, maximum, and average of each half.
4. Toggle a debug pin in the DMA interrupts. Measure the interrupt rate with the logic analyzer.

### Exercise 5.3: UART transmit with DMA

1. Change the UART driver so that it sends with DMA.
2. Measure the CPU load before and after the change. Method: count loop iterations in the idle loop over one second.

### Exercise 5.4: SD card

1. Initialize the SD card in SPI mode. Print the response of each command.
2. Read block 0. Find the partition table or the FAT boot sector.
3. Integrate FatFs. Create a file, write a line, close it. Read the file on your PC.

### Exercise 5.5: RTC

1. Read and set the time on the DS3231. Add the commands `time` and `settime` to the command line.
2. Configure the internal RTC. Configure a wake-up every 10 seconds.

### Exercise 5.6: Low power

1. Measure the MCU current at 100 MHz in the main loop.
2. Add `__WFI()` to the idle loop. Measure again.
3. Enter Stop mode. Wake up with the RTC every 10 seconds. Blink LD2 for 10 ms after each wake-up. Measure the current in Stop mode.
4. Set all unused pins to analog mode. Measure again.
5. Calculate the battery life with a 2000 mAh battery.

### Exercise 5.7: Watchdog

1. Start the IWDG with a timeout of 2 seconds.
2. Add a command `hang` that enters an endless loop. Show that the MCU resets.
3. Print the reset cause at startup.

### Exercise 5.8: WS2812B with DMA

1. Generate the bit stream with a timer and DMA.
2. Do a check of the bit timing with the logic analyzer.
3. Show a rainbow animation. Show the temperature as a color.

### Exercise 5.9 (side project): Small oscilloscope

1. Sample PA0 at 100 kHz with the timer, ADC, and DMA.
2. Show the waveform on the TFT. Add a trigger on a rising edge.
3. Use PWM from another timer as the test signal.

## Project: Data logger, stage 2

See [../projects/data-logger.md](../projects/data-logger.md), stage 2.

## Done criteria

- [ ] I can sample an analog signal at an exact rate with a timer, the ADC, and DMA.
- [ ] I can use DMA in circular mode with double buffering.
- [ ] My logger writes CSV files with time stamps to the SD card.
- [ ] I measured the current in Run, Sleep, and Stop mode.
- [ ] My firmware uses the watchdog and logs the reset cause.

## Common problems

| Problem | Cause |
|---|---|
| The ADC value is noisy. | The source impedance is high and the sample time is short. Add a 100 nF capacitor at the input, or increase the sample time. |
| The DMA does not start. | Wrong stream or channel, flags from the last transfer are not cleared, or the peripheral DMA request is not enabled. |
| The SD card does not answer `CMD0`. | The SPI clock is too fast for initialization, or the card did not get the 74 clock cycles with CS high. |
| The clock is wrong after a wake-up. | The PLL was not configured again after Stop mode. |
| The current in Stop mode is high. | The debugger is connected, pins float, or other parts of the board use the measured supply. |

## Next

[Phase 6: Software architecture and RTOS](phase-6-architecture-rtos.md)
