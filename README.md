# Embedded C Course

A self-study course in embedded programming with C. You learn electronics basics, microcontroller architecture, interrupts, timers, serial protocols, DMA, low power, RTOS, and professional practices. You build a real device: an environmental data logger.

- **For:** anyone who can program in at least one language. You do not need to know C or electronics.
- **Hardware:** a 32-bit microcontroller board that you select, and modules and tools for approximately $150–250 in total. The documents use the STM32 Nucleo-F411RE as the reference board. You can use a different board: the AI mentor suggests boards for your goals and budget, and adapts the course to your board. See [docs/boards.md](docs/boards.md) and [docs/hardware.md](docs/hardware.md).
- **Time:** approximately 6 months for phases 0–7 at 6–8 hours per week. Phase 8 is open-ended.

You can use the course alone, or with an AI coding agent as your mentor. The mentor assesses your knowledge, adapts the explanations to your background, reviews your work, and tracks your progress.

## Quick start with an AI mentor

1. Clone the repository:

   ```sh
   git clone https://github.com/ethno2405/embedded-c-course.git
   cd embedded-c-course
   ```

2. Open an AI coding agent in the repository root. The agent must be able to read and write files. Examples: Claude Code (`claude`), OpenAI Codex CLI, GitHub Copilot in agent mode, Cursor, Gemini CLI. The agent reads its instructions from [AGENTS.md](AGENTS.md) (Claude Code reads [CLAUDE.md](CLAUDE.md), which imports `AGENTS.md`).

3. Type the first prompt:

   ```
   Start the course.
   ```

4. The mentor offers three assessment modes. Select one:

   | Mode | Time | Content |
   |---|---|---|
   | Quick | ~5 minutes | Questions about your background only |
   | Standard | ~15–20 minutes | Background questions and a short adaptive quiz |
   | Full | ~45 minutes | Background questions, a longer quiz, and practical items (code reading, circuit calculations) |

5. The mentor saves your profile and your progress in the `learner/` directory. Then it tells you where to start.

In each later session, open the agent in the repository root and type `Continue the course.` The agent has no memory of earlier sessions. It reads your profile and progress from `learner/`.

## Your data

- The `learner/` directory contains `profile.md` (background, levels, preferences, mentor observations) and `progress.md` (position, completed work, session log).
- **Git ignores `learner/`.** Your data stays on your computer. It is not committed or pushed.
- Make a backup of `learner/` yourself if you want to keep it, for example before you delete the clone or move to a different computer.
- You can read and edit both files. The mentor uses the content that it finds.
- Your code in `labs/` is **not** ignored. At the end of each session, the mentor asks whether you want to commit your changes in `labs/`. It pushes only when you agree, and only to a remote that you own (for example your fork).

## Export and import your data

Your state is in two places: the `learner/` directory (local files) and the `labs/` directory (your code, in git).

**Export** (from the old clone):

1. End the session with `Save my progress and end the session.` The mentor updates both files and offers to commit your code.
2. Copy the full `learner/` directory (both `profile.md` and `progress.md`) to a safe location.
3. Push your commits in `labs/` to a remote that you own (your fork, or your own repository).

**Import** (into the new clone or fork):

1. Clone your fork, or pull your code into the new clone.
2. Copy the `learner/` directory into the root of the new clone.
3. Start the agent and type `Continue the course.` The mentor continues from the "Current position" in `progress.md`.

**Partial data:**

| You copied | Result |
|---|---|
| `profile.md` and `progress.md` | The mentor continues where you stopped. |
| Only `progress.md` | The mentor keeps your position and offers a short assessment to rebuild your profile. |
| Only `profile.md` | The mentor asks for your current phase and completed exercises, and makes a new progress file. |
| Nothing | The mentor asks whether you started before. If not, it starts a new assessment. |

**Course version:** the progress file refers to phase and exercise numbers. If the new clone has a different version of the course documents, the mentor checks the position and corrects it with you.

## Start from scratch

A fork of a fork can contain the code of the previous learner in `labs/`. It never contains their `learner/` directory, because git ignores it. To start from scratch, reset your data.

