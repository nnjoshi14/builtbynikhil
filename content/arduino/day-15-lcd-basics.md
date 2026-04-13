---
title: "Day 15: LCD Screen — Hello Anish!"
date: 2026-04-11T23:45:00+05:30
lastmod: 2026-04-11T23:45:00+05:30
tags: ["arduino", "beginner", "kids", "lcd", "i2c", "display", "library"]
categories: ["arduino"]
summary: "Day 15 of Learn Coding with Arduino — hook up a 16×2 LCD via I2C and display your first text. Arduino gets a screen!"
---

Hi Anish! Until now, Arduino has only been able to talk back to you through the Serial Monitor on your laptop. Today, Arduino gets its own **screen** — a 16×2 LCD display. That means **16 columns** and **2 rows** of text. Not big, but enough to show sensor readings, a menu, or a friendly greeting. And it doesn't need a laptop — it works standalone.

## What you need today

- Arduino Uno + USB cable
- Breadboard
- **1 × 16×2 LCD display with I2C module** on the back (the small board soldered to the back of the LCD — has 4 pins: GND, VCC, SDA, SCL)
- 4 **female-to-male jumper wires**

## Why the I2C module?

A bare 16×2 LCD (without the I2C add-on) needs **16 wires** to connect. It's a mess. The **I2C module** (the small green or blue board soldered to the back of the LCD) shrinks that down to just **4 wires** — power, ground, and two signal wires. It uses a protocol called **I2C** (pronounced "eye-squared-C" or "eye-two-C") to do all the communication.

Don't worry about *how* I2C works under the hood. The library handles it. You just need to wire 4 pins correctly.

## The circuit

{{< mermaid >}}
graph LR
    V5["5V"] --> VCC["LCD VCC"]
    GND["GND"] --> LG["LCD GND"]
    A4["Arduino A4"] --> SDA["LCD SDA"]
    A5["Arduino A5"] --> SCL["LCD SCL"]
{{< /mermaid >}}

**Wiring:**

1. LCD **GND** → Arduino **GND**
2. LCD **VCC** → Arduino **5V**
3. LCD **SDA** → Arduino **A4**
4. LCD **SCL** → Arduino **A5**

The **SDA** and **SCL** pins are fixed on the Arduino Uno — they are physically pins **A4** and **A5**. Even though they look like analog pins, when we use I2C they become data lines. (The Uno board may also have labeled "SDA" and "SCL" pins above the digital area — those are actually the same pins as A4/A5, just broken out to different spots on the board.)

## Install the library

We need a library called **LiquidCrystal_I2C**. It doesn't come with the IDE by default — you have to install it.

1. Open the Arduino IDE.
2. Click **Tools → Manage Libraries...**
3. In the search box, type `LiquidCrystal_I2C`.
4. Find the one by **Frank de Brabander** (most common) or Marco Schwartz. Click **Install**.

Once it's installed, you can use it in any sketch by `#include`-ing it at the top.

## First code: Hello Anish!

```cpp
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

// The 0x27 is the "address" of the LCD on the I2C bus.
// Most modules use 0x27 or 0x3F. If yours doesn't work, try 0x3F.
LiquidCrystal_I2C lcd(0x27, 16, 2);

void setup() {
  lcd.init();             // start the LCD
  lcd.backlight();        // turn on the backlight
  lcd.print("Hello Anish!");
}

void loop() {
  // Nothing — the text stays on the screen
}
```

Upload. The LCD backlight should come on (blue, usually), and "Hello Anish!" should appear on the top row.

**If you see blocks or nothing:** Try changing `0x27` to `0x3F` in the line `LiquidCrystal_I2C lcd(0x27, 16, 2);` and upload again. That's the second most common address.

**If you see very faint text or nothing at all:** There's a tiny blue trimmer (a little screw) on the back of the I2C module. That's the **contrast adjustment**. Turn it slowly with a small screwdriver until the text is clear.

## What's new in this code?

### `#include <Wire.h>` and `#include <LiquidCrystal_I2C.h>`

Two libraries this time:

- **`Wire.h`** — the built-in Arduino library that handles I2C communication under the hood. The LCD library uses it, so we include it too.
- **`LiquidCrystal_I2C.h`** — the library we just installed. Gives us the `LiquidCrystal_I2C` object type.

Remember Day 12's `#include <Servo.h>`? Same idea — pull in a library so you can use its commands.

### `LiquidCrystal_I2C lcd(0x27, 16, 2);`

This creates an **LCD object** called `lcd`. The three arguments configure it:

- **`0x27`** — the I2C address of this specific LCD module. Every I2C device has a unique address. 0x27 is the most common for these little LCDs.
- **`16`** — number of columns (character spaces per row).
- **`2`** — number of rows.

