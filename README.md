![Calarts Creative Computing Logo](https://i.imgur.com/XZpdi2e.png)


# Seraph



**Seraph – Open-Source Teensy MIDI Controller Platform**  

An open-source platform for building USB-MIDI controllers and sensor-based interactive art projects. This repository provides a PCB design and sample demo code to help you build and customize your own MIDI devices.

---


**Getting Started Guide**

What You'll Learn

1 — Board Overview & Hardware Setup 

2 — First Wiring Tutorial: Button & LED 

3 — Analog Sensors: Potentiometers & FSRs 

4 — I²C Devices: IMUs, Displays & More 

5 — MIDI Mapping in Code 

6 — Connecting to a DAW

---

# 01 — Board Overview & Hardware Setup


Seraph is a breakout board for the Teensy 4.1 microcontroller. It handles all the messy wiring, power regulation, and I/O routing so you can focus on building your instrument or installation — not on debugging circuits.

## What's on the Board
![](https://i.imgur.com/RNhljU7.png)


![](https://i.imgur.com/WtuQLjl.png)

## First-Time Setup

### Step 1 — Install Arduino IDE & Teensyduino

Seraph uses the Arduino IDE with the Teensyduino add-on, which gives your computer the tools to compile code and talk to the Teensy 4.1.

1.  Download Arduino IDE from arduino.cc/en/software
    
2.  Download Teensyduino from pjrc.com/teensy/teensyduino.html
    
3.  Run the Teensyduino installer — it will find your Arduino IDE automatically
    
4.  Open Arduino IDE. You should now see Teensy boards in Tools > Board > Teensyduino
    

  

### Step 2 — Mount the Teensy on Seraph

⚠️ Always use header pins when mounting the Teensy. Soldering it directly means you cannot reuse it in other projects.

5.  Solder male header pins to the Teensy 4.1 (if not pre-soldered)
    
6.  Press the Teensy firmly into the MCU socket on the Seraph board — the USB port should face toward the board edge
    
7.  Double-check alignment: all pins seated, none bent

### Step 3 — Configure USB-MIDI Mode

This is the most important setting. Without it, your computer won't recognise the Teensy as a MIDI device.

1.  In Arduino IDE, go to Tools > Board and select Teensy 4.1
    
2.  Go to Tools > USB Type and select Serial + MIDI
    
3.  Connect the Teensy to your computer via USB
    
4.  You should see it appear in your system as both a serial port and a MIDI device

💡 On macOS, check Audio MIDI Setup (Applications > Utilities) to confirm the device appears. On Windows, check Device Manager.

    
# 02 — First Wiring Tutorial: Button & LED

This is your first circuit. You'll wire a push button and an LED to Seraph, then upload code that lights the LED when the button is pressed and sends a MIDI Note On message to your computer.

## What You'll Need

-   Seraph board with Teensy 4.1 mounted
    
-   1x tactile push button (momentary)
    
-   1x LED (any color, 3mm or 5mm)
    
-   1x 470Ω resistor (for LED current limiting)
    
-   Jumper wires
  
## Wiring Diagram

![](https://i.imgur.com/f4wfXpq.png)

![Descriptive alt text](https://i.imgur.com/JGOtWVS.png)
💡 Each channel strip on the digital bank has three pins in a row: + (power/3.3V), – (ground), and S (signal). For buttons, you only need S and –. For LEDs, S and –, with a resistor in series.

## The Code
Create a new sketch in Arduino IDE and paste in the following code. This is adapted from the Seraph_ButtonDemo in the GitHub repository.

```cpp
// Seraph — Button + LED Demo
// Digital Channel 1 = Button input
// Digital Channel 2 = LED output

const int BUTTON_PIN = 1;   // Change to match your wiring
const int LED_PIN    = 2;   // Change to match your wiring

const int MIDI_CHANNEL = 1;  // MIDI channel 1
const int MIDI_NOTE    = 60; // Middle C
const int MIDI_VELOCITY = 100;

int lastButtonState = HIGH; // Buttons default HIGH (internal pullup)

void setup() {
  pinMode(BUTTON_PIN, INPUT_PULLUP); // Enable internal pullup resistor
  pinMode(LED_PIN, OUTPUT);
}

void loop() {
  int buttonState = digitalRead(BUTTON_PIN);

  // Button pressed (LOW because of pullup)
  if (buttonState == LOW && lastButtonState == HIGH) {
    digitalWrite(LED_PIN, HIGH); // Turn LED on
    usbMIDI.sendNoteOn(MIDI_NOTE, MIDI_VELOCITY, MIDI_CHANNEL);
  }

  // Button released
  if (buttonState == HIGH && lastButtonState == LOW) {
    digitalWrite(LED_PIN, LOW); // Turn LED off
    usbMIDI.sendNoteOff(MIDI_NOTE, 0, MIDI_CHANNEL);
  }

  lastButtonState = buttonState;

  // Always flush MIDI at the end of loop()
  while (usbMIDI.read()) {}
}
```    
**Key Concepts**


### INPUT_PULLUP

When you use INPUT_PULLUP, the Teensy internally connects the pin to 3.3V through a resistor. This means the pin reads HIGH when nothing is connected, and LOW when the button pulls it to ground. This is why you only need two wires for a button — no external resistor needed.

### Event-Based MIDI

Notice the code only sends MIDI when the button state changes — not every loop iteration. This is called event-based transmission, and it's important: sending MIDI every millisecond would flood the connection with redundant messages. Always track the previous state and only transmit on transitions.

### usbMIDI.read()

The line while (usbMIDI.read()) {} at the end of loop() flushes any incoming MIDI data. Even if you're not receiving MIDI, the USB library needs this to maintain a stable connection. Always include it.

## Testing

1.  Select Tools > USB Type > Serial + MIDI in Arduino IDE
    
2.  Upload the sketch (Ctrl+U / Cmd+U)
    
3.  Open a MIDI monitor app (e.g. MIDI Monitor on macOS, MIDI-OX on Windows) to confirm notes are being sent
    
4.  Press the button — the LED should light and you should see a Note On message for Middle C
    

  

💡 If nothing happens, check that the Teensy appears as a MIDI device in your system, and that your pin numbers in the code match where you physically wired the components.

# 03 — Analog Sensors: Potentiometers & FSRs

Analog sensors output a continuously varying voltage rather than just on/off. Seraph's analog bank reads these voltages and converts them to numbers your code can use. This section covers the two most common types: potentiometers (knobs) and FSRs (force-sensitive resistors).

  

## 3A — Potentiometer (No Pulldown Required)

Potentiometers are the simplest analog sensors — they're essentially volume knobs. They have three leads: power, ground, and a wiper that outputs a voltage proportional to rotation.

## Wiring Diagram

![](https://i.imgur.com/VT83PWb.png)
![](https://i.imgur.com/CjAuKit.png)
💡  The left/right orientation depends on how you're holding the pot. If your value goes the wrong direction when you turn it, just swap the + and – leads.

```cpp
// Seraph — Potentiometer Demo
// Maps pot position to MIDI CC (Control Change)

const int POT_PIN      = A1;  // Analog channel 1
const int MIDI_CC      = 74;  // CC 74 = Filter Cutoff (common choice)
const int MIDI_CHANNEL = 1;

int lastCCValue = -1; // Track last sent value to avoid redundant messages

void setup() {
  // No pinMode needed for analog inputs
}

void loop() {
  int rawValue = analogRead(POT_PIN);  // 0-1023

  // Map 10-bit ADC range to 7-bit MIDI CC range (0-127)
  int ccValue = map(rawValue, 0, 1023, 0, 127);

  // Only send if value has changed
  if (ccValue != lastCCValue) {
    usbMIDI.sendControlChange(MIDI_CC, ccValue, MIDI_CHANNEL);
    lastCCValue = ccValue;
  }

  while (usbMIDI.read()) {}
}
```
## 3B — FSR / LDR (Pulldown Resistor Required)

Force-Sensitive Resistors (FSRs) and Light-Dependent Resistors (LDRs) change their resistance based on physical input. Unlike potentiometers, they only have two leads — you need to add a pulldown resistor to create the voltage divider that makes them readable.

### Why a Pulldown Resistor?

An FSR is just a variable resistor. To read it as a voltage, you pair it with a fixed resistor to form a voltage divider. Seraph has a resistor footprint on each analog channel strip for exactly this purpose — no breadboard required.

## Wiring Diagram

![](https://i.imgur.com/dIB1qCb.png)

![](https://i.imgur.com/BHXqAIf.png
)
⚠️ Place the pulldown resistor in the resistor slot on the same channel row as your FSR. The Seraph PCB has a dedicated footprint for this — it connects the signal line to ground through the resistor without any extra wiring.


### Recommended Resistor Values

![](https://i.imgur.com/k0FOzIM.png)


### Code
```cpp
// Seraph — FSR Demo
// FSR maps pressure to MIDI velocity / aftertouch

const int FSR_PIN      = A2;
const int MIDI_CHANNEL = 1;

// Calibration — adjust these to match your sensor's actual range
const int FSR_MIN = 50;   // Value when barely touched
const int FSR_MAX = 900;  // Value at maximum pressure

int lastCCValue = -1;

void loop() {
  int rawValue = analogRead(FSR_PIN);

  int ccValue = map(
    constrain(rawValue, FSR_MIN, FSR_MAX),
    FSR_MIN,
    FSR_MAX,
    0,
    127
  );

  if (ccValue != lastCCValue) {
    // CC 11 = Expression — works well for pressure control
    usbMIDI.sendControlChange(11, ccValue, MIDI_CHANNEL);
    lastCCValue = ccValue;
  }

  while (usbMIDI.read()) {}
}
```
💡 Use constrain() before map() when your sensor's real-world range doesn't match the theoretical 0–1023. This prevents the mapped value from going outside 0–127.

# 04 — I²C Devices: IMUs, Displays & More
I²C (Inter-Integrated Circuit) is a communication protocol that lets you connect multiple sensors to just two wires: SDA (data) and SCL (clock). Seraph exposes three I²C blocks (A, B, C), all defaulting to the Teensy's I2C0 port for maximum compatibility.

## Common I²C Devices for NIME Projects

-   IMU (MPU-6050, BNO055) — 6-axis or 9-axis motion sensing for gesture control
    
-   OLED Display (SSD1306) — small screen for parameter feedback
    
-   ADC Expander (ADS1115) — adds 4 extra high-resolution analog inputs
    
-   Distance Sensor (VL53L0X) — time-of-flight distance measuring
    
-   Color Sensor (TCS34725) — reads RGB color values
    

  

## Wiring
![](https://i.imgur.com/VopP0T5.png)
All I²C devices use the same four-pin connection regardless of which sensor you're using:
### 🔌 I²C Device (e.g. MPU-6050 IMU)

  

| Component Pin | → | Wire Color | → | Seraph Header |



  

| Sensor — SDA | → | Blue | → | I²C Block A — SDA |

  

| Sensor — SCL | → | Yellow | → | I²C Block A — SCL |

  

| Sensor — VCC | → | Red | → | I²C Block A — Power (+) |

  

| Sensor — GND | → | Black | → | I²C Block A — GND (−) |

💡 If your sensor has additional pins (INT, ADDR, etc.) beyond the four core I²C pins, refer to your sensor's datasheet. ADDR pins often let you change the I²C address to avoid conflicts when using multiple devices of the same type.

## Using Multiple I²C Devices

Multiple I²C devices can share the same two wires as long as each has a unique address. Seraph's three I²C blocks (A, B, C) are all wired to the same I2C0 bus — they're just convenient connection points, not separate buses.

-   Connecting two MPU-6050s: bridge the ADDR pin on one to 3.3V to change its address from 0x68 to 0x69
    
-   Connecting an OLED + IMU: different device types usually have different addresses by default — no configuration needed
    

  

## Code Example: MPU-6050 IMU to MIDI Pitch Bend

Install the Adafruit MPU6050 library via Tools > Manage Libraries in Arduino IDE before uploading this sketch.

```cpp
// Seraph — IMU (MPU-6050) to MIDI Pitch Bend Demo

#include "Wire.h"
#include "Adafruit_MPU6050.h"

Adafruit_MPU6050 mpu;

const int MIDI_CHANNEL = 1;

void setup() {
  Wire.begin();

  mpu.begin();

  mpu.setAccelerometerRange(MPU6050_RANGE_2_G);
  mpu.setGyroRange(MPU6050_RANGE_250_DEG);
}

void loop() {
  sensors_event_t accel, gyro, temp;

  mpu.getEvent(&accel, &gyro, &temp);

  // Map X-axis tilt (-10 to +10 m/s^2)
  // to MIDI pitch bend (-8192 to +8191)
  int pitchBend = map(
    constrain(accel.acceleration.x * 100, -1000, 1000),
    -1000,
    1000,
    -8192,
    8191
  );

  usbMIDI.sendPitchBend(pitchBend, MIDI_CHANNEL);

  delay(10); // ~100 updates/second

  while (usbMIDI.read()) {}
}
```
## Troubleshooting I²C
![enter image description here](https://i.imgur.com/9vpSpay.png)

# 05 — MIDI Mapping in Code

This section explains the core MIDI message types you'll use in Seraph projects and how to send them using the Teensyduino USB-MIDI library. Understanding these building blocks lets you map any sensor to any parameter in your DAW or audio software.
## Core MIDI Message Types
![enter image description here](https://i.imgur.com/09eKWef.png)
## usbMIDI Cheat Sheet

These are the most commonly used usbMIDI functions in Seraph projects:
```cpp
// Note On — play a note
usbMIDI.sendNoteOn(note, velocity, channel);

//   note:     0–127  (60 = Middle C)
//   velocity: 0–127  (0 = silent, 127 = max)
//   channel:  1–16


// Note Off — stop a note
usbMIDI.sendNoteOff(note, 0, channel);


// Control Change — continuous parameter
usbMIDI.sendControlChange(cc_number, value, channel);

//   cc_number: 0–127 (see MIDI CC list below)
//   value:     0–127


// Pitch Bend — continuous pitch movement
usbMIDI.sendPitchBend(value, channel);

//   value: -8192 to +8191
//   0 = center / no bend


// Program Change — switch patches
usbMIDI.sendProgramChange(program, channel);

//   program: 0–127


// Always flush MIDI messages at end of loop()
while (usbMIDI.read()) {}
```

## Useful MIDI CC Numbers

CC numbers are standardized. Here are the ones most useful for NIME and music production:
![](https://i.imgur.com/8wxerxz.png)
## Combining Multiple Sensors

Real Seraph projects typically combine several sensors sending different MIDI messages. Here's the structure to follow:
```cpp
// Multi-sensor Seraph template

const int POT_PIN    = A1;
const int FSR_PIN    = A2;
const int BUTTON_PIN = 1;

int lastPotCC = -1;
int lastFSRCC = -1;
int lastBtn   = HIGH;

void setup() {
  pinMode(BUTTON_PIN, INPUT_PULLUP);
}

void loop() {

  // --- Potentiometer -> CC 74 (Filter) ---
  int potCC = map(
    analogRead(POT_PIN),
    0,
    1023,
    0,
    127
  );

  if (potCC != lastPotCC) {
    usbMIDI.sendControlChange(74, potCC, 1);
    lastPotCC = potCC;
  }


  // --- FSR -> CC 11 (Expression) ---
  int fsrCC = map(
    constrain(analogRead(FSR_PIN), 50, 900),
    50,
    900,
    0,
    127
  );

  if (fsrCC != lastFSRCC) {
    usbMIDI.sendControlChange(11, fsrCC, 1);
    lastFSRCC = fsrCC;
  }


  // --- Button -> Note On/Off ---
  int btn = digitalRead(BUTTON_PIN);

  if (btn == LOW && lastBtn == HIGH) {
    usbMIDI.sendNoteOn(60, 100, 1);
  }

  if (btn == HIGH && lastBtn == LOW) {
    usbMIDI.sendNoteOff(60, 0, 1);
  }

  lastBtn = btn;


  // Always flush MIDI messages
  while (usbMIDI.read()) {}
}
```
# 06 — Connecting to a DAW

Once your Seraph is sending MIDI, the final step is getting that data into your DAW or audio software. This section covers Ableton Live, Max/MSP, and general MIDI routing — the three most common environments in NIME and music production contexts.

  

## 6A — Ableton Live

### Enable MIDI Input

1.  Go to Live > Preferences (Mac) or Options > Preferences (PC)
    
2.  Click the Link / Tempo / MIDI tab
    
3.  Find your Teensy in the MIDI Ports list — it will appear as something like "Teensy MIDI"
    
4.  Turn ON Track under the Input column for your Teensy
    
5.  Optionally turn ON Remote if you want to use it for Live's remote control
    

  

💡 If your Teensy doesn't appear, check that USB Type is set to Serial + MIDI in Arduino IDE and that the sketch is running (the board is powered and the code is uploaded).

  

### Mapping CC to a Parameter

6.  Enter MIDI Map Mode (Cmd+M on Mac, Ctrl+M on PC) — everything mappable turns blue
    
7.  Click the parameter you want to control (e.g. a filter knob on an instrument)
    
8.  Turn your potentiometer on Seraph — Live will detect the CC and assign it
    
9.  Exit MIDI Map Mode (Cmd+M / Ctrl+M again)
    

  

💡 In MIDI Map Mode you can also set min/max ranges for each mapping. This lets you limit a potentiometer to only sweep part of a parameter's range.

  

### Triggering Clips with Buttons

10.  Enter MIDI Map Mode
    
11.  Click a clip slot in Session View
    
12.  Press your button on Seraph — Live assigns it
    
13.  Exit MIDI Map Mode
    

  

## 6B — Max/MSP

Max/MSP is the most common environment for custom interactive music systems. The Lumaphone and H.A.I. projects from the Seraph case studies both used Max for audio processing.

### Receiving MIDI in Max

Use the midiin or notein / ctlin objects to receive MIDI from Seraph:
```cpp
// In a Max patch, create these objects:

[notein]          // receives Note On/Off from any MIDI device
[ctlin]           // receives Control Change messages
[bendin]          // receives Pitch Bend


// To receive from Seraph specifically:

[notein Teensy MIDI]    // specify device name in the argument
[ctlin Teensy MIDI]


// Example: map CC 74 to a filter cutoff

[ctlin 74 Teensy MIDI]  // only listens to CC 74 from Teensy
        |
     [/ 127.]           // normalize 0-127 to 0.0-1.0
        |
    [cutoff~]           // connect to your audio object
```
💡 If Max doesn't list your Teensy as a MIDI device, go to Max > Preferences > MIDI and click Refresh. Make sure the Teensy is plugged in and the sketch is running.

### Serial vs USB-MIDI in Max

Seraph defaults to USB-MIDI, but some advanced Seraph projects (like the Lumaphone) communicate over serial instead, sending raw data rather than formatted MIDI messages. For USB-MIDI, use notein/ctlin as above. For serial, use the serial object and parse the data manually.

  

## 6C — General MIDI Routing (Any DAW)

Any DAW that accepts MIDI input will work with Seraph. The general process is the same across tools:
## 🎛️ DAW / Software MIDI Setup

| DAW / Software | Where to Enable MIDI |
|---|---|
| **Ableton Live** | Preferences → MIDI → Enable **Track Input** for Teensy |
| **Logic Pro** | File → Project Settings → MIDI → Input Devices |
| **Bitwig Studio** | Preferences → Controllers → Add Controller → Generic MIDI |
| **Reaper** | Options → Preferences → MIDI Devices → Enable Teensy Input |
| **GarageBand** | Auto-detects USB MIDI — just connect and play |
| **Pure Data** | Use `[notein]` / `[ctlin]` objects (same as Max/MSP) |
| **SuperCollider** | `MIDIClient.init;` → `MIDIIn.connectAll;` → `MIDIdef.cc(...)` |

## Common DAW Issues

-   MIDI device not appearing: Confirm USB Type is Serial + MIDI in Arduino IDE and re-upload the sketch
    
-   Notes stuck on: Add usbMIDI.sendNoteOff() for every Note On — always pair them
    
-   CC values jumpy or noisy: Add smoothing in your sketch (see below), or adjust the sensor's pulldown resistor value
    
-   Wrong channel: Make sure the MIDI channel in your sketch matches what the DAW is listening on
    

  

### Adding Smoothing to Reduce Jitter
```cpp
// Exponential moving average — reduces sensor noise

float smoothed = 0;

const float ALPHA = 0.1;
// Lower = smoother, less responsive
// Higher = faster response, more jitter

void loop() {

  float raw = analogRead(A1);

  // Smooth incoming sensor data
  smoothed = ALPHA * raw + (1.0 - ALPHA) * smoothed;

  // Convert analog range (0-1023)
  // into MIDI range (0-127)
  int ccValue = map(
    (int)smoothed,
    0,
    1023,
    0,
    127
  );

  // Send MIDI as normal
  // usbMIDI.sendControlChange(CC, ccValue, CHANNEL);

  while (usbMIDI.read()) {}
}
```
You're ready to build.


Creative Computing at California Institute of the Arts is a forward-thinking interdisciplinary program that fuses the power of computational engineering skills with the limitless possibilities of artistic expression. This innovative degree encourages students to explore the intersection of technology and creativity, using computational tools to craft work that is both personally and culturally meaningful, while preparing them for industry. Our program is designed to provide an integrative learning experience that equips students with the skills to push the boundaries of art, music, and technology. With a strong foundation in computer science, electrical engineering, signal processing, and emerging technologies including virtual/augmented reality, robotics, and machine learning, students will be empowered to innovate, experiment, and reimagine what technology can do in artistic contexts.

<p align="center">
  <a href="https://creativecomputing.calarts.edu/">Learn More</a>
</p>


