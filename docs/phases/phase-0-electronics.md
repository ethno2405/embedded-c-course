# Phase 0: Electronics Basics

**Time:** 2–3 weeks
**Prerequisites:** None
**Hardware:** Breadboard, jumper wires, 2 × AA battery holder, component kit, multimeter. A microcontroller board is optional in this phase.

## Goal

At the end of this phase, you can:

- Calculate and measure voltage, current, and resistance in a simple circuit.
- Select a resistor for an LED.
- Explain why a digital input needs a pull-up or a pull-down resistor.
- Explain logic levels and why 5 V can damage a 3.3 V microcontroller.
- Use a transistor as a switch.
- Read a simple schematic.
- Find the important data in a datasheet.

Software bugs crash a program. Hardware mistakes can destroy a component. This phase teaches you the rules that keep your hardware safe in the later phases.

## Reading

| Source | Part |
|---|---|
| Platt, *Make: Electronics* | The first experiments: voltage, current, resistance, switches, capacitors, transistors |
| SparkFun tutorials | "Voltage, Current, Resistance, and Ohm's Law", "How to Use a Breadboard", "How to Use a Multimeter", "Pull-up Resistors", "Logic Levels", "How to Read a Schematic", "Light-Emitting Diodes (LEDs)", "Transistors" |
| White, *Making Embedded Systems* | The chapter about reading schematics and datasheets |
| Scherz and Monk, *Practical Electronics for Inventors* | Use as a reference when a topic is not clear |

## Topics

### 0.1 Voltage, current, resistance, and power

- **Voltage (V, volts)** is the difference in electric potential between two points. Always measure voltage between two points. Usually one point is ground (GND, 0 V).
- **Current (I, amperes)** is the flow of charge through a conductor. The same current flows through all parts of a series circuit.
- **Resistance (R, ohms, Ω)** limits the current.
- **Ohm's law:** `V = I × R`. If you know two values, you can calculate the third value.
- **Power (P, watts):** `P = V × I = I² × R`. A component gets hot when it dissipates power. A standard through-hole resistor has a rating of 0.25 W.

**Example:** A 1 kΩ resistor is connected to 3.3 V. The current is `3.3 V / 1000 Ω = 3.3 mA`. The power is `3.3 V × 3.3 mA ≈ 11 mW`. This value is much less than 0.25 W.

### 0.2 Series, parallel, and the voltage divider

- **Series:** `R_total = R1 + R2`. The same current flows through each resistor.
- **Parallel:** `1 / R_total = 1 / R1 + 1 / R2`. Each resistor has the same voltage.
- **Kirchhoff's voltage law:** The sum of the voltages around a closed loop is zero.
- **Kirchhoff's current law:** The current that flows into a node is equal to the current that flows out of it.
- **Voltage divider:** Two resistors in series between `Vin` and GND. The voltage at the middle point is:

  ```
  Vout = Vin × R2 / (R1 + R2)
  ```

  R2 is the resistor between the middle point and GND. A potentiometer is a variable voltage divider. In phase 5 you read a potentiometer with the ADC.

### 0.3 Resistor values and the color code

- Resistors come in standard series. The E12 series has these values in each decade: 10, 12, 15, 18, 22, 27, 33, 39, 47, 56, 68, 82.
- A 4-band resistor has two digits, a multiplier, and a tolerance band. Example: brown-black-red-gold = 1, 0, × 100 = 1 kΩ, ±5 %.
- Always measure a resistor with the multimeter when you are not sure of its value.

### 0.4 LEDs

- An LED is a diode. Current flows only from the anode (+, long leg) to the cathode (−, short leg, flat side of the case).
- An LED has a **forward voltage** (`Vf`). Approximate values: red 1.8–2.0 V, yellow and green 2.0–2.2 V, blue and white 2.8–3.2 V. Find the exact value in the datasheet.
- **An LED always needs a series resistor.** Without the resistor, the current is too high and the LED or the GPIO pin can be damaged.
- Resistor formula:

  ```
  R = (Vsupply − Vf) / I_led
  ```

**Example:** Supply 3.3 V, red LED with `Vf = 2.0 V`, current 5 mA:
`R = (3.3 − 2.0) / 0.005 = 260 Ω`. Use the next higher E12 value: 270 Ω. Modern LEDs are bright at 2–5 mA. You do not need 20 mA.

