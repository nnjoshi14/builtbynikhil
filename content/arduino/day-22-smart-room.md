---
title: "Day 22: Project — Smart Room System"
date: 2026-04-11T23:38:00+05:30
lastmod: 2026-04-11T23:38:00+05:30
tags: ["arduino", "beginner", "kids", "project", "smart-home", "integration"]
categories: ["arduino"]
summary: "Day 22 of Learn Coding with Arduino — the big one. Build a Smart Room System that combines IR, sound, LCD, light sensor, LED, and buzzer into one coherent program."
---

Hi Anish! Project time — and this is the biggest one yet. The **Smart Room System** combines almost everything we've built so far:

- **LDR** — senses room darkness
- **IR remote** — for mode switching and manual control
- **Sound sensor** — clap-to-toggle
- **LCD** — status display
- **LED** — the "room light" (stand-in for a real lamp)
- **Buzzer** — alerts when modes change

Three modes:

1. **AUTO** — LED turns on automatically when the room is dark
2. **MANUAL** — remote controls the LED directly
3. **CLAP** — a clap toggles the LED

This is the kind of thing a working home-automation prototype looks like. By the end, you'll have built a gadget with four inputs, three outputs, and a coherent user interface. Real engineering.

## What you need today

- Arduino Uno + USB cable
- Breadboard (ideally big, or two breadboards side by side)
- **LCD with I2C** (Day 15)
- **LDR + 10kΩ resistor** (Day 9)
- **IR receiver** (Day 19)
- **Sound sensor** (Day 20)
- **1 LED + 220Ω resistor**
- **Buzzer** (Day 11)
- Lots of jumper wires

## Pin plan

Write this down or tape it to your breadboard — you'll thank yourself later.

| Pin | What |
|---|---|
| A0 | LDR (analog) |
| A4 | LCD SDA (I2C) |
| A5 | LCD SCL (I2C) |
| 2 | Sound D0 (digital in) |
| 8 | LED |
| 10 | Buzzer |
| 11 | IR receiver OUT |

Power: 5V and GND rails shared across all modules.

## The circuit

{{< mermaid >}}
graph TB
    subgraph Sensors["Sensors (inputs)"]
      PIN_A0["A0 ← LDR"]
      PIN_2["Pin 2 ← Sound D0"]
      PIN_11["Pin 11 ← IR OUT"]
    end
    subgraph Actuators["Actuators (outputs)"]
      PIN_8["Pin 8 → LED (+220Ω)"]
      PIN_10["Pin 10 → Buzzer"]
    end
    subgraph Display["Display"]
      A4["A4 ↔ LCD SDA"]
      A5["A5 ↔ LCD SCL"]
    end
{{< /mermaid >}}

Wire one module at a time and use a Serial.println after each to sanity-check. "LCD wired — print Hello, works. Now add LDR — read A0, works. Now add IR..." etc. If you try to wire everything at once and it doesn't work, you won't know which part broke.

## The code

This is a long one. Read it top to bottom, then we'll break it down.

```cpp
#include <IRremote.hpp>
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

// === Pins ===
const int LDR_PIN   = A0;
const int SOUND_PIN = 2;
const int LED_PIN   = 8;
const int BUZZ_PIN  = 10;
const int IR_PIN    = 11;

// === Your remote codes — replace with what you discovered on Day 19 ===
const int MODE_CYCLE_CODE = 0x45;  // cycles AUTO → MANUAL → CLAP
const int LED_ON_CODE     = 0x40;  // works only in MANUAL
const int LED_OFF_CODE    = 0x19;

// === Tuning ===
const int DARK_THRESHOLD  = 400;
const int CLAP_DEBOUNCE_MS = 300;

// === Modes ===
const int MODE_AUTO   = 0;
const int MODE_MANUAL = 1;
const int MODE_CLAP   = 2;
int mode = MODE_AUTO;

// === State ===
int lastClapTime = 0;
bool ledState    = false;

LiquidCrystal_I2C lcd(0x27, 16, 2);

// === Forward declarations ===
void handleIR();
void handleSound();
void handleLight();
void updateLED();
void showMode();
void beep(int freq, int duration);

// ============ setup / loop ============

void setup() {
  pinMode(LED_PIN, OUTPUT);
  pinMode(BUZZ_PIN, OUTPUT);
  pinMode(SOUND_PIN, INPUT);
  Serial.begin(9600);

  IrReceiver.begin(IR_PIN);

  lcd.init();
  lcd.backlight();
  showMode();
  Serial.println("Smart Room System ready");
}

void loop() {
  handleIR();
  if (mode == MODE_AUTO)   handleLight();
  if (mode == MODE_CLAP)   handleSound();
  updateLED();
}

// ============ Handlers ============

void handleIR() {
  if (!IrReceiver.decode()) return;

  int code = IrReceiver.decodedIRData.command;
  Serial.print("IR: 0x");
  Serial.println(code, HEX);

  if (code == MODE_CYCLE_CODE) {
    mode = (mode + 1) % 3;   // cycle through 0, 1, 2
    showMode();
    beep(1500, 100);
  } else if (code == LED_ON_CODE && mode == MODE_MANUAL) {
    ledState = true;
  } else if (code == LED_OFF_CODE && mode == MODE_MANUAL) {
    ledState = false;
  }

  IrReceiver.resume();
}

void handleSound() {
  if (digitalRead(SOUND_PIN) == HIGH) {
    int now = millis();
    if (now - lastClapTime > CLAP_DEBOUNCE_MS) {
      ledState = !ledState;
      beep(800, 50);
      lastClapTime = now;
    }
  }
}

void handleLight() {
  int light = analogRead(LDR_PIN);
  ledState = (light < DARK_THRESHOLD);
}

// ============ Output ============

void updateLED() {
  digitalWrite(LED_PIN, ledState ? HIGH : LOW);
}

void showMode() {
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Mode:");
  lcd.setCursor(0, 1);
  if (mode == MODE_AUTO)   lcd.print("AUTO (light)");
  if (mode == MODE_MANUAL) lcd.print("MANUAL (IR)");
  if (mode == MODE_CLAP)   lcd.print("CLAP");
}

void beep(int freq, int duration) {
  tone(BUZZ_PIN, freq, duration);
}
```

