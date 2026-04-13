---
title: "Day 4: Variables and Loops"
date: 2026-04-11T23:56:00+05:30
lastmod: 2026-04-11T23:56:00+05:30
tags: ["arduino", "beginner", "kids", "variables", "loops", "for-loop"]
categories: ["arduino"]
summary: "Day 4 of Learn Coding with Arduino — meet variables (labeled boxes that hold numbers) and for loops (repeat without retyping). Blink an LED 5 times in a row with just a few lines."
---

Hi Anish. So far your code has been straight lines: do this, then this, then this. Today you learn the two ideas that will let you write **less code that does more**:

1. **Variables** — labeled boxes where Arduino stores numbers you can reuse
2. **For loops** — a way to say "do this N times" without typing the same line N times

These two ideas are the foundation of every serious program in the world.

## What you need today

- Arduino Uno + USB cable
- Breadboard
- 1 **LED**
- 1 **220Ω resistor**
- 2 **jumper wires**

Same wiring as Day 2: LED on **pin 8** with a 220Ω resistor in series. You can leave the button from Day 3 plugged in if you want — we just won't use it today.

## The circuit

{{< mermaid >}}
graph LR
    PIN8["Pin 8"] --> R["220Ω"]
    R --> LEDp["LED long leg"]
    LEDp --> LEDn["LED short leg"]
    LEDn --> GND["GND"]
{{< /mermaid >}}

Same as Day 2. Nothing new to wire.

## What is a variable?

Here's a picture. Imagine your bedroom shelf has a label taped to it: **"homework pages"**, and sitting on that shelf is a sticky note that says **"3"**.

- The **label** is the name: *homework pages*
- The **sticky note** is the value: *3*

If you get more homework tomorrow, you can walk over, swap the "3" for a "5", and the label stays the same. Next time somebody asks "how many homework pages do I have?", they look at the same shelf and see the new value.

That is exactly what a variable is in code: **a named box that holds a number you can look up or change.**

In Arduino, you make a variable like this:

```cpp
int homeworkPages = 3;
```

Let's break every part of that down:

- **`int`** — short for "integer", which is just a fancy word for a whole number (1, 2, 3, 500, -7, etc.). It is telling Arduino "this box will only hold whole numbers." There are other kinds of boxes too (for decimal numbers, for words, for true/false), but `int` is the one we use most.
- **`homeworkPages`** — the name of the box. You pick it. It has to start with a letter, can have numbers in it, and can't have spaces. Convention: the first letter is lowercase and the next words start with uppercase, like `homeworkPages` or `ledBrightness`. This is called *camelCase* because it looks like a camel's humps.
- **`=`** — "put this into the box." Remember on Day 3 I said **one** equal sign means "copy," and **two** equal signs (`==`) mean "compare"? This is the one-equal-sign version. We are copying `3` into the box.
- **`3`** — the value we are putting in.
- **`;`** — semicolon, end of the sentence, just like always.

Now anywhere in our code, if we write `homeworkPages`, Arduino reads it as "3". And if we write `homeworkPages = 5;`, we've updated the box to hold 5 instead.

### Why bother? Why not just write `3`?

Two big reasons:

1. **It explains what the number means.** `homeworkPages` reads like English. Just writing `3` tells you nothing. Future-you (reading your own code next month) will thank present-you.
2. **Change one place, not ten.** If your code uses `3` in twelve different spots and tomorrow the number needs to become `5`, you have to find and change twelve things. If you used a variable, you change **one line** and everything updates.

Variables are lazy-but-smart. We love lazy-but-smart.

## Using a variable in real Arduino code

Let's use a variable to hold the **delay time** for our blink. That way, if we want to make the blink faster or slower, we change one line.

```cpp
int delayTime = 500;  // half a second

void setup() {
  pinMode(8, OUTPUT);
}

void loop() {
  digitalWrite(8, HIGH);
  delay(delayTime);        // ← using the variable, not a hard number
  digitalWrite(8, LOW);
  delay(delayTime);
}
```

Upload it. The LED blinks every half second. Now change the **first line** to `int delayTime = 100;` and upload again. Super fast. Change it to `int delayTime = 2000;` — slow blink. One number at the top of the file controls the whole program.

Notice we put `int delayTime = 500;` **outside** of `setup()` and `loop()`, at the very top of the file. That's called a **global variable** — it lives for the entire time the program is running, and both `setup()` and `loop()` can see it.

## What is a `for` loop?

Now imagine I told you: *"Blink the LED 5 times, then pause."* You could do it the long way:

```cpp
digitalWrite(8, HIGH); delay(200); digitalWrite(8, LOW); delay(200);
digitalWrite(8, HIGH); delay(200); digitalWrite(8, LOW); delay(200);
digitalWrite(8, HIGH); delay(200); digitalWrite(8, LOW); delay(200);
digitalWrite(8, HIGH); delay(200); digitalWrite(8, LOW); delay(200);
digitalWrite(8, HIGH); delay(200); digitalWrite(8, LOW); delay(200);
```

