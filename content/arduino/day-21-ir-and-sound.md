---
title: "Day 21: Dual-Mode Control — IR Remote + Sound"
date: 2026-04-11T23:39:00+05:30
lastmod: 2026-04-11T23:39:00+05:30
tags: ["arduino", "beginner", "kids", "ir", "sound", "lcd", "modes", "integration"]
categories: ["arduino"]
summary: "Day 21 of Learn Coding with Arduino — combine the IR remote and sound sensor into one sketch with manual and auto modes, with the LCD showing the current state."
---

Hi Anish! Today is integration day. You're going to take the **IR remote** from Day 19, the **sound sensor** from Day 20, and the **LCD** from Days 15-18, and mash them into a single sketch that has **two modes**:

- **MANUAL mode** — IR remote controls the LED directly. Clap detection is off.
- **AUTO mode** — claps toggle the LED. Remote switches modes but doesn't control the LED.

One remote button switches between modes. The LCD shows which mode you're in. No new concepts today — just combining everything you already know.

## What you need today

- Arduino Uno + USB cable
- Breadboard
- **IR receiver** (Day 19)
- **Sound sensor** (Day 20)
- **LCD with I2C** (Day 15)
- **1 LED** + **220Ω resistor**
- Jumper wires

This is a lot of components on one breadboard. Take your time. Test each subsystem alone first if you're not sure it's wired right.

## The circuit

{{< mermaid >}}
graph TB
    subgraph Power["Power & LCD"]
      V5["5V"]
      GND["GND"]
      A4["A4 → LCD SDA"]
      A5["A5 → LCD SCL"]
    end
    subgraph IR["IR receiver"]
      V5 --> IRV["IR VCC"]
      GND --> IRG["IR GND"]
      PIN11["Pin 11"] --> IRO["IR OUT"]
    end
    subgraph Sound["Sound sensor"]
      V5 --> SV["Sound VCC"]
      GND --> SG["Sound GND"]
      PIN2["Pin 2"] --> SD["Sound D0"]
    end
    subgraph LED["LED"]
      PIN8["Pin 8"] --> R220["220Ω"] --> LEDp["LED"] --> GND
    end
{{< /mermaid >}}

**Pin assignments:**

- **Pin 2** — Sound sensor digital output
- **Pin 8** — LED
- **Pin 11** — IR receiver OUT
- **A4/A5** — LCD I2C

Power rails (5V and GND) on the breadboard shared by all three modules.

## The code

```cpp
#include <IRremote.hpp>
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

// Pin assignments
const int LED_PIN   = 8;
const int SOUND_PIN = 2;
const int IR_PIN    = 11;

// === YOUR remote codes — from Day 19 ===
const int MODE_BTN_CODE   = 0x45;  // e.g., Power button — toggles mode
const int LED_ON_CODE     = 0x40;  // e.g., Vol Up — turns LED on in manual
const int LED_OFF_CODE    = 0x19;  // e.g., Vol Down — turns LED off in manual

// Mode: false = MANUAL, true = AUTO
bool autoMode = false;

// Sound debouncing
int lastClapTime = 0;

LiquidCrystal_I2C lcd(0x27, 16, 2);

void setup() {
  pinMode(LED_PIN, OUTPUT);
  pinMode(SOUND_PIN, INPUT);
  Serial.begin(9600);

  IrReceiver.begin(IR_PIN);

  lcd.init();
  lcd.backlight();
  showMode();
}

void loop() {
  handleIR();
  if (autoMode) {
    handleSound();
  }
}

// === Handles IR button presses ===
void handleIR() {
  if (IrReceiver.decode()) {
    int code = IrReceiver.decodedIRData.command;
    Serial.print("IR code: 0x");
    Serial.println(code, HEX);

    if (code == MODE_BTN_CODE) {
      autoMode = !autoMode;
      showMode();
    } else if (code == LED_ON_CODE && !autoMode) {
      digitalWrite(LED_PIN, HIGH);
    } else if (code == LED_OFF_CODE && !autoMode) {
      digitalWrite(LED_PIN, LOW);
    }

    IrReceiver.resume();
  }
}

// === Handles clap detection (only in auto mode) ===
void handleSound() {
  if (digitalRead(SOUND_PIN) == HIGH) {
    int now = millis();
    if (now - lastClapTime > 300) {    // ignore echoes
      digitalWrite(LED_PIN, !digitalRead(LED_PIN));
      Serial.println("Clap -> toggle");
      lastClapTime = now;
    }
  }
}

// === Updates the LCD to show current mode ===
void showMode() {
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Mode:");
  lcd.setCursor(0, 1);
  lcd.print(autoMode ? "AUTO (clap)" : "MANUAL (remote)");
}
```

