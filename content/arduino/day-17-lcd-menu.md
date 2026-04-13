---
title: "Day 17: Build an LCD Menu You Can Navigate"
date: 2026-04-11T23:43:00+05:30
lastmod: 2026-04-11T23:43:00+05:30
tags: ["arduino", "beginner", "kids", "lcd", "menu", "ui", "arrays", "strings"]
categories: ["arduino"]
summary: "Day 17 of Learn Coding with Arduino — build a scrollable menu on the 16x2 LCD. Two buttons: one to cycle through options, one to select. Arduino gets a real user interface."
---

Hi Anish! Today your Arduino gets a **real user interface**. We're going to build a menu on the LCD — a list of options you can scroll through with a button, and another button to pick one. Press one button to move down the list. Press the other to activate the current option. This is how a microwave, a vending machine, and most kitchen gadgets work.

## What you need today

- Arduino Uno + USB cable
- Breadboard
- **LCD with I2C module**
- **2 push buttons** (we'll use `INPUT_PULLUP`)
- **1 LED** (the "action" light) + **220Ω resistor**
- Jumper wires

## The circuit

{{< mermaid >}}
graph TB
    subgraph LCD["LCD"]
      V5a["5V"] --> LV["VCC"]
      Ga["GND"] --> LG["GND"]
      A4["A4"] --> SDA["SDA"]
      A5["A5"] --> SCL["SCL"]
    end
    subgraph BTNS["Buttons (INPUT_PULLUP)"]
      PIN2["Pin 2 (Next)"] --> BT1a["Button 1"]
      BT1a -.-> BT1b["Button 1"] --> Gb["GND"]
      PIN3["Pin 3 (Select)"] --> BT2a["Button 2"]
      BT2a -.-> BT2b["Button 2"] --> Gc["GND"]
    end
    subgraph LED["LED (action indicator)"]
      PIN8["Pin 8"] --> R220["220Ω"] --> LEDp["LED +"] --> LEDn["LED −"] --> Gd["GND"]
    end
{{< /mermaid >}}

**Pin assignments:**

- **A4, A5** — LCD (I2C)
- **Pin 2** — "Next" button (cycles to the next menu option)
- **Pin 3** — "Select" button (activates the current option)
- **Pin 8** — indicator LED

## The code

```cpp
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

LiquidCrystal_I2C lcd(0x27, 16, 2);

// The menu options
String menu[] = {"LED ON", "LED OFF", "Blink"};
int menuSize  = 3;
int currentOption = 0;

// Button state tracking for "press detection"
int lastNext   = HIGH;
int lastSelect = HIGH;

void setup() {
  lcd.init();
  lcd.backlight();
  pinMode(2, INPUT_PULLUP);
  pinMode(3, INPUT_PULLUP);
  pinMode(8, OUTPUT);

  showMenu();
}

void loop() {
  // === NEXT button ===
  int next = digitalRead(2);
  if (lastNext == HIGH && next == LOW) {
    currentOption = (currentOption + 1) % menuSize;
    showMenu();
    delay(50);
  }
  lastNext = next;

  // === SELECT button ===
  int sel = digitalRead(3);
  if (lastSelect == HIGH && sel == LOW) {
    runOption(currentOption);
    delay(50);
  }
  lastSelect = sel;
}

void showMenu() {
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Select:");
  lcd.setCursor(0, 1);
  lcd.print(menu[currentOption]);
}

void runOption(int option) {
  if (option == 0) {
    digitalWrite(8, HIGH);            // LED ON
  } else if (option == 1) {
    digitalWrite(8, LOW);             // LED OFF
  } else if (option == 2) {
    // Blink 3 times
    for (int i = 0; i < 3; i++) {
      digitalWrite(8, HIGH);
      delay(200);
      digitalWrite(8, LOW);
      delay(200);
    }
  }
}
```

Upload. You should see:

```
Select:
LED ON
```

Press **Next** (pin 2 button). The second line changes to:

```
Select:
LED OFF
```

Press again:

```
Select:
Blink
```

Press again — it wraps around back to "LED ON". That's the menu.

Press **Select** (pin 3 button) when the display shows the option you want:

- On "LED ON" → LED turns on
- On "LED OFF" → LED turns off
- On "Blink" → LED blinks 3 times

## What's new in this code?

Lots. This is the most complex sketch so far. Take it slowly.

### `String` — variable that holds text

```cpp
String menu[] = {"LED ON", "LED OFF", "Blink"};
```

Up to now, variables have held numbers: `int`, `bool`. `String` is a variable type that holds **text** — a sequence of characters. The text goes in **double quotes**.

Like `int` and `bool`, a `String` is just a box — but instead of holding `5` or `true`, it holds `"Blink"`. You can `lcd.print(someString)` just like you'd print a number.

The `[]` after `menu` makes it an **array of strings** — a list of 3 pieces of text. `menu[0]` is `"LED ON"`, `menu[1]` is `"LED OFF"`, `menu[2]` is `"Blink"`. Same zero-indexing as always.

### `(currentOption + 1) % menuSize`

```cpp
currentOption = (currentOption + 1) % menuSize;
```

The `%` operator is called **modulo** (or "remainder"). It gives you the remainder after division. For example:

- `7 % 3` = 1 (because 7 ÷ 3 = 2 remainder 1)
- `10 % 5` = 0
- `4 % 3` = 1

Here we're using `% menuSize` (where `menuSize = 3`) to **wrap around** to 0 when we go past the end. Watch:

- `(0 + 1) % 3` = `1 % 3` = **1**
- `(1 + 1) % 3` = `2 % 3` = **2**
- `(2 + 1) % 3` = `3 % 3` = **0** ← wraps!
- `(0 + 1) % 3` = **1**
- ...

So `currentOption` cycles through `0, 1, 2, 0, 1, 2, ...` forever. `%` is the clean way to wrap around. You'll use it a lot.

### Two helper functions — `showMenu()` and `runOption()`

```cpp
void showMenu() {
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Select:");
  lcd.setCursor(0, 1);
  lcd.print(menu[currentOption]);
}

void runOption(int option) {
  if (option == 0) { ... }
  else if (option == 1) { ... }
  else if (option == 2) { ... }
}
```

Look at this — we defined our own functions! Not just `setup()` and `loop()`, but **custom** functions with names we picked. This is a huge step.

Why do it? Because otherwise, `loop()` would become a giant mess of LCD code mixed with button-reading code mixed with action code. Pulling each "job" into its own function makes the code much easier to read.

The rules of writing a function:

```cpp
returnType functionName(arguments) {
  // the code that runs when this function is called
}
```

- **`void showMenu()`** — "a function called `showMenu` that returns nothing and takes no arguments."
- **`void runOption(int option)`** — "a function called `runOption` that takes one integer argument called `option`."

Once a function is defined, you **call** it just like you call built-in ones:

```cpp
showMenu();           // no arguments
runOption(currentOption);  // one argument
```

Arduino jumps into the function, runs its code, and comes back. Clean.

### Where to put your functions

Your custom functions go **outside** of `setup()` and `loop()`, usually below them. Arduino compiles everything in the file and links it up. Order doesn't matter as long as everything is at the top level of the file (not inside another function).

### `else if` — chain multiple conditions

```cpp
if (option == 0) {
  ...
} else if (option == 1) {
  ...
} else if (option == 2) {
  ...
}
```

Remember `else if` from Day 9? It lets you check multiple conditions in order. Perfect for "which menu option was picked?" Each branch handles one case.

For 3 options, `else if` works great. For 10 options, there's a cleaner tool called `switch/case` which we'll meet later. For now, `else if` is fine.

## Try this

1. **Add a 4th option** called "Fade" that smoothly fades the LED from 0 to full brightness using PWM (`analogWrite`, Day 8). Remember to move the LED to a `~` pin like 9.
2. **Show a `>` cursor** on row 1 before the current option name: `> LED ON`. Just prepend `"> "` in `showMenu`.
3. **Two-line menu** — show **two options at once** on row 0 and row 1, with a `>` next to the current one. Hint: you'll need a `for` loop inside `showMenu()` to print each line.
4. **Go backwards.** Add a third button on pin 4 called "Previous" that decreases `currentOption`. When you go below 0, wrap to the last option. Hint: use `(currentOption + menuSize - 1) % menuSize`.

## What you learned today

- **`String`** — a variable that holds text
- **Array of strings** — `String menu[] = {"A", "B", "C"};`
- **`%`** modulo operator — get the remainder; use it to wrap around lists
- How to write your own **custom functions** (not just `setup`/`loop`)
- **Function signature**: `returnType name(arguments) { body }`
- How to **call** your custom functions from `loop()` or elsewhere
- **`else if`** chains for handling multiple cases

## What is next

[Day 18](/arduino/day-18-weather-station/) — the finale of the LCD week. We build a **mini Weather Station**: LCD, LDR, and a temperature sensor (or simulated temperature), with custom weather icons drawn pixel-by-pixel on the screen. Nearly a real gadget.

Great work, Anish. You just built your first user interface.
