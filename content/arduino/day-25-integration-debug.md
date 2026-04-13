---
title: "Day 25: Integrate and Debug"
date: 2026-04-11T23:35:00+05:30
lastmod: 2026-04-11T23:35:00+05:30
tags: ["arduino", "beginner", "kids", "project", "debugging", "millis", "functions"]
categories: ["arduino"]
summary: "Day 25 of Learn Coding with Arduino — stitch all the modules from Day 24 into one coherent sketch, break loop() into helper functions, and debug integration issues."
---

Hi Anish! Today you **integrate** — combine all the separately-tested modules into one big sketch. This is often the hardest day, because things that worked alone sometimes act strangely when glued together. You'll learn the tools to debug those issues.

The good news: you've already done the hard parts. All the components work. All the thresholds are measured. All the wiring is in place. Today is about gluing them together cleanly.

## The structure of a good integrated sketch

Every Arduino project, once it has more than one module, should follow this shape:

```cpp
// === Includes ===
#include <...>

// === Pins ===
const int ...

// === Tuning constants ===
const int ...

// === State variables ===
int state ...

// === Object setup (LCD, servo, IR, etc.) ===
LiquidCrystal_I2C lcd(...);

// === Helper functions (defined below) ===
void readSensors();
void updateDisplay();
void decideActions();

// === setup() ===
void setup() {
  // init pins, Serial, objects
}

// === loop() — short! ===
void loop() {
  readSensors();
  decideActions();
  updateDisplay();
}

// === Helper function definitions ===
void readSensors() { ... }
void decideActions() { ... }
void updateDisplay() { ... }
```

Notice how **tiny** `loop()` is. Just 3 lines. Each line calls a helper that does one job. This is the shape of every serious Arduino project.

For the Smart Plant Monitor, we might structure it like this:

```cpp
void loop() {
  readSensors();
  handleButton();
  decidePlantStatus();
  updateDisplay();
  updateAlerts();
}
```

Five verbs. Read top-to-bottom, understand in 2 seconds.

## Writing your integrated sketch

Open a new sketch. Copy your Day 24 module code in one chunk at a time, using the structure above. Your goal: **make `loop()` small and readable** by moving logic into helpers.

Here's a full example for the Smart Plant Monitor (adapt to your own project):

```cpp
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

// === Pins ===
const int MOIST_PIN = A0;
const int LDR_PIN   = A1;
const int BUTTON_PIN = 2;
const int RED_LED    = 8;
const int GREEN_LED  = 9;
const int BUZZ_PIN   = 10;

// === Tuning ===
const int MOIST_DRY_THRESHOLD = 400;
const int NIGHT_THRESHOLD     = 200;
const int BEEP_INTERVAL_MS    = 5000;
const int SNOOZE_DURATION_MS  = 10UL * 60 * 1000;  // 10 minutes

// === State ===
int  moisture  = 0;
int  light     = 0;
bool thirsty   = false;
bool isNight   = false;
unsigned long lastBeepTime  = 0;
unsigned long snoozeUntil   = 0;
int  lastButtonState = HIGH;

// === Objects ===
LiquidCrystal_I2C lcd(0x27, 16, 2);

// === Helpers ===
void readSensors();
void handleButton();
void decidePlantStatus();
void updateDisplay();
void updateAlerts();

// ============ setup / loop ============

void setup() {
  pinMode(RED_LED, OUTPUT);
  pinMode(GREEN_LED, OUTPUT);
  pinMode(BUZZ_PIN, OUTPUT);
  pinMode(BUTTON_PIN, INPUT_PULLUP);
  Serial.begin(9600);

  lcd.init();
  lcd.backlight();
  lcd.print("Plant Monitor");
  delay(1000);
  lcd.clear();
}

void loop() {
  readSensors();
  handleButton();
  decidePlantStatus();
  updateDisplay();
  updateAlerts();
}

// ============ Helpers ============

void readSensors() {
  moisture = analogRead(MOIST_PIN);
  light    = analogRead(LDR_PIN);
  isNight  = (light < NIGHT_THRESHOLD);
}

void handleButton() {
  int b = digitalRead(BUTTON_PIN);
  if (lastButtonState == HIGH && b == LOW) {
    snoozeUntil = millis() + SNOOZE_DURATION_MS;
    Serial.println("Snoozed");
  }
  lastButtonState = b;
}

void decidePlantStatus() {
  thirsty = (moisture < MOIST_DRY_THRESHOLD);
}

void updateDisplay() {
  lcd.setCursor(0, 0);
  lcd.print("Moist: ");
  lcd.print(moisture);
  lcd.print("   ");

  lcd.setCursor(0, 1);
  if (thirsty) {
    lcd.print("THIRSTY!       ");
  } else {
    lcd.print("Plant happy :) ");
  }
}

void updateAlerts() {
  if (thirsty) {
    digitalWrite(RED_LED, HIGH);
    digitalWrite(GREEN_LED, LOW);
  } else {
    digitalWrite(RED_LED, LOW);
    digitalWrite(GREEN_LED, HIGH);
  }

  // Beep only in day, when thirsty, and not snoozed
  bool shouldBeep = thirsty && !isNight && millis() > snoozeUntil;

  if (shouldBeep && millis() - lastBeepTime > BEEP_INTERVAL_MS) {
    tone(BUZZ_PIN, 1500, 200);
    lastBeepTime = millis();
  }
}
```