### 0.5 Switches and pull-up/pull-down resistors

- A digital input has a very high input impedance. If nothing drives the input, the input **floats**. It reads random values because of electrical noise.
- A **pull-up resistor** connects the input to the supply. The input reads 1 when the button is open. When you push the button, the button connects the input to GND, and the input reads 0. This is "active low".
- A **pull-down resistor** connects the input to GND. The button connects the input to the supply.
- Typical value: 10 kΩ. A smaller value uses more current when the button is pressed. A larger value is more sensitive to noise.
- The STM32 has internal pull-up and pull-down resistors (approximately 40 kΩ). You enable them in a register in phase 2.
- **Switch bounce:** A mechanical switch does not close cleanly. It opens and closes many times in the first 1–20 ms. Software must ignore these transitions. Phase 3 teaches debounce.

### 0.6 Logic levels

- The STM32F411 uses **3.3 V logic**. Logic 0 is near 0 V. Logic 1 is near 3.3 V.
- The datasheet gives thresholds: `VIL` (maximum voltage that the input reads as 0) and `VIH` (minimum voltage that the input reads as 1). Between these values, the result is not defined.
- Many older modules and Arduino boards use **5 V logic**.
- **Some STM32 pins are 5 V tolerant ("FT" in the datasheet pin table). Other pins are not.** A 5 V signal on a pin that is not 5 V tolerant can damage the MCU. 5 V tolerance applies only when the pin is an input or an open-drain output, and only when the MCU has power.
- Use a level shifter or a voltage divider to connect a 5 V output to a 3.3 V input.

### 0.7 Capacitors

- A capacitor stores charge. It blocks DC and passes changes in voltage.
- **Decoupling capacitors** (typically 100 nF ceramic) are near each supply pin of a chip. They supply short current peaks and reduce noise. Every board that you see has them.
- **Bulk capacitors** (10–1000 µF electrolytic) keep the supply voltage stable when the load changes, for example when a motor starts.
- **Electrolytic capacitors have a polarity.** If you connect one with the wrong polarity, it can be damaged or burst.
- **RC time constant:** `τ = R × C`. A capacitor charges through a resistor to approximately 63 % of the supply voltage in one `τ`, and to approximately 99 % in 5 `τ`. Example: 10 kΩ × 100 µF = 1 s.
- Hardware debounce uses an RC filter. Some boards have one on the user button.

### 0.8 Diodes

- A diode lets current flow in one direction only. A silicon diode has a forward voltage of approximately 0.6–0.7 V.
- **Flyback diode:** A motor or a relay coil makes a high voltage spike when you switch it off. A diode in parallel with the coil (reverse direction) absorbs the spike. Always use one with inductive loads.

### 0.9 Transistors as switches

- A GPIO pin can supply only a small current. The STM32F411 datasheet gives the limits per pin and for all pins together. Find them in the "Absolute maximum ratings" section. **Do not use a GPIO pin to drive a motor, a relay, or a long LED strip directly.**
- An **NPN transistor** as a low-side switch:
  - The load connects between the supply and the collector.
  - The emitter connects to GND.
  - The GPIO pin connects to the base through a resistor (typically 1–4.7 kΩ).
  - A small base current switches a larger collector current.
- A **logic-level N-channel MOSFET** does the same job for larger currents. Its gate needs almost no current. Select a MOSFET that switches fully on at 3.3 V (look at `RDS(on)` at `VGS = 2.5 V` or `3.3 V` in the datasheet).
- **Common ground:** The MCU and the load supply must share GND. Without a common ground, the circuit does not work.

### 0.10 Power on the Nucleo board

- USB supplies 5 V. A regulator on the board makes 3.3 V for the MCU.
- The board has `5V`, `3V3`, and `GND` pins on the headers. You can use them to supply small circuits.
- **A short circuit between a supply pin and GND can damage the board or the USB port of your computer.** Disconnect the USB cable before you change a circuit.

### 0.11 Schematics

- A schematic shows the electrical connections. It does not show the physical layout.
- Learn the symbols: resistor, capacitor (with and without polarity), diode, LED, switch, NPN transistor, MOSFET, ground, supply.
- A **net label** connects points that have the same name. Two wires with the label `SDA` are connected, even if no line joins them.
- A **solder bridge** (SB) on a Nucleo board is a small pad that you can connect or disconnect with solder. It changes the board configuration.

