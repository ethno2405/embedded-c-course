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

5. The mentor asks for your name and makes your learner branch, `learner/<name>`. It saves your profile and your progress in the `learner/` directory on this branch. Then it tells you where to start.

In each later session, open the agent in the repository root and type `Continue the course.` The agent has no memory of earlier sessions. It finds your learner branch, and reads your profile and progress from `learner/`.

## Your data

- The `learner/` directory contains `profile.md` (background, levels, preferences, mentor observations) and `progress.md` (position, completed work, session log).
- The `labs/` directory contains your code.
- You can read and edit all files. The mentor uses the content that it finds.

### Branches

The `master` branch is the course baseline. It contains only the course documents. Each learner has a branch `learner/<name>` that contains the course documents, the learner data, and the code of one learner.

| Branch | Content |
|---|---|
| `master` | Course documents |
| `learner/<name>` | Course documents, `learner/`, and `labs/` |

- At the end of each session, the mentor asks whether you want to commit your data and code. It commits only on your learner branch.
- On `master`, the file `learner/.gitignore` ignores all files in `learner/`, so the files cannot go to `master` by accident. When the mentor makes a learner branch, it clears this file in the first commit of the branch. After that, git tracks the learner data on the branch.
- The mentor pushes only when you agree, and only to a remote that you own (for example your fork), or to a remote where the owner permits learner branches.
- **Your profile contains personal data.** All persons who can read the remote can read your learner branch.
- Never merge a learner branch into `master`.

### Get course updates

The course documents change on `master`. To get the changes on your learner branch, merge `master`:

```sh
git switch master
git pull                       # if master has a remote
git switch learner/<name>
git merge master
```

A learner branch does not change the course documents, so the merge has no conflicts.

### Share your data with git

**Send** (from the old clone or computer):

1. End the session with `Save my progress and end the session.` The mentor updates your data and offers to commit it.
2. Push your learner branch: `git push -u origin learner/<name>`.

**Receive** (in the new clone, or for a different person):

1. Clone the repository, or fetch: `git fetch origin`.
2. Switch to the learner branch: `git switch learner/<name>`. Git makes a local branch from `origin/learner/<name>`.
3. Start the agent and type `Continue the course.` The mentor continues from the "Current position" in `progress.md`.

To see all learner branches, run `git branch -a --list "learner/*" "*/learner/*"`.

**Before you switch branches,** commit your changes. If `learner/` contains files other than `.gitignore` on `master` (local data from an older version of the course), copy the directory to a safe location first. A checkout can overwrite ignored files and does not give a warning.

**Partial data:**

| The branch contains | Result |
|---|---|
| `profile.md` and `progress.md` | The mentor continues where you stopped. |
| Only `progress.md` | The mentor looks for an earlier version in the git history. If there is none, it keeps your position and offers a short assessment to rebuild your profile. |
| Only `profile.md` | The mentor looks for an earlier version in the git history. If there is none, it asks for your current phase and completed exercises, and makes a new progress file. |
| Nothing | The mentor asks whether you started before. If not, it starts a new assessment. |

**Course version:** the progress file refers to phase and exercise numbers. If you merge a different version of the course documents, the mentor checks the position and corrects it with you.

## Start from scratch

The easiest way: make a new learner branch from `master`. Your old branch does not change.

```sh
git switch -c learner/<new-name> master
echo "# Track learner data on this branch." > learner/.gitignore
git add -- learner/.gitignore
git commit -m "Track learner data on this branch"
```

**With the mentor:**

```
Reset the course. I want to start from scratch.
Reset only my progress.
Delete my code in labs/ and start phase 2 again.
```

The mentor asks what to reset (profile, progress, code, or all), shows what it will delete, and offers a backup. It deletes nothing before you confirm. It resets only a learner branch, never `master`. The backup is a git tag `backup/<name>-<date>`.

**Without the mentor** (run in the repository root, on your learner branch):

Bash:

```sh
git tag "backup/<name>-$(date +%F)"                 # optional backup of committed data and code
git rm -r -q --ignore-unmatch -- learner/profile.md learner/progress.md labs   # delete profile, progress, and code
rm -rf learner/profile.md learner/progress.md labs   # delete the remaining local files
git commit -m "Remove learner data to start the course again"
```

PowerShell:

```powershell
git tag "backup/<name>-$(Get-Date -Format yyyy-MM-dd)"   # optional backup of committed data and code
git rm -r -q --ignore-unmatch -- learner/profile.md learner/progress.md labs   # delete profile, progress, and code
Remove-Item -Recurse -Force learner/profile.md, learner/progress.md, labs -ErrorAction SilentlyContinue   # delete the remaining local files
git commit -m "Remove learner data to start the course again"
```

Uncommitted changes are lost. Commit them before the reset if you want to keep them in the backup tag. Skip the `git commit` command if the branch had no committed learner files.

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
| `labs/` | Your code (created during the course, only on your learner branch) |
| `learner/` | Your profile and progress (only on your learner branch). On `master`, only `.gitignore`. |
| `learner-backup-<date>/` | Local backups of learner data (not committed) |

## License

[MIT](LICENSE). Third-party code and vendor documents that you add during the course (CMSIS, FreeRTOS, FatFs, datasheets) keep their own licenses.
