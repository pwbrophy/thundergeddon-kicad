# Thundergeddon

## What is Thundergeddon?

**Thundergeddon** is a hybrid tabletop/RC tank battle game: real, small robotic tanks fight in the physical world, while a **Unity companion app** runs the match logic and controls the robots over the local network.

* **Physical side:**
  * Custom ESP32 electronics, motor + turret control, and an IR “shooting / hit detection” system plus FPV camera streaming
  
  * Robots measure about 13cm long, 80cm wide and 80cm high
  * Robots body is 3d printed PLA, created in OnShape
  * Electronics designed in KiCad, manufactured by JLC PCB
  
* **Digital side (Unity app):** discovers/hosts robots on Wi-Fi (UDP discovery + WebSocket control), lets you select and drive a tank with an on-screen joystick, shows a live video feed per robot, and runs **turn-based rules** like player/alliance turns, action points, and a coordinated “shoot” sequence.

## To-do

- [ ] Test reverse polarity protection



## Hardware

I have designed two boards: hull and turret.  Both boards are standard 2-sided, default JLC PCB boards.  The hull will have two sided assembly, while the turret will only be assembled on the top side.  Both boards have a GND pour on the bottom layer.  

### Hull Board

The hull board connects to the battery, motors, IR receivers and turret board via a slip-ring.

#### **Power**

Two 18350 li-ion batteries power the robot

* Connected via a JST XH 2-pin connector (J2 C158012)
* Positive terminal connected to Q1 source pin via 1.2mm width copper trace
* Negative connects to GND

#### **Reverse Polarity Protection**

I use a P-channel MOSFET to protect the board from reversed batteries (Q1 C15127)

+ **Source** connects to J2

- **Drain** connects to C15 via 1.2mm width copper trace
- **Gate** connects to GND via inline 100kΩ capacitor (R4 C25803)
- Source and drain both have an an area of copper under them around 5mm×5mm with several vias to a similar size area on the 

#### **Bulk Capacitor**

I'm using a 220uF bulk capacitor (C15 C2887273). 

* Anode connects to:
  * Q1 Drain
  * PS1 EN and IN
  * U2 and U3 VM
  * Slip Ring

#### 3.3V Voltage Step-down

#### I2C Expander

#### Motor Drivers

#### IR Receivers

#### Slip Ring Connector

#### Other

* Status LED
* Expansion
* 

### Turret Board

### Software

### Unreal Prototype
