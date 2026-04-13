---
title: "Day 23: Pick Your Project — Brainstorm and Plan"
date: 2026-04-11T23:37:00+05:30
lastmod: 2026-04-11T23:37:00+05:30
tags: ["arduino", "beginner", "kids", "project", "planning", "design"]
categories: ["arduino"]
summary: "Day 23 of Learn Coding with Arduino — the first day of your final project. Brainstorm ideas, pick one that excites you, and sketch the system diagram before writing any code."
---

Hi Anish! Today marks a big shift. For the last 22 days, I've been telling you what to build. Starting today, **you** pick what to build. The final project is yours.

Over the next 4 days (Day 23 → Day 26), you'll:

- **Day 23** — Brainstorm and pick a project. Sketch how it will work.
- **Day 24** — Build and test each piece separately.
- **Day 25** — Stitch it all together and debug.
- **Day 26** — Demo it to your family. 🎉

Today is **all planning**. No code, no wiring. Just thinking and sketching. That might feel weird — you've been making things for 22 days straight. But here's the thing: **every real engineer plans before they build.** If you skip planning, you waste hours wiring something that doesn't work.

## What you need today

- A notebook
- A pen or pencil
- Maybe an eraser (plans change)
- A pile of your Arduino components nearby, so you know what's available

## Step 1: Brainstorm

Pick a quiet 10 minutes. Write down **at least 5 project ideas**, no matter how silly. Don't filter — just dump everything that pops into your head. Some seed ideas to get you started:

- **Smart Plant Monitor** — tells you when your plant needs water (moisture sensor + LCD + buzzer)
- **Intruder Alarm** — detects motion or sound and flashes LEDs (sound sensor + LEDs + buzzer)
- **Clap Night Light** — clap in the dark to turn on a light; clap again to turn it off
- **Reaction Game** — a game where you press a button as fast as possible when an LED lights up, and it shows your time on the LCD
- **Morse Code Practice** — tap a button to enter Morse; LCD decodes what you tapped
- **Auto Curtain / Blind Opener** — a servo that rotates when the sun comes up (LDR + servo)
- **Room Temperature Display** — LM35 + LCD + color-coded LEDs (blue/green/red for cold/comfy/hot)
- **Mini Piano** — 5 buttons play 5 different notes through the buzzer
- **Toy Robot Arm** — two servos controlled by two potentiometers
- **Parking Assist** — distance sensor + buzzer that beeps faster as you get closer to a wall

These are ideas to inspire you. **Feel free to invent your own.** The best projects are the ones you actually care about.

## Step 2: Pick one

Look at your list. Cross off anything that:

- Needs a part you don't have and can't easily get
- Would take more than 4 days (keep it small — you can always make v2 later)
- Doesn't actually excite you

Now circle the one you like most. That's your project.

For the rest of this post, I'm going to use **Smart Plant Monitor** as a running example so you can see how the planning works. You'll do the same steps for whatever you picked.

## Step 3: Write the "one-liner"

In ONE sentence, describe what your project does:

> "A gadget that monitors how much water my plant has and beeps at me when it's thirsty."

This is harder than it sounds. If you can't fit your project into one sentence, it's probably too complex. Simplify.

## Step 4: List the inputs and outputs

Every Arduino project boils down to:

- **Inputs** — things the Arduino reads (sensors, buttons, remotes)
- **Outputs** — things the Arduino controls (LEDs, motors, buzzers, screens)

Write them down. For our Smart Plant Monitor:

**Inputs:**

- Soil moisture sensor (connected to A0) → reads how wet the soil is
- Button (pin 2, INPUT_PULLUP) → "snooze" button to silence the alarm
- LDR (A1) → knows if it's night (to avoid beeping at 2am)

**Outputs:**

- LCD (A4/A5) → shows moisture level + status message
- Red LED (pin 8) → lights up when plant is thirsty
- Green LED (pin 9) → lights up when plant is happy
- Buzzer (pin 10) → beeps when plant is thirsty (but only during the day)

If you can list your inputs and outputs, you're halfway to done.

## Step 5: Sketch the system diagram

Take your notebook and draw a box in the middle labeled **Arduino**. Around it, draw smaller boxes for each input and output. Connect them to the Arduino with arrows showing the direction of data flow.