**With the mentor:**

```
Reset the course. I want to start from scratch.
Reset only my progress.
Delete my code in labs/ and start phase 2 again.
```

The mentor asks what to reset (profile, progress, code, or all), shows what it will delete, and offers a backup. It deletes nothing before you confirm. Backups go to `learner-backup-<date>/` (ignored by git) and to a git tag `labs-backup-<date>`.

**Without the mentor** (run in the repository root):

Bash:

```sh
cp -r learner "learner-backup-$(date +%F)"         # optional backup of profile and progress
git tag "labs-backup-$(date +%F)"                  # optional backup of committed code
rm -rf learner                                     # delete profile and progress
git rm -r -q --ignore-unmatch -- labs && rm -rf labs   # delete all code
git commit -m "Remove learner code to start the course again"
```

PowerShell:

```powershell
$d = Get-Date -Format yyyy-MM-dd
Copy-Item -Recurse learner "learner-backup-$d"      # optional backup of profile and progress
git tag "labs-backup-$d"                            # optional backup of committed code
Remove-Item -Recurse -Force learner                 # delete profile and progress
git rm -r -q --ignore-unmatch -- labs; Remove-Item -Recurse -Force labs -ErrorAction SilentlyContinue   # delete all code
git commit -m "Remove learner code to start the course again"
```

Uncommitted changes in `labs/` are lost. Commit them before the reset if you want to keep them in the backup tag. Skip the `git commit` command if `labs/` had no committed files.

## Example prompts

### First session

```
Start the course.
Start the course. Use the quick assessment.
Start the course. I already know C well, so use the full assessment for electronics only.
```

### Each session

```
Continue the course.
What is my next step?
Summarize my progress.
Save my progress and end the session.
Commit my code in labs/.
```

### Learn a topic

```
Explain pull-up resistors at my level.
Explain the reset sequence of my microcontroller with a diagram.
Compare C structs and function pointers with the language that I know best.
Why does this peripheral need a clock enable bit?
```

### Exercises

```
I finished exercise 0.3. My measurements are: red LED 2.9 mA at 1.95 V, blue LED 0.4 mA at 2.8 V. Check them.
Give me a hint for exercise 2.6. Do not give the solution.
I am blocked on exercise 4.4. Show me the full solution and explain it.
```

### Code review

```
Review my code in labs/phase-2-boot.
Check my BME280 driver against the datasheet.
```

### Debugging

```
My I2C scanner finds no devices. Help me find the cause step by step.
The board does not start after I changed the clock configuration. What do I check first?
Here is my logic analyzer capture of the UART line: ... What is wrong?
```

### Quizzes and reviews

```
Quiz me on phase 0.
I completed phase 2. Do the phase review.
Re-assess my C knowledge.
```

### Profile and progress

```
Mark exercise 0.4 as done.
Update my profile: I now prefer full solutions.
I bought the phase 0–3 hardware. Update my progress.
Reset the course. I want to start from scratch.
```

### Hardware and setup

```
Help me choose a board. My budget is $30 and I want to learn Bluetooth later.
I already have a Raspberry Pi Pico 2. Can I use it for the course?
What must I buy for the next phase?
Help me install the toolchain on my computer.
Is it safe to connect this 5 V sensor output to PA0?
```

## Use the course without an AI mentor

Start with [docs/README.md](docs/README.md). It has the roadmap, and each phase document has its reading, exercises, and done criteria.

## Repository

| Path | Content |
|---|---|
| [AGENTS.md](AGENTS.md) | Instructions for the AI mentor |
| [docs/](docs/) | Course documents |
| [docs/mentor/](docs/mentor/) | Assessment and teaching instructions for the AI mentor |
| `labs/` | Your code (created during the course) |
| `learner/` | Your profile and progress (local, not committed) |
| `learner-backup-<date>/` | Backups made by a reset (local, not committed) |

## License

[MIT](LICENSE). Third-party code and vendor documents that you add during the course (CMSIS, FreeRTOS, FatFs, datasheets) keep their own licenses.
