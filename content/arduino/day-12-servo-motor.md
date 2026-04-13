---
title: "Day 12: Servo Motor — Rotate to Any Angle"
date: 2026-04-11T23:48:00+05:30
lastmod: 2026-04-11T23:48:00+05:30
tags: ["arduino", "beginner", "kids", "servo", "motor", "library"]
categories: ["arduino"]
summary: "Day 12 of Learn Coding with Arduino — hook up an SG90 servo motor, sweep it from 0° to 180°, and control its position with a potentiometer."
---

Hi Anish! Today is your first motor. A **servo motor** is a tiny motor that can rotate to any exact angle you tell it — 0 degrees, 90 degrees, 180 degrees, anything in between. That makes it perfect for robot arms, little doors, a wagging tail, a steering mechanism, a dial gauge, and a million other things.

## What you need today

- Arduino Uno + USB cable
- Breadboard
- **1 SG90 servo motor** (the most common hobby servo — blue or orange)
- 3 **jumper wires** (female-to-male might help if your servo has male pins)
- (Optional) a potentiometer for the bonus section

## What is a servo?

A regular DC motor just spins continuously when you give it power. A **servo** is smarter — it has a little built-in controller that lets you tell it: *"rotate to exactly 90 degrees and hold there."* It remembers the position and holds still until you tell it otherwise.

A hobby servo like the SG90 can rotate from **0° to 180°** (half a full circle). It has three wires:

- **Red** — 5V (power)
- **Brown or black** — GND
- **Orange or yellow** — signal wire (where Arduino tells it what angle to go to)

## The circuit

{{< mermaid >}}
graph LR
    V5["5V"] --> RED["Servo Red<br/>(power)"]
    GND["GND"] --> BLK["Servo Brown<br/>(ground)"]
    PIN9["Pin 9"] --> ORG["Servo Orange<br/>(signal)"]
{{< /mermaid >}}

**Wiring:**

1. Jumper from **5V** to the **red** wire of the servo.
2. Jumper from **GND** to the **brown** (or black) wire.
3. Jumper from **pin 9** to the **orange** (or yellow) wire.

No breadboard needed if you have female-to-male jumpers — you can plug them directly. If you only have male-to-male, push the servo wires into the breadboard first and use male-to-male jumpers from there.

## The code — sweep back and forth

```cpp
#include <Servo.h>

Servo myServo;

void setup() {
  myServo.attach(9);   // tell the servo library we're using pin 9
}

void loop() {
  // Sweep from 0 to 180 degrees
  for (int pos = 0; pos <= 180; pos++) {
    myServo.write(pos);
    delay(15);
  }

  // Sweep back from 180 to 0 degrees
  for (int pos = 180; pos >= 0; pos--) {
    myServo.write(pos);
    delay(15);
  }
}
```

Upload. The servo arm should sweep slowly from one extreme to the other, then back, forever. Like a windshield wiper in slow motion.

## What's new in this code?

There's a lot to unpack even though it looks short. This is the first time we've used a **library**, which is a huge concept.

### `#include <Servo.h>`

This is a **preprocessor directive** — a special line that starts with `#`. `#include <Servo.h>` tells Arduino: *"before you compile my code, grab the `Servo` library and pull it in so I can use the commands from it."*

A **library** is a pre-written bundle of code that handles a specific thing. Someone (in this case, the Arduino team) wrote the complicated low-level stuff for controlling a servo, packaged it into a library, and now you can use it without knowing the details. Libraries are the reason you can do complex things in just a few lines.

The Arduino IDE already has the `Servo` library installed by default — you don't need to do anything to get it. Later in the series we'll install other libraries (for the LCD screen and the IR remote, for example).

### `Servo myServo;`

This creates a **servo object**. Think of `Servo` like a new kind of variable — the same way `int` is a variable type that holds numbers. `Servo` is a variable type that represents a physical servo motor.

We're saying: *"give me a `Servo` variable and call it `myServo`."* From now on, `myServo` is how we talk to our servo.

You can have multiple servos with multiple names:

```cpp
Servo leftArm;
Servo rightArm;
Servo head;
```

### `myServo.attach(9);`