For our Smart Plant Monitor:

{{< mermaid >}}
graph LR
  subgraph Inputs
    MOIST["Moisture sensor"]
    BTN["Snooze button"]
    LDR["LDR (day/night)"]
  end
  subgraph Arduino
    UNO["Arduino Uno"]
  end
  subgraph Outputs
    LCD["LCD display"]
    RED["Red LED"]
    GREEN["Green LED"]
    BUZZ["Buzzer"]
  end
  MOIST --> UNO
  BTN --> UNO
  LDR --> UNO
  UNO --> LCD
  UNO --> RED
  UNO --> GREEN
  UNO --> BUZZ
{{< /mermaid >}}

This is called a **block diagram** or **system diagram**. It doesn't show the wires yet — it shows *what talks to what*. Don't skip this step even if your project feels simple. It will save you time tomorrow.

## Step 6: Write the rules (pseudo-code)

Before writing actual Arduino code, write the logic in **plain English**. This is called **pseudo-code** — fake code that describes what the real code will do.

For the Smart Plant Monitor:

```
Every 1 second:
  Read the moisture level.
  Read the LDR to see if it's day or night.
  Show moisture on the LCD.

  If moisture is LOW (plant thirsty):
    Turn on the red LED.
    Turn off the green LED.
    If it's DAY and the snooze button is NOT pressed:
      Beep the buzzer.
  Else (plant happy):
    Turn on the green LED.
    Turn off the red LED.
    No beep.

  If the snooze button is pressed:
    Silence the buzzer for 10 minutes.
```

This is the "recipe" for your code. Anybody who reads this will understand what you're building, even someone who doesn't know Arduino.

**Tip:** Write pseudo-code before code. It's the cheapest way to find logic bugs. If the pseudo-code looks confused, the real code will be 10x more confused.

## Step 7: List the components you need

From the inputs/outputs list and the block diagram, write down EVERY part you need:

- 1 soil moisture sensor
- 1 LDR + 10kΩ resistor
- 1 push button
- 1 LCD with I2C module
- 1 red LED + 220Ω resistor
- 1 green LED + 220Ω resistor
- 1 buzzer
- 1 breadboard
- ~10 jumper wires

Check your kit. Do you have everything? If not, can you either:

- Order the missing parts today, or
- Find a substitute (e.g., a potentiometer instead of a moisture sensor — turn the knob manually to simulate wet/dry), or
- Redesign the project to use only what you have?

**Don't skip this check.** Every kid (and every adult) has wasted a day building half a project and then realized halfway through that they were missing a part.

## Step 8: Define "done"

Before you build anything, decide what "finished" means. For the Smart Plant Monitor:

- [ ] LCD shows the moisture reading in real time
- [ ] Red LED turns on when moisture drops below threshold
- [ ] Green LED turns on when moisture is above threshold
- [ ] Buzzer beeps intermittently in thirsty-mode during the day
- [ ] Buzzer stays silent at night
- [ ] Snooze button silences the buzzer for 10 minutes

That's 6 specific things. If all 6 are working by Day 25, you're done. Don't add more features until you've nailed those.

**Good engineers ship small things that work.** Bad engineers plan huge things that don't.

## For the rest of today

Spend the rest of this session:

1. Finalizing your one-liner
2. Drawing your block diagram cleanly
3. Writing your pseudo-code
4. Checking that you have all the parts
5. Writing your "done" checklist

Show it to your dad / a parent and explain your plan. If you can explain it in under 2 minutes without confusing them, you're ready for tomorrow. If not, simplify.

## What you learned today

- Why **planning before building** saves time and frustration
- How to write a **project one-liner** (the elevator pitch)
- **Inputs vs outputs** as the core map of any Arduino project
- How to draw a **block diagram** of subsystems
- **Pseudo-code** — writing your logic in plain English before coding it
- A **component checklist** to catch missing parts early
- Defining a **"done" checklist** so you know when to stop

## What is next

[Day 24](/arduino/day-24-build-core/) — now you start **building** the individual pieces of your project. Don't try to wire everything at once. One thing at a time, tested and working, before moving to the next.

Take a break, Anish. Tomorrow you start building.
