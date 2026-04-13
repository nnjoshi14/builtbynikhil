---
title: "Day 14: Project — Auto Fan System"
date: 2026-04-11T23:46:00+05:30
lastmod: 2026-04-11T23:46:00+05:30
tags: ["arduino", "beginner", "kids", "project", "motor", "sensor", "auto-fan"]
categories: ["arduino"]
summary: "Day 14 of Learn Coding with Arduino — build an automatic fan that turns on when the room gets too bright or too hot, with a buzzer alert. Everything from Days 1-13 in one project."
---

Hi Anish! Project day. We're going to build an **Auto Fan System** — a fan that automatically turns on when the room gets bright (or hot, if you have a temperature sensor). When the fan comes on, the buzzer from Day 11 plays a short alert. When the room cools down, the fan stops. Fully automatic, no human needed.

This is the third real project in the series, and it uses components from Day 9 (LDR), Day 11 (buzzer), and Day 13 (motor + transistor). No new concepts today — just combining old ones.

## What you need today

- Arduino Uno + USB cable
- Breadboard
- **DC motor + transistor + diode + 1kΩ resistor** (the full Day 13 setup)
- **LDR + 10kΩ resistor** (the Day 9 setup) — OR a temperature sensor like an LM35 if you have one
- **Buzzer** (Day 11)
- Jumper wires

## What we're building

Rules:

1. Read the light sensor (or temperature sensor) every 100ms.
2. If the reading says "it's too bright / too hot," turn the fan on at full speed and play a short warning beep.
3. If the reading says "it's cool enough," turn the fan off.
4. Print the current state and reading to the Serial Monitor so you can see what's happening.

## The circuit

Three subsystems, one Arduino.

{{< mermaid >}}
graph TB
    subgraph Sensor["Light sensor (Day 9)"]
      V5["5V"] --> LDR["LDR"] --> JUNC["junction"] --> A0["A0"]
      JUNC --> R10k["10kΩ"] --> G1["GND"]
    end
    subgraph Fan["Motor + Transistor (Day 13)"]
      V5b["5V"] --> MOTp["Motor +"]
      MOTp --> MOTn["Motor −"] --> COLL["Transistor C"]
      COLL --> TRANS["NPN Transistor"] --> EMIT["Transistor E"] --> G2["GND"]
      PIN9["Pin 9"] --> R1k["1kΩ"] --> BASE["Transistor B"]
      MOTp -.->|flyback diode| MOTn
    end
    subgraph Buzz["Buzzer (Day 11)"]
      PIN8["Pin 8"] --> BZp["Buzzer +"] --> BZn["Buzzer −"] --> G3["GND"]
    end
{{< /mermaid >}}

**Pin summary:**

- **A0** — LDR voltage divider (reads light)
- **Pin 8** — buzzer
- **Pin 9** — motor (through the transistor)

Wire each subsystem on the same breadboard. They all share the same GND rail and 5V rail. Be extra careful that the diode's stripe points toward the **+ side** of the motor.

## The code

```cpp
// === Tweakable settings ===
int brightThreshold = 600;    // above this = too bright
int fanSpeed        = 200;    // 0-255 PWM
int beepFreq        = 1500;   // Hz
int beepLength      = 200;    // ms

// === State ===
bool fanOn = false;

void setup() {
  pinMode(9, OUTPUT);    // Motor
  pinMode(8, OUTPUT);    // Buzzer
  Serial.begin(9600);
}

void loop() {
  int light = analogRead(A0);
  Serial.print("Light: ");
  Serial.print(light);

  if (light > brightThreshold) {
    if (!fanOn) {
      // Fan was off — turn it on and beep
      tone(8, beepFreq, beepLength);
      fanOn = true;
    }
    analogWrite(9, fanSpeed);
    Serial.println("  |  FAN ON");
  } else {
    if (fanOn) {
      // Fan was on — turn it off and beep low
      tone(8, beepFreq / 2, beepLength);
      fanOn = false;
    }
    analogWrite(9, 0);
    Serial.println("  |  FAN OFF");
  }

  delay(100);
}
```

