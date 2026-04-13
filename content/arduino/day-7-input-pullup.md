---
title: "Day 7: The Easier Button — INPUT_PULLUP"
date: 2026-04-11T23:53:00+05:30
lastmod: 2026-04-11T23:53:00+05:30
tags: ["arduino", "beginner", "kids", "button", "input", "pullup", "toggle"]
categories: ["arduino"]
summary: "Day 7 of Learn Coding with Arduino — simplify your button circuit using the Arduino's built-in pull-up. One resistor less, and learn how to toggle an LED with each press."
---

Hi Anish! Remember Day 3 when we wired a button with that extra **10kΩ pull-down resistor** so the pin wouldn't "float"? Today I'm going to show you a cleaner way that **removes that extra resistor** entirely. The trick: Arduino has a pull-up resistor **built into every pin**, and you just have to turn it on.

## What you need today

- Arduino Uno + USB cable
- Breadboard
- 1 **push button**
- 1 **LED**
- 1 **220Ω resistor** (for the LED)
- Jumper wires

Notice what's missing: **no 10kΩ resistor this time.** We're letting Arduino handle it internally.

## What is a pull-up resistor?

On Day 3 we used a **pull-down** resistor. It "pulled" the pin gently down to 0V when the button wasn't pressed, so the pin read `LOW` by default. Pressing the button connected the pin to 5V, flipping it to `HIGH`.

A **pull-up** resistor is the opposite. It pulls the pin gently *up* to 5V when the button isn't pressed, so the pin reads `HIGH` by default. Pressing the button connects the pin directly to GND, dragging it down to `LOW`.

**Important flip — remember this:**

> With `INPUT_PULLUP`, the pin reads **HIGH when NOT pressed**, and **LOW when pressed**.

It feels backward. Press the button → pin goes LOW? Yes — because pressing connects the pin to GND, and GND is `LOW`. Say it out loud until it sticks: **"pullup means pressed equals low."**

## The circuit

This is the reason `INPUT_PULLUP` is beautiful — the circuit is simpler.

{{< mermaid >}}
graph LR
    PIN8["Pin 8"] --> R220["220Ω"] --> LEDp["LED long leg"] --> LEDn["LED short leg"] --> G1["GND"]
    PIN2["Pin 2"] --> BTNa["Button side A"]
    BTNa -.->|pressed| BTNb["Button side B"]
    BTNb --> G2["GND"]
{{< /mermaid >}}

**Wiring the button (only two wires this time!):**

1. Put the button on the breadboard, straddling the middle gap.
2. Jumper from **Arduino pin 2** to one side of the button.
3. Jumper from the **other side** of the button to **GND**.

That's it. No 5V wire. No pull-down resistor. Two wires and a button. Compare that to Day 3 (which needed 3 wires plus a 10kΩ resistor) — much cleaner.

The LED is wired exactly like Day 2: pin 8 → 220Ω → LED + → LED − → GND.

## The code

```cpp
void setup() {
  pinMode(8, OUTPUT);
  pinMode(2, INPUT_PULLUP);   // ← the magic word
}

void loop() {
  if (digitalRead(2) == LOW) {
    // Button IS pressed (remember: pullup flips the logic)
    digitalWrite(8, HIGH);
  } else {
    // Button NOT pressed
    digitalWrite(8, LOW);
  }
}
```

Upload. Press the button — LED on. Release — LED off.

### What changed from Day 3?

Just two things:

1. `pinMode(2, INPUT)` → **`pinMode(2, INPUT_PULLUP)`**. That single word tells Arduino "turn on the internal pull-up resistor for this pin." You don't need a physical resistor on the breadboard — Arduino has one inside the chip.
2. The if-condition flipped: `== HIGH` → **`== LOW`**. Because with a pull-up, **pressed = LOW**.

Everything else is identical.

## Toggle: press once to turn on, press again to turn off

The code above only keeps the LED on *while you're holding the button*. Boring. Real switches work by **toggling**: press once to turn on, press again to turn off.

