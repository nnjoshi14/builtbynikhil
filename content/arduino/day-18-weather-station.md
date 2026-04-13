---
title: "Day 18: Project — Mini Weather Station"
date: 2026-04-11T23:42:00+05:30
lastmod: 2026-04-11T23:42:00+05:30
tags: ["arduino", "beginner", "kids", "project", "lcd", "weather", "custom-chars"]
categories: ["arduino"]
summary: "Day 18 of Learn Coding with Arduino — build a mini weather station that shows light level, simulated temperature, and custom sun and moon icons on the LCD."
---

Hi Anish! Finale of the display week. We're going to build a **Mini Weather Station** that shows:

- **Row 1:** Current temperature (simulated) and the light level
- **Row 2:** A word describing the weather ("Sunny", "Dark", "Cloudy") and a custom **sun icon** drawn pixel-by-pixel

Plus: you'll learn how to **draw your own characters** on the LCD — your first taste of pixel art in code.

## What you need today

- Arduino Uno + USB cable
- Breadboard
- **LCD with I2C module**
- **LDR + 10kΩ resistor** (Day 9 wiring)
- Jumper wires
- (Optional) an LM35 temperature sensor — if you don't have one, we'll fake it with the potentiometer

If you don't have an LM35, use a **potentiometer** as a fake temperature knob — turn the knob to pretend the temperature changes. Pedagogically it's just as good.

## The circuit

{{< mermaid >}}
graph TB
    subgraph LCD["LCD"]
      V5a["5V"] --> LV["VCC"]
      Ga["GND"] --> LG["GND"]
      A4a["A4"] --> SDA["SDA"]
      A5a["A5"] --> SCL["SCL"]
    end
    subgraph LIGHT["LDR (Day 9)"]
      V5b["5V"] --> LDR["LDR"] --> JUNC["junction"]
      JUNC --> A0["A0"]
      JUNC --> R10k["10kΩ"] --> Gb["GND"]
    end
    subgraph TEMP["Temp (pot stand-in)"]
      V5c["5V"] --> P1["Pot"] 
      P2["Pot middle"] --> A1["A1"]
      P3["Pot"] --> Gc["GND"]
    end
{{< /mermaid >}}

Same setup as Day 16 (LCD + LDR + pot), just relabeled.

## The code

```cpp
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

LiquidCrystal_I2C lcd(0x27, 16, 2);

// Custom sun icon — 5 columns wide, 8 rows tall, each row is a binary pattern
byte sunIcon[8] = {
  0b00100,
  0b10101,
  0b01110,
  0b11111,
  0b01110,
  0b10101,
  0b00100,
  0b00000
};

// Custom moon icon
byte moonIcon[8] = {
  0b00000,
  0b01110,
  0b11100,
  0b11000,
  0b11100,
  0b01110,
  0b00000,
  0b00000
};

void setup() {
  lcd.init();
  lcd.backlight();

  // Register the custom characters into slots 0 and 1
  lcd.createChar(0, sunIcon);
  lcd.createChar(1, moonIcon);

  // Labels (print once, they never change)
  lcd.setCursor(0, 0);
  lcd.print("T:");
  lcd.setCursor(8, 0);
  lcd.print("L:");
}

void loop() {
  int lightRaw = analogRead(A0);
  int potRaw   = analogRead(A1);

  // Simulate temperature from the pot: map to 0-45 degrees
  int temperature = map(potRaw, 0, 1023, 0, 45);

  // Temperature value on row 0
  lcd.setCursor(2, 0);
  lcd.print("   ");
  lcd.setCursor(2, 0);
  lcd.print(temperature);
  lcd.print("C");

  // Light value on row 0
  lcd.setCursor(10, 0);
  lcd.print("    ");
  lcd.setCursor(10, 0);
  lcd.print(lightRaw);

  // Row 1: weather word + icon
  lcd.setCursor(0, 1);
  if (lightRaw > 700) {
    lcd.print("Sunny    ");
    lcd.setCursor(15, 1);
    lcd.write(byte(0));       // sun icon
  } else if (lightRaw > 300) {
    lcd.print("Cloudy   ");
    lcd.setCursor(15, 1);
    lcd.print(" ");
  } else {
    lcd.print("Dark     ");
    lcd.setCursor(15, 1);
    lcd.write(byte(1));       // moon icon
  }

  delay(200);
}
```

