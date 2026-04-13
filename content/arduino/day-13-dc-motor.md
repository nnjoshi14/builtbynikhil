---
title: "Day 13: DC Motor, Transistor, and Speed Control"
date: 2026-04-11T23:47:00+05:30
lastmod: 2026-04-11T23:47:00+05:30
tags: ["arduino", "beginner", "kids", "dc-motor", "transistor", "pwm", "speed-control"]
categories: ["arduino"]
summary: "Day 13 of Learn Coding with Arduino — drive a DC motor with a transistor and control its speed using PWM. Meet the diode and understand why Arduino can't power motors directly."
---

Hi Anish! Yesterday we used a servo, which is polite and precise. Today we use a **DC motor** — the kind that just **spins** when you give it power, like in a toy car or a fan. Simple, loud, powerful.

There's a catch: a DC motor draws a lot of current, way more than an Arduino pin can safely give. If you plug one straight into pin 9, you'll damage your Arduino. The solution: a **transistor** — a tiny electronic switch that lets Arduino control lots of power with very little effort. Today you meet your first transistor.

## What you need today

- Arduino Uno + USB cable
- Breadboard
- **1 small DC motor** (the little yellow-geared kind, or any 3-6V hobby motor)
- **1 NPN transistor** (like a 2N2222 or PN2222 — looks like a tiny black half-cylinder with 3 legs)
- **1 × 1kΩ resistor** (brown-black-red stripes) — for the transistor base
- **1 diode** (like 1N4001 or 1N4148 — looks like a small cylinder with a stripe on one end)
- Jumper wires
- (Optional) potentiometer for the speed-control part

## Why can't Arduino power a motor directly?

Every Arduino pin can only safely output a small amount of current — about **20 milliamps (mA)**. That's enough for LEDs, buzzers, and signals. A typical small DC motor needs **100-500 mA** — 10 to 25 times more than a pin can give.

If you connect a motor straight to a pin, one of two things happens:

1. The motor barely twitches because it's not getting enough current.
2. Your Arduino pin burns out because you're trying to pull too much from it.

The fix: a **transistor**. It's like a valve. A tiny signal from Arduino (through the transistor's "base" pin) opens the valve and lets a big flow of power from 5V run through the motor. Arduino says "open," transistor says "yes sir," and a huge current flows — but not through Arduino itself.

## Meet the NPN transistor

An NPN transistor (like the 2N2222) has **3 legs**. Hold it with the flat side facing you:

{{< mermaid >}}
graph TB
    FLAT["Flat side facing you"] --> LEGS["Legs: E — B — C<br/>(Emitter — Base — Collector)"]
{{< /mermaid >}}

The legs from left to right (when looking at the flat side) are:

- **E — Emitter** → goes to **GND**
- **B — Base** → goes to **Arduino pin** (through a 1kΩ resistor)
- **C — Collector** → goes to the **motor's GND side**

The idea: when Arduino sends HIGH to the base, the transistor lets current flow from collector to emitter. When the base is LOW, the transistor blocks the flow. It's a tiny electronic switch, and the Arduino signal controls it.

## Why the diode?

Motors are coils of wire. When you suddenly stop the current flowing through a coil, the coil's magnetic field collapses and spits back a brief high-voltage spike in the **reverse** direction. That spike can fry your transistor and Arduino.

A **diode** is a one-way valve for electricity. We put it across the motor in the **reverse** direction — during normal operation it does nothing, but if the motor tries to spit back a reverse voltage, the diode catches it safely. This is called a **flyback diode** (because it stops the "flyback" voltage).

The diode has a **stripe** on one end (the cathode, the minus side). Important: **the stripe goes toward the + side of the motor power.**

## The circuit

This is the most complex circuit we've wired so far. Read slowly.

{{< mermaid >}}
graph TB
    V5["5V"] --> MOT_POS["Motor + side"]
    MOT_POS --> MOT_NEG["Motor − side"]
    MOT_NEG --> COLL["Transistor Collector (C)"]
    COLL --> TRANS["NPN Transistor"]
    TRANS --> EMIT["Transistor Emitter (E)"]
    EMIT --> G1["GND"]
    PIN9["Pin 9"] --> R1k["1kΩ"]
    R1k --> BASE["Transistor Base (B)"]
    MOT_POS -.->|diode stripe toward +| DIODE["Diode<br/>(flyback protection)"]
    DIODE -.-> MOT_NEG
{{< /mermaid >}}

**Step by step:**

