# Phase 3: Interrupts and Timers

**Time:** 3 weeks
**Prerequisites:** Phase 2
**Hardware:** Your board (reference: Nucleo-F411RE), LEDs, resistors, push buttons, passive buzzer, SG90 servo, logic analyzer
**Board:** The board details in this phase (pins, registers, manual sections, tool commands) are for the reference board. On a different board, translate them with your board documents, or ask the AI mentor. See [../boards.md](../boards.md).

## Goal

At the end of this phase, you can:

- Configure the NVIC and write interrupt handlers.
- Share data between an interrupt handler and the main code safely.
- Make a 1 ms system tick with SysTick.
- Generate PWM signals and measure pulses with hardware timers.
- Debounce a button in software.

## Reading

| Source | Part |
|---|---|
| Yiu, *Definitive Guide* | The chapters about exceptions, the NVIC, and SysTick |
| PM0214 | Sections about the NVIC, SCB, and SysTick |
| RM0383 | Sections: "Interrupts and events" (EXTI), "System configuration controller" (SYSCFG), "General-purpose timers (TIM2 to TIM5)" |
| White, *Making Embedded Systems* | The chapters about interrupts and about timers |
| Quantum Leaps (Samek) | The lessons about interrupts and race conditions |

## Topics

### 3.1 The exception model

- An **exception** stops the normal program flow and runs a handler. Interrupts from peripherals are a type of exception.
- The core saves `R0`–`R3`, `R12`, `LR`, `PC`, and `xPSR` on the stack automatically (the "stack frame"). A handler is therefore a normal C function.
- `LR` gets a special value (`EXC_RETURN`). When the handler returns, the core restores the registers.
- **Tail chaining:** If a second interrupt waits, the core starts its handler without a full restore and save.
- The vector table has the handler addresses. The CMSIS startup file gives each handler a name, for example `EXTI15_10_IRQHandler`. Your startup code must use the same names.

### 3.2 The NVIC and priorities

- `NVIC_EnableIRQ(irq)` enables an interrupt. `NVIC_SetPriority(irq, prio)` sets its priority.
- The STM32F4 uses 4 priority bits: priorities 0–15. **A lower number is a higher priority.**
- A higher-priority interrupt can preempt a lower-priority handler ("nesting").
- `__disable_irq()` and `__enable_irq()` change `PRIMASK`. They block all interrupts with configurable priority.
- `BASEPRI` blocks only the interrupts below a given priority. An RTOS uses it (phase 6).

### 3.3 External interrupts (EXTI)

To make an interrupt from PC13:

1. Enable the SYSCFG clock (`RCC->APB2ENR`).
2. Select port C for EXTI line 13 in `SYSCFG->EXTICR[3]`.
3. Select the edge in `EXTI->RTSR` (rising) or `EXTI->FTSR` (falling).
4. Unmask line 13 in `EXTI->IMR`.
5. Enable `EXTI15_10_IRQn` in the NVIC.
6. In the handler, clear the pending bit: write `1` to bit 13 of `EXTI->PR`. **If you do not clear it, the handler runs again and again.**

### 3.4 Data shared with an interrupt handler

- Declare shared variables `volatile`.
- A 32-bit aligned read or write is atomic on Cortex-M. A read-modify-write is **not** atomic. A 64-bit value is **not** atomic.
- **Race condition example:** The main code does `counter++`. An interrupt also does `counter++` between the read and the write of the main code. One increment is lost.
- Solutions:
  - A short critical section: save `PRIMASK`, disable interrupts, do the operation, restore `PRIMASK`.
  - A single-writer design: for example the ring buffer from exercise 1.6.
  - The exclusive access instructions `LDREX`/`STREX` (CMSIS: `__LDREXW`, `__STREXW`).
- **Rules for handlers:** keep them short, do not wait in a loop, do not call `printf`. Set a flag or put data in a buffer. Do the work in the main loop.

### 3.5 SysTick

- SysTick is a 24-bit down counter in the Cortex-M core.
- Load `SysTick->LOAD` with `HCLK / 1000 - 1` for a 1 ms tick. CMSIS has `SysTick_Config(ticks)`.
- In `SysTick_Handler`, increment a `volatile uint32_t` counter. This gives you a `millis()` function.
- **Wrap-around:** a 32-bit millisecond counter wraps after approximately 49.7 days. Always compare time with subtraction: `if ((uint32_t)(now - start) >= timeout)`. This code works across the wrap.

### 3.6 Hardware timers