Upload. Watch the Serial Monitor. The output looks something like:

```
Light: 400  |  FAN OFF
Light: 421  |  FAN OFF
Light: 650  |  FAN ON       ← buzzer beeps
Light: 680  |  FAN ON
Light: 550  |  FAN OFF      ← buzzer lower beep
```

Shine a flashlight on the LDR — the fan spins, buzzer beeps. Cover the LDR — fan stops, lower beep.

## What's worth noticing in this code

No new functions, but let me point out a few things you might have missed.

### `if (!fanOn)` — reacting to changes, not states

```cpp
if (light > brightThreshold) {
  if (!fanOn) {
    tone(8, beepFreq, beepLength);  // beep ONCE when the fan turns on
    fanOn = true;
  }
  analogWrite(9, fanSpeed);
  ...
}
```

Why the inner `if (!fanOn)`? Because the outer `if` runs **every single time** the loop checks and finds the room is bright — that's 10 times per second. If we beeped every time, we'd get a constant stream of annoying beeps.

Instead, we remember whether the fan is *currently* on (`fanOn` variable), and only beep **when the state changes from off to on**. Same trick as Day 7's state change detection — but for a sensor threshold instead of a button press.

This is a really important pattern: **"do X only when something changes."** You'll use it in almost every project from here on.

### Settings at the top of the file

```cpp
int brightThreshold = 600;
int fanSpeed        = 200;
int beepFreq        = 1500;
int beepLength      = 200;
```

I put every tweakable number at the top so future-you can adjust the behavior without hunting through the code. This is the same habit from Day 10. Do it in every project.

### `beepFreq / 2` — math in arguments

```cpp
tone(8, beepFreq / 2, beepLength);
```

You can do math inside function arguments! `beepFreq / 2` is `1500 / 2 = 750`. So the "fan off" beep is half the frequency of the "fan on" beep — a lower pitch. Gives the two events different sounds so you can tell them apart by ear. All the basic math operators work: `+`, `-`, `*`, `/`, `%` (remainder).

## Tuning for your room

The **`brightThreshold = 600`** is a guess. You'll need to tune it for YOUR room:

1. Open the Serial Monitor with the default code running.
2. Watch what value `light` reads in normal conditions.
3. Cover/uncover the LDR to see the range.
4. Pick a threshold that's clearly above the "normal" reading but below the "really bright" reading.

Same for **`fanSpeed`** — start at `200` and adjust. Some motors need `150`, some need `255`. Too low and the motor doesn't spin; too high wastes power. Find a sweet spot.

## Try this

1. **Temperature instead of light.** If you have an LM35 temperature sensor, replace the LDR. LM35 wiring is simple: + to 5V, GND to GND, signal to A0. Use `analogRead(A0) * 0.48828125` to get temperature in Celsius roughly.
2. **Variable fan speed.** Instead of on/off at a fixed speed, use `map()` to make the fan spin faster as the room gets brighter. Rich people have fans like this.
3. **Manual override button.** Add a button (INPUT_PULLUP on pin 2) that forces the fan ON regardless of the sensor. Copy the mode-switching pattern from Day 10.
4. **Shut-off timer.** Once the fan has been on for 10 seconds, force it to pause for 5 seconds, then resume. Good for preventing the motor from overheating.

## What you learned today

- How to **combine** multiple subsystems (sensor + motor + buzzer) into one project
- **State-change detection** applied to a sensor threshold (not just button presses)
- Why you should beep/alert on *changes* not on *states*
- Math operators inside function arguments: `freq / 2`
- The habit of putting **all tweakable settings at the top** of your sketch

## What is next

[Day 15](/arduino/day-15-lcd-basics/) — we start a new chapter: **displays**. You'll hook up a **16×2 LCD screen** and make Arduino say "Hello Anish!" in text. Next step: Arduino can talk back to you without needing the Serial Monitor or your laptop.

Great work, Anish. Every project from here on uses the patterns you learned today.