1. Place the NPN transistor on the breadboard with its **flat side** facing you. Each leg goes into its own column.
2. Jumper from the **Emitter** (leftmost leg) to **GND**.
3. Jumper from **pin 9** through a **1kΩ resistor** to the **Base** (middle leg). The resistor protects the transistor's base from too much current.
4. Connect the **motor's − (negative)** wire to the **Collector** (rightmost leg).
5. Connect the **motor's + (positive)** wire to **5V**.
6. Place the **diode across the motor**, with its **stripe** pointing toward the + side of the motor.

Double-check step 6 — it's the one that's easiest to get wrong. Stripe = cathode = points toward positive. If you install the diode backward, it shorts your power supply and bad things happen.

## Part 1: Turn motor on and off

```cpp
void setup() {
  pinMode(9, OUTPUT);
}

void loop() {
  digitalWrite(9, HIGH);   // Motor ON
  delay(2000);
  digitalWrite(9, LOW);    // Motor OFF
  delay(2000);
}
```

Upload. The motor should spin for 2 seconds, stop for 2 seconds, repeat. Just like Day 1's blink, but with a motor instead of an LED. Same `digitalWrite` — the transistor does the heavy lifting.

The code looks almost identical to Day 1 Blink. That's the beautiful part: **from Arduino's point of view, driving a motor through a transistor feels the same as blinking an LED.** The transistor handles all the power complexity invisibly.

## Part 2: Speed control with PWM

Remember PWM from Day 8? We used it to dim an LED. The exact same trick works here — `analogWrite` on a `~` pin to control motor speed. The motor doesn't glow bright/dim; instead, it spins fast/slow (or not at all).

Add a potentiometer on A0 (same wiring as Day 8).

```cpp
void setup() {
  pinMode(9, OUTPUT);
  Serial.begin(9600);
}

void loop() {
  int val = analogRead(A0);
  int speed = map(val, 0, 1023, 0, 255);
  analogWrite(9, speed);
  Serial.print("Speed: ");
  Serial.println(speed);
  delay(100);
}
```

Upload. Turn the pot — motor speeds up and slows down smoothly. `0` = stopped, `255` = full speed.

Watch the Serial Monitor to see the exact PWM value being sent.

### Note: motors have a "minimum speed"

At very low PWM values (say, 0 to 40), the motor might not spin at all — it needs a minimum push to overcome friction and startup resistance. Below that, it just sits there buzzing. Above some threshold it suddenly starts spinning. That's normal for DC motors.

You can tune the range so the knob is more useful:

```cpp
int speed = map(val, 0, 1023, 60, 255);
```

This way, even at the lowest knob setting, you send `60` (enough to start turning), and full knob sends `255`. Cleaner feel.

## Why today is a big deal

Compare Day 1 (blink LED) to today:

- **Day 1** — Arduino directly controls a 20 mA LED.
- **Day 13** — Arduino controls a 300 mA motor by *telling a transistor what to do*.

You just learned the pattern for controlling ANY high-power device: fan, pump, strip of LEDs, solenoid, relay. Different parts, same pattern. Arduino → transistor → big thing.

You'll use this pattern for the rest of your Arduino life.

## Try this

1. **Fixed speed, blink style.** Set the motor to medium speed (`analogWrite(9, 150)`) for 3 seconds, then stop, then repeat. No pot needed.
2. **Ramp up and down.** Use a for loop to smoothly go from 0 to 255 over 2 seconds, then back. Motor spins up like a jet engine starting, then spins down.
3. **Button control.** Add a button (INPUT_PULLUP, pin 2). Motor only runs while the button is held. Release → stop.
4. **Light sensor controls speed.** Replace the pot with the LDR from Day 9. Motor spins faster when the room is brighter.

## What you learned today

- Why Arduino pins can't safely drive motors directly (current limit ~20 mA)
- What a **transistor** is: an electronic switch that lets a small signal control a big flow
- The 3 legs of an **NPN transistor**: Emitter, Base, Collector
- Why a **flyback diode** is needed to protect from coil voltage spikes
- A **1kΩ resistor** on the base protects the transistor
- The Arduino code for a motor looks almost like the code for an LED — because the transistor hides the difference
- **PWM speed control** (`analogWrite`) — same as LED dimming, but for motors

## What is next

[Day 14](/arduino/day-14-auto-fan/) — combine the DC motor with a sensor to build an **Auto Fan System**: the motor automatically runs when it's hot (or bright). First proper "smart" mechanical project.

Great work, Anish. You just leveled up from lights to physics.
