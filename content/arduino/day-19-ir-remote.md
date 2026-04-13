---
title: "Day 19: IR Remote — Control Arduino with a TV Remote"
date: 2026-04-11T23:41:00+05:30
lastmod: 2026-04-11T23:41:00+05:30
tags: ["arduino", "beginner", "kids", "ir", "remote", "wireless", "library"]
categories: ["arduino"]
summary: "Day 19 of Learn Coding with Arduino — hook up an IR receiver and use any TV or DVD remote to control LEDs. Your first wireless input."
---

Hi Anish! Today, Arduino goes **wireless**. We're going to plug in a tiny **infrared (IR) receiver** and use any TV or DVD remote in your house to control the LEDs. Press the volume-up button — LED on. Press volume-down — LED off. You can use any remote; Arduino reads the codes and acts.

This is your first **wireless input**. Everything up to now has needed physical wires — today, you push a button from across the room.

## What you need today

- Arduino Uno + USB cable
- Breadboard
- **1 × VS1838B IR receiver** (or any 3-pin IR receiver — looks like a small dome with 3 legs)
- **1 LED** + **220Ω resistor**
- **Any TV, DVD, or cheap universal remote** — it doesn't matter which brand
- Jumper wires

## What is IR?

**IR** stands for **infrared** — light that's just beyond what our eyes can see (below red on the color spectrum). TV remotes work by blinking an infrared LED at the front, very fast, in a specific pattern. The pattern encodes which button was pressed.

An **IR receiver** (like the VS1838B) is a little sensor that picks up these IR blinks and converts them into electrical pulses that Arduino can read. The library we install decodes those pulses into button codes.

Why IR instead of Bluetooth or Wi-Fi? Because it's **cheap**, **simple**, and there are **billions of remotes** in the world already. Your old TV remote, a hotel AC remote, a cheap dollar-store one — they all work.

## The VS1838B pinout

Hold the IR receiver with the rounded side (the "eye") facing you. From left to right, the pins are:

- **OUT** — signal (goes to an Arduino pin)
- **GND** — ground
- **VCC** — 5V