### 0.12 Datasheets

A datasheet usually has these sections. Learn where to find each one:

| Section | What it tells you |
|---|---|
| Features / description | What the chip does |
| Pinout and pin description | What each pin does |
| **Absolute maximum ratings** | Limits that destroy the chip if you exceed them. Never operate the chip at these limits. |
| **Recommended operating conditions** | The range where the chip works correctly |
| Electrical characteristics | Thresholds, currents, timings, with minimum/typical/maximum values |
| Timing diagrams | The order and duration of signals (very important for protocols) |
| Register map | The control registers (for chips that you program) |
| Typical application circuit | A circuit that the manufacturer recommends. Start with it. |
| Package information | Physical size and pins |

**Rule:** Design with the minimum or maximum values, not with the typical values.

### 0.13 Tools

- **Breadboard:** Each row of 5 holes is connected. The two halves are not connected. The long supply rails at the sides are connected along their length. On some breadboards the rails have a break in the middle. Measure with the continuity function to be sure.
- **Multimeter:**
  - **Voltage:** Probes in parallel with the part. Black probe on GND.
  - **Current:** The meter goes **in series** with the circuit. Move the red probe to the current input (`mA` or `A`). **Never connect the meter in current mode across a supply. This makes a short circuit and can blow the fuse of the meter.** Move the red probe back to the `V` input after the measurement.
  - **Resistance:** Measure only when the circuit has no power. Remove the part from the circuit for an accurate value.
  - **Continuity:** Finds connections and broken wires. The meter beeps when the resistance is low.
  - **Diode mode:** Shows the forward voltage. A small LED glows dimly in this mode. Use it to find the polarity of an LED.

### 0.14 Safety

- Disconnect the power before you change a circuit.
- Do not short-circuit batteries. AA batteries can get hot. Lithium batteries can catch fire.
- Touch a grounded metal object before you handle chips. Electrostatic discharge (ESD) can damage them.
- Use a soldering iron in a ventilated area. Put the iron in its stand. Wash your hands after you use leaded solder.

## Exercises

Write each calculation, each measurement, and the difference between them in your lab notebook. Use the 2 × AA battery holder (approximately 3.0 V) as the supply, unless the exercise tells you to use something else.

### Exercise 0.1: Resistors

1. Take 5 resistors from your kit. Read the color code of each resistor.
2. Measure each resistor with the multimeter.
3. Calculate the difference in percent. Compare it with the tolerance band.

### Exercise 0.2: Ohm's law

1. Connect a 1 kΩ resistor across the battery supply.
2. Measure the supply voltage.
3. Calculate the current.
4. Measure the current with the multimeter in series. Compare it with the calculated value.
5. Repeat with 470 Ω and 10 kΩ.

### Exercise 0.3: LED and resistor

1. Measure the forward voltage of a red LED and a blue or white LED in diode mode.
2. Calculate the resistor for 3 mA from the 3.0 V supply for each LED.
3. Build the circuit. Measure the current and the voltage across the LED.
4. Change the resistor to 2 times and to 0.5 times the value. Write down the change in brightness and current.
5. Question: Why does a blue LED at 3.0 V give much less current than a red LED with the same resistor?

### Exercise 0.4: Voltage divider

1. Build a divider with two 10 kΩ resistors. Calculate and measure `Vout`.
2. Change R2 to 4.7 kΩ. Calculate and measure again.
3. Replace the resistors with a 10 kΩ potentiometer. Measure `Vout` at several positions.
4. Connect a 1 kΩ load resistor from `Vout` to GND. Measure `Vout` again. Explain the change. (Hint: the load is in parallel with R2.)

### Exercise 0.5: Pull-up resistor and a button

1. Connect a 10 kΩ resistor from the supply to a node. Connect a push button from the node to GND.
2. Measure the node voltage with the button open and with the button closed.
3. Calculate the current through the resistor when the button is closed.
4. Remove the resistor. Measure the node voltage with the button open. Touch the node with your finger. Write down what you see. This is a floating input.
5. Question: Why does a pull-up of 100 Ω waste energy? Why can a pull-up of 1 MΩ cause problems?

### Exercise 0.6: RC time constant

