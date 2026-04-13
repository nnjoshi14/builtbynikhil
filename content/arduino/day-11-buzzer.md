---
title: "Day 11: Buzzer — Make Noise and a Simple Melody"
date: 2026-04-11T23:49:00+05:30
lastmod: 2026-04-11T23:49:00+05:30
tags: ["arduino", "beginner", "kids", "buzzer", "tone", "sound", "melody"]
categories: ["arduino"]
summary: "Day 11 of Learn Coding with Arduino — connect a piezo buzzer and use tone() to play beeps, notes, and a short melody."
---

Hi Anish! Today your Arduino starts making **sound**. We'll connect a **piezo buzzer** — a tiny speaker that can play tones — and learn how to play single beeps, long beeps, and even a short melody. By the end, your board can *sing*.

## What you need today

- Arduino Uno + USB cable
- Breadboard
- **1 piezo buzzer** (passive buzzer — looks like a small black cylinder with a hole)
- 2 **jumper wires**

## Passive vs active buzzers

There are two kinds of buzzers in most kits:

- **Active buzzer** — has a built-in circuit. Apply power and it just beeps at one fixed sound. Useful but boring.
- **Passive buzzer** — you send it different frequencies and it plays different notes. This is what we want. More flexible.

If your kit has both, the passive one usually has a hole on top (where sound comes out) and the active one is sealed. When in doubt, try both — whichever one plays different notes with the code below is the passive one.

Passive buzzers have a **+ side** (usually marked with `+` on top) and a **−** side. The + leg is positive, − is GND.

## The circuit

{{< mermaid >}}
graph LR
    PIN8["Pin 8"] --> BZp["Buzzer +"]
    BZp --> BZn["Buzzer −"]
    BZn --> GND["GND"]
{{< /mermaid >}}

**Wiring:**

1. Jumper from **pin 8** to the **+ leg** of the buzzer.
2. Jumper from the **− leg** of the buzzer to **GND**.

That's it. No resistor needed for a buzzer (though some people add one for safety — not required).

## Part 1: Simple beep

```cpp
void setup() {
  pinMode(8, OUTPUT);
}

void loop() {
  tone(8, 1000);    // Play a 1000 Hz tone on pin 8
  delay(500);
  noTone(8);        // Silence
  delay(500);
}
```

Upload. You should hear: **beep** (half a second), silence (half a second), **beep**, silence, forever.

### What is `tone()`?

`tone(pin, frequency)` tells Arduino to send a **square wave** to the pin at the given frequency (in Hz, which is "cycles per second"). The buzzer's membrane vibrates at that frequency and you hear a pitch. Higher numbers = higher pitch.

- `tone(8, 200)` — very low, almost a rumble
- `tone(8, 1000)` — middle, a clear tone
- `tone(8, 4000)` — high squeak

### What is `noTone()`?

`noTone(pin)` stops the sound. That's all. You need it because `tone()` by itself keeps playing forever until you tell it to stop.

You can also call `tone(pin, frequency, duration)` with **three arguments** — Arduino plays the note for that many milliseconds and then stops on its own. We'll use that style in the melody below.

## Part 2: Rising pitch siren

```cpp
void setup() {
  pinMode(8, OUTPUT);
}

void loop() {
  // Sweep from low to high
  for (int freq = 200; freq <= 2000; freq += 10) {
    tone(8, freq);
    delay(5);
  }
  noTone(8);
  delay(500);
}
```

Upload. You hear a rising siren — 200 Hz up to 2000 Hz in tiny steps. That's a `for` loop where the counter `freq` goes up by **10 each time** (`freq += 10` is shorthand for `freq = freq + 10`). Day 4's for-loop idea, applied to sound.

## Part 3: Play a short melody

Let's play the first few notes of Twinkle Twinkle Little Star. Each note is a specific frequency.

