---
title: "Day 3: Button Input — Making the Arduino Listen"
date: 2026-04-11T23:57:00+05:30
tags: ["arduino", "beginner", "kids", "button", "input", "digitalread", "if-else"]
categories: ["arduino"]
summary: "Day 3 of Learn Coding with Arduino — connect a push button, read it with digitalRead, and use your first if/else to turn on an LED only when pressed."
lastmod: 2026-04-11T23:57:00+05:30
---

Hi Anish! So far the Arduino has only been **talking** (sending power out to LEDs). Today, it starts **listening**. We are going to connect a button, and the Arduino will light an LED **only when you press it**. That's your first real **input**, and your first piece of code that makes a **decision**.

## What you need today

- Arduino Uno + USB cable
- Breadboard
- 1 **push button** (4-legged tactile switch from your kit)
- 1 **LED** (any color)
- 1 **220Ω resistor** (for the LED — red-red-brown stripes)
- 1 **10kΩ resistor** (for the button — brown-black-orange stripes) — this is the **pull-down resistor**
- 5 **jumper wires**

## First, a quick detour: what is voltage?

Before we wire a button, it helps to have a picture of what "voltage" even means. Here are two ways to think about it.

### Water pipe analogy

Voltage is like **water pressure in a pipe**. Imagine two tanks of water:

- One is high up on a hill — lots of pressure, water shoots out fast
- One is on the ground — low pressure, water trickles

The hill-tank pushes water harder. A battery with more volts pushes electricity harder, just like the hill-tank pushes water harder. A 9V battery pushes harder than a 5V one. A dead battery (0 volts) has no push at all — nothing flows.

### Electric train track analogy

Voltage is like **how much power a toy train gets from its track**:

- 5V = train runs at a nice speed
- 9V = train zips like crazy
- 0V = train doesn't move

More volts = more push. Simple.

So when the Arduino says `5V`, it means "a gentle, steady push of electricity — enough to light an LED or drive a tiny sensor."

## How Arduino reads a button

A button has only two states: **pressed** or **not pressed**. The Arduino reads a pin by measuring its voltage:

- **HIGH** = about 5 volts = "there's a push"
- **LOW** = about 0 volts = "there's nothing"

So we wire the button so that when it is **pressed**, the Arduino pin sees 5V (`HIGH`), and when it is **not pressed**, the pin sees 0V (`LOW`).

The tricky part: what happens when the button is **not pressed**? If the wire is just floating in mid-air, the pin doesn't see 5V *or* 0V — it sees random garbage, because the wire picks up electrical noise like an antenna. The Arduino would read `HIGH` sometimes and `LOW` sometimes, totally at random. Chaos.

The fix: a **pull-down resistor**. It "pulls" the pin gently down to 0V whenever nothing else is happening. When the button is pressed, the pin gets pushed up to 5V by a direct wire (much stronger than the weak pull-down), so the pin reads `HIGH`. When the button is released, the pull-down quietly drags the pin back down to `LOW`. No more chaos.

Today we use a **10kΩ resistor** as our pull-down. "Pull-down" just tells you its job in the circuit.

## The circuit

We will wire **two things** on the same breadboard:

1. An **LED** on **pin 8** (same as Day 2)
2. A **button** on **pin 7**, with a 10kΩ pull-down resistor

{{< mermaid >}}
graph LR
    subgraph LED_side["LED part"]
        PIN8["Pin 8"] --> R220["220Ω"]
        R220 --> LED_POS["LED long leg"]
        LED_POS --> LED_NEG["LED short leg"]
        LED_NEG --> GND1["GND"]
    end
    subgraph BTN_side["Button part"]
        V5["5V"] --> BTN_A["Button side A"]
        BTN_A -.->|connects when pressed| BTN_B["Button side B"]
        BTN_B --> PIN7["Pin 7"]
        BTN_B --> R10K["10kΩ pull-down"]
        R10K --> GND2["GND"]
    end
{{< /mermaid >}}

**Wiring steps — LED first (same as Day 2):**

1. Place the LED on the breadboard (long leg and short leg in different columns).
2. Resistor (220Ω) from pin 8 jumper to the long leg of the LED.
3. Jumper from the short leg of the LED to GND.

**Wiring steps — Button:**

4. Place the button on the breadboard so that it straddles the middle gap. The 4 legs should end up in 4 different columns (2 on top of the gap, 2 below).
5. Jumper from **5V** on the Arduino to one side of the button.
6. Jumper from the **other side** of the button to **pin 7** on the Arduino.
7. From that same "other side" column, also plug in the **10kΩ resistor**, with its other end going to **GND**.

The third wire is the trick: the same column of the breadboard now connects to three things at once — the button, pin 7, and the 10kΩ resistor. That is the whole point of a breadboard — one column = one electrical "node".

