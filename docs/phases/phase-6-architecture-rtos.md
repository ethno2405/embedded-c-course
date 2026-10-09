# Phase 6: Software Architecture and RTOS

**Time:** 3–4 weeks
**Prerequisites:** Phase 5
**Hardware:** Your board (reference: Nucleo-F411RE) with the stage 2 data logger hardware, logic analyzer
**Board:** The board details in this phase (pins, registers, manual sections, tool commands) are for the reference board. On a different board, translate them with your board documents, or ask the AI mentor. See [../boards.md](../boards.md).

## Goal

At the end of this phase, you can:

- Select an architecture for firmware: super loop, state machine, event-driven, or RTOS.
- Write flat and hierarchical state machines.
- Use FreeRTOS tasks, queues, semaphores, mutexes, and notifications.
- Find and fix concurrency problems: race conditions, deadlocks, and priority inversion.
- Measure stack usage, CPU load, and timing jitter.

## Reading

| Source | Part |
|---|---|
| White, *Making Embedded Systems* | The chapters about software architecture and state machines |
| Barry, *Mastering the FreeRTOS Real Time Kernel* | All chapters |
| freertos.org | "FreeRTOS on ARM Cortex-M" (`RTOS-Cortex-M3-M4.html`): interrupt priorities |
| Quantum Leaps (Samek) | Lessons about RTOS, state machines, and active objects |
| DigiKey / Shawn Hymel | "Introduction to RTOS" series |
| Yiu, *Definitive Guide* | The chapter about OS support: SVC, PendSV, and the PSP |

## Topics

### 6.1 Architectures without an RTOS

- **Super loop:** `while (1) { task_a(); task_b(); }`. Simple. The response time depends on the slowest function. No function can wait.
- **Super loop with interrupts and flags:** handlers set flags; the loop does the work. This is your design until now.
- **Cooperative scheduler:** a table of functions and periods. Each function runs to completion and returns quickly.
- **Event-driven design:** events go into a queue. A dispatcher sends each event to the handler. With `__WFI()` when the queue is empty, this design uses little power.

### 6.2 State machines

- A **finite state machine** has states, events, transitions, and actions.
- Implementation styles:
  - `switch` on the state, then `switch` on the event.
  - A table of `(state, event) → (action, next state)`.
  - A function pointer for each state: the state is the function that handles events.
- **Hierarchical state machines (HSM, UML statecharts):** a sub-state inherits the transitions of its parent state. This removes duplicate transitions. The QP framework from Quantum Leaps implements them.
- **Active object:** a thread or task with a private event queue and a state machine. No shared data. Objects communicate only with events.

### 6.3 RTOS concepts

- **Task:** a function with its own stack. It runs as if it has the CPU alone.
- **Scheduler:** selects the ready task with the highest priority. **Preemptive:** a higher-priority task that becomes ready runs immediately.
- **Tick:** a periodic interrupt (usually 1 ms) for delays and time slices.
- **Task states:** Running, Ready, Blocked (waits for time or for an event), Suspended.
- **Context switch on Cortex-M:** the `PendSV` exception saves `R4`–`R11` (and the FPU registers if necessary) on the task stack, changes the `PSP`, and restores the registers of the next task. `SVC` starts the first task.

### 6.4 FreeRTOS objects

| Object | Use it for |
|---|---|
| Task | An independent activity: `xTaskCreate` or `xTaskCreateStatic` |
| `vTaskDelay` / `vTaskDelayUntil` | Wait for a relative time / a fixed period without drift |
| Queue | Send copies of data between tasks, and from an interrupt to a task |
| Binary semaphore | Signal an event |
| Counting semaphore | Count events or free resources |
| Mutex | Protect a shared resource. Has **priority inheritance**. |
| Task notification | A fast, light signal to one task. Good for "interrupt → task". |
| Event group | Wait for a combination of events |
| Software timer | Call a function after a time, in the timer service task |
| Stream / message buffer | Send a stream of bytes (for example UART data) from one writer to one reader |

### 6.5 Interrupts and FreeRTOS

- In an interrupt handler, use only the API functions that end with `FromISR`. Then call `portYIELD_FROM_ISR(higher_priority_task_woken)`.
- `configMAX_SYSCALL_INTERRUPT_PRIORITY` (or `configMAX_API_CALL_INTERRUPT_PRIORITY`): an interrupt that calls a FreeRTOS function must have a **numerically equal or higher** priority value (a **logically lower or equal** priority). This is the most common FreeRTOS bug on Cortex-M. Enable `configASSERT` to find it.
- Interrupts with a higher logical priority than this limit are never delayed by FreeRTOS. Use them for very fast events, but they must not call FreeRTOS functions.
- The STM32 HAL also uses SysTick. If you use FreeRTOS and the HAL together, use a different timer for the HAL time base.

### 6.6 Memory and stack

