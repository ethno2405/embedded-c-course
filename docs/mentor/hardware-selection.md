# Hardware Selection

This document tells the AI mentor how to help the learner select a board, and how to adapt the course to the selected board. The learner makes the decision. The mentor suggests.

## When to do the selection

| Situation | Action |
|---|---|
| At the end of the first assessment | Offer the selection. The learner can postpone it. |
| The learner asks ("Help me choose a board", "What must I buy?") | Do the selection. |
| The "Hardware" section of the profile says "Not selected", and the learner starts phase 2 or wants to order hardware | Do the selection before you continue. Phases 0 and 1 do not need a board. |
| The learner wants to change the board | Do the selection again. Record the old board in the notes. |
| The learner starts phase 8, module 8.9 (FPGAs) | Suggest an FPGA board from the "FPGA boards" section of `docs/boards.md`. Record it as a second board in the "Hardware" section of the profile. |

An FPGA board is never a main board for phases 2–7. If a learner asks to use only an FPGA board, explain that phases 2–7 need a microcontroller.

## Step 1: Ask

Ask these questions. Use the answers that are already in the profile, and do not ask them again.

1. Do you already have a microcontroller board? Which one?
2. What is your budget for the board and the debug probe?
3. Do you have a goal that needs a specific feature? (Bluetooth, Wi-Fi, CAN, USB, low power, motor control, a specific job or industry)
4. Do you prefer the easiest path through the documents, or do you want to adapt the course to a different board?
5. In which country do you buy? (Some boards are hard to get in some countries.)

## Step 2: Suggest

1. Use the comparison in `docs/boards.md`.
2. Suggest **2–3 boards**. Put the reference board first, unless the answers give a clear reason for a different board.
3. For each board, give: the price, the fit with the documents, the advantages for the learner's goal, and the disadvantages (extra probe, extra adaptation, availability).
4. If the learner has a board already, tell them its fit. If it is not suitable (for example an 8-bit board), explain why and suggest an alternative.
5. Do not invent specifications. If a board is not in `docs/boards.md`, tell the learner to do a check of its specifications at the vendor, and help them compare.

## Step 3: Record

When the learner selects a board, fill in the "Hardware" section of `learner/profile.md`:

- The board, the MCU, and the core
- The debug probe
- The reference documents for the board: reference manual, datasheet, programming manual of the core, board manual (with document numbers)
- The pin map for the signals that the course uses: user LED, user button, console UART (TX, RX), I2C (SCL, SDA), SPI (SCK, MISO, MOSI), an ADC input, a PWM output
- The flash and debug tool and its configuration (for example the OpenOCD board file)

Find the pins together with the learner in the board manual. Do not guess them. If the learner does not have the board yet, mark the pin map "to be checked".

Then update the hardware list in `learner/progress.md`: the items to buy for the selected board, from `docs/hardware.md`.

## Adapt the course to the board

In every session, if the board in the profile is not the reference board (STM32 Nucleo-F411RE):

1. Read the phase document as usual.
2. Before you give the learner a board-specific detail (a pin, a register, a bit, a manual section, a clock value, a tool command), translate it to the learner's board.
3. Use the reference documents in the profile. If you are not sure of a translated detail, tell the learner, and ask them to do a check in the named document.
4. If a feature does not exist on the learner's board (for example a peripheral), tell the learner. Suggest an alternative exercise that teaches the same concept.
5. If the translation of a phase is large, suggest a topic document in `docs/topics/` for the board (for example `board-pico2-phase-2.md`). Other learners with the same board can use it.

For the reference board, use the documents as they are.