- The STM32F411 has advanced timers (TIM1), general-purpose timers (TIM2–TIM5, TIM9–TIM11). TIM2 and TIM5 have 32-bit counters.
- Main registers: `PSC` (prescaler), `ARR` (auto-reload, the period), `CNT` (counter), `CCRx` (capture/compare for channel x).
- **Update frequency:** `f = f_timer / ((PSC + 1) × (ARR + 1))`.
- **Timer clock:** If the APB prescaler is not 1, the timer clock is two times the APB clock. Look at the clock tree in RM0383.
- Modes:
  - **Update interrupt:** periodic interrupt.
  - **PWM:** the output is high while `CNT < CCR` (PWM mode 1). Duty cycle = `CCR / (ARR + 1)`.
  - **Input capture:** the timer copies `CNT` into `CCR` on an input edge. Use it to measure a period or a pulse width.
  - **Output compare:** the timer changes an output when `CNT == CCR`.
- Connect a timer channel to a pin with the alternate function. Example: PA5 has `TIM2_CH1` on AF1. So LD2 can show PWM.
- Set the `ARPE` and `OCxPE` bits (preload). Then a new `ARR` or `CCR` value takes effect only at the next update event. This prevents glitches.

### 3.7 Debounce

- A button bounces for 1–20 ms. One press can make many EXTI interrupts.
- A good method: sample the button every 1–5 ms (in the SysTick handler or with a timer). Accept a new state only after N equal samples.
- Write the debounce as a small state machine: `RELEASED`, `MAYBE_PRESSED`, `PRESSED`, `MAYBE_RELEASED`.
- Add events: press, release, long press.

## Exercises

### Exercise 3.1: SysTick and non-blocking blink

1. Make a 1 ms tick with SysTick. Write `uint32_t millis(void)`.
2. Blink LD2 at 2 Hz without a delay loop. The main loop must not wait.
3. Add a second LED on another pin that blinks at 3 Hz. Both blink independently.

### Exercise 3.2: Button interrupt

1. Configure an EXTI interrupt on PC13, falling edge.
2. Count the interrupts in a `volatile` counter. Toggle LD2 in the main loop when the counter changes.
3. Connect an external button (without hardware debounce) to another pin. Count the interrupts for one press. Capture the bounce with the logic analyzer. Write down the bounce time.

### Exercise 3.3: Debounce

1. Write the debounce state machine from topic 3.7.
2. Detect short press and long press (more than 1 s).
3. Write unit tests for the state machine on your PC. Feed it sample sequences.

### Exercise 3.4: Race condition

1. Increment a global `uint32_t` in the main loop and in a 10 kHz timer interrupt. Do not use a critical section.
2. After 10 seconds, compare the total with the expected total. Find the lost increments.
3. Fix the problem with a critical section. Then fix it with `LDREX`/`STREX`.

### Exercise 3.5: PWM

1. Generate a 1 kHz PWM on PA5 with TIM2 channel 1. LD2 fades in and out.
2. Measure the frequency and the duty cycle with the logic analyzer.
3. Play tones on the passive buzzer. Change `ARR` to change the frequency. Keep 50 % duty.
4. Control the SG90 servo: 50 Hz period, pulse 1–2 ms. Move it from end to end. **Supply the servo from 5 V, not from 3.3 V. Connect the grounds.**

### Exercise 3.6: Input capture

1. Generate a PWM on one timer.
2. Connect it with a wire to an input capture channel of another timer.
3. Measure the period and the pulse width. Print the values in the debugger (or in phase 4, over UART).

### Exercise 3.7: Priorities

1. Make two timer interrupts with different priorities. The low-priority handler waits 2 ms in a loop (only for this experiment).
2. Toggle a different pin at the start and at the end of each handler. Capture the pins with the logic analyzer.
3. Show that the high-priority handler preempts the low-priority handler. Then give both the same priority. What changes?

## Side project: Simon game

- 4 LEDs, 4 buttons, a passive buzzer.
- The game plays a random sequence of LEDs and tones. The player repeats it. Each round adds one step.
- Use non-blocking timing (no delay loops), your debounce module, and PWM tones.
- Random seed: the time in microseconds of the first button press.

## Done criteria

- [ ] I can configure an EXTI interrupt and a timer interrupt.
- [ ] I can explain the stack frame and `EXC_RETURN`.
- [ ] I can show a race condition and fix it.
- [ ] My `millis()` comparison works across the 32-bit wrap.
- [ ] I can calculate `PSC` and `ARR` for a given frequency.
- [ ] I can generate PWM and measure it with the logic analyzer.
- [ ] My debounce module has unit tests.

## Common problems

| Problem | Cause |
|---|---|
| The handler runs continuously. | The pending flag is not cleared. |
| The handler never runs. | The handler name does not match the vector table name, or the NVIC line is not enabled, or the peripheral interrupt is not enabled. |
| The timer frequency is two times the expected value. | The timer clock is doubled because the APB prescaler is not 1. |
| The PWM has glitches when you change the duty cycle. | Preload is not enabled. |
| The servo shakes. The board resets. | The servo uses too much current from the board supply. Use a separate 5 V supply with a common ground. |

## Next

[Phase 4: Serial protocols](phase-4-serial-protocols.md)