1. Connect a 10 kΩ resistor and a 100 µF electrolytic capacitor in series across the supply. Observe the polarity.
2. Calculate `τ`.
3. Measure the capacitor voltage with the multimeter while it charges. Write down the voltage every second for 5 seconds.
4. Compare the values with the expected curve: 63 % at `τ`, 86 % at `2τ`, 95 % at `3τ`.
5. Disconnect the supply. Discharge the capacitor through a resistor before you remove it.

### Exercise 0.7: Transistor switch

1. Build an NPN low-side switch. Load: LED with its resistor. Base resistor: 4.7 kΩ.
2. Connect the base resistor to the supply with a wire. The LED turns on. Connect it to GND. The LED turns off.
3. Measure the base current and the collector current. Calculate the current gain.
4. Question: In phase 3, a GPIO pin drives this base resistor. Why does this protect the pin?

### Exercise 0.8: Read the Nucleo schematic

Open UM1724 (Nucleo-64 user manual). Find the schematic of the board at the end of the document. Answer these questions. If you use a different board, use the manual and the schematic of your board, and answer the same questions for it.

1. Which MCU pin connects to the user LED LD2? Which resistor limits its current? Calculate the LED current.
2. Which MCU pin connects to the user button B1? Is there an external pull-up resistor? Is there a capacitor? What is the level of the pin when the button is pressed?
3. Which MCU pins connect to the ST-LINK virtual COM port?
4. What is the source of the HSE clock (the external high-speed clock) on your board revision? Look at the section about the oscillator configuration and the solder bridges.
5. What does jumper JP6 (IDD) do?
6. Find the MCU supply pins. How many decoupling capacitors do you find near them?

### Exercise 0.9: Read the STM32F411 datasheet

Open the STM32F411xC/E datasheet. Answer these questions. If you use a different board, use the datasheet of your MCU, and find the matching pins.

1. What is the absolute maximum supply voltage `VDD`?
2. What is the maximum current that one I/O pin can source or sink? What is the maximum total current for all I/O pins?
3. Find the pin table. Is PA5 5 V tolerant? Is PC13? What does the note about PC13 current say?
4. What are `VIL` and `VIH` for a standard I/O pin at `VDD = 3.3 V`?
5. Which alternate functions does PA2 have?

### Exercise 0.10 (optional): Soldering

1. Solder the headers on one module, for example the SSD1306 OLED.
2. Put the long side of the header into the breadboard. Put the module on top. The breadboard holds the pins straight.
3. Solder one pin. Do a check that the module is straight. Then solder the other pins.
4. Do a check of each joint with the continuity function. Also do a check that no two neighbor pins are connected.

## Project: Hardware reference card

Make a one-page reference card for your bench. Write it in your lab notebook or in [../topics/](../topics/). It contains:

- Ohm's law, the power formula, the voltage divider formula, and the LED resistor formula
- The forward voltages that you measured for your LEDs
- The resistor color code
- The Nucleo pins for LD2, B1, and the virtual COM port
- The absolute maximum ratings of the STM32F411 that you found
- The list of 5 V tolerant pins that you will use

## Done criteria

- [ ] I can calculate a current or a voltage with Ohm's law and confirm it with a measurement.
- [ ] I can select a resistor for an LED at a given current.
- [ ] I can explain a floating input and how a pull-up resistor fixes it.
- [ ] I can measure current safely with the multimeter in series.
- [ ] I can explain why a 5 V signal can damage a 3.3 V pin.
- [ ] I can switch a load with an NPN transistor.
- [ ] I can find the LED, button, and UART pins in the Nucleo schematic.
- [ ] I can find the absolute maximum ratings and logic thresholds in a datasheet.

## Common problems

| Problem | Cause |
|---|---|
| The LED does not turn on. | The LED has the wrong polarity, or the circuit uses the wrong breadboard rows. |
| The multimeter shows 0 A. | The red probe is in the `V` input, or the meter is not in series. The fuse can also be blown. |
| The measured voltage changes when you touch the circuit. | A node floats. Add a pull-up or a pull-down resistor. |
| The circuit works on one half of the breadboard only. | The supply rail has a break in the middle. |

## Next

[Phase 1: C for embedded systems](phase-1-c-for-embedded.md). You can do phase 1 in parallel with this phase.