**Check:** When the button is NOT pressed, pin 7 is connected only to GND through the 10kΩ resistor → pin 7 sees 0V = `LOW`. When the button IS pressed, pin 7 is connected directly to 5V → pin 7 sees 5V = `HIGH`. 

## The code

```cpp
void setup() {
  pinMode(8, OUTPUT);  // LED output (from Day 2)
  pinMode(7, INPUT);   // Button input — NEW! We are listening, not talking
}

void loop() {
  if (digitalRead(7) == HIGH) {
    // Button is pressed
    digitalWrite(8, HIGH);   // LED ON
  } else {
    // Button is not pressed
    digitalWrite(8, LOW);    // LED OFF
  }
}
```

Click **Upload**. Now press the button. The LED lights up *only while you are holding it down*. Let go — it turns off. Press again — on. Release — off.

## What is new in today's code?

### `pinMode(7, INPUT)`

On Day 1 and Day 2, every pin we used was an `OUTPUT` — we sent power *out*. Today, pin 7 is an `INPUT` — we are listening, not talking. That one word tells Arduino: "don't send power out of pin 7, just measure what's on it."

### `digitalRead(7)`

`digitalRead` is the mirror of `digitalWrite`. Where `digitalWrite` *puts* HIGH or LOW on a pin, `digitalRead` *checks* if the pin is currently HIGH or LOW. It gives you back one of those two words.

```cpp
digitalRead(7)   // → HIGH  (if button pressed)
digitalRead(7)   // → LOW   (if button not pressed)
```

### `if` and `else` — making decisions

This is the big new idea of Day 3. Until now, our code has just run straight through, top to bottom, no choices. Today we make the Arduino **decide**.

```cpp
if (digitalRead(7) == HIGH) {
  // do this
} else {
  // do that instead
}
```

In English: *"IF the button is being pressed, turn the LED on. OTHERWISE, turn it off."*

Break it down:

- **`if (...)`** — "IF this thing in the parentheses is true..."
- **`==`** — this is how you ask "are these two things equal?" Note: **two equal signs**. One equal sign (`=`) means "copy this value into this variable" (we'll see that on Day 4). Two equal signs (`==`) mean "compare these — are they the same?" If you ever mix them up, the code will not do what you expect.
- **`HIGH`** — we are comparing the result of `digitalRead(7)` to the word `HIGH`.
- **`{ ... }`** — the curly braces hold the instructions that run *if the condition is true*.
- **`else { ... }`** — a second set of braces with instructions that run *if the condition is false*.

Only one of the two blocks runs each time the loop goes around. Never both.

```cpp
if (condition) {
  // runs ONLY if condition is true
} else {
  // runs ONLY if condition is false
}
```

This is the most important pattern in programming. Pretty much every app, game, and website in the world is built on top of `if` statements making decisions millions of times a second.

## What is `INPUT` vs `OUTPUT`, really?

| Mode | What the pin does |
|---|---|
| `OUTPUT` | The Arduino **pushes** power out of the pin (for LEDs, motors, buzzers) |
| `INPUT` | The Arduino **listens** to the pin (for buttons, sensors) |

You pick the mode once, in `setup()`, with `pinMode`. You can't listen and talk on the same pin at the same time.

## Try this

1. **Flip the logic.** Change `== HIGH` to `== LOW`. Now the LED is on when the button is NOT pressed. Weird, but educational.
2. **Pick a different pin** for the button — say pin 6. You need to: move the jumper wire from pin 7 to pin 6, and change `pinMode(7, INPUT)` to `pinMode(6, INPUT)` and `digitalRead(7)` to `digitalRead(6)`. Three edits, one move.
3. **Two LEDs, one button.** Add a second LED on pin 9 (resistor + wire, just like Day 2). Make the button **switch** which LED is on: when pressed, LED 1 off, LED 2 on; when not pressed, LED 1 on, LED 2 off.

## What you learned today

- What voltage is (a push), and why HIGH = 5V, LOW = 0V
- How a push button works (just two pieces of metal that touch when you press)
- Why a **pull-down resistor** is needed (to keep the pin from floating)
- `pinMode(pin, INPUT)` — tell Arduino to *listen* instead of talk
- `digitalRead(pin)` — check whether a pin is HIGH or LOW
- **`if` / `else`** — the most important decision-making tool in all of programming
- **`==`** (two equal signs) — "are these equal?"

## What is next

[Day 4](/arduino/day-4-variables-and-loops/) — we introduce **variables** (little labeled boxes that hold numbers) and **`for` loops** (tell the Arduino to do something N times without typing the same line N times). It is the trick that makes your code smaller and smarter.

See you tomorrow, Anish.
