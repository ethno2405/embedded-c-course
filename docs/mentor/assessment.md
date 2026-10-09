# Assessment

This document tells the AI mentor how to assess the knowledge of a learner. The result is the learner profile, `learner/profile.md`. The mentor uses the profile in every session.

## When to assess

| Situation | Action |
|---|---|
| `learner/profile.md` does not exist | Do a full assessment: mode selection, part A, part B, result |
| `learner/profile.md` does not exist, but `learner/progress.md` exists | Rebuild the profile. Recommend the quick mode. Keep the progress file and the current position. |
| The learner asks to re-assess one area ("Re-assess my C knowledge") | Do part B for that area only. Update the level after the learner agrees. |
| The learner completes a phase | Do the phase review (last section of this document) |

## Rules for the assessment

- Tell the learner that the assessment is not a test. Its only purpose is to select the correct start point and the correct explanation level.
- Ask **one question at a time**. Wait for the answer.
- Tell the learner that "I do not know" is a good answer. A guess gives a wrong level.
- Ask the learner not to look up answers.
- Do not explain the correct answer during part B, unless the learner asks. Give all corrections at the end, in the result summary.
- Keep a short record of each question and the result. You write it into the profile as evidence.

## Step 1: Mode selection

Show the three modes to the learner and ask them to select one:

| Mode | Time | Content | Accuracy |
|---|---|---|---|
| **Quick** | ~5 minutes | Part A only (self-report) | Low. The mentor trusts the answers and corrects the levels later from the learner's work. |
| **Standard** | ~15–20 minutes | Part A, then 2–4 adaptive questions for each area in part B | Good for most learners |
| **Full** | ~45 minutes | Part A, then 5–8 questions for each area in part B, with the practical items | High. Use it if the learner wants an exact start point. |

If the learner does not select a mode, use **standard**.

## Step 2: Part A, self-report (all modes)

Ask these questions. You can ask related questions together in one message.

1. Which programming languages do you know? For how many years? Which one do you know best?
2. How much C do you know? (none / I can read it / I wrote small programs / I use it at work)
3. Do you know how a CPU executes a program: registers, memory, the stack, interrupts? (no / a little / yes)
4. How much electronics do you know? (none / school level / hobby projects / professional)
5. Which tools do you use: git, a terminal, a debugger with breakpoints, a serial terminal?
6. Which operating system do you use?
7. Which hardware do you already have: a microcontroller board, tools (multimeter, logic analyzer, soldering iron), components? What is your budget for hardware?
8. How many hours per week can you work on the course?
9. What is your goal? (for example: hobby projects, a job in embedded systems, a specific project, or a feature such as Bluetooth, Wi-Fi, or low power)
10. How do you want the mentor to help when you are blocked? (hints first, and a solution only when I ask / a full solution with an explanation)
11. How detailed must the explanations be? (short / normal / very detailed)

In **quick** mode, set the levels from the self-report with the rubric below, and mark them "self-reported". Then go to step 4.

## Step 3: Part B, knowledge check (standard and full modes)

### Adaptive rule

For each area:

1. Use the self-report to select the first question. Self-report "none" → start at level 1. Self-report "a little" or "basic" → start at level 2. Self-report "good" or "professional" → start at level 3.
2. A correct answer → ask the next question at a higher level (or another question at level 3).
3. A wrong answer or "I do not know" → ask the next question at a lower level.
4. Stop when the level is clear. Standard mode: 2–4 questions for each area. Full mode: 5–8 questions for each area, including the practical items.
5. A partly correct answer counts as half. Ask one more question at the same level to decide.

You can change the wording of a question. You can make new questions at the same level. The expected answers below are the minimum for "correct".

### Area C: C language

| Level | Question | Expected answer |
|---|---|---|
| 1 | What does this code do? `int x = 1; int *p = &x; *p = 5;` | `p` holds the address of `x`. `*p = 5` writes 5 into `x`. |
| 1 | How does C know where a string ends? | A `'\0'` (zero) byte ends the string. |
| 1 | `int a[4]; a[4] = 1;` What happens? | Write outside the array. C does not check bounds. Undefined behavior. |
| 2 | Write one expression that sets bit 3 and clears bit 5 of `x`. | `x = (x \| (1u << 3)) & ~(1u << 5);` or the same with two statements |
| 2 | What does `static` mean at file scope? What does it mean on a local variable? | File scope: the name is private to the file. Local: the variable keeps its value between calls (static storage). |
| 2 | A function has the parameter `int arr[10]`. What is `sizeof(arr)` in the function? | The size of a pointer. The array parameter is a pointer. |
| 2 | What does `volatile` do? Give one use. | The compiler must do every read and write. Use: hardware registers, variables shared with an interrupt handler. |
| 3 | `uint8_t a = 0x80;` Is `(a << 1) == 0` true or false? Why? | False. `a` is promoted to `int`. The result is `0x100`. |
| 3 | On a CPU with 32-bit `int`: `uint16_t x = 0xFFFF; uint32_t y = x * x;` Is this well defined? | No. `x` is promoted to signed `int`. The product overflows `int`. Undefined behavior. |
| 3 | A `volatile uint32_t counter` is incremented in the main loop and in an interrupt handler. Is this safe? | No. `counter++` is read-modify-write. The interrupt can occur between the read and the write. Use a critical section or atomic operations. |
| 3 | Why is `float f = *(float *)&u32;` a problem? What is the correct method? | It breaks the strict aliasing rule. Use `memcpy`. |

