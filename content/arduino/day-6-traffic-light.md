---
title: "Day 6: Build a Traffic Light System"
date: 2026-04-11T23:54:00+05:30
lastmod: 2026-04-11T23:54:00+05:30
tags: ["arduino", "beginner", "kids", "led", "project", "traffic-light"]
categories: ["arduino"]
summary: "Day 6 of Learn Coding with Arduino — build a full red-yellow-green traffic light with realistic timing, then add a pedestrian button that overrides the cycle."
---

Hi Anish! Today is your **first real project**. No new concepts — we're going to combine everything from Days 1-5 to build a traffic light. Red, yellow, green, with realistic timing. And if you feel brave, we'll add a **pedestrian button** that interrupts the cycle when someone wants to cross.

This is the kind of project a working engineer might actually build (with fancier parts). You're about to ship one with an Arduino, 3 LEDs, and 20 lines of code.

## What you need today

- Arduino Uno + USB cable
- Breadboard
- 3 LEDs — **red, yellow, and green**
- 3 × **220Ω resistors**
- 1 **push button** (for the pedestrian button — optional)
- 1 **10kΩ resistor** (pull-down for the button — optional)
- 6-8 **jumper wires**

## The circuit

Same 3-LED setup as Day 5, plus an optional button wired like Day 3.

{{< mermaid >}}
graph LR
    PIN8["Pin 8"] --> R1["220Ω"] --> RED["Red LED"] --> G1["GND"]
    PIN9["Pin 9"] --> R2["220Ω"] --> YEL["Yellow LED"] --> G2["GND"]
    PIN10["Pin 10"] --> R3["220Ω"] --> GRN["Green LED"] --> G3["GND"]
    V5["5V"] --> BTNa["Button"] -.->|pressed| BTNb["Button"]
    BTNb --> PIN2["Pin 2"]
    BTNb --> R10k["10kΩ pull-down"] --> G4["GND"]
{{< /mermaid >}}

**Pin assignments:**

- **Pin 8** — Red LED
- **Pin 9** — Yellow LED
- **Pin 10** — Green LED
- **Pin 2** — Pedestrian button (input, with 10kΩ pull-down)

Wire the 3 LEDs exactly like Day 5. Wire the button exactly like Day 3 — but this time plug it into **pin 2** instead of pin 7. (Why pin 2? Because later in the series we'll use pins 2 and 3 for a special feature called "interrupts" — good habit to start now.)

## Part 1: Basic traffic light (no button)

How does a real traffic light work?

- **Green** stays on for a long time (cars flow freely)
- **Yellow** flashes briefly as a warning ("get ready to stop!")
- **Red** stays on for a long time (stopped)
- Then back to green. Repeat forever.

In code:

```cpp
void setup() {
  pinMode(8, OUTPUT);   // Red
  pinMode(9, OUTPUT);   // Yellow
  pinMode(10, OUTPUT);  // Green
}

void loop() {
  // GREEN — cars go
  digitalWrite(10, HIGH);
  delay(3000);            // 3 seconds
  digitalWrite(10, LOW);

  // YELLOW — warning
  digitalWrite(9, HIGH);
  delay(1000);            // 1 second
  digitalWrite(9, LOW);

  // RED — cars stop
  digitalWrite(8, HIGH);
  delay(3000);            // 3 seconds
  digitalWrite(8, LOW);
}
```

Upload. You should see: **green for 3 seconds → yellow for 1 second → red for 3 seconds → back to green**, forever.

Nothing new in this code — you already know `pinMode`, `digitalWrite`, `delay`. We just chained them together in a real-world pattern.

Try tweaking the timings. Real-life traffic lights use about **25 seconds** for green/red and **4 seconds** for yellow. The ratio is what matters: **green and red are roughly equal, yellow is short.**

## Part 2: Adding the pedestrian button

Now the fun part. When someone presses the button, the light should **switch to red immediately** so the pedestrian can cross.

```cpp
void setup() {
  pinMode(8, OUTPUT);   // Red
  pinMode(9, OUTPUT);   // Yellow
  pinMode(10, OUTPUT);  // Green
  pinMode(2, INPUT);    // Button
}

void loop() {
  // GREEN — unless someone presses the button
  digitalWrite(10, HIGH);
  for (int t = 0; t < 30; t++) {      // 30 × 100ms = 3 seconds
    if (digitalRead(2) == HIGH) {
      break;                           // ← button pressed, jump out of for-loop
    }
    delay(100);
  }
  digitalWrite(10, LOW);

  // YELLOW — warning (always 1 second)
  digitalWrite(9, HIGH);
  delay(1000);
  digitalWrite(9, LOW);

  // RED — let people cross
  digitalWrite(8, HIGH);
  delay(3000);
  digitalWrite(8, LOW);
}
```

Upload. Now during the green phase, press the button. Within a fraction of a second, the light jumps to yellow, then red. Pedestrians cross safely.

### Wait — what is `break;`?

New keyword today! `break` means **"stop this loop right now and jump out."** It only works inside a `for` or `while` loop. When Arduino hits `break;`, it exits the loop immediately and runs the next instruction after it.

Here, we're using it to say: *"Normally loop 30 times (waiting 100ms each = 3 seconds total), BUT if at any point the button is pressed, bail out early."*

### Why not just use `delay(3000)`?

Because `delay(3000)` is **blocking**. While Arduino is counting 3 seconds, it can't do anything else — including checking the button. The whole board just sits there, deaf to the world.

The trick is: instead of **one long `delay(3000)`**, we do **30 short `delay(100)`s in a for loop**, and check the button between each one. We're still waiting 3 seconds in total, but now we peek at the button 30 times along the way.

This is the first taste of a really important idea in Arduino: **if you want to do multiple things at once, don't use long delays.** Break them into small chunks with work in between. On Day 25 we'll learn a fancier way using `millis()`, but the "break a big delay into small delays" trick is a solid starting point.

## Try this

1. **Make yellow blink** before going red. Flash yellow on/off 3 times during the warning phase. (Hint: a for loop inside the yellow section.)
2. **Add a blinking red "walk" signal** — after showing red, blink a 4th LED (on pin 11) to simulate a "WALK" signal for 2 seconds before going back to green.
3. **Change the timings to look realistic.** Use `int greenTime = 5000;` and `int redTime = 5000;` at the top of the file. Variables make tweaking easier.
4. **Two-way traffic.** Wire a second set of 3 LEDs on pins 5, 6, 7. When one direction is green, the other is red. Like a real intersection.

## What you learned today

- How to combine multiple LEDs with realistic timing
- How to read a button in the middle of a timing loop (check between small delays)
- **`break;`** — exit a loop early
- Why long `delay()` calls are "blocking" and limit what Arduino can do
- Your first real **project**: a traffic light with pedestrian override

## What is next

[Day 7](/arduino/day-7-input-pullup/) — we revisit the button, but with a much **cleaner** way to wire it. You'll learn about `INPUT_PULLUP`, which saves you an entire resistor and simplifies the circuit. Same idea as Day 3, but grown up.

Brilliant work, Anish. You just built a real thing.
