---
title: "Day 2: Blinking an External LED & Understanding the Breadboard"
date: 2026-04-11T23:58:00+05:30
lastmod: 2026-04-11T23:58:00+05:30
tags: ["arduino", "beginner", "kids", "led", "breadboard", "resistor"]
categories: ["arduino"]
summary: "Day 2 of Learn Coding with Arduino — wire up your own LED on a breadboard, learn why resistors matter, and blink it from the Uno."
---

Welcome back, Anish. Yesterday we made the tiny built-in LED on the Arduino blink. Cool, but small. What if we want to blink a **bigger LED** of our own? That's where we go today. We're going to wire up an external LED on a breadboard and blink it with the same kind of code from Day 1.

## What you need today

- Arduino Uno + USB cable
- **Breadboard**
- 1 **LED** (any color — red is classic)
- 1 **220Ω resistor** (red-red-brown-gold stripes)
- 2 **jumper wires** (male-to-male)

## What is a breadboard?

A breadboard is a flat plastic board full of tiny holes. It is basically a **LEGO board for electricity**. Under the plastic, **some holes are already connected to each other** with little metal strips. That means if you plug a wire into hole A and an LED leg into a *connected* hole B, they are now touching — without any soldering.

{{< mermaid >}}
graph TB
    TOP["+ rail — all connected left-to-right (use for power)"]
    subgraph Middle["Middle area"]
        COL["Each vertical column of 5 holes<br/>is connected top-to-bottom<br/>(with a gap in the very middle)"]
    end
    BOT["− rail — all connected left-to-right (use for GND)"]
{{< /mermaid >}}

**Two rules to remember:**

1. The long rails on the top and bottom (marked `+` and `−`) are connected along the whole length. Use them for power and ground.
2. In the middle, each **column of 5 holes** is connected. The gap in the very middle splits the board into a top half and bottom half.

Once you see it a few times, it becomes obvious.

## Why do we need a resistor?

An LED is like a thirsty plant. If you give it too much water (too much electricity), it dies — sometimes with a little pop. The Uno sends out **5 volts** from each pin, which is way more than an LED can handle on its own.

The **resistor** is like a narrow straw. It slows the electricity down so the LED gets just the right amount. A **220Ω resistor** is a safe, gentle value for most LEDs with a 5V Arduino.

Rule: **every LED needs a resistor in series with it.** No resistor = burnt-out LED. Do not skip this.

## The LED has a + side and a − side

An LED only works **one way around**. Look at your LED carefully:

- The **longer leg** is the **positive** side (called the **anode**, `+`)
- The **shorter leg** is the **negative** side (called the **cathode**, `−`)

There is also a **flat spot** on the plastic rim near the shorter leg, in case the legs get bent.

If you plug the LED in backward, it won't blow up — it just stays dark. Flip it and try again.

## The circuit

We're going to use **pin 8** this time, not pin 13. Why? Because pin 13 is already wired to the built-in LED on the board. If we use pin 13, both LEDs would blink — the built-in one *and* our new one. Our external LED would not feel special. On any pin from 2 to 12, our LED gets the spotlight to itself.

{{< mermaid >}}
graph LR
    PIN8["Arduino<br/>Pin 8"] -->|jumper wire| R["220Ω<br/>Resistor"]
    R --> ANODE["LED long leg<br/>( + anode )"]
    ANODE --> CATHODE["LED short leg<br/>( − cathode )"]
    CATHODE -->|jumper wire| GND["Arduino<br/>GND"]
{{< /mermaid >}}

**Step-by-step on the breadboard:**

1. Plug the LED into the breadboard so the two legs are in **different columns** (e.g. long leg in column 10, short leg in column 11). If both legs were in the same column, they'd be shorted — nothing would light.
2. Plug one end of the 220Ω resistor into the same column as the **long leg** of the LED. Plug the other end of the resistor into an empty column (e.g. column 8).
3. Use a jumper wire from **Arduino pin 8** to column 8 on the breadboard (where the free end of the resistor is).
4. Use a second jumper wire from the **GND pin** on the Arduino to the column with the LED's **short leg**.

That's the whole circuit. Electricity flows: **Pin 8 → resistor → LED long leg → LED short leg → GND**. A loop, all the way back to the Uno.

## The code

Open a new sketch in the Arduino IDE and type this in:

```cpp
void setup() {
  pinMode(8, OUTPUT);     // Pin 8 is an output now (not pin 13)
}

void loop() {
  digitalWrite(8, HIGH);  // LED ON
  delay(500);             // wait half a second
  digitalWrite(8, LOW);   // LED OFF
  delay(500);             // wait half a second
}
```

Click **Upload**. Your external LED should blink on and off every half second.

If nothing happens:

- Is the LED plugged in the right way around? (long leg toward the resistor)
- Is the resistor in the same column as the long leg?
- Is the GND wire going to a GND pin on the Uno (not 5V)?
- Is the jumper going from pin 8 (not pin 13)?

## What is new in today's code?

Notice how **all four instructions** from Day 1 are still here: `pinMode`, `digitalWrite`, `delay`. The only thing that changed is the **pin number**:

- Day 1: `pinMode(13, OUTPUT)` → controls the built-in LED
- Day 2: `pinMode(8, OUTPUT)` → controls *our* LED wired to pin 8

That's it. You already knew how to blink; we just moved the blink to a different pin. This is the big idea: **Arduino code for any output pin looks almost identical**. Once you can do one, you can do them all.

Also notice we used `delay(500)` instead of `delay(1000)`. That's half a second instead of a full second — twice as fast. Same instruction, different number.

## Try this

1. **Move the LED** to pin 9. Change `pinMode(8, OUTPUT)` to `pinMode(9, OUTPUT)` and both `digitalWrite(8, ...)` lines to `digitalWrite(9, ...)`. Unplug the jumper wire that went to pin 8, move it to pin 9. Upload. Still blinks, right? You just changed pins with a tiny code edit.
2. **Reverse the LED** (take it out, flip it, plug it back in). What happens? Nothing — it stays dark. That's polarity. Put it back the right way.
3. **Remove the resistor** — but only for 1 second. Touch the LED with your finger as you upload. Do you feel it getting hot? Put the resistor back right away. LEDs without resistors die fast.

## Bonus: Two LEDs blinking alternately

If you have a second LED and a second 220Ω resistor, try this. Add a second LED on **pin 12** exactly the same way you wired the first one.

```cpp
void setup() {
  pinMode(8, OUTPUT);
  pinMode(12, OUTPUT);
}

void loop() {
  digitalWrite(8, HIGH);   // LED 1 ON
  digitalWrite(12, LOW);   // LED 2 OFF
  delay(500);
  digitalWrite(8, LOW);    // LED 1 OFF
  digitalWrite(12, HIGH);  // LED 2 ON
  delay(500);
}
```

Two LEDs taking turns — like a police car light. See how we just **added more instructions** inside the same `loop()`? Arduino runs them top to bottom, then starts the loop over. That's how you build anything: a little more, a little more, a little more.

## What you learned today

- How a breadboard is wired inside
- Why every LED needs a **resistor** (so you don't burn it out)
- LEDs have a **+ side (long leg)** and a **− side (short leg)**
- Any digital pin on the Arduino can control an external LED
- You can control **multiple pins** by calling `pinMode` and `digitalWrite` for each one
- Code that worked for pin 13 works for pin 8 — just change the number

## What is next

[Day 3](/arduino/day-3-button-input/) — we flip it around. Instead of the Arduino telling something what to do, the Arduino **listens** to a button and reacts. You will learn your first **input** and your first **if statement**.

Nice work, Anish.