### Area A: computer architecture

| Level | Question | Expected answer |
|---|---|---|
| 1 | What is the difference between RAM and flash memory? | RAM loses its content without power and is fast to write. Flash keeps its content and holds the program. |
| 1 | What does the program counter (PC) hold? | The address of the next instruction. |
| 2 | What is the stack used for? What happens when it overflows? | Local variables, return addresses, saved registers. Overflow corrupts other memory or causes a fault. |
| 2 | What happens to the running program when an interrupt occurs? | The CPU saves its state, runs the interrupt handler, then continues the program where it stopped. |
| 2 | What is memory-mapped I/O? | Peripheral registers have memory addresses. The program reads and writes them like memory. |
| 3 | Which steps happen on a microcontroller between reset and `main()`? | The CPU loads the stack pointer and the reset vector. Startup code copies `.data`, clears `.bss`, sets up clocks (optional), then calls `main()`. |
| 3 | In a little-endian CPU, `0x12345678` is at address `0x100`. Which byte is at `0x100`? | `0x78` |
| 3 | What is a race condition between an interrupt handler and the main loop? How do you prevent it? | Both change shared data, and the order of the operations is not controlled. Use critical sections, atomic operations, or a single-writer design. |

### Area E: electronics

| Level | Question | Expected answer |
|---|---|---|
| 1 | 3.3 V is across a 1 kΩ resistor. What is the current? | 3.3 mA |
| 1 | Why does an LED need a series resistor? | To limit the current. Without it, the current is too high and damages the LED or the driver. |
| 2 | Calculate the resistor for an LED: supply 3.3 V, forward voltage 2.0 V, current 5 mA. | 260 Ω. Use the next standard value, 270 Ω. |
| 2 | A button connects a digital input to GND. With the button open, the input reads random values. Why? How do you fix it? | The input floats. Add a pull-up resistor (or enable the internal pull-up). |
| 2 | How do you measure current with a multimeter? | Connect the meter in series, with the probe in the current input. |
| 3 | A divider of two 10 kΩ resistors is on 3.3 V. A 10 kΩ load connects from the output to GND. What is the output voltage? | The load is in parallel with the lower resistor: 5 kΩ. Output = 3.3 × 5 / 15 = 1.1 V. |
| 3 | Why must a GPIO pin not drive a motor directly? What do you use? | The current is too high for the pin, and the motor makes voltage spikes. Use a transistor or a driver, and a flyback diode. |
| 3 | A sensor has a 5 V output. The MCU uses 3.3 V. What do you check before you connect them? | Whether the pin is 5 V tolerant, in the datasheet. If it is not, use a level shifter or a voltage divider. |

### Area T: tools

| Level | Question | Expected answer |
|---|---|---|
| 1 | What is the difference between the compiler and the linker? | The compiler makes object files from source files. The linker combines object files and libraries into one program. |
| 1 | What does `git commit` do? | It records the staged changes as a new version in the local repository. |
| 2 | How do you find the value of a variable at a specific line with a debugger? | Set a breakpoint on the line, run, and inspect the variable when the program stops. |
| 2 | What is a serial terminal? What is a baud rate? | A program that sends and receives text over a serial port. The baud rate is the number of symbols per second. Both sides must use the same rate. |
| 3 | What is cross-compilation? | Compilation on one machine (the host) for a different CPU (the target). |
| 3 | What is a linker script? What is a map file? | The linker script tells the linker where to put sections in memory. The map file lists where each symbol went and its size. |
| 3 | Why is debugging harder with `-O2` than with `-O0` or `-Og`? | The optimizer removes, reorders, and combines code and variables. Lines and variables do not map directly. |

### Practical items (full mode only)

Ask 1–2 items for each area, at the level of the learner.

**C, code reading.** Find the bugs:

```c
char name[8];
strcpy(name, "temperature");
```

Expected: the string needs 12 bytes, the buffer has 8. Buffer overflow.