Upload. The screen shows something like:

```
T:23C   L:640
Cloudy         ☀
```

Turn the pot — "T" changes. Cover the LDR — the bottom row changes to "Dark" and a moon appears. Shine a flashlight — "Sunny" and a sun icon.

## How custom characters work

The LCD screen is made of little 5×8 pixel boxes (5 wide, 8 tall). By default, the character set is preprogrammed — A, B, C, numbers, symbols. But the LCD has **8 user-defined slots** (numbered 0 to 7) where you can define your own character by specifying which pixels should be on.

### Drawing a sun

```cpp
byte sunIcon[8] = {
  0b00100,   // row 0 — just the middle pixel (top of the sun)
  0b10101,   // row 1 — left, middle, right (rays)
  0b01110,   // row 2 — middle three (top of ball)
  0b11111,   // row 3 — all five (wide middle)
  0b01110,   // row 4
  0b10101,   // row 5
  0b00100,   // row 6
  0b00000    // row 7 — bottom blank
};
```

Each row is 8 rows of a 5-column wide character. The `0b` prefix means **binary** — you're writing the bits directly. Each `1` is a pixel that's on, each `0` is off. Since the character is 5 wide, we only use the last 5 bits of each byte (the leading `0b00`... is just padding).

If I draw it out:

```
..#..    ← 00100
#.#.#    ← 10101
.###.    ← 01110
#####    ← 11111
.###.    ← 01110
#.#.#    ← 10101
..#..    ← 00100
.....    ← 00000
```

There's your sun with rays.

### `lcd.createChar(0, sunIcon);`

Registers the pixel pattern into the LCD's slot **0**. Do this once in `setup()`.

### `lcd.write(byte(0));`

Displays the character in slot 0 at the current cursor position. The `byte(0)` cast is necessary because `lcd.write(0)` would be ambiguous — `write(0)` could mean "print a zero" OR "print slot 0". Writing `byte(0)` makes it clear we mean slot 0.

You have **8 slots total** (0-7), so you can have up to 8 custom characters at once.

## Designing your own icon

Grab graph paper or just write out a 5×8 grid on paper. Mark the pixels you want on with `1`s, off with `0`s. Then read each row as a binary number and write it in `0b...` form. That's the whole recipe.

You can also use online tools — search for "LCD custom character generator" and you'll find drag-and-drop editors that spit out the C code for you.

## Small padding trick

Notice I padded the weather word with trailing spaces: `"Sunny    "`, `"Cloudy   "`, `"Dark     "`. Why?

Because without padding, if the current word is "Cloudy" (6 characters) and the next update changes it to "Dark" (4 characters), the old letters "dy" would stick around on the screen. By padding each word to the same width, we overwrite the leftover characters with spaces.

It's the same "overwrite old content" idea from Day 16, applied to strings instead of numbers.

## Tuning the thresholds

The thresholds `700` and `300` are for bright/normal/dark. Measure your room with the Serial Monitor (or print the raw value on the LCD for a moment) and adjust these numbers so they match real conditions.

## Try this

1. **Real temperature.** Replace the pot with an LM35 temperature sensor and calculate real temperature: `float tempC = analogRead(A1) * 0.48828125;` — prints the actual room temperature in Celsius.
2. **More icons.** Design a custom cloud icon, a raindrop, a thermometer. Put them in slots 2, 3, 4. Show the thermometer icon next to the temperature.
3. **Celsius / Fahrenheit toggle.** Add a button that switches between units. `F = C * 9/5 + 32`.
4. **Temperature alert.** If `temperature > 35`, show `"HOT!"` on row 1 with a custom warning icon.

## What you learned today

- How to design a **custom character** by specifying a pixel grid in binary
- **`byte iconName[8] = {...};`** — an 8-row pixel pattern
- **`0b...`** prefix for binary numbers
- **`lcd.createChar(slot, pattern)`** — register a custom char (in setup)
- **`lcd.write(byte(slot))`** — display a custom char
- Why you need **trailing spaces** when overwriting strings of different lengths
- You have **8 custom character slots** available

## What is next

[Day 19](/arduino/day-19-ir-remote/) — we start a new chapter: **wireless control**. You'll hook up an **IR receiver** and use a TV remote (yes, any TV remote) to control your Arduino. Press the volume-up button and your Arduino reacts. Magic.

Great work, Anish. Your board can now tell you about the world in its own words.
