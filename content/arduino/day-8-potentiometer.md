---
title: "Day 8: Potentiometer, Serial Monitor, and Dimming an LED"
date: 2026-04-11T23:52:00+05:30
lastmod: 2026-04-11T23:52:00+05:30
tags: ["arduino", "beginner", "kids", "analog", "potentiometer", "serial", "pwm", "map"]
categories: ["arduino"]
summary: "Day 8 of Learn Coding with Arduino — your first analog input. Read a potentiometer knob, print numbers to the Serial Monitor, and use PWM to dim an LED smoothly."
---

Hi Anish! Today is a big day. Until now, every pin we've read has been **digital** — only HIGH or LOW, on or off, two choices. Today we meet **analog**, where a pin can give us **1,024 different values** instead of just 2. We'll use a **potentiometer** (a knob), print its value to your laptop screen with the **Serial Monitor**, and use it to smoothly dim an LED.

## What you need today

- Arduino Uno + USB cable
- Breadboard
- **1 potentiometer** (looks like a knob with 3 legs — usually 10kΩ)
- 1 **LED**
- 1 **220Ω resistor**
- 5 **jumper wires**

## What is a potentiometer?

A **potentiometer** (or "pot" for short) is a knob you can turn. Inside, it's a resistor with a sliding contact. As you twist the knob, the contact slides along the resistor, changing how much voltage comes out the middle leg.

It has **3 legs**:

- **Left leg** → 5V
- **Right leg** → GND
- **Middle leg** → variable voltage from 0V to 5V, depending on knob position

When the knob is turned all the way left, the middle leg gives 0V. All the way right, 5V. Anywhere in between, some value in between.

The key idea: the middle leg is now a **tunable voltage source**. And the Arduino can *read that voltage* using an analog pin.

## Digital vs Analog pins

Arduino has **two kinds of input pins**:

| Type | Pin labels | Values it can read |
|---|---|---|
| Digital | 0 through 13 | `HIGH` or `LOW` (2 values) |
| Analog | **A0 through A5** | **0 to 1023** (1,024 values) |

The analog pins are labeled **A0, A1, A2, A3, A4, A5** on the board — on the opposite side from the digital pins. They can read any voltage between 0V and 5V and convert it to a number:

- 0V → `0`
- 2.5V → `511` (roughly half)
- 5V → `1023`

In between, you get every number. That's 10x finer than digital.

## The circuit

{{< mermaid >}}
graph LR
    V5["5V"] --> POTa["Pot left leg"]
    POTb["Pot middle leg"] --> A0["Pin A0"]
    POTc["Pot right leg"] --> GND["GND"]
    PIN9["Pin 9 (PWM)"] --> R["220Ω"] --> LEDp["LED +"] --> LEDn["LED −"] --> GND2["GND"]
{{< /mermaid >}}

**Wiring the potentiometer:**

1. Plug the pot into the breadboard so its 3 legs sit in 3 different columns.
2. Jumper from **5V** to the **left leg**.
3. Jumper from **GND** to the **right leg**.
4. Jumper from the **middle leg** to **A0**.

**Wiring the LED:** same as always — **pin 9** → 220Ω → LED + → LED − → GND. We use pin 9 for a specific reason (see below).

## Why pin 9 for the LED?

Because pin 9 supports **PWM**, and PWM is how we'll dim the LED. Only certain pins have it.

On the Arduino Uno, **PWM pins** are marked with a tiny **`~`** symbol next to the number. They are pins **3, 5, 6, 9, 10, 11**. If you use a non-PWM pin with `analogWrite`, dimming won't work. So remember: for smooth brightness, use a `~` pin.

What's PWM? We'll get to it in a second — just wire pin 9 for now.

## Part 1: Read the knob with Serial Monitor

First, let's read the potentiometer and watch the values change. Upload this:

```cpp
void setup() {
  Serial.begin(9600);   // Open the pipeline to your laptop
}

void loop() {
  int val = analogRead(A0);
  Serial.println(val);
  delay(100);
}
```

Upload. Now click the **magnifying glass icon** in the top-right corner of the Arduino IDE (or press Ctrl+Shift+M / Cmd+Shift+M). That opens the **Serial Monitor** — a window where Arduino can print messages.

You should see a stream of numbers scrolling: `0`, `0`, `15`, `412`, `823`, `1023`, `1023`, `560`... Turn the knob slowly and watch the number change. Turn it all the way to one end — `0`. All the way to the other — `1023`. In the middle — about `512`.

**You're seeing what the Arduino sees.**

### New things in this code

#### `Serial.begin(9600);`

`Serial` is Arduino's way of talking back to your laptop over the USB cable — the same cable we use to upload code. `Serial.begin(9600)` opens the "pipeline" and sets the speed to 9,600 bits per second (that's the `9600`). You only need to do it once, in `setup()`.

