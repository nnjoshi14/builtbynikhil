---
title: "Day 1: Intro to Arduino & Your First Code"
date: 2026-04-11T23:59:00+05:30
lastmod: 2026-04-11T23:59:00+05:30
tags: ["arduino", "beginner", "kids", "blink", "led"]
categories: ["arduino"]
summary: "Day 1 of Learn Coding with Arduino — meet your Uno, install the IDE, and make the little built-in LED blink."
---

Hi Anish! Welcome to **Day 1**. Imagine a tiny brain inside a robot. That brain can turn lights on, make sounds, move motors, and listen to buttons — but only if somebody writes instructions for it. That "somebody" is going to be **you**, starting today.

By the end of this post you will have made a real computer chip blink a light, just by typing a few lines of code. High-five yourself in advance. It is going to take about 20 minutes.

## What is an Arduino?

An **Arduino Uno** is a small blue board with a little computer on it. Think of it as a super simple brain you can program.

It can:

- Turn lights on and off
- Read buttons, sensors, temperature, light, sound
- Control motors, speakers, and servos
- Talk to other gadgets

You are going to teach it — one small step at a time.

## What is on the board?

Unbox your kit and look at the Uno. These are the important parts:

{{< mermaid >}}
graph TB
    subgraph Uno["Arduino Uno Board"]
        USB["USB Port<br/>(power + code come in here)"]
        POWER["Power Jack<br/>(for a battery, if you want)"]
        CHIP["The Brain<br/>(the big black chip)"]
        LED13["Built-in LED<br/>marked 'L' near pin 13"]
        DIGITAL["Digital Pins<br/>numbered 0 to 13"]
        ANALOG["Analog Pins<br/>A0 to A5"]
        GND["GND pins<br/>'ground', like minus on a battery"]
        V5["5V pin<br/>sends out power"]
    end
{{< /mermaid >}}

The one we care about today is the **built-in LED** — the tiny yellow-orange light near the letter **L** on the board. It is wired to **pin 13**. That is our target.

## What you need today

- Arduino Uno board
- USB cable (one end goes into the Uno, the other into your laptop)
- A laptop

No wires, no breadboard, no extra LEDs. Just the Uno and the cable.

## Install the Arduino IDE

The **IDE** (which stands for "Integrated Development Environment" — fancy name, but really it is just *the app where you write code*) is where we type our instructions and send them to the Uno.

1. Go to **arduino.cc/en/software**
2. Download **Arduino IDE 2** for your operating system (Windows, Mac, or Linux)
3. Install it like any normal app
4. Open it

When it opens, a window shows up with some empty code already filled in. That window is the editor — that's where we type.

## Plug in your Arduino

Connect the USB cable from the Uno to your laptop. You should see a tiny green light turn on near the letter **ON** — that means the board has power.

Now in the IDE:

1. Click **Tools → Board → Arduino Uno**
2. Click **Tools → Port** — pick the one that says "Arduino Uno"
   - Windows: it looks like `COM3`
   - Mac: it looks like `/dev/cu.usbmodem...`
   - Linux: it looks like `/dev/ttyUSB0` or `/dev/ttyACM0`

If the port is missing, unplug and replug the cable, or try a different USB port.

## Your first program: Blink

Delete anything that is already in the editor and type this in (or copy-paste it):

```cpp
void setup() {
  pinMode(13, OUTPUT);     // Tell the Uno pin 13 is for sending out power
}

void loop() {
  digitalWrite(13, HIGH);  // Turn LED ON
  delay(1000);             // Wait 1 second
  digitalWrite(13, LOW);   // Turn LED OFF
  delay(1000);             // Wait 1 second
}
```

That's it — 10 lines of code. Click the **→ (right arrow)** button in the top-left corner. That is the **Upload** button. The IDE compiles your code, sends it over USB, and a few seconds later says **"Done uploading"**.

Look at the Uno. The little **L** light is now blinking: on for 1 second, off for 1 second. Forever.

**You just programmed a computer.** Congratulations.

## Let's break the code down — every symbol, every word

Every tiny thing in that code has a job. Let's walk through it slowly so you actually understand what you typed.

### Comments: `//`

See the stuff after `//` on each line? Those are **comments**. They are notes for humans to read. The Uno ignores everything after `//`. You can write anything there — it's like sticky notes for your future self.

```cpp
// This is a comment. The Uno doesn't care what I write here.
digitalWrite(13, HIGH);   // But this part — before the // — is a real instruction.
```

### Semicolons: `;`

Every instruction in Arduino code ends with a **semicolon** (`;`). Think of it as the **period at the end of a sentence**. Without it, the Uno doesn't know where one instruction ends and the next begins — and it will refuse to run your program.

```cpp
digitalWrite(13, HIGH);  // ← semicolon = end of sentence
```