- Heap schemes: `heap_1` (no free), `heap_2`, `heap_3` (`malloc` wrapper), `heap_4` (free with merge, most common), `heap_5` (many regions). Or use static allocation for all objects (`configSUPPORT_STATIC_ALLOCATION`).
- Each task has its own stack. A stack that is too small corrupts memory without a clear error.
- `configCHECK_FOR_STACK_OVERFLOW = 2` and `vApplicationStackOverflowHook` find most overflows.
- `uxTaskGetStackHighWaterMark()` gives the smallest free stack space that the task had.
- `printf` and `snprintf` with `float` use much stack. Measure it.
- newlib is not thread-safe by default. Configure `configUSE_NEWLIB_REENTRANT`, or do not call newlib functions from many tasks.

### 6.7 Concurrency problems

- **Race condition:** two tasks change shared data without protection. Use a mutex, or do not share the data (send it in a queue).
- **Deadlock:** task A holds mutex 1 and waits for mutex 2. Task B holds mutex 2 and waits for mutex 1. Fix: always take mutexes in the same order. Use timeouts.
- **Priority inversion:** a low-priority task holds a resource. A high-priority task waits for it. A medium-priority task runs and blocks the low-priority task. The high-priority task waits for a long time. The Mars Pathfinder lander had this problem in 1997. Fix: a mutex with priority inheritance.
- **Starvation:** a high-priority task never blocks. Lower-priority tasks never run.

### 6.8 Measurements

- **CPU load:** enable run-time statistics (`configGENERATE_RUN_TIME_STATS`) with a fast timer. `vTaskGetRunTimeStats()` prints the time for each task.
- **Timing and jitter:** toggle a GPIO pin when a task starts and ends. Measure with the logic analyzer.
- **SEGGER SystemView:** shows a timeline of tasks and interrupts. It needs a J-Link or RTT support. Optional.

## Exercises

### Exercise 6.1: State machine without an RTOS

1. Write the data logger control logic as a state machine: `IDLE`, `MEASURING`, `WRITING`, `ERROR`, `SLEEPING`.
2. Use an event queue and a dispatcher.
3. Draw the state diagram in `docs/topics/` before you write the code.

### Exercise 6.2: First FreeRTOS project

1. Add the FreeRTOS kernel source and the `GCC/ARM_CM4F` port to the template project.
2. Write `FreeRTOSConfig.h`. Enable `configASSERT`, stack overflow checks, and static allocation.
3. Make two tasks that blink two LEDs at different rates with `vTaskDelayUntil`.

### Exercise 6.3: Producer and consumer

1. Task A reads the BME280 every second and sends the result to a queue.
2. Task B takes values from the queue and prints them.
3. Make the queue small and task B slow. What happens when the queue is full? Use a timeout and count the lost values.

### Exercise 6.4: Interrupt to task

1. Change the UART RX handler. It now writes to a stream buffer.
2. A command line task reads from the stream buffer and blocks when it is empty.
3. Set the UART interrupt priority wrong on purpose. Show that `configASSERT` finds the error.

### Exercise 6.5: Priority inversion

1. Make three tasks with low, medium, and high priority. The low and high tasks share a resource protected by a binary semaphore. The medium task uses the CPU for a long time.
2. Show the inversion with debug pins and the logic analyzer.
3. Change the binary semaphore to a mutex. Show that the inversion is solved.

### Exercise 6.6: Stack and CPU measurement

1. Print the stack high water mark of each task with a command `tasks`.
2. Enable run-time statistics. Print the CPU time of each task.
3. Reduce the stack of one task until the overflow hook runs.

### Exercise 6.7: Low power with FreeRTOS

1. Enable the tickless idle mode (`configUSE_TICKLESS_IDLE`).
2. Measure the current. Compare it with the result of phase 5.

## Project: Data logger, stage 3

See [../projects/data-logger.md](../projects/data-logger.md), stage 3.

## Done criteria

- [ ] I can explain when a super loop is enough and when an RTOS is better.
- [ ] I can write a state machine in two different styles.
- [ ] I can explain how the Cortex-M switches between tasks.
- [ ] I can configure interrupt priorities for FreeRTOS correctly.
- [ ] I showed priority inversion and fixed it.
- [ ] I measured the stack usage and the CPU load of each task.
- [ ] The data logger runs on FreeRTOS.

## Common problems

| Problem | Cause |
|---|---|
| HardFault when the first task starts. | The `SVC`, `PendSV`, and `SysTick` handler names do not match the FreeRTOS port names. Map them in `FreeRTOSConfig.h` or in the vector table. |
| `configASSERT` fails in `vPortValidateInterruptPriority`. | An interrupt that calls a FreeRTOS function has a priority that is too high. |
| Random crashes. | A task stack is too small. |
| A task never runs. | A higher-priority task never blocks. |
| `printf` output from two tasks is mixed. | Two tasks write to the UART at the same time. Use a mutex or one logging task with a queue. |

## Next

[Phase 7: Professional practices](phase-7-professional-practices.md)