The challenge with toggles: the `loop()` runs thousands of times per second. If you just flip the LED whenever the button is pressed, the state would flip thousands of times during a single press — the LED would appear to be half-on (actually flickering so fast you can't tell). We need to detect **the moment the button changes from not-pressed to pressed** — not every instant it's being held.

Meet **state change detection**:

```cpp
int ledState = LOW;             // is the LED currently on or off?
int lastButtonState = HIGH;     // what was the button doing last time we checked?

void setup() {
  pinMode(8, OUTPUT);
  pinMode(2, INPUT_PULLUP);
}

void loop() {
  int buttonState = digitalRead(2);

  // Did the button JUST get pressed?
  // (Was HIGH last time, is LOW now = just pressed)
  if (lastButtonState == HIGH && buttonState == LOW) {
    // Flip the LED state
    if (ledState == LOW) {
      ledState = HIGH;
    } else {
      ledState = LOW;
    }
    digitalWrite(8, ledState);
    delay(50);  // tiny delay to "debounce" — ignore bouncy button noise
  }

  lastButtonState = buttonState;
}
```

Upload. Now: press once → LED on. Press again → LED off. Press again → LED on. Each press flips it. This is how a light switch in real life works.

### There's a lot here — let's unpack it

#### Two variables at the top

```cpp
int ledState = LOW;
int lastButtonState = HIGH;
```

We're using variables to **remember** things between loops. Remember Day 4: a variable is a box. `ledState` remembers whether the LED should be on or off right now. `lastButtonState` remembers what the button was doing the **previous time** through the loop. Without remembering, we can't detect changes.

### `&&` means "AND"

```cpp
if (lastButtonState == HIGH && buttonState == LOW) {
```

The `&&` is a new operator (two ampersands, looks like this: `&&`). It means **AND**. The whole condition is true **only if both halves are true**:

- `lastButtonState == HIGH` → *last time through the loop, the button was NOT pressed*
- `buttonState == LOW` → *this time, the button IS pressed*

Both together mean: *"The button just went from not-pressed to pressed — right this very moment."*

There's also `||` (two pipes) which means **OR** — true if either side is true. We'll meet it later.

### Flipping `ledState`

```cpp
if (ledState == LOW) {
  ledState = HIGH;
} else {
  ledState = LOW;
}
```

"If the LED is currently off, turn it on. Otherwise, turn it off." Each button press flips the state. That's a toggle.

### `digitalWrite(8, ledState);`

Wait — we're passing a **variable** as the second argument? Yes! `digitalWrite` just needs a HIGH or LOW, and it doesn't care whether you typed it directly or stored it in a variable first. Variables and values are interchangeable.

### `lastButtonState = buttonState;`

At the end of every loop, we update `lastButtonState` so the next time around, we know what the button was doing **last time**. This line is critical — without it, `lastButtonState` would be stuck on `HIGH` forever and the toggle would fire repeatedly while you held the button.

### `delay(50)` — debounce

Real buttons are mechanical. When you press them, the metal contacts bounce slightly for a few milliseconds, creating multiple "press" signals from one actual press. The 50ms delay smooths that out. We'll learn better ways to debounce later; for now, this works.

## Try this

1. **Different LED.** Move the LED to pin 11 and update the code. Same behavior, different pin.
2. **Two buttons, two LEDs.** Add a second button on pin 3 and a second LED on pin 11. Each button toggles its own LED independently.
3. **"Hold the button" detection.** Instead of toggling on a press, make the LED turn on only if you hold the button for **1 full second**. You'll need a timer variable. (This is tricky — come back to it if you get stuck.)

## When to use pull-up vs pull-down

| You want... | Use |
|---|---|
| Fewer parts, cleaner circuit | **Pull-up** (`INPUT_PULLUP`) — no extra resistor needed |
| Pressed = HIGH feels more natural | Pull-down (external 10kΩ, like Day 3) |
| Most real projects | **Pull-up** — it's the default choice for pros |

From now on, unless we have a good reason, we'll use `INPUT_PULLUP`.

## What you learned today

- **`INPUT_PULLUP`** — Arduino has a built-in pull-up resistor, just turn it on
- With pull-up, **pressed = LOW**, **not-pressed = HIGH** (backward from Day 3)
- **State change detection** — using a "last state" variable to detect the *moment* of a press
- **`&&`** — the AND operator (both conditions must be true)
- You can pass variables (like `ledState`) as arguments to functions like `digitalWrite`
- **Debouncing** — why we add a small `delay()` after a button event
- How to build a **toggle switch** in code

## What is next

[Day 8](/arduino/day-8-potentiometer/) — your first **analog input**. Until now, pins have read HIGH or LOW (two states). A potentiometer (the knob from your kit) gives **1024 different values**. You'll also meet the **Serial Monitor**, which lets Arduino print messages to your laptop screen.

Great work, Anish.
