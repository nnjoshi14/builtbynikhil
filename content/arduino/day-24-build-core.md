---
title: "Day 24: Build Your Project — One Piece at a Time"
date: 2026-04-11T23:36:00+05:30
lastmod: 2026-04-11T23:36:00+05:30
tags: ["arduino", "beginner", "kids", "project", "building", "testing"]
categories: ["arduino"]
summary: "Day 24 of Learn Coding with Arduino — take the project plan from Day 23 and start building. One module at a time, tested and working, before moving to the next."
---

Hi Anish! Yesterday you planned. Today you **build**. But not everything at once — we're going to follow one rule that pros live by:

> **Make one thing work before starting the next.**

This sounds obvious but it's the single most common mistake people make when building projects. They wire up everything at once, upload their full sketch, and... nothing works. Then they spend hours trying to figure out which part is broken. Don't do that.

## What we're doing today

Take your project plan from Day 23 and split it into **modules**. A module is one chunk that you can test on its own. For our Smart Plant Monitor example, the modules are:

1. **LCD module** — "can I display text?"
2. **Moisture sensor module** — "can I read moisture values and print to Serial?"
3. **LED module** — "can I turn the LEDs on and off?"
4. **Button module** — "can I detect button presses?"
5. **Buzzer module** — "can I make it beep?"
6. **LDR module** — "can I read day/night?"

Each module is a tiny sketch you already know how to write (from earlier days). You're going to make each one work by itself, then mash them together tomorrow.

## Step 1: Lay out the breadboard

Wire **everything** you'll need for the full project, but don't connect anything to Arduino yet — except power (5V) and ground rails. Have the parts physically in place so you don't have to redo wiring tomorrow.

Draw your wiring plan first. Use this as a checklist:

- [ ] Power rails: 5V and GND, connected to Arduino
- [ ] Each module's components placed on the breadboard
- [ ] Jumper wires ready but not yet connected to Arduino pins

## Step 2: Test each module individually

Here's the key: upload a **tiny sketch** that tests ONE module. If it works, move on to the next. Never try to test two at once.

Below is how the Smart Plant Monitor testing would go. Adapt the same pattern to your own project.

### Module 1: LCD

Wire the LCD to A4/A5 and run this:

```cpp
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
LiquidCrystal_I2C lcd(0x27, 16, 2);

void setup() {
  lcd.init();
  lcd.backlight();
  lcd.print("Plant Monitor");
  lcd.setCursor(0, 1);
  lcd.print("Test OK");
}

void loop() {}
```

Upload. Does it show "Plant Monitor / Test OK"? **If yes**, LCD module works, move on. **If no**, go back to Day 15 troubleshooting. Don't move on until this is fixed. Mark it done in your notebook.

### Module 2: Moisture sensor (or pot stand-in)

Connect the moisture sensor signal to A0.

```cpp
void setup() { Serial.begin(9600); }
void loop() {
  int m = analogRead(A0);
  Serial.print("Moisture: ");
  Serial.println(m);
  delay(500);
}
```

Upload, open Serial Monitor. Push the sensor into wet soil (or a cup of water, carefully). The number changes. Lift it out, dry it — the number changes again. Write down the wet number and the dry number. **You will need these as thresholds later.**

Don't just stare at the numbers — actually test the full range. How low can it go? How high? What's the number when it's "just barely wet enough"? Those are the numbers that determine your threshold.

### Module 3: LEDs

Wire the red LED to pin 8 and the green to pin 9, each with a 220Ω resistor.

```cpp
void setup() {
  pinMode(8, OUTPUT);
  pinMode(9, OUTPUT);
}

void loop() {
  digitalWrite(8, HIGH);
  digitalWrite(9, LOW);
  delay(500);
  digitalWrite(8, LOW);
  digitalWrite(9, HIGH);
  delay(500);
}
```

Upload. LEDs should alternate. Both work? Check. Move on.

### Module 4: Button

