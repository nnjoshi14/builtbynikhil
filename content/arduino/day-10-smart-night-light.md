---
title: "Day 10: Project — Smart Night Light"
date: 2026-04-11T23:50:00+05:30
lastmod: 2026-04-11T23:50:00+05:30
tags: ["arduino", "beginner", "kids", "project", "ldr", "led", "button", "night-light"]
categories: ["arduino"]
summary: "Day 10 of Learn Coding with Arduino — build a smart night light that turns on automatically when it gets dark, with a manual override button."
---

Hi Anish! Your **second real project**. Today we combine the LDR from Day 9 with the button from Day 7 and an LED to build a **Smart Night Light** — one that turns itself on when the room gets dark, with a button that lets you override it.

This is the first project that's genuinely useful. You could actually set this on a shelf in your room and it would work.

## What you need today

- Arduino Uno + USB cable
- Breadboard
- **1 LDR** + **1 × 10kΩ resistor** (for the voltage divider, like Day 9)
- **1 push button** (INPUT_PULLUP style, like Day 7 — no extra resistor)
- **1 LED** (ideally a bright white one for the night-light feel)
- **1 × 220Ω resistor** (for the LED)
- Jumper wires

## What we're building

The rules:

1. When the room is **dark**, the LED turns ON automatically.
2. When the room is **bright**, the LED stays OFF.
3. If you press the **button**, it toggles **manual mode**. In manual mode, the LED is always ON (no matter the light).
4. Press the button again → back to automatic mode.

So there are **two modes**: AUTO (follows the LDR) and MANUAL (always on). The button switches between them.

## The circuit

Everything from the last few days on one breadboard.

{{< mermaid >}}
graph TB
    V5["5V"] --> LDR1["LDR"]
    LDR1 --> JUNC["junction"]
    JUNC --> A0["Pin A0"]
    JUNC --> R10k["10kΩ"] --> G1["GND"]
    PIN8["Pin 8"] --> R220["220Ω"] --> LEDp["LED +"] --> LEDn["LED −"] --> G2["GND"]
    PIN2["Pin 2<br/>(INPUT_PULLUP)"] --> BTNa["Button"]
    BTNa -.->|pressed| BTNb["Button"] --> G3["GND"]
{{< /mermaid >}}

**Summary:**

- **A0** — LDR with 10kΩ pull-down (Day 9 wiring)
- **Pin 8** — LED with 220Ω (Day 2 wiring)
- **Pin 2** — button to GND (Day 7 wiring, INPUT_PULLUP — no extra resistor)

## The code

```cpp
// Modes
bool manualMode = false;   // false = AUTO, true = MANUAL (always on)

// Button state tracking
int lastButtonState = HIGH;

// Light threshold — adjust based on YOUR room
int darkThreshold = 400;

void setup() {
  pinMode(8, OUTPUT);
  pinMode(2, INPUT_PULLUP);
  Serial.begin(9600);
}

void loop() {
  // Check button for state change (press detection)
  int buttonState = digitalRead(2);
  if (lastButtonState == HIGH && buttonState == LOW) {
    manualMode = !manualMode;              // flip the mode
    Serial.print("Mode switched: ");
    Serial.println(manualMode ? "MANUAL" : "AUTO");
    delay(50);                             // debounce
  }
  lastButtonState = buttonState;

  // Decide LED state
  if (manualMode) {
    digitalWrite(8, HIGH);                 // manual — always on
  } else {
    int light = analogRead(A0);
    if (light < darkThreshold) {
      digitalWrite(8, HIGH);               // auto — dark, turn on
    } else {
      digitalWrite(8, LOW);                // auto — bright enough, off
    }
  }
}
```

Upload. Here's what should happen:

- Cover the LDR with your hand → LED turns on.
- Uncover → LED turns off.
- Press the button once → LED stays on permanently, even in bright light. Serial Monitor says "Mode switched: MANUAL".
- Press again → back to automatic. LED follows the LDR again. Serial says "Mode switched: AUTO".

## What's new in this code?

Almost everything here you've seen before, but there are a few new tricks.

### `bool` — true or false

```cpp
bool manualMode = false;
```

`bool` (short for "boolean") is a new kind of variable. Instead of holding a whole number like `int`, it holds either **`true`** or **`false`**. Just two values — like a switch.

We use `bool` when something has only two states. "Am I in manual mode?" Yes or no. "Is the door open?" Yes or no. "Has the button been pressed?" Yes or no.

In fact, `true` is basically another name for `HIGH` (or 1) and `false` is another name for `LOW` (or 0). But when you use `bool` / `true` / `false`, your code reads more like English, which is nicer.

### `!manualMode` — the NOT operator

```cpp
manualMode = !manualMode;
```