Upload. Initial screen says `Mode: AUTO (light)`. Cover the LDR — LED turns on. Press the remote's mode button — beep, screen changes to `MANUAL (IR)`. Press volume-up — LED on. Vol-down — off. Press mode button again — beep, `CLAP` mode. Clap — LED toggles. Cycle again back to AUTO.

## What's worth noticing

Read the code structure. This is what **good Arduino code** looks like.

### Named constants at the top

```cpp
const int MODE_AUTO   = 0;
const int MODE_MANUAL = 1;
const int MODE_CLAP   = 2;
```

Instead of writing `if (mode == 0)` and `if (mode == 1)` throughout the code, we give names to the mode numbers. Now `if (mode == MODE_AUTO)` reads like English. Future-you can tell at a glance what the code is checking.

This is how pros handle "magic numbers" — constants with meaningful names.

### Forward declarations

```cpp
void handleIR();
void handleSound();
// ...
```

These are lines that tell Arduino *"these functions exist; you'll see them defined below."* Technically not required if you define your functions before `setup()` and `loop()`, but good practice for longer sketches — it's a clean summary of all the functions in this file, right at the top.

### Clean `loop()`

```cpp
void loop() {
  handleIR();
  if (mode == MODE_AUTO)   handleLight();
  if (mode == MODE_CLAP)   handleSound();
  updateLED();
}
```

**Four lines.** That's all. Each one is a verb: "handle IR", "handle light (if auto)", "handle sound (if clap)", "update the LED". If you read this to a human engineer, they'd understand the program in 5 seconds.

This is a huge contrast to an amateur sketch that crams 100 lines of logic into `loop()`. Break things into functions; your code becomes readable.

### `updateLED()` is the single source of truth

Notice how every handler only writes to `ledState` (the variable), never directly to the LED pin. The actual `digitalWrite(LED_PIN, ...)` happens in exactly **one place**: `updateLED()`.

This is a pattern called **single source of truth**. It means:

- You always know who's deciding the LED state (whichever handler set `ledState` most recently)
- You don't have to hunt through the code looking for `digitalWrite(LED_PIN, ...)` calls when debugging
- If you want to add something like "blink the LED fast instead of steady-on", you change `updateLED()` once and every mode picks it up

As projects get bigger, this pattern saves you from bugs where two different pieces of code fight over the same output.

### Reusable helper: `beep()`

```cpp
void beep(int freq, int duration) {
  tone(BUZZ_PIN, freq, duration);
}
```

Two lines. Seems pointless — why not just call `tone(BUZZ_PIN, freq, duration)` directly? Because `beep(1500, 100)` reads more clearly, AND if I ever change the buzzer pin I only update one spot. Small win, big habit.

## Try this

1. **Add a 4th mode: OFF.** LED stays off regardless of input. Cycle becomes AUTO → MANUAL → CLAP → OFF → AUTO.
2. **Show live light reading** on the LCD in AUTO mode (row 0, after "Mode:"). Like a dashboard.
3. **Different beep pitches per mode.** Each mode change beeps at a different frequency — AUTO beeps at 500 Hz, MANUAL at 1000 Hz, CLAP at 2000 Hz. Use a `switch` inside `showMode()`:

    ```cpp
    switch (mode) {
      case MODE_AUTO:   beep(500,  100); break;
      case MODE_MANUAL: beep(1000, 100); break;
      case MODE_CLAP:   beep(2000, 100); break;
    }
    ```

    `switch/case` is a cleaner alternative to a long chain of `if/else if`. The `break` keyword jumps out of the switch when the case matches.
4. **Auto-off timer.** In CLAP mode, if the LED has been on for 30 seconds and no new claps, turn it off automatically. Use `millis()` to track when the LED last turned on.

## What you learned today

- **How to architect** a multi-input, multi-output project
- **Named mode constants** (`const int MODE_AUTO = 0;`)
- **Forward declarations** for a clean function list at the top
- A tiny, readable `loop()` that calls helpers
- **Single source of truth** — one function owns the output
- Simple helpers (like `beep()`) improve readability
- **`switch / case`** as an alternative to `else if` chains (in the challenges)

## What is next

You've now completed the **sensors and smart control chapter**. From here on, it's about **your own ideas**. For Days 23-26, we'll plan, build, debug, and demo a final project *you* design. No more new components — just combining what you already know into something you thought of yourself.

[Day 23](/arduino/day-23-project-planning/) starts with **brainstorming**. Pick a project that excites you. It can be anything.

Massively great work, Anish. You just built a smart home prototype from a handful of electronic parts.
