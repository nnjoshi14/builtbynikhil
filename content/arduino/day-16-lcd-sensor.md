---
title: "Day 16: Show Sensor Values on the LCD"
date: 2026-04-11T23:44:00+05:30
lastmod: 2026-04-11T23:44:00+05:30
tags: ["arduino", "beginner", "kids", "lcd", "sensor", "ldr", "potentiometer"]
categories: ["arduino"]
summary: "Day 16 of Learn Coding with Arduino — display live potentiometer and LDR readings on the 16x2 LCD without flicker. Arduino gets a dashboard."
---

Hi Anish! Yesterday your LCD said "Hello Anish!" Today it becomes a **live dashboard** — we're going to show real-time sensor values right on the screen. No laptop needed.

## What you need today

- Arduino Uno + USB cable
- Breadboard
- **LCD with I2C module** (Day 15 wiring)
- **1 potentiometer** (Day 8 wiring)
- **1 LDR + 10kΩ resistor** (Day 9 wiring)
- Jumper wires

You'll have a lot going on the breadboard today. Take a deep breath, wire it slowly, test one thing at a time.

## The circuit

Three subsystems.

{{< mermaid >}}
graph TB
    subgraph LCD["LCD (I2C)"]
      V5a["5V"] --> LV["LCD VCC"]
      Ga["GND"] --> LG["LCD GND"]
      A4["A4"] --> SDA["LCD SDA"]
      A5["A5"] --> SCL["LCD SCL"]
    end
    subgraph POT["Potentiometer"]
      V5b["5V"] --> P1["Pot left"]
      P2["Pot middle"] --> A0["A0"]
      P3["Pot right"] --> Gb["GND"]
    end
    subgraph LDR["LDR (voltage divider)"]
      V5c["5V"] --> L1["LDR"]
      L1 --> JUNC["junction"]
      JUNC --> A1["A1"]
      JUNC --> R10k["10kΩ"] --> Gc["GND"]
    end
{{< /mermaid >}}

**Pin assignments:**

- **A4, A5** — LCD (I2C, fixed)
- **A0** — potentiometer middle leg
- **A1** — LDR voltage divider

This is the first time we've used **two analog inputs at once** (A0 and A1). Arduino has 6 of them (A0-A5, minus A4/A5 which we're using for the LCD), so we can have up to 4 sensors running simultaneously.

## The code

```cpp
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

LiquidCrystal_I2C lcd(0x27, 16, 2);

void setup() {
  lcd.init();
  lcd.backlight();

  // Print the labels ONCE — they never change
  lcd.setCursor(0, 0);
  lcd.print("Pot:");
  lcd.setCursor(0, 1);
  lcd.print("Light:");
}

void loop() {
  int pot = analogRead(A0);
  int light = analogRead(A1);

  // Update just the number portions — labels stay
  lcd.setCursor(6, 0);
  lcd.print("    ");        // clear old value
  lcd.setCursor(6, 0);
  lcd.print(pot);

  lcd.setCursor(6, 1);
  lcd.print("    ");
  lcd.setCursor(6, 1);
  lcd.print(light);

  delay(200);
}
```

Upload. The LCD shows:

```
Pot:   512
Light: 640
```

Turn the pot — the top number changes live. Cover the LDR — the bottom number drops. No flicker, no clearing the whole screen.

## The no-flicker trick

Yesterday I warned you that calling `lcd.clear()` every loop makes the screen flicker. Here's the clever solution:

### 1. Print the **labels** once, in `setup()`

```cpp
void setup() {
  ...
  lcd.setCursor(0, 0);
  lcd.print("Pot:");        // label, doesn't change
  lcd.setCursor(0, 1);
  lcd.print("Light:");      // label, doesn't change
}
```

Because labels never change, we write them once in `setup()` and never touch them again. The LCD remembers them.

### 2. Only overwrite the **numbers**, in `loop()`

```cpp
lcd.setCursor(6, 0);      // jump to column 6 (just past "Pot: ")
lcd.print("    ");        // 4 spaces — wipes any leftover digits
lcd.setCursor(6, 0);      // jump back
lcd.print(pot);           // new value
```

Three moves:

1. Move cursor to where the number should go.
2. Print 4 spaces to wipe the old value (4 spaces covers a number from 0 to 9999 — more than enough for 0-1023).
3. Move the cursor **back** to the same spot and print the new number.

Without the "print spaces to wipe" step, old digits would hang around. For example, if the pot used to read `1023` and drops to `5`, you'd see `5023` on the screen — the old `023` didn't get cleared.

This is the standard trick for flicker-free updating LCDs. Every sensor dashboard uses it.

## Why not just use `lcd.clear()` once per loop?

You could. It's just *obviously* wrong visually — you'd see the whole screen go blank for a split second on every update, 5 times per second. Looks like a broken TV.

The "overwrite only what changes" trick is smoother because the LCD keeps most of its content stable. Your brain doesn't notice updates happening if they're localized.

## Add a unit

Let's show percentages instead of raw 0-1023 values for the pot:

```cpp
int potPercent = map(pot, 0, 1023, 0, 100);

lcd.setCursor(6, 0);
lcd.print("    ");
lcd.setCursor(6, 0);
lcd.print(potPercent);
lcd.print("%");
```

Now the top row reads like `Pot: 42%`. Much more human.

Remember `map()` from Day 8? Same trick: rescale from 0-1023 to 0-100.

## Try this

1. **Three sensors.** Add a second LDR on A2 (same wiring pattern) and show three live values by using row 1 for two fields, split in half.
2. **Reading the time.** Arduino has a function `millis()` that returns milliseconds since the board started. Show `millis() / 1000` on row 2 so you see seconds running up. It's a live clock.
3. **Battery icon.** Show a simple text "bar" that grows with the pot value: `[####    ]` for 50%, `[########]` for 100%. Use a for loop to print the right number of `#` characters.
4. **Alert on threshold.** When `pot > 900`, print `"HIGH!"` on the right side of row 0. Otherwise, print 5 spaces to clear it.

## Tip: too many `setCursor` calls get tedious

You'll notice that updating even two fields takes 6 LCD calls. As your dashboards get bigger, that adds up. One trick is to **write a helper function**:

```cpp
void printAt(int col, int row, int value) {
  lcd.setCursor(col, row);
  lcd.print("    ");
  lcd.setCursor(col, row);
  lcd.print(value);
}
```

Now your `loop()` becomes much cleaner:

```cpp
void loop() {
  printAt(6, 0, analogRead(A0));
  printAt(6, 1, analogRead(A1));
  delay(200);
}
```

**Helper functions** are a big idea — we'll explore them more on Day 25. For now, recognize the pattern: *if you find yourself typing the same 3-4 lines of code over and over, you can group them into a named function and call that function instead.*

## What you learned today

- How to read **multiple analog sensors** at once (A0, A1, ...)
- The **no-flicker LCD update** pattern: labels in `setup()`, values in `loop()`, overwrite in place
- Why you shouldn't call `lcd.clear()` in a fast loop
- Printing **numbers** with `lcd.print(someVar)`
- Adding **units** with a second `lcd.print("%")` after the number
- Brief preview of **helper functions** for cleaner code

## What is next

[Day 17](/arduino/day-17-lcd-menu/) — a **scrollable menu** on the LCD with buttons. You'll press a button to cycle through options like "Fan ON", "Light AUTO", "Buzzer ON", and select one to trigger an action. Arduino gets a user interface.

Great work, Anish. Your screen is alive.
