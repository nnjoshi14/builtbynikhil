---
title: "Day 5: Three LEDs — Dance, Chase, and Random"
date: 2026-04-11T23:55:00+05:30
lastmod: 2026-04-11T23:55:00+05:30
tags: ["arduino", "beginner", "kids", "led", "for-loop", "arrays", "random"]
categories: ["arduino"]
summary: "Day 5 of Learn Coding with Arduino — wire up 3 LEDs and control them as a group. Chaser patterns, random blinks, and the trick of looping over pin numbers."
---

Hi Anish! Yesterday we met for-loops. Today we take that idea and use it on **three LEDs at once**. By the end of this post, you'll have a mini light show: LEDs that dance in sequence, chase each other, and blink at random.

## What you need today

- Arduino Uno + USB cable
- Breadboard
- **3 LEDs** (ideally different colors — red, yellow, green is perfect for Day 6 tomorrow)
- **3 × 220Ω resistors**
- 5 **jumper wires**

## The circuit

We're going to wire three LEDs to pins **8, 9, and 10**. Each LED needs its own 220Ω resistor.

{{< mermaid >}}
graph LR
    PIN8["Pin 8"] --> R1["220Ω"] --> L1["LED 1 (Red)"] --> G1["GND"]
    PIN9["Pin 9"] --> R2["220Ω"] --> L2["LED 2 (Yellow)"] --> G2["GND"]
    PIN10["Pin 10"] --> R3["220Ω"] --> L3["LED 3 (Green)"] --> G3["GND"]
{{< /mermaid >}}

**Tip:** you don't need three separate GND jumper wires. Run **one** jumper from the Arduino's GND pin to the breadboard's `−` rail, then plug all three LED short legs into that `−` rail. Same effect, fewer wires.

## Program 1: LED dance (the long way)

Let's start with the obvious version — three `digitalWrite` calls in a row:

```cpp
void setup() {
  pinMode(8, OUTPUT);
  pinMode(9, OUTPUT);
  pinMode(10, OUTPUT);
}

void loop() {
  digitalWrite(8, HIGH);
  delay(300);
  digitalWrite(8, LOW);

  digitalWrite(9, HIGH);
  delay(300);
  digitalWrite(9, LOW);

  digitalWrite(10, HIGH);
  delay(300);
  digitalWrite(10, LOW);
}
```

Upload. The LEDs light up one at a time: 8, 9, 10, 8, 9, 10... A dance.

Look at the code. Notice anything? Every block of three lines is **almost identical** — only the pin number changes. Whenever you find yourself copying-and-pasting code and tweaking one number, that's a clue you can use a **loop** instead.

## Program 2: LED dance (the smart way)

Watch what happens when we loop over the pin numbers:

```cpp
void setup() {
  for (int i = 8; i <= 10; i++) {
    pinMode(i, OUTPUT);
  }
}

void loop() {
  for (int i = 8; i <= 10; i++) {
    digitalWrite(i, HIGH);
    delay(300);
    digitalWrite(i, LOW);
  }
}
```

Upload. Same dance — but look at the code. Instead of three `pinMode` calls we have one for-loop with `i` going 8, 9, 10. Instead of nine lines in `loop()` we have four.

### Wait — a for loop from 8 to 10?

On Day 4 our for loops counted from 0. Here we count from **8 to 10**. Is that allowed?

Yes! The three parts of a for loop are just numbers you pick:

```cpp
for (int i = 8; i <= 10; i++)
```

- **`int i = 8`** — start the counter at 8
- **`i <= 10`** — keep going while `i` is **less than or equal to 10**
- **`i++`** — add 1 each time

So `i` takes the values `8, 9, 10`. And when we write `digitalWrite(i, HIGH)`, Arduino substitutes in whatever `i` is right now — so we get `digitalWrite(8, HIGH)`, then `digitalWrite(9, HIGH)`, then `digitalWrite(10, HIGH)`. **The variable IS the pin number.**