```c
uint8_t i;
for (i = 0; i < 300; i++) {
    process(i);
}
```

Expected: `i` can never reach 300. The loop never ends.

```c
int *make_buffer(void) {
    int buf[16];
    return buf;
}
```

Expected: returns a pointer to a local variable. The memory is not valid after the return.

**Architecture.** Explain what this code does and why `volatile` is necessary:

```c
#define REG (*(volatile uint32_t *)0x40020014u)
REG |= (1u << 5);
```

Expected: read the register at that address, set bit 5, write it back. Without `volatile`, the compiler can remove or combine the accesses.

**Electronics, circuit.** A 10 kΩ pull-up connects an input to 3.3 V. A button connects the input to GND. What is the input voltage with the button open and closed? What is the current with the button closed?

Expected: open 3.3 V, closed 0 V, current 0.33 mA.

**Tools.** Explain the steps from a `.c` file to a program that runs on a microcontroller.

Expected: preprocess, compile, assemble, link with a linker script, convert to a binary or hex file (optional), flash it with a programmer or debugger.

## Level rubric

Give each area (C, A, E, T) a level from 0 to 3:

| Level | Name | Description |
|---|---|---|
| 0 | None | Cannot answer level 1 questions. |
| 1 | Basic | Answers most level 1 questions. Fails most level 2 questions. |
| 2 | Working | Answers most level 2 questions. Fails most level 3 questions. |
| 3 | Strong | Answers most level 3 questions. |

In quick mode, use the self-report: "none" → 0, "a little" / "I can read it" / "school level" → 1, "I wrote small programs" / "hobby projects" → 2, "I use it at work" / "professional" → 3. Mark these levels "self-reported".

## Learning path

Use the levels to make the learning path. Write the decisions in the profile.

| Condition | Decision |
|---|---|
| C = 0 or 1 | Phase 1 starts with topic 1.0. Plan 4–6 weeks for phase 1. |
| C = 2 | Skip topic 1.0. Do all other topics and exercises of phase 1. Plan 2 weeks. |
| C = 3 | Read phase 1 quickly. Do exercises 1.2, 1.4, 1.6, and 1.8 only. Plan 1 week. |
| E = 0 or 1 | Do all of phase 0. Explain every electrical term at first use. |
| E = 2 | In phase 0, read topics 0.6, 0.9, 0.12, and 0.13. Do exercises 0.5, 0.7, 0.8, and 0.9. |
| E = 3 | In phase 0, do exercises 0.8 and 0.9 only. |
| A = 0 or 1 | Before phase 2, watch the first lessons of the Quantum Leaps course. Explain the reset sequence and the memory map with diagrams. |
| A = 3 | In phases 2 and 3, focus on the STM32-specific details. |
| T = 0 or 1 | Before phase 2, teach git, the terminal, and the build steps. Spend extra time on `docs/software-setup.md`. |
| No hardware yet | Start phase 0 and phase 1 topics that do not need hardware. Help the learner select a board (`docs/mentor/hardware-selection.md`) and order the phase 0–3 items. |
| A board that is not the reference board | Plan extra time in phases 2–7 for the adaptation. The more the board is different, the more time. |
| Fewer than 4 hours per week | Tell the learner to double the phase times in `docs/README.md`. |

## Step 4: Result

1. Write `learner/profile.md` from `docs/templates/profile-template.md`. Fill in all sections. Put the evidence for each level (the questions and the results).
2. Show the learner a summary: the level of each area, the learning path, and the corrections for the questions that they answered wrong.
3. Ask the learner whether the summary is correct. Change the profile if the learner disagrees with good reasons.
4. If `learner/progress.md` does not exist, create it from `docs/templates/progress-template.md`. Set the start date to today.
5. Offer the hardware selection (`docs/mentor/hardware-selection.md`). If the learner postpones it, set the "Hardware" status in the profile to "Not selected". If the profile already has a selected board, skip this step.
6. If `learner/progress.md` already existed before the assessment, do not change the current position. Tell the learner their next step from the progress file. Otherwise, tell the learner the first step of their learning path.

## Phase review

When the learner says that they completed a phase:

1. Ask 3–5 questions from the "Done criteria" list of the phase. Use questions that need an explanation, not a yes or no.
2. Ask for evidence of the practical criteria: measurements, logic analyzer captures, or code in `labs/`.
3. If the learner meets all criteria, mark the phase "Done" in `learner/progress.md`.
4. If some criteria are not met, tell the learner which ones, and suggest the exercises to repeat.
5. If the results show that a level in the profile changed, propose the new level to the learner. Change it after the learner agrees. Add the review to the "Assessment history" in the profile.