If you forget a semicolon, the IDE will yell at you with a red error at the bottom. That's OK — just add the semicolon and try again.

### Curly braces: `{` and `}`

Curly braces are like **the walls of a room**. Everything between `{` and `}` belongs to one group. In our code, the curly braces hold the instructions inside `setup()` and inside `loop()`.

```cpp
void setup() {        // ← door opens
  pinMode(13, OUTPUT);
}                     // ← door closes
```

Everything between `{` and `}` is "the stuff inside setup()". Simple.

### `void setup()` — runs ONE TIME

```cpp
void setup() {
  pinMode(13, OUTPUT);
}
```

`setup()` is a **function**. A function is just a named group of instructions — like a recipe with a name. Arduino gives you a special function called `setup` that it runs **exactly once**, the moment the Uno turns on. Think of it as "getting ready in the morning" — put on shoes, grab backpack, check lunch. You only do it once before leaving for school.

What about the word `void`? `void` means **"this function doesn't give anything back."** Some functions give back a number (like "what time is it?" gives back a time). `setup()` doesn't need to give anything back — it just *does* stuff. So we write `void` in front of it.

And the empty `()`? Those are the parentheses where you'd pass extra info into a function if it needed any. `setup()` doesn't need any, so they stay empty.

Inside setup, we wrote:

```cpp
pinMode(13, OUTPUT);
```

`pinMode` is another function — but this one is **built into Arduino**, so we just call it. We are telling the Uno: *"Hey, I am going to use pin 13. I want it as an **OUTPUT** — meaning I will send power OUT of this pin."* The two things inside the parentheses (`13` and `OUTPUT`) are called **arguments** — they are the extra info `pinMode` needs to know what to do.

If we wanted to *read* something from pin 13 instead (like a button), we would use `INPUT` instead of `OUTPUT`. We will do that on Day 3.

### `void loop()` — runs FOREVER

```cpp
void loop() {
  digitalWrite(13, HIGH);
  delay(1000);
  digitalWrite(13, LOW);
  delay(1000);
}
```

`loop()` is another special Arduino function. Unlike `setup()`, it runs **over and over, forever**, as long as the Uno has power. When it reaches the end, it goes back to the top and starts again. And again. And again. Millions of times.

Think of it like brushing your teeth — except instead of once a day, you brush your teeth a million times a second. That's what `loop()` does with the LED.

Inside `loop()` there are four instructions:

- **`digitalWrite(13, HIGH);`** — "Turn pin 13 ON." Because pin 13 is wired to the LED, this lights up the LED.
- **`delay(1000);`** — "Wait 1000 milliseconds." That's 1 second. (1 second = 1000 milliseconds. So `delay(500)` would be half a second, `delay(2000)` would be 2 seconds.)
- **`digitalWrite(13, LOW);`** — "Turn pin 13 OFF." The LED goes dark.
- **`delay(1000);`** — "Wait another 1 second."

Then the loop starts over from the top — LED on, wait, LED off, wait, LED on, wait, forever.

### HIGH and LOW

`HIGH` and `LOW` are special words Arduino understands:

- **`HIGH`** = "power is ON" = 5 volts coming out of the pin = LED lights up
- **`LOW`** = "power is OFF" = 0 volts = LED goes dark

That's all HIGH and LOW mean. ON and OFF. Don't overthink it.

## Try this

Change the code, click **Upload**, and see what happens:

1. **Challenge 1** — Change both `delay(1000)` values to `delay(200)`. The LED blinks super fast, like a strobe.
2. **Challenge 2** — Make it blink fast on, slow off: try `delay(100)` after the HIGH and `delay(900)` after the LOW. Looks like a heartbeat, doesn't it?
3. **Challenge 3** — Invent your own pattern: two fast blinks, one long pause. Copy the four lines and play.

Every time you change the code, click **Upload** again. The Uno forgets what it was doing and runs the new instructions.

## Quick reference — what you learned today

| Thing | What it means |
|---|---|
| Arduino Uno | Tiny brain you can program |
| `setup()` | Runs once when the Uno turns on |
| `loop()` | Runs forever, over and over |
| `pinMode(13, OUTPUT)` | "I will send power OUT of pin 13" |
| `digitalWrite(13, HIGH)` | Turn pin 13 ON |
| `digitalWrite(13, LOW)` | Turn pin 13 OFF |
| `delay(1000)` | Wait 1 second (1000 milliseconds) |
| `HIGH` / `LOW` | ON / OFF |
| `//` | A note for humans; the Uno ignores it |
| `;` | End of an instruction (like a period) |
| `{` and `}` | The walls that group instructions together |

## What is next

On [Day 2](/arduino/day-2-external-led-breadboard/), we stop using the built-in LED. You will wire up your **own** LED on a breadboard with a resistor, and learn why you should *never* plug an LED straight into the board.

Great work, Anish. See you on Day 2.