Upload. You should see:

```
Mode:
MANUAL (remote)
```

Press the mode button on the remote → display changes to:

```
Mode:
AUTO (clap)
```

Clap → LED toggles. Press the mode button again → back to manual. Press vol-up on remote → LED on. Press vol-down → LED off. In auto mode, the remote volume buttons do nothing (only clap works).

## What's worth noticing

No new language features, but **three important patterns** are on display. Look at the `loop()`:

```cpp
void loop() {
  handleIR();
  if (autoMode) {
    handleSound();
  }
}
```

**That's it.** Three lines. No giant blob of code. The actual work is broken out into three helper functions: `handleIR()`, `handleSound()`, `showMode()`. The `loop()` just coordinates them.

### Pattern 1: Small `loop()`, big helpers

A good Arduino `loop()` should read almost like English: *"Handle IR. If we're in auto mode, handle sound. Done."* Each helper function does exactly **one job**. This is the structure real embedded engineers use — it's also how you'll organize the final project on Day 25.

If your `loop()` starts to grow past ~20 lines, it's usually a sign you should break a chunk out into a helper function.

### Pattern 2: Mode-dependent behavior

```cpp
if (autoMode) {
  handleSound();
}
```

We skip the entire sound handler when we're in manual mode. Why? Because in manual mode, we don't WANT sound to do anything. Instead of wrapping the sound handling with an `if` deep inside `handleSound()`, we decide **at the `loop()` level** whether to call it at all.

Inside `handleIR()`, we do something similar:

```cpp
} else if (code == LED_ON_CODE && !autoMode) {
  digitalWrite(LED_PIN, HIGH);
}
```

The `&& !autoMode` means: *"only turn on the LED from a remote button IF we're in manual mode."* In auto mode, the volume buttons are ignored. The `&&` (AND) operator from Day 7.

### Pattern 3: Non-blocking sound detection with `millis()`

```cpp
void handleSound() {
  if (digitalRead(SOUND_PIN) == HIGH) {
    int now = millis();
    if (now - lastClapTime > 300) {
      digitalWrite(LED_PIN, !digitalRead(LED_PIN));
      lastClapTime = now;
    }
  }
}
```

Notice there's **no `delay()`** in the sound handler. Instead, we track when the last clap happened with `lastClapTime` (a global variable), and ignore any new sound for 300ms after the last valid one.

Why? Because a `delay(300)` would also stall the IR handler — you wouldn't be able to receive remote presses during those 300ms. Using `millis()` means both handlers run simultaneously without blocking each other.

This is the **cornerstone** of writing Arduino code that handles multiple things at once. You'll see it again and again.

## Try this

1. **Show the last action.** Add a third row of LCD info — print "Last: remote" or "Last: clap" depending on what just happened.
2. **Clap-to-mode.** Map a **triple clap** to change modes (Day 20 clap counter pattern). No more fumbling for the remote.
3. **Servo control.** Replace the LED with a servo (Day 12). In manual mode, remote buttons rotate the servo. In auto mode, each clap nudges it by 10°.
4. **Sound meter fallback.** In auto mode, if there's been no clap for 30 seconds, turn the LED off automatically (like an auto-off bedside lamp).

## What you learned today

- How to **integrate multiple subsystems** (IR + Sound + LCD + LED) into one sketch
- How to keep `loop()` **clean** by using helper functions
- How to **gate behavior by mode** with `if (mode)` and `&& !mode` checks
- Why `delay()` is bad when multiple things need to run at once
- Using `millis()` to **debounce without blocking**
- The skeleton of a real embedded program: **mode state + multiple handlers + display**

## What is next

[Day 22](/arduino/day-22-smart-room/) — we take everything and wrap it into the **Smart Room System**, the grand project of the sensor chapter. LCD status, IR remote for modes, claps for triggers, LDR for auto-darkness, buzzer for alerts. It's a lot, but you already know how to do every piece.

Great work, Anish. You just shipped your most complex sketch yet.