Some receivers have a different order — check the datasheet or the markings on the PCB if you have one. If in doubt, try different orders (never connect VCC to GND directly; that's the only combination that can damage it).

## The circuit

{{< mermaid >}}
graph LR
    V5["5V"] --> VCC["IR VCC"]
    GND["GND"] --> GND_IR["IR GND"]
    OUT["IR OUT"] --> PIN11["Pin 11"]
    PIN8["Pin 8"] --> R220["220Ω"] --> LEDp["LED +"] --> LEDn["LED −"] --> GND2["GND"]
{{< /mermaid >}}

**Wiring the IR receiver:**

1. Jumper from **5V** to the receiver's **VCC**.
2. Jumper from **GND** to the receiver's **GND**.
3. Jumper from the receiver's **OUT** pin to Arduino **pin 11**.

**Wiring the LED:** standard — pin 8 → 220Ω → LED → GND.

## Install the IR library

We need a library called **IRremote** by **Rafi Khan / shirriff** (the most popular one).

1. **Tools → Manage Libraries...**
2. Search for `IRremote`.
3. Install the one by **shirriff, z3t0, ArminJo** (it's usually the top result).

The newer versions of this library changed the API quite a bit. The code below uses the modern (v3+) style.

## Part 1: Discover your remote's codes

Every remote button sends a different code. Before you can *use* a button, you need to know its code. Upload this sketch to **discover** the codes:

```cpp
#include <IRremote.hpp>

const int IR_PIN = 11;

void setup() {
  Serial.begin(9600);
  IrReceiver.begin(IR_PIN);
}

void loop() {
  if (IrReceiver.decode()) {
    Serial.print("Code: 0x");
    Serial.println(IrReceiver.decodedIRData.command, HEX);
    IrReceiver.resume();
  }
}
```

Upload. Open the Serial Monitor. Now point a remote at the receiver and press buttons, one at a time. For each button, you'll see something like:

```
Code: 0x40
Code: 0x45
Code: 0x46
Code: 0x44
```

Write down which code corresponds to which button. For example:

- Power button → `0x45`
- Volume up → `0x40`
- Volume down → `0x19`
- Channel up → `0x07`

Your numbers will be different for your remote. **That's totally normal.**

### What is `0x45`?

`0x` means **hexadecimal** — a number system programmers use because it's compact for raw data. You don't need to understand hex fully. Just know: `0x45` is a number, and we compare to it with `==` like any other number. Copy-paste it exactly.

## Part 2: Control the LED with the remote

Now use the codes you discovered to trigger actions:

```cpp
#include <IRremote.hpp>

const int IR_PIN  = 11;
const int LED_PIN = 8;

// === Replace these with YOUR remote's codes ===
const int POWER_CODE    = 0x45;
const int VOL_UP_CODE   = 0x40;
const int VOL_DOWN_CODE = 0x19;

void setup() {
  pinMode(LED_PIN, OUTPUT);
  Serial.begin(9600);
  IrReceiver.begin(IR_PIN);
}

void loop() {
  if (IrReceiver.decode()) {
    int code = IrReceiver.decodedIRData.command;

    if (code == VOL_UP_CODE) {
      digitalWrite(LED_PIN, HIGH);
      Serial.println("LED ON");
    } else if (code == VOL_DOWN_CODE) {
      digitalWrite(LED_PIN, LOW);
      Serial.println("LED OFF");
    } else if (code == POWER_CODE) {
      // Toggle
      digitalWrite(LED_PIN, !digitalRead(LED_PIN));
      Serial.println("TOGGLE");
    }

    IrReceiver.resume();   // ready for the next press
  }
}
```

Upload. Point your remote at the receiver. Press the buttons you mapped:

- Volume up → LED turns on
- Volume down → LED turns off
- Power → LED toggles

You're now controlling your Arduino **wirelessly**.

## What's new in this code?

### `#include <IRremote.hpp>`

New library — the IRremote one. Note the `.hpp` extension (not `.h`). That's the modern API.

### `const int`

```cpp
const int IR_PIN = 11;
const int POWER_CODE = 0x45;
```

**`const`** means **"this variable cannot change."** It's like `int`, but once you set it, nobody can modify it later in the program. Using `const` for fixed values (like pin numbers and key codes) is a good habit — if you accidentally try to write `IR_PIN = 5` later, the compiler will stop you with an error, which saves debugging time.

### `IrReceiver.begin(IR_PIN)`

Just like `Serial.begin(9600)` and `myServo.attach(9)` — initialize the IR receiver object on pin 11. Do it once in setup.

### `IrReceiver.decode()`

Returns `true` when a new IR code has arrived and been decoded. If no new signal, returns `false`. That's why we wrap everything in `if (IrReceiver.decode()) { ... }` — do nothing until there's a real button press.

### `IrReceiver.decodedIRData.command`

This is the actual button code, as a number. The dots are how you drill into an object's fields — `IrReceiver` has a member called `decodedIRData`, which has a member called `command`. Don't worry about the dots right now — just copy the line.

### `IrReceiver.resume()`

After handling one button press, tell the library "OK, I'm done, start listening again." Without this, you'd only receive one press forever.

### `!digitalRead(LED_PIN)` — toggle with NOT

```cpp
digitalWrite(LED_PIN, !digitalRead(LED_PIN));
```

Clever trick. Read the current state of the LED, flip it with `!` (NOT, from Day 10), then write it back. If the LED was HIGH, it becomes LOW. If LOW, becomes HIGH. One-line toggle, no extra variable needed.

## Why `0x` hex codes instead of decimal?

Good question. IR codes are often shown in hex because that's the native format the IR protocol uses internally. You could convert them — `0x45` equals `69` in decimal — but hex makes the codes easier to group mentally and easier to look up in datasheets.

You can freely mix hex and decimal in Arduino code. `if (code == 0x45)` and `if (code == 69)` do exactly the same thing. We stick with hex because that's what the Serial Monitor printed.

## Button repeat / holding a button

If you hold a button down on most remotes, the receiver gets a special "repeat" code instead of the original code. The modern IRremote library handles this automatically, but you may see the `protocol` field say "UNKNOWN" when a repeat comes in. If you want to act on holds differently from taps, check `IrReceiver.decodedIRData.flags & IRDATA_FLAGS_IS_REPEAT`. That's a deeper topic for later.

## Try this

1. **Control 3 LEDs.** Map three remote buttons to three LEDs on pins 8, 9, 10. Press '1' to toggle red, '2' for yellow, '3' for green.
2. **Speed control.** Map volume-up to speed up a blink, volume-down to slow it down. Use a variable `blinkDelay` that starts at 500 and goes up/down by 50 each press.
3. **Remote + servo.** Hook up the Day 12 servo. Use the left-arrow and right-arrow buttons to sweep the servo. Each press moves it by 10°.
4. **Print the button name.** Use a big `if/else if` chain or `switch/case` to print a human name for each code you recognize: `"Power"`, `"Vol Up"`, `"Ch Down"`, etc.

## What you learned today

- What **IR** (infrared) is, and how TV remotes work
- How to wire a **VS1838B IR receiver** (OUT, GND, VCC)
- Installing the **IRremote** library and including `IRremote.hpp`
- **`IrReceiver.begin(pin)`** / **`IrReceiver.decode()`** / **`IrReceiver.resume()`**
- Reading **hex codes** from a remote and matching them with `==`
- **`const int`** — make a variable that can't be accidentally changed
- One-line toggle with `!digitalRead(pin)`

## What is next

[Day 20](/arduino/day-20-sound-sensor/) — your Arduino learns to hear. We'll wire up a **sound sensor** and make it react to claps. Combined with today's IR, you'll have two totally different ways to trigger an action — remote or voice.

Great work, Anish. Your Arduino is now wireless.
