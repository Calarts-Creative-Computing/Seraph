![Calarts Creative Computing Logo](https://i.imgur.com/XZpdi2e.png)


# Seraph



**Seraph – Open-Source Teensy MIDI Controller Platform**  
Welcome to Seraph!

Seraph is an open platform for developing Teensy-based MIDI controllers and interactive musical interfaces. This repository provides a PCB design and sample demo code to help you build and customize your own MIDI devices.

<p align="center">
  <a href="https://github.com/Calarts-Creative-Computing/Seraph/stargazers">
    <img src="https://img.shields.io/github/stars/Calarts-Creative-Computing/Seraph?style=social" alt="Stars">
  </a>
  <a href="https://github.com/Calarts-Creative-Computing/Seraph/watchers">
    <img src="https://img.shields.io/github/watchers/Calarts-Creative-Computing/Seraph?style=social" alt="Watchers">
  </a>
  <a href="https://github.com/Calarts-Creative-Computing/Seraph/network/members">
    <img src="https://img.shields.io/github/forks/Calarts-Creative-Computing/Seraph?style=social" alt="Forks">
  </a>
</p>

<p align="center">
  <a href="https://github.com/Calarts-Creative-Computing/Seraph/blob/main/LICENSE">
    <img src="https://img.shields.io/github/license/Calarts-Creative-Computing/Seraph" alt="License">
  </a>
  <a href="https://github.com/Calarts-Creative-Computing/Seraph/issues">
    <img src="https://img.shields.io/github/issues/Calarts-Creative-Computing/Seraph" alt="Issues">
  </a>
  <a href="https://github.com/Calarts-Creative-Computing/Seraph/pulls">
    <img src="https://img.shields.io/github/issues-pr/Calarts-Creative-Computing/Seraph" alt="Pull Requests">
  </a>
  <a href="https://github.com/Calarts-Creative-Computing/Seraph/commits/main">
    <img src="https://img.shields.io/github/last-commit/Calarts-Creative-Computing/Seraph" alt="Last Commit">
  </a>
</p>

<p align="center">
  <a href="https://github.com/Calarts-Creative-Computing/Seraph">
    <img src="https://img.shields.io/badge/View_on-GitHub-181717?logo=github&logoColor=white" alt="View on GitHub">
  </a>
  <a href="https://github.com/Calarts-Creative-Computing/Seraph/archive/refs/heads/main.zip">
    <img src="https://img.shields.io/badge/Download-ZIP-28a745?logo=github" alt="Download ZIP">
  </a>
</p>

---

### What's Inside?

This repository includes:

- **Seraph PCB design** – A hardware platform for connecting various sensors and controls.  
- **Demo code** – Examples showcasing how to interface different sensors with the Seraph PCB.  
- **Customization support** – Easily modify the code to suit your specific project needs.  

---

### Quick Start

1. **Solder your Teensy to the board**  
   ◦ Use headers instead of soldering the Teensy directly to the PCB.  
   ◦ This allows you to easily remove and reuse your Teensy for future projects.  

2. **Install the required library**  
   ◦ Most of our demo code uses the Teensy USBMIDI library.  
   ◦ Download it via the Arduino Library Manager before running the code.  

3. **Set your board mode**  
   ◦ Ensure your Teensy is in Serial + MIDI mode in the Arduino IDE.  
   ◦ This allows your computer to recognize it as a MIDI device.  

4. **Upload and experiment!**  
   ◦ Flash the code to your Teensy and start exploring interactive MIDI control.  

---

### Get Creative!

Seraph is designed to be flexible and open-ended. Whether you're building a MIDI controller, sound-reactive installation, or experimental music interface, this platform is here to support your creativity.

<p align="center"><strong>Have fun and make something amazing!</strong></p>

---

![Seraph Screenshot](https://i.imgur.com/n9ZP12J.png)


**<span style="text-decoration:underline;">Seraph Quickstart Tutorials + Diagrams</span>**

**<span style="text-decoration:underline;">Connecting an Analog Sensor (No Pulldown Resistor Required)</span>**

For simple analog sensors like potentiometers, which do not require pulldown resistors:

1: Connect your power lead to any + (power) via.

2: Connect your ground lead to any – (ground) via.

3: Connect your data lead to any available analog input via, and make sure to modify your code to match the analog input you chose






![Diagram1](https://i.imgur.com/gaAru7W.png "image_tooltip")


*The code for this circuit can be found in the Seraph Github repository under the name **Seraph_PotentiometerDemo***

---


### **<span style="text-decoration:underline;">Connecting an Analog Sensor *with* a Pulldown Resistor</span>**

For sensors such as LDRs (Light Dependent Resistors) or FSRs (Force Sensitive Resistors) that *do* require a pulldown resistor:



1. Connect your power lead to any + (power) via. 

2. Connect your data lead to your desired analog input via, and make sure to reflect this choice in your code. 

3. Place your pulldown resistor in the allocated resistor area on the PCB, and make sure it is in the same row as your chosen analog input. This connects the sensor’s data line to ground. The built-in resistor footprint on the PCB makes this easy and clean.






![alt_text](https://i.imgur.com/6BPQGN4.png "image_tooltip")


*The code for this circuit can be found in the Seraph Github repository under the name **Seraph_FSRDemo***
---


**<span style="text-decoration:underline;">Connecting a Digital Sensor</span>**

For basic digital input devices like buttons that don’t require pullup or pulldown resistors:



1. Connect one lead of the button to any - (ground) via.
2. Connect the other lead to the desired digital input via, and make sure to reflect this choice in your code.
3. To connect an LED to this circuit, simply place an LED in one of the allocated LED slots, and place your resistor in a resistor slot in the same row.






![alt_text](https://i.imgur.com/5UsLsT5.png) 


*The code for this circuit can be found in the Seraph Github repository under the name **Seraph_ButtonDemo***

---



### **<span style="text-decoration:underline;">Connecting an I²C Sensor</span>**

To connect an I²C sensor (such as IMUs or environmental sensors):



1. Connect the SDA (Data) pin on your sensor to an SDA pin on Seraph.  

2. Connect the SCL (Clock) pin on your sensor to an SCL pin on Seraph. 

3. Connect the Power (VCC) pin on your sensor to a + (power) via. 

4. Connect the Ground (GND) pin on your sensor to a – (ground) via.

							






![figure4](https://i.imgur.com/VybPBKZ.png "image_tooltip")

---


<p align="center">
  <img src="https://www.hanoverresearch.com/wp-content/uploads/2020/05/CALARTS-01.png" alt="Calarts Creative Computing Logo" width="300"/>
</p>

Creative Computing at California Institute of the Arts is a forward-thinking interdisciplinary program that fuses the power of computational engineering skills with the limitless possibilities of artistic expression. This innovative degree encourages students to explore the intersection of technology and creativity, using computational tools to craft work that is both personally and culturally meaningful, while preparing them for industry. Our program is designed to provide an integrative learning experience that equips students with the skills to push the boundaries of art, music, and technology. With a strong foundation in computer science, electrical engineering, signal processing, and emerging technologies including virtual/augmented reality, robotics, and machine learning, students will be empowered to innovate, experiment, and reimagine what technology can do in artistic contexts.

<p align="center">
  <a href="https://creativecomputing.calarts.edu/">Learn More</a>
</p>