Wire the button to pin 2 with INPUT_PULLUP.

```cpp
void setup() {
  pinMode(2, INPUT_PULLUP);
  Serial.begin(9600);
}

void loop() {
  if (digitalRead(2) == LOW) Serial.println("Pressed!");
  delay(100);
}
```

Upload. Press button. See "Pressed!"? Good.

### Module 5: Buzzer

Wire the buzzer to pin 10.

```cpp
void setup() { pinMode(10, OUTPUT); }
void loop() {
  tone(10, 1000, 200);
  delay(1000);
}
```

Upload. Beep every second? Done.

### Module 6: LDR

Wire the LDR voltage divider to A1.

```cpp
void setup() { Serial.begin(9600); }
void loop() {
  int l = analogRead(A1);
  Serial.print("Light: ");
  Serial.println(l);
  delay(500);
}
```

Upload. Cover/uncover — number changes. Write down your "day" and "night" values.

## Step 3: Write everything down

After each module passes, write in your notebook:

- ✅ Module name
- The pin it's on
- Any threshold numbers you discovered (e.g., "dry soil ≈ 250, wet soil ≈ 800, threshold = 400")
- Any weird behavior ("LED flickers slightly when LCD updates — probably fine, but note it")

These notes are **gold** tomorrow when you write the combined sketch. You won't have to re-measure anything.

## Why this is better than building the whole thing at once

If you wire everything at once and upload your full sketch and it doesn't work, you have **no idea** which of the 6 modules is broken. The sensor? The wiring? The button? The threshold? Pin conflict? Power issue? You'll waste hours.

If you test one module at a time and each one works, then when the combined sketch fails, you know the problem is **not in the modules themselves — it's in the way they interact**. That's a much smaller bug space to search.

Pros call this **bring-up**. Every embedded engineer does it.

## "But what if a module fails?"

That's the point of testing them alone — to find the problems **while the code is simple**. When you're stuck:

1. **Read the error or lack of response carefully.** The Serial Monitor is your friend.
2. **Check wiring first.** 70% of "broken modules" are actually just a loose wire or wrong pin.
3. **Check polarity.** Long leg, short leg, stripe direction — all the things that can be put in backward.
4. **Check power.** Is 5V actually reaching the module? Ground?
5. **If all else fails**, go back to the "Day X" post where you first learned that component. The wiring and code there work; see what's different.

Don't feel bad if you get stuck. Debugging is 80% of real software engineering. **You're learning the most valuable skill of all.**

## Pin conflicts

One thing to watch for: **no two things on the same pin**. Look at your plan:

| Pin | Who uses it |
|---|---|
| A0 | Moisture sensor |
| A1 | LDR |
| A4 | LCD SDA (shared I2C) |
| A5 | LCD SCL (shared I2C) |
| 2  | Button |
| 8  | Red LED |
| 9  | Green LED |
| 10 | Buzzer |

If two things collide on a pin, neither will work right. Double-check before you wire. Adjust your plan if you find a conflict.

## Document as you go

Take a photo of your breadboard after each module works. That way, if something later gets accidentally moved, you have a reference to get back to a known-good state.

Also: write 2 sentences in your notebook:

- **"What worked?"**
- **"What didn't work, and how did I fix it?"**

Even just two lines per day makes a huge difference when you come back to this project in a month and don't remember anything.

## What you learned today

- The rule: **make one thing work before starting the next**
- How to split a big project into **modules**
- How to write a **tiny test sketch** for each module to confirm it works alone
- Why this approach makes debugging much easier
- How to **measure thresholds** live with the Serial Monitor
- The habit of **writing down what you learned** as you go
- The concept of **pin conflicts** and how to avoid them

## What is next

[Day 25](/arduino/day-25-integration-debug/) — now we **combine** the modules into the final sketch, debug any integration issues, and polish the user experience. The hardest day of the project, but you've done all the preparation work.

You've got this, Anish. Keep going.