This tells `myServo` which pin it's connected to. The dot (`.`) means "do something *to* this object." `myServo.attach(9)` means "tell myServo to attach itself to pin 9."

You call this **once in `setup()`**, to hook the software servo to the physical one.

### `myServo.write(pos);`

This tells the servo to rotate to the angle `pos` (which should be between 0 and 180). If `pos` is 0, the servo goes to one extreme. If 180, the other extreme. 90 is exactly in the middle.

Note: `myServo.write(90)` is NOT the same as `digitalWrite(9, HIGH)` or `analogWrite(9, 90)`. The Servo library handles all the low-level timing tricks automatically. You just say "go to 90" and it figures out how.

### `delay(15)` inside the sweep

Why 15ms between each 1-degree step? Because servos can only rotate so fast. If you change the angle too quickly, the servo can't keep up — you'd basically tell it "0, 180, 0, 180" faster than it can physically move. 15ms per degree gives it time to catch up. You can try `delay(5)` for a faster sweep and see what happens.

## Bonus: Knob controls the servo

Add a potentiometer (like Day 8) on pin A0. Now you can turn the knob to point the servo wherever you want.

```cpp
#include <Servo.h>

Servo myServo;

void setup() {
  myServo.attach(9);
}

void loop() {
  int val = analogRead(A0);                 // 0 to 1023
  int angle = map(val, 0, 1023, 0, 180);   // squish to 0-180
  myServo.write(angle);
  delay(15);
}
```

Upload. Turn the knob — the servo follows. That's it. Turn the knob quickly, servo whips around. Turn slowly, servo creeps. You just built a **manually-controlled robotic joint** in 10 lines of code.

Notice how I used `map()` from Day 8 to convert the pot's 0-1023 range into the servo's 0-180 range. This is exactly the same trick, with different numbers.

## Try this

1. **Servo as dial gauge.** Tape a paper arrow to the servo arm and draw a scale on paper behind it (0° on the left, 180° on the right). Now the pot+servo becomes a physical meter. Use it to show the LDR reading from Day 9 — bright room = arm right, dark room = arm left.
2. **Three positions.** Ignore the pot. Make the servo rotate to 0°, 90°, 180°, 90°, 0°, 90°, 180°... holding each position for 1 second. Use `delay(1000)` between each `myServo.write()`.
3. **Slow motion.** In the sweep code, change `delay(15)` to `delay(100)`. Super slow sweep. Good for a wagging tail or a slowly opening door.
4. **Two servos.** If you have a second servo, attach it to pin 10 and make them sweep in opposite directions at the same time. Call it `Servo servoB;` and use it just like `myServo`.

## Key idea: objects have behavior

Here's something worth noticing. An `int` is a variable that holds a number. That's all it does. But `myServo` is a different kind of variable — it **holds a state** (*what pin am I on? what's my current angle?*) and it **has behaviors** (`attach`, `write`, `read`, `detach`). You call those behaviors with the dot (`myServo.write(pos)`).

These fancier variables are called **objects**, and each object type has its own set of behaviors. We'll see this again on Day 15 with the LCD display — there's an `LiquidCrystal_I2C` object with methods like `lcd.print()` and `lcd.setCursor()`.

Don't worry about the big picture yet — just learn the pattern: **object name**, **dot**, **behavior name**, **parentheses with arguments**. That's how you use any library.

## What you learned today

- What a **servo motor** is and how it differs from a regular motor
- Servo has 3 wires: **power (red), GND (brown), signal (orange)**
- **`#include`** — how to pull in a library before your code compiles
- What a **library** is and why they're useful
- **Objects** — variables that have state AND behaviors
- **`Servo myServo;`** — create a servo object
- **`myServo.attach(pin)`** — hook it up in setup
- **`myServo.write(angle)`** — send it to an angle (0-180)
- How to control physical position with a knob via `map()`

## What is next

[Day 13](/arduino/day-13-dc-motor/) — a **regular DC motor**. Unlike a servo, a DC motor just spins continuously, and controlling its speed needs a new trick: a **transistor**. You'll learn why Arduino can't power motors directly and how transistors solve that.

Great work, Anish. Things are moving now.
