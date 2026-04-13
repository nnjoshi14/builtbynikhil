---
title: "Day 9: Light Sensor — Arduino Learns to See"
date: 2026-04-11T23:51:00+05:30
lastmod: 2026-04-11T23:51:00+05:30
tags: ["arduino", "beginner", "kids", "sensor", "ldr", "analog", "threshold"]
categories: ["arduino"]
summary: "Day 9 of Learn Coding with Arduino — wire up a photoresistor (LDR) to read how bright the room is, and use a threshold to decide if it's dark enough to turn on a light."
---

Hi Anish! Yesterday we read a knob. Today we read **light**. A tiny component called an **LDR** acts like a light-powered knob — when the room is bright, it sends one value to Arduino; when the room is dark, it sends another. Arduino is about to "see."

## What you need today

- Arduino Uno + USB cable
- Breadboard
- **1 LDR** (photoresistor — looks like a round disc with a squiggly line on top)
- **1 × 10kΩ resistor** (not the 220Ω we've been using for LEDs!)
- 1 **LED**
- 1 **220Ω resistor** (for the LED)
- Jumper wires

## What is an LDR?

**LDR** stands for **Light-Dependent Resistor** — also called a **photoresistor**. Its resistance changes based on how much light hits it:

- **Bright light** → low resistance (lets lots of current through)
- **Dark** → high resistance (blocks current)

By itself, an LDR doesn't give you a voltage. To turn it into a voltage Arduino can read, we build a **voltage divider** — a fancy name for two resistors in a row. The LDR is one resistor. A fixed **10kΩ resistor** is the other.

Don't panic about the theory. The practical rule is: **LDR on top, 10kΩ on the bottom, read the voltage from the middle.**

## The circuit

{{< mermaid >}}
graph TB
    V5["5V"] --> LDR1["LDR leg 1"]
    LDR1 --> MID["Junction<br/>(middle point)"]
    MID --> A0["Pin A0"]
    MID --> R10k["10kΩ resistor"]
    R10k --> G1["GND"]
    PIN8["Pin 8"] --> R220["220Ω"] --> LEDp["LED +"] --> LEDn["LED −"] --> G2["GND"]
{{< /mermaid >}}

**Wiring the LDR (as a voltage divider):**

1. Plug the LDR into the breadboard — doesn't matter which leg goes where (LDRs aren't polarized).
2. Jumper from **5V** to **one leg** of the LDR.
3. The **other leg** of the LDR goes into an empty column — let's call it "the junction."
4. In that same junction column, plug in the **10kΩ resistor**, with its other end going to **GND**.
5. Jumper from the junction column to **A0**.

Now A0 is reading the voltage in the middle of the divider. In bright light, the LDR lets voltage through, so A0 reads a **high** number. In darkness, the LDR blocks it and A0 reads a **low** number. (Or it could be the other way around, depending on your LDR — we'll check in code.)

**Wiring the LED:** pin 8 → 220Ω → LED → GND. Same as always.

## Part 1: Read the light level

Let's start by just printing the LDR value and watching it change.

```cpp
void setup() {
  Serial.begin(9600);
}

void loop() {
  int light = analogRead(A0);
  Serial.println(light);
  delay(200);
}
```

Open the Serial Monitor. You should see numbers scrolling by. Wave your hand over the LDR — the number changes. Cover it with a finger to block the light — number drops (or rises, depending on your LDR wiring). Shine a flashlight on it — jumps the other way.

Write down roughly what value you see in a **normal room**, and what value you see when you **cover the LDR**. You'll need those numbers for the next step.

For example, you might see:

- Normal room: around **600**
- Covered (dark): around **200**
- Flashlight: around **950**

Your numbers will be different — every LDR and every room is unique. That's why we measure first, then decide.

## Part 2: Turn on the LED when it's dark

Now we use the reading to make a decision.

```cpp
void setup() {
  pinMode(8, OUTPUT);
  Serial.begin(9600);
}

void loop() {
  int light = analogRead(A0);
  Serial.println(light);

  if (light < 400) {
    // It's dark — turn on the LED
    digitalWrite(8, HIGH);
  } else {
    // It's bright enough — LED off
    digitalWrite(8, LOW);
  }

  delay(100);
}
```

Upload. Cover the LDR with your hand or a book — the LED turns on! Uncover — LED off. Automatic light.

### The threshold number

The magic number here is `400` — the **threshold**. If the light value is **below 400**, we call it "dark." Above, we call it "bright." You need to pick this number based on **your** measurements. Too high and the LED will be on all the time. Too low and it will never turn on.

Use the Serial Monitor readings from Part 1 to pick something **between** your "normal" and "covered" values. If normal is 600 and covered is 200, try `400` or `450`. Adjust until it feels right.

### My LDR is backward!

Some LDRs wired some ways read *low* in bright light and *high* in darkness — the opposite of what I described. If yours behaves like that, just flip the comparison: `if (light > 600)` instead of `if (light < 400)`. Same idea, flipped.

The Serial Monitor is your friend — always print the raw value first so you know which way your sensor works.

## Nothing new about the code, really

Look at this code and notice what's there:

- `analogRead(A0)` — Day 8
- `Serial.println(...)` — Day 8
- `if/else` — Day 3
- `digitalWrite` — Day 1

**No new functions today.** We're just combining what you already know — reading an analog sensor (Day 8 idea) and making a decision from it (Day 3 idea). That's how most Arduino projects actually work: you stack familiar tools in new ways.

The one new concept is the **sensor** itself — and the skill of **measuring first, then writing the threshold**.

## Try this

1. **Print with a label.** Change the Serial output to `Serial.print("Light: "); Serial.println(light);` so it's more readable.
2. **Three zones.** Print `"Dark"`, `"Normal"`, or `"Very bright"` based on the reading. Use `if / else if / else`:

    ```cpp
    if (light < 300) {
      Serial.println("Dark");
    } else if (light < 700) {
      Serial.println("Normal");
    } else {
      Serial.println("Very bright");
    }
    ```
3. **Smooth LED brightness.** Use `map()` (from Day 8) to make the LED brighter when the room is darker. Wire the LED to a `~` pin (like 9). Hint: `int b = map(light, 0, 1023, 255, 0); analogWrite(9, b);`
4. **Two-LED alarm.** Red LED (pin 8) on when dark. Green LED (pin 9) on when bright. Both off in the normal range.

## Meet `else if`

Challenge 2 above introduced a new thing: **`else if`**. It lets you check multiple conditions in order:

```cpp
if (condition1) {
  // runs if condition1 is true
} else if (condition2) {
  // runs if condition1 was false AND condition2 is true
} else if (condition3) {
  // runs if both above were false AND condition3 is true
} else {
  // runs if none of the above were true
}
```

Think of it like a waterfall of questions. Arduino tries each one from top to bottom and stops at the first one that's true. Useful for "categories" — like mapping a number into dark/normal/bright zones.

You can have as many `else if` as you want.

## What you learned today

- What an **LDR** (photoresistor) is and how it responds to light
- Why sensors like the LDR need a **voltage divider** to work with Arduino
- How to **measure first** (with Serial.println) and **then pick a threshold**
- **`else if`** — chain multiple conditions together
- That today's code had **no new functions** — just old ideas in a new shape

## What is next

[Day 10](/arduino/day-10-smart-night-light/) — combine everything from this week into a **Smart Night Light**. The LED turns on automatically when the room gets dark, with a button to override it manually. Your first project that's genuinely useful.

Great work, Anish. Your Arduino can now see.
