---
title: "Day 20: Sound Sensor — Clap to Control"
date: 2026-04-11T23:40:00+05:30
lastmod: 2026-04-11T23:40:00+05:30
tags: ["arduino", "beginner", "kids", "sound", "sensor", "ky-038", "clap"]
categories: ["arduino"]
summary: "Day 20 of Learn Coding with Arduino — wire up a KY-038 sound sensor, detect claps, and toggle an LED with a sound."
---

Hi Anish! Today, Arduino learns to **hear**. We'll hook up a tiny microphone module (KY-038 or similar) and make the Arduino react to **claps**. Two claps → LED toggles. You get to build a real clap-switch.

## What you need today

- Arduino Uno + USB cable
- Breadboard
- **1 × KY-038 sound sensor module** (or any 4-pin sound sensor with a digital and analog output)
- **1 LED** + **220Ω resistor**
- Jumper wires

## What's on the KY-038 module?

The KY-038 is a small circuit board with a tiny **electret microphone**, a **sensitivity potentiometer** (the blue screw), and a built-in comparator chip. It has **4 pins**:

- **A0** — analog output (voltage changes with sound level — we can read it with `analogRead`)
- **D0** — digital output (goes HIGH when sound is above a threshold, LOW otherwise — we can read it with `digitalRead`)
- **GND** — ground
- **VCC** — 5V

The digital output is easier to use — it's basically a "clap detected: yes or no" signal. The threshold is set by turning the blue screw on the module, by hand. We'll use the digital output today.

## The circuit

{{< mermaid >}}
graph LR
    V5["5V"] --> VCC["Sound VCC"]
    GND1["GND"] --> SGND["Sound GND"]
    D0["Sound D0"] --> PIN2["Pin 2"]
    PIN8["Pin 8"] --> R220["220Ω"] --> LEDp["LED +"] --> LEDn["LED −"] --> GND2["GND"]
{{< /mermaid >}}

**Wiring the sound sensor:**

1. **VCC** → 5V
2. **GND** → GND
3. **D0** → pin 2 (digital input)

We're ignoring the A0 pin today — just the digital is enough.

**LED:** pin 8 → 220Ω → LED → GND as always.

## Tune the sensitivity

The sensitivity knob (the little blue screw on the module) is critical. If it's too sensitive, the sensor triggers on every whisper. If it's too deaf, you'll have to scream.

To find the right setting: upload the code below, open the Serial Monitor, and turn the screw with a tiny screwdriver while clapping. Watch the output. Turn the screw until **talking normally does nothing** but **clapping triggers** reliably.

## Part 1: Simple clap detection

```cpp
void setup() {
  pinMode(2, INPUT);
  pinMode(8, OUTPUT);
  Serial.begin(9600);
}

void loop() {
  int sound = digitalRead(2);

  if (sound == HIGH) {
    // Clap detected!
    digitalWrite(8, !digitalRead(8));  // toggle LED
    Serial.println("Clap!");
    delay(200);                         // ignore further sound for 200ms
  }
}
```

Upload. Clap near the microphone. The LED should toggle. Clap again — toggles back.

### Why `delay(200)`?

When you clap, the sound sensor fires **multiple times** in a row (the echo of the clap can last 100-150ms). Without a delay, the LED would flicker rapidly as the sensor fires maybe 10 times in one clap. The 200ms delay makes the code **ignore new claps for 200ms** after detecting one — just long enough for the echo to pass.

This is the simplest form of **debouncing** — same idea as the button debounce on Day 7, but for sound. A more elaborate version uses `millis()` to avoid blocking the rest of the code, which we'll cover on Day 25.

## Part 2: Two-clap detection

Toggling on every clap is fun but fragile — random noises might trigger it. A smarter pattern is **"two claps in a row"** — only toggle if you hear **two** claps within 1 second.

This needs a little state tracking.