If you had a 20×4 LCD, you'd write `LiquidCrystal_I2C lcd(0x27, 20, 4);`. Same library, different size.

The `0x` prefix means "hexadecimal" — a number system programmers use for low-level stuff. `0x27` equals 39 in regular decimal. You don't need to understand hex right now. Just copy the number.

### `lcd.init();`

Initializes the LCD — wakes it up, clears it, tells it to start fresh. Call it once in `setup()`.

### `lcd.backlight();`

Turns on the LCD's backlight so you can actually see the text. There's also `lcd.noBacklight();` if you want to turn it off (useful for a dark mode or to save power).

### `lcd.print("Hello Anish!");`

Prints text at the current cursor position. Right after `init()`, the cursor is at column 0, row 0 (top-left). The text flows from left to right.

Notice how this looks exactly like `Serial.print(...)`? Same pattern — print text to a "thing." The only difference is *which* thing we're talking to (`Serial` for the laptop, `lcd` for the screen).

## Position the cursor

By default, `lcd.print` starts from wherever the cursor currently is. You can move the cursor with `lcd.setCursor(col, row)`:

```cpp
void setup() {
  lcd.init();
  lcd.backlight();
  lcd.setCursor(0, 0);
  lcd.print("Hello Anish!");
  lcd.setCursor(0, 1);
  lcd.print("Day 15 rocks");
}
```

Note: columns and rows **start at 0**, not 1. Top-left is `(0, 0)`, bottom-right on a 16×2 is `(15, 1)`. Same zero-indexing habit as for loops and arrays.

Upload. Two lines of text, one on each row.

## Blinking text

Let's make the text flash on and off.

```cpp
void setup() {
  lcd.init();
  lcd.backlight();
}

void loop() {
  lcd.setCursor(0, 0);
  lcd.print("Hello Anish!");
  delay(500);

  lcd.clear();         // wipe the screen
  delay(500);
}
```

Upload. The text shows for half a second, disappears, shows again, disappears. The new function is `lcd.clear()` — wipes the whole screen back to blank.

## Common LCD commands

Here are the main things you'll use with the `lcd` object:

| Command | What it does |
|---|---|
| `lcd.init()` | Wake up the LCD (call in setup) |
| `lcd.backlight()` | Turn backlight on |
| `lcd.noBacklight()` | Turn backlight off |
| `lcd.clear()` | Erase everything |
| `lcd.setCursor(col, row)` | Move the cursor |
| `lcd.print("...")` | Print text at cursor |
| `lcd.print(42)` | Print a number |

Just like the Serial library, you can `print` either text (in quotes) or a number (no quotes) or a variable.

## Try this

1. **Print your name** on the top row and "loves Arduino" on the bottom row.
2. **Scrolling text.** Use `lcd.scrollDisplayLeft();` inside `loop()` with a delay to slide text across the screen. (Look it up in the library docs if you want a challenge.)
3. **Count up.** Create an `int counter = 0;` global, increment it in the loop, and print it on row 2 with a label: `"Count: 42"`. Don't forget `lcd.clear()` or `setCursor()` to overwrite the old number.
4. **Show Day 9's light sensor value** on the LCD instead of in the Serial Monitor. Wire the LDR like before, read it, and print the value. (This is Day 16's project, but you can start now.)

## Don't call `lcd.clear()` in a fast loop

One warning: `lcd.clear()` is slow — it takes a few milliseconds. If you call it on every single loop iteration, your display will **flicker badly**. A better approach: only call `clear()` when something actually changes, or use `lcd.setCursor()` to *overwrite* the old text in place. We'll do the overwriting trick tomorrow.

## What you learned today

- What a **16×2 LCD** is and what the numbers mean
- Why the **I2C module** simplifies wiring (16 pins → 4 pins)
- How to **install a library** (Tools → Manage Libraries)
- **`#include <Wire.h>`** and **`#include <LiquidCrystal_I2C.h>`**
- Creating an LCD object: `LiquidCrystal_I2C lcd(0x27, 16, 2);`
- **`lcd.init()`**, **`lcd.backlight()`**, **`lcd.clear()`**
- **`lcd.setCursor(col, row)`** — position the cursor (0-indexed)
- **`lcd.print(...)`** — print text or numbers
- Why `lcd.clear()` in a fast loop causes flicker

## What is next

[Day 16](/arduino/day-16-lcd-sensor/) — show **real-time sensor data** on the LCD. We'll read the potentiometer and the LDR and display their values live, without flicker. Arduino gets its own dashboard.

Great work, Anish. Now the Arduino has a face.