Notice I also used `<=` instead of `<`. `<=` means "less than or equal to" — so the loop includes 10. If I had written `i < 10`, the loop would have stopped at 9 and we'd only dance with two LEDs.

This is the trick that lets you scale up. Want **10 LEDs** on pins 2 through 11? Just change the numbers:

```cpp
for (int i = 2; i <= 11; i++) { ... }
```

Same code, ten LEDs. Magic.

## Program 3: Random blink

Sometimes you don't want a pattern — you want chaos. Arduino has a `random()` function that gives you a random number.

```cpp
void setup() {
  randomSeed(analogRead(A0));   // shake the dice bag before rolling
  for (int i = 8; i <= 10; i++) {
    pinMode(i, OUTPUT);
  }
}

void loop() {
  int led = random(8, 11);     // pick a random number: 8, 9, or 10
  digitalWrite(led, HIGH);
  delay(200);
  digitalWrite(led, LOW);
}
```

Upload. The LEDs blink in random order — no pattern, just chaos. Every time it's a coin toss which one lights next.

Three new things here:

### `random(min, max)`

`random(8, 11)` gives you a random whole number that is **at least 8** and **less than 11**. So it returns one of: 8, 9, or 10. Yes, the second number is **not included** — that's how `random` works in Arduino. If you want "a number from 1 to 6 for a dice roll", you write `random(1, 7)`.

### `randomSeed(...)`

Computers aren't really random — they follow a secret recipe that *looks* random. If you don't shake that recipe up, every time you reset the Arduino, it will produce the **exact same sequence** of "random" numbers. Boring.

`randomSeed(...)` mixes up the recipe. We pass it whatever is on analog pin `A0` — which is typically a floating pin picking up random electrical noise from the air. That noise is different every time the Arduino starts, so the random sequence is different every time. Good enough for our light show.

### `int led = random(8, 11);`

This is a **variable assignment** using a function's result. `random(8, 11)` gives you back a number — we grab that number and store it in a new variable called `led`. Then we use `led` on the next line.

## Bonus: Chaser pattern

A **chaser** is when LEDs light up one after another in a wave — think airport runway lights. You can do it with the same for-loop idea, but you light each LED briefly and move on:

```cpp
void setup() {
  for (int i = 8; i <= 10; i++) {
    pinMode(i, OUTPUT);
  }
}

void loop() {
  for (int i = 8; i <= 10; i++) {
    digitalWrite(i, HIGH);
    delay(100);
    digitalWrite(i, LOW);
  }
}
```

Shorter delay (`100` instead of `300`) makes it look like a light travelling across the row.

## Try this

1. **Reverse direction.** Make the chaser go from pin 10 → 9 → 8 instead. Hint: `for (int i = 10; i >= 8; i--)`. The `--` is the opposite of `++` — it subtracts 1 each loop.
2. **Bounce.** Go forward, then backward: 8 → 9 → 10 → 9 → 8 → 9 → 10... Like Knight Rider. You'll need two for loops back to back.
3. **Speed up as it goes.** Use `i` in the delay: `delay(50 * (i - 7))`. Each LED lights longer than the previous.
4. **All on, all off.** Turn all 3 LEDs on for 300ms, then all off for 300ms. Use a for loop inside the loop to do the `digitalWrite` calls.

## What you learned today

- How to wire **3 LEDs** on different pins
- How to **loop over pin numbers** with a for loop (a for loop's counter can be anything — a pin number, an index, whatever)
- `<=` means "less than or equal to"
- `random(min, max)` — get a random whole number
- `randomSeed(...)` — shake the dice bag so you don't get the same sequence every time
- You can store a function's result in a variable: `int x = random(1, 10);`
- `i--` decreases the counter instead of increasing it

## What is next

[Day 6](/arduino/day-6-traffic-light/) — we take those same three LEDs (red, yellow, green) and build a real **traffic light system** with proper timing. And for bonus points, we add a pedestrian button that says "I want to cross!" You'll use everything from Days 1-5.

See you tomorrow, Anish.