```cpp
int lastClapTime = 0;       // when was the last clap?
int clapCount    = 0;       // how many recent claps?

void setup() {
  pinMode(2, INPUT);
  pinMode(8, OUTPUT);
  Serial.begin(9600);
}

void loop() {
  int sound = digitalRead(2);

  if (sound == HIGH) {
    int now = millis();

    // If previous clap was less than 1 second ago, count them together
    if (now - lastClapTime < 1000) {
      clapCount++;
    } else {
      clapCount = 1;   // reset — this is a new burst
    }
    lastClapTime = now;

    if (clapCount == 2) {
      // Two claps detected!
      digitalWrite(8, !digitalRead(8));
      Serial.println("Double clap — toggle!");
      clapCount = 0;  // reset so we don't count three-in-a-row
    }

    delay(150);  // small debounce
  }
}
```

Upload. Single claps do nothing. **Two claps within 1 second** toggle the LED. Three claps in a second count as one toggle plus one leftover. Try it out.

### What's `millis()`?

`millis()` is a built-in Arduino function that returns the number of **milliseconds since the board started**. It keeps counting forever — or at least for 49 days, after which it wraps back to 0.

We use it to **measure time between events** without `delay()` blocking everything:

```cpp
int now = millis();
if (now - lastClapTime < 1000) { ... }
```

This asks: *"How long has it been since the last clap? Less than 1 second?"* If yes, count it as a double-clap. If no (a whole second passed with silence), reset the count.

`millis()` is one of the most important functions in all of Arduino. You're going to use it a lot from now on. **It's the right way to handle time when you need to do multiple things at once.** We'll use it heavily on Day 25.

## Part 3: Claps with LCD status

Let's add a display update for fun. If you still have the LCD wired, show "Claps: N" that resets every second of silence. (This part is optional.)

```cpp
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

LiquidCrystal_I2C lcd(0x27, 16, 2);

int lastClapTime = 0;
int clapCount    = 0;

void setup() {
  pinMode(2, INPUT);
  pinMode(8, OUTPUT);
  lcd.init();
  lcd.backlight();
  lcd.setCursor(0, 0);
  lcd.print("Claps: 0");
}

void loop() {
  int sound = digitalRead(2);
  int now = millis();

  if (sound == HIGH) {
    if (now - lastClapTime < 1000) {
      clapCount++;
    } else {
      clapCount = 1;
    }
    lastClapTime = now;

    lcd.setCursor(7, 0);
    lcd.print("   ");
    lcd.setCursor(7, 0);
    lcd.print(clapCount);

    delay(150);
  }

  // Reset count if no claps for 2 seconds
  if (clapCount > 0 && now - lastClapTime > 2000) {
    clapCount = 0;
    lcd.setCursor(7, 0);
    lcd.print("0  ");
  }
}
```

Upload. Clap and watch the number go up on the LCD. Stop clapping, and after 2 seconds it resets to 0.

This sketch runs the sound detection AND the LCD update AND a time-based reset **all at the same time**, without any long `delay()` stalling anything. That's the power of `millis()`.

## Try this

1. **Triple clap.** Require **three** claps in 1.5 seconds before doing anything. Good for filtering out random noise.
2. **Different actions per count.** 1 clap → LED on. 2 claps → LED off. 3 claps → blink 5 times. Use `if/else if`.
3. **Noise meter.** Use `analogRead(A0)` on the sensor's A0 pin to read the raw sound level, and show it as a bar on the LCD: more sound = longer bar. Fun visualization of clapping.
4. **Combine with the light sensor.** Only respond to claps when the room is dark (from the LDR). Clap in the dark → turn on a night light. Clap in daylight → nothing happens.

## What you learned today

- What an **electret microphone** / sound sensor module is and its 4 pins
- How to tune the **sensitivity knob** on a KY-038
- Reading the **digital output** of the sound sensor with `digitalRead`
- **Debouncing sound** with a short delay
- **`millis()`** — the built-in clock that lets you time events without blocking
- How to detect **double-clap** patterns by counting events within a time window

## What is next

[Day 21](/arduino/day-21-ir-and-sound/) — we combine today's sound sensor with yesterday's IR remote into **dual-mode control**. Manual mode uses the remote; auto mode triggers on claps. One button switches modes, and the LCD shows which mode you're in.

Great work, Anish. Your Arduino now hears you clap.
