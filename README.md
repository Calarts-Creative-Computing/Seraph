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
![](https://i.imgur.com/CjAuKit.png
)




Creative Computing at California Institute of the Arts is a forward-thinking interdisciplinary program that fuses the power of computational engineering skills with the limitless possibilities of artistic expression. This innovative degree encourages students to explore the intersection of technology and creativity, using computational tools to craft work that is both personally and culturally meaningful, while preparing them for industry. Our program is designed to provide an integrative learning experience that equips students with the skills to push the boundaries of art, music, and technology. With a strong foundation in computer science, electrical engineering, signal processing, and emerging technologies including virtual/augmented reality, robotics, and machine learning, students will be empowered to innovate, experiment, and reimagine what technology can do in artistic contexts.

<p align="center">
  <a href="https://creativecomputing.calarts.edu/">Learn More</a>
</p>


https://i.imgur.com/CjAuKit.png