Ugh. Five copies of the same two lines. Now imagine I asked for 100 blinks. Your fingers would fall off.

A **`for` loop** says: "do this same block of code, N times." Like climbing a staircase with 5 steps — each step you do the same thing: lift your foot, put it down. Same action, 5 times.

Here is the for-loop version:

```cpp
for (int i = 0; i < 5; i++) {
  digitalWrite(8, HIGH);
  delay(200);
  digitalWrite(8, LOW);
  delay(200);
}
```

Two lines of real work, run 5 times. Let me decode that scary-looking first line:

```cpp
for (int i = 0; i < 5; i++)
```

The `for` has **three parts** inside the parentheses, separated by semicolons:

1. **`int i = 0;`** — "Start a counter called `i` at 0." This happens once, at the beginning. (Yes, `i` is a variable! It's just a very short name. Programmers use `i` for "index" — the counter that tracks how many times we've looped.)

2. **`i < 5;`** — "Keep looping **as long as** `i` is less than 5." This is the condition. Every time the loop is about to run, Arduino checks: is `i` still less than 5? If yes, go. If no, stop.

3. **`i++`** — "Add 1 to `i` **at the end of each loop**." `i++` is shorthand for `i = i + 1`. (Yes, really — the `++` adds one to whatever is in front of it.)

So the counter `i` goes through the values `0, 1, 2, 3, 4` — that's 5 values — and then stops when it hits 5.

### "Why does it start at 0, not 1?"

You will ask this. It's weird. In programming, counters almost always start at 0 instead of 1. It feels strange for a day and then feels normal forever. Just accept it for now. (If it really bugs you: you could write `for (int i = 1; i <= 5; i++)` — starting at 1, stopping at "less than or equal to 5". That also gives 5 loops. Both work. The `i = 0; i < 5` version is the standard.)

## Putting it all together

```cpp
int delayTime = 200;  // speed of each blink

void setup() {
  pinMode(8, OUTPUT);
}

void loop() {
  // Blink 5 times in a row
  for (int i = 0; i < 5; i++) {
    digitalWrite(8, HIGH);
    delay(delayTime);
    digitalWrite(8, LOW);
    delay(delayTime);
  }

  // Then take a big pause before repeating
  delay(2000);
}
```

Upload. The LED blinks 5 times fast, pauses for 2 seconds, then blinks 5 times again, forever. All with a `for` loop inside a `loop()`.

Read that code slowly and trace through it in your head:

- Program starts. `delayTime` is set to 200.
- `setup()` runs — pin 8 is an output.
- `loop()` starts.
  - `for` loop begins. `i = 0`. Is `i < 5`? Yes. Blink once.
  - End of first trip through the `for`. `i++` makes `i` become 1. Is `i < 5`? Yes. Blink again.
  - `i = 2`, blink. `i = 3`, blink. `i = 4`, blink.
  - `i++` makes `i` become 5. Is `i < 5`? **No.** Exit the for loop.
  - `delay(2000)` — wait 2 seconds.
- End of `loop()`. Start `loop()` again from the top. `i` resets to 0. Blink 5 more times.

## Try this

1. **Change `delayTime`** — try 50, 500, 1000. Observe how the blink speed changes.
2. **Make it blink 10 times** instead of 5. Change just one thing. (Hint: `i < 5` → `i < 10`.)
3. **Add a second variable** `int pauseTime = 2000;` at the top and use it in the `delay()` after the for loop. Now you have two knobs you can turn.
4. **Tricky one: use `i` inside the loop.** Try `delay(50 * (i + 1));` instead of `delay(delayTime);`. Each blink gets slower and slower because `i` grows from 0 to 4 and the delay multiplies. Upload and watch.

The last challenge is the first time you've used a variable that **changes** on its own. Up until now, `delayTime` has sat there at 500 forever. But `i` inside a for loop is a variable whose value keeps changing — 0, 1, 2, 3, 4. That idea of *"a variable that changes over time"* is the second foundation of all programming.

## What you learned today

- What a **variable** is — a labeled box holding a number
- `int` — a box that holds a whole number
- **camelCase** naming convention
- **Global** variables (declared at the top, outside functions, visible everywhere)
- One `=` means "copy", two `==` means "compare"
- `for` loops — repeat a block of code N times
- The three parts of a `for`: start counter, keep-going condition, advance counter
- `i++` is short for `i = i + 1`
- How to trace through a for loop in your head

## What is next

[Day 5](/arduino/day-5-led-sequences/) — we use a for loop to control **three LEDs at once**. Chaser patterns, random blinks, the beginnings of a light show. Bring three LEDs, three resistors, and a handful of jumper wires.

Great job, Anish. You just met the two biggest ideas in programming and lived to tell the tale.