Upload. The flow:

1. Read sensors. 
2. Read button. If pressed, set a 10-minute silence timer.
3. Decide if the plant is thirsty.
4. Update the LCD.
5. Light the right LED and, if all conditions are met, beep every 5 seconds.

## Key patterns in this code

### `millis()` for non-blocking timing

Look at the beep logic:

```cpp
if (shouldBeep && millis() - lastBeepTime > BEEP_INTERVAL_MS) {
  tone(BUZZ_PIN, 1500, 200);
  lastBeepTime = millis();
}
```

No `delay()` anywhere. Instead:

- `lastBeepTime` remembers **when the last beep happened**.
- Every loop, we check: *"has enough time passed since the last beep?"*
- If yes, beep and update `lastBeepTime`.

This pattern is called **non-blocking timing**. It's how you do "every N milliseconds, do X" without freezing the rest of the code. `delay()` would stall the sensor reads, the button, the display — everything. `millis()` lets you do many timed things in parallel.

### `unsigned long` for time

```cpp
unsigned long lastBeepTime = 0;
unsigned long snoozeUntil  = 0;
```

Notice I used `unsigned long` instead of `int` for anything holding `millis()`. Why? Because `int` on Arduino Uno is only 16 bits (holds up to ~32,000). `millis()` counts milliseconds — it hits 32,000 in just 32 seconds. `unsigned long` is 32 bits and holds over 4 billion — good for ~49 days of running.

**Rule:** any variable holding a `millis()` value must be `unsigned long`. Don't use `int`.

### Padding strings for clean LCD

```cpp
lcd.print("THIRSTY!       ");
lcd.print("Plant happy :) ");
```

Trailing spaces to overwrite the previous (possibly longer) string. Same trick as Day 18. Critical when two status strings have different lengths.

### State variables

```cpp
int  moisture  = 0;
int  light     = 0;
bool thirsty   = false;
bool isNight   = false;
```

Global variables at the top hold the **state** of the system — values computed by the sensor reader, decided by the status function, and used by the display and alert functions. Each helper function only reads/writes a few of them, and the overall data flow is easy to follow.

## Debugging integration issues

Things that often go wrong when you glue everything together:

### Issue 1: "Everything works alone but fails when combined"

Usually a **pin conflict** you missed. Double-check that no pin is used by two things. Especially watch for A4/A5 (both I2C and regular analog), pins 0/1 (used for Serial), pin 13 (used by the built-in LED).

### Issue 2: "The LCD freezes for a second every few seconds"

You probably have a `delay()` somewhere that's blocking everything. Replace it with a `millis()`-based check.

### Issue 3: "Random garbage on the LCD"

Likely a **power issue** — too many modules on 5V at once. The IR receiver, the LCD, and the sound sensor all draw a little power. If you're running on USB power and you've got a bunch of modules, consider plugging in a separate 9V battery or USB wall adapter.

### Issue 4: "Button sometimes registers twice"

Bounce. Add a small delay in the button handler (`delay(50)` after detecting a press) or track the last-press time with `millis()`.

### Issue 5: Serial Monitor prints nothing

Is `Serial.begin(9600)` in `setup()`? Is the Serial Monitor set to 9600 baud? Is the Arduino selected and the port correct? These are the 3 usual suspects.

## The debug print trick

When in doubt, **print everything**. Add `Serial.println` lines that say what each function is doing:

```cpp
void decidePlantStatus() {
  thirsty = (moisture < MOIST_DRY_THRESHOLD);
  Serial.print("Moisture = ");
  Serial.print(moisture);
  Serial.print(", thirsty = ");
  Serial.println(thirsty);
}
```

Run the Serial Monitor while interacting with the project. You'll see exactly which functions run in what order and what values they're working with. Most integration bugs become obvious as soon as you print the data flow.

Delete the debug prints once the sketch is stable. Or leave them in — they don't slow anything important down and they help future-you understand the code.

## Checklist before Demo Day

Before tomorrow, go through every item on your "done" list from Day 23. Check them off as you verify:

- [ ] Does every core feature work reliably?
- [ ] Does the LCD show what I expect, with no flicker or garbage?
- [ ] Are there any weird edge cases? (What happens if I press the button during an alert?)
- [ ] Have I written down which remote button does what?
- [ ] Can I explain in 60 seconds what each part of the code does?

If something feels flaky, fix it now. Tomorrow is Demo Day — you don't want to debug in front of an audience.

## Optional polish

- Add a **splash screen** in `setup()` — "Smart Plant Monitor" + your name for 2 seconds before the main display starts.
- Clean up the wires — shorter jumpers, tucked neatly, labeled.
- Write a tiny laminated card saying what each pin does. Pros do this.

## What you learned today

- How to **integrate** modules into one sketch using the "tiny `loop()`, big helpers" pattern
- **`unsigned long`** for time — why you can't use `int` with `millis()`
- **Non-blocking timing** with `millis()` — how to do timed actions without freezing other code
- Common integration bugs: pin conflicts, `delay()` blocking, power issues, bouncing
- The **debug print trick** — add Serial.println lines everywhere when stuck
- The habit of going through your **"done" checklist** before declaring finished

## What is next

[Day 26](/arduino/day-26-demo-day/) — **Demo Day!** You present your project to your family. Tomorrow is the celebration of 26 days of hard work.

Go get some rest, Anish. Tomorrow is the big day.