The `!` (exclamation mark) means **NOT**. It flips `true` to `false` and vice versa:

- If `manualMode` is `false`, `!manualMode` is `true`.
- If `manualMode` is `true`, `!manualMode` is `false`.

So `manualMode = !manualMode;` means "flip the mode to its opposite." Perfect for toggling. This is the cleanest way to flip a bool — shorter than an `if/else`.

### The ternary operator — `condition ? A : B`

```cpp
Serial.println(manualMode ? "MANUAL" : "AUTO");
```

This one looks weird. The `?` and `:` together form a **mini if-statement** you can use inline. Read it as: *"Is `manualMode` true? If yes, use `"MANUAL"`. If no, use `"AUTO"`."* It's a shortcut for:

```cpp
if (manualMode) {
  Serial.println("MANUAL");
} else {
  Serial.println("AUTO");
}
```

Same thing, one line instead of five. Use it when you want to pick between two small values. Don't use it for big multi-line logic — that's what `if/else` is for.

### Variables at the top of the file

Look at how I set up the variables:

```cpp
bool manualMode = false;
int lastButtonState = HIGH;
int darkThreshold = 400;
```

These are all **global variables** (remember Day 4) — declared at the top, visible everywhere. `darkThreshold = 400` is a particularly important one: it's the magic number you'll tweak for your room. Keeping it at the top makes it easy to find.

**Good habit:** always put your "tweakable numbers" (like thresholds, delays, pin numbers) at the very top of the file. Future-you can find them in 2 seconds.

## Tracing the flow

Let's walk through one run of the loop, slowly:

1. `buttonState = digitalRead(2)` — check button. Let's say it's `HIGH` (not pressed).
2. Is `lastButtonState == HIGH && buttonState == LOW`? HIGH and HIGH — no, the second half is false. Skip the mode flip.
3. `lastButtonState = buttonState;` — remember current button state for next loop.
4. Is `manualMode` true? Let's say no, it's still `false` (auto mode).
5. Read `light = analogRead(A0)` — say it's 350.
6. Is `350 < 400`? Yes. `digitalWrite(8, HIGH)` — LED on.
7. End of loop. Start again from step 1.

Now imagine the button gets pressed. On some loop:

1. `buttonState = LOW`.
2. Is `lastButtonState == HIGH && buttonState == LOW`? Yes! (Last time it was HIGH, now it's LOW — the moment of the press.) Enter the if block.
3. `manualMode = !manualMode;` — was false, now true.
4. Print "Mode switched: MANUAL". `delay(50)` for debounce.
5. `lastButtonState = LOW`.
6. Is `manualMode` true? Yes. `digitalWrite(8, HIGH)` — LED on.
7. From now on, the code skips the LDR check entirely until the button is pressed again.

## Try this

1. **Add a second LED** on pin 9 as a mode indicator. When in AUTO mode, pin 9 LED is off. When in MANUAL mode, pin 9 LED is on. Now you can tell which mode you're in just by looking.
2. **Smooth dimming.** Instead of `digitalWrite(8, HIGH)` in auto mode, use `analogWrite(9, brightness)` where brightness is mapped from the LDR — darker room = brighter LED. Move the LED to a `~` pin first.
3. **Off mode.** Add a third mode: OFF (LED always off, regardless of light). Cycle with each button press: AUTO → MANUAL → OFF → AUTO → ... You'll need an `int` instead of a `bool`, since you have 3 modes now.
4. **Hysteresis.** Notice how the LED flickers on and off if the light is right at the threshold? Add two thresholds: `int onBelow = 350;` and `int offAbove = 450;`. LED turns on when light drops below 350, and turns off when light rises above 450. That 100-unit gap stops the flicker.

## Why today mattered

Look at what you just built. It's **one program** that:

- Reads a sensor
- Reads a button with state change detection
- Makes decisions based on modes
- Controls an LED
- Prints debug messages
- Has a user-tweakable configuration at the top

This is not "toy Arduino." This is what **real embedded software** looks like. Real engineers write programs with this exact shape. Today you learned the skeleton of every interactive gadget you will ever build.

## What you learned today

- **`bool`** — a variable that holds `true` or `false`
- **`!x`** — the NOT operator, flips true/false
- **Ternary** `x ? a : b` — inline if-else expression
- How to structure an interactive sketch with **modes**
- Tweakable configuration at the top of the file
- How to combine inputs (LDR + button) with outputs (LED) into one coherent program
- **Hysteresis** (in the challenges) — using two thresholds to prevent flicker

## What is next

[Day 11](/arduino/day-11-buzzer/) — time to make some noise. You'll wire up a **buzzer**, play simple tones, and code a short melody. Arduino goes from seeing to speaking.

Seriously great work, Anish. You just shipped a useful gadget.