`9600` is a standard speed — just remember it as "the normal one." If you change the speed in your code, you must also change the speed in the Serial Monitor window (there's a dropdown in the corner). Otherwise you'll see garbage characters instead of numbers.

#### `int val = analogRead(A0);`

`analogRead(A0)` gives back a number between `0` and `1023` based on the voltage on pin A0. We store that number in a variable called `val`. This is where the magic happens — **the knob position becomes a number in your code.**

#### `Serial.println(val);`

`Serial.println(...)` prints its argument to the Serial Monitor and adds a new line after. So each reading shows on its own line. `Serial.print(...)` (without `ln`) prints without starting a new line — useful for building up labels like `Serial.print("Pot: "); Serial.println(val);`.

**Try this:** change the loop to print something more helpful:

```cpp
Serial.print("Pot value: ");
Serial.println(val);
```

Now you see `Pot value: 512` instead of just a number. `Serial.println` is your best friend for debugging.

## Part 2: Dim the LED with the knob

Reading numbers is cool. Making something *happen* based on those numbers is cooler. Let's use the knob to control LED brightness.

```cpp
void setup() {
  pinMode(9, OUTPUT);
  Serial.begin(9600);
}

void loop() {
  int val = analogRead(A0);              // 0 to 1023
  int brightness = map(val, 0, 1023, 0, 255);  // squish to 0-255
  analogWrite(9, brightness);
  Serial.println(brightness);
  delay(20);
}
```

Upload. Turn the knob. The LED glows brighter or dimmer smoothly — not just on/off, but every shade in between. You're now controlling real-world brightness from a knob using code.

### What is `analogWrite`?

`analogWrite(pin, value)` sends a value from `0` (fully off) to `255` (fully on) to a pin. Anything in between is a partial brightness.

"But wait," you say, "earlier you said pins are only HIGH or LOW. How can a pin be 60% on?"

Great question. The answer is **PWM — Pulse Width Modulation**. The pin is actually still either HIGH or LOW, but Arduino switches it on and off *super fast* — hundreds of times per second. If it's HIGH 50% of the time and LOW 50% of the time, the LED appears 50% bright (your eyes can't see the flickering; it happens too fast). If it's HIGH 10% of the time, 10% bright. And so on.

**`analogWrite(9, 128)` really means "keep pin 9 HIGH about half the time."** Your eyes average it out to "half brightness."

This is why PWM only works on the specific **`~` pins** — those pins have the special hardware to flicker fast enough to fool your eyes.

### What is `map()`?

`analogRead` gives us 0-1023. But `analogWrite` needs 0-255. They don't match! We need to **squish** one range into the other.

```cpp
map(val, 0, 1023, 0, 255)
```

The `map` function says: *"take the value `val`, which is on a scale from 0 to 1023, and rescale it to a scale from 0 to 255."* If `val` is 0, you get 0. If `val` is 1023, you get 255. If `val` is in the middle (512), you get half (roughly 127).

`map(value, fromLow, fromHigh, toLow, toHigh)` — five arguments, rescaling from one range to another. Super useful when two parts of your project speak different number ranges.

## Try this

1. **Print both values.** Change the Serial output to show both `val` (the raw reading) and `brightness` (the scaled version), side by side. Use `Serial.print()` and `Serial.println()`.
2. **Reverse the direction.** Make the LED brightest when the knob is all the way left instead of right. Hint: swap the last two arguments in `map()` — `map(val, 0, 1023, 255, 0)`.
3. **Knob controls delay.** Go back to blinking — but use the pot to control the blink speed. Read the pot, map it to something like `50` to `1000`, then use that number as your delay. Turn the knob, speed changes live.
4. **Dim two LEDs in opposite directions.** Wire a second LED on pin 10. Make one brighter as the other gets dimmer, controlled by the same knob.

## What you learned today

- **Analog** input vs **digital** input
- Arduino has 6 analog pins: **A0-A5**, each reading 0 to 1023
- **`analogRead(A0)`** — read a voltage as a number 0-1023
- **`Serial.begin(9600)`** — open the text pipeline to your laptop
- **`Serial.print(...)`** and **`Serial.println(...)`** — print messages and values
- How to open the **Serial Monitor** in the Arduino IDE
- **PWM** (Pulse Width Modulation) — the trick to "analog output" from a digital pin
- **`analogWrite(pin, 0-255)`** — fade a pin smoothly (only works on `~` pins)
- **`map(val, fromLow, fromHigh, toLow, toHigh)`** — rescale a number from one range to another

## What is next

[Day 9](/arduino/day-9-light-sensor/) — we replace the knob with a **light sensor** (an LDR). Arduino learns to "see" — it reads how bright the room is and reacts. Next step toward making an automatic night light on Day 10.

Great work, Anish. You just unlocked analog. Every sensor from here on uses this idea.