| Note | Frequency (Hz) |
|---|---|
| C (middle C) | 262 |
| D | 294 |
| E | 330 |
| F | 349 |
| G | 392 |
| A | 440 |
| B | 494 |
| C (high) | 523 |

Twinkle Twinkle goes: **C C G G A A G** ("Twinkle twinkle little star")

```cpp
void setup() {
  pinMode(8, OUTPUT);
}

void loop() {
  tone(8, 262, 400);  // C
  delay(450);         // wait a bit longer than the note
  tone(8, 262, 400);  // C
  delay(450);
  tone(8, 392, 400);  // G
  delay(450);
  tone(8, 392, 400);  // G
  delay(450);
  tone(8, 440, 400);  // A
  delay(450);
  tone(8, 440, 400);  // A
  delay(450);
  tone(8, 392, 800);  // G (longer — it's a held note)
  delay(850);

  delay(2000);        // big pause before repeating
}
```

Upload. You should hear the opening line of Twinkle Twinkle. Not a symphony, but recognizable.

### Why `tone(8, 262, 400)` with 3 arguments?

The third number (`400`) tells `tone()` how long to play the note — 400 milliseconds in this case. After 400ms the sound stops automatically. No `noTone()` needed.

### Why `delay(450)` instead of `delay(400)`?

Because if you `delay(400)` right after a 400ms note, the next note starts the *instant* the first one ends — notes blend into each other with no gap. The extra 50ms gives a tiny pause so you can hear individual notes clearly.

## Try this

1. **Faster melody.** Change all the `400`s to `200`s and all the `450`s to `250`s. Twinkle Twinkle, espresso edition.
2. **New melody.** Mary Had a Little Lamb: **E D C D E E E** (all the same length). Look up the frequencies above and write the `tone` calls.
3. **Use variables.** Make `int noteLength = 400;` at the top and use it in every tone call. Changing one number changes the whole tempo. (This is Day 4's variable idea, applied to music.)
4. **Two-tone siren.** Alternate between 500 Hz and 1000 Hz every 200ms. Sounds like an ambulance.

## Storing notes in arrays

If you do Challenge 2 above, you'll notice it's tedious to write 7 `tone()` calls. Here's a fancier way using an **array** — a variable that holds a list of numbers.

```cpp
int melody[] = {262, 262, 392, 392, 440, 440, 392};
int noteCount = 7;

void setup() {
  pinMode(8, OUTPUT);
}

void loop() {
  for (int i = 0; i < noteCount; i++) {
    tone(8, melody[i], 400);
    delay(450);
  }
  delay(2000);
}
```

- **`int melody[] = {262, 262, 392, ...};`** — this creates a variable called `melody` that holds a **list** of numbers. The `[]` after the name means "this is an array."
- **`melody[0]`** is the first number (262), **`melody[1]`** is the second (262), **`melody[2]`** is the third (392), etc. Arrays are indexed from 0, just like for-loop counters.
- Inside the for loop, `melody[i]` grabs the `i`th note and plays it.

Arrays are the right way to handle "lots of things of the same kind." You'll use them constantly from here on.

## What you learned today

- What a **piezo buzzer** is (active vs passive)
- **`tone(pin, frequency)`** — play a pitch on a pin
- **`tone(pin, frequency, duration)`** — play for a set time
- **`noTone(pin)`** — stop playing
- **Frequency = pitch.** Higher Hz = higher note.
- **Musical notes as Hz values** (C = 262, G = 392, A = 440, etc.)
- **`+=`** — shorthand for "add to variable" (`freq += 10` is `freq = freq + 10`)
- **Arrays** — `int melody[] = {...};` — a variable that holds a list

## What is next

[Day 12](/arduino/day-12-servo-motor/) — your first **motor**. A servo motor can rotate to any angle you pick. We'll sweep it back and forth like a wagging tail, and then use the potentiometer to control it with a knob.

Great work, Anish. Now your Arduino can beep, sing, and tell you stories.
