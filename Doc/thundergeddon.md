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
- [ ] Add hull board LED on spare GPIO



## Hardware

I have designed two boards: hull and turret.  Both boards are standard 2-sided, default JLC PCB boards.  The hull will have two sided assembly, while the turret will only be assembled on the top side.  Both boards have a GND pour on the bottom layer.  

------

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
- Source and drain both have an an area of copper under them around 5mm×5mm with several vias to a similar size area on the other side of the board

#### **Bulk Capacitor**

I'm using a 220uF bulk capacitor (C15 C2887273). 

* Anode connects to:
  * Q1 Drain
  * +BATT
    * PS1 EN and IN
    * U2 and U3 VM 
    * Slip Ring
* Cathode connects to GND

#### 3.3V Voltage Step-down

* Step down regulator SY8113BADC (PS1 C78989)
* EN and IN connect to C15 with a 10uF decoupling capacitor to GND (C10 C15850)
* FB connects to:
  * 22kΩ to GND (R16 C31850)
  * 100kΩ to +3V3_HULL with 22pF in parallel (C14 C1653)
* GND connected to GND
* BS connects to 100nf (C13 C14663) inline to LX
* LX connects
  * to 4.7uH inductor (L1 C167220)
  * Two decoupling capacitors 100nf (C7 C14663) and 47uF (C9 C16780)
  * Out to +3V3_HULL

#### I2C Expander

PCA9555PW is used to provide GPIOs for motor control and IR receivers (U8 C128392).  

* SCL and SDA connect to slip ring (pull ups are on the turret board)
* INT no connection
* A0, A1, A2 connect to GND
* VSS to GND

| IO   | Connects to            | Pull up to +3V3_HULL |
| ---- | ---------------------- | -------------------- |
| 0_0  | IR Receive N           | 10kΩ (R8 C25804)     |
| 0_1  | IR Receive NE          | 10kΩ (R9 C25804)     |
| 0_2  | IR Receive E           | 10kΩ (R13 C25804)    |
| 0_3  | IR Receive SE          | 10kΩ (R14 C25804)    |
| 0_4  | IR Receive S           | 10kΩ (R19 C25804)    |
| 0_5  | IR Receive SW          | 10kΩ (R18 C25804)    |
| 0_6  | IR Receive W           | 10kΩ (R11 C25804)    |
| 0_7  | IR Receive NW          | 10kΩ (R10 C25804)    |
| 1_0  | Left Motor Control 1   |                      |
| 1_1  | Left Motor Control 2   |                      |
| 1_2  | Turret Motor Control 1 |                      |
| 1_3  | Turret Motor Control 2 |                      |
| 1_4  | Motors Sleep           |                      |
| 1_5  | Right Motor Control 2  |                      |
| 1_6  | Right Motor Control 1  |                      |
| 1_7  | No connection          |                      |

#### Motor Drivers

##### Right Track

Motor Controller DRV8833PWP (U2 C50506) controls the right track.  

* VM connects to +BATT
  * Decoupling capacitors 100nf (C4 C14663) and 10uF (C5 C15850)
* VINT bypassed to GND with 2.2uF (C3 C23630)
* FAULT, AIN1, AIN2, AOUT1, AOUT2: no connection
* BIN1 connected to Right Motor Control 2
* BIN2 connected to Right Motor Control 1
* BOUT1 connected to Right Drive Motor 1
* BOUT2 connected to Right Drive Motor 2
* GND, BISEN and AISEN connected to GND
* VCP High-side gate drive voltage connects to VM via inline 0.01uF (C1 C57112)
* SLEEP connected to MOTORS_SLEEP

Right Drive Motor 1 and 2 connect to 2-pin JST PH (J3 C131337) which connects to a 050 motor with a 118:1 gearbox for driving the right track.  

##### Left Track and Turret

Motor Controller DRV8833PWP (U3 C50506) controls the right track.  

* VM connects to +BATT
  * Decoupling capacitors 100nf (C11 C14663) and 10uF (C12 C15850)
* VINT bypassed to GND with 2.2uF (C3 C23630)
* FAULT: no connection
* AIN1 connected to Turret Motor Control 1
* AIN2 connected to Turret Motor Control 2
* AOUT1 connected to Turret Motor Control 2
* AOUT2 connected to Turret Motor Control 1
* BIN1 connected to Left Motor Control 1
* BIN2 connected to Left Motor Control 2
* BOUT1 connected to Left Drive Motor 1
* BOUT2 connected to Left Drive Motor 2
* GND, BISEN and AISEN connected to GND
* VCP High-side gate drive voltage connects to VM via inline 0.01uF (C6 C57112)
* SLEEP connected to MOTORS_SLEEP

Left Drive Motor 1 and 2 connect to 2-pin JST PH (J5 C131337) which connects to a 050 motor with a 118:1 gearbox for driving the right track.  

Turret Motor 1 and 2 connect to 2-pin JST PH (J4 C131337) which connects to a N20 motor with a 1030:1 gearbox for driving the turret.  

#### IR Receivers

There are eight infrared receivers IRM-V838M3-C/TR1 (C499566) at 45 degree increments along the edge of the board, facing outwards.  

* OUT pin connects to a GPIO on the PCA9555
* VCC has an inline 47Ω resistor (C23182)
* VCC is decoupled with 100nF (C14663) and 4.7uF (C1779)

| GPIO Connection | IR Receiver | 100nF | 4.7uF | 47Ω  |
| --------------- | ----------- | ----- | ----- | ---- |
| IR Receive N    | U4          | C16   | C2    | R1   |
| IR Receive NE   | U5          | C18   | C17   | R2   |
| IR Receive E    | U6          | C19   | C26   | R5   |
| IR Receive SE   | U12         | C20   | C27   | R6   |
| IR Receive S    | U11         | C22   | C28   | R12  |
| IR Receive SW   | U10         | C23   | C29   | R20  |
| IR Receive W    | U9          | C24   | C30   | R21  |
| IR Receive NW   | U7          | C25   | C31   | R22  |

#### Slip Ring Connector

#### Other

* Status LED
* Expansion
* Test Points

------

### Turret Board

#### Slip Ring

#### ESP32

#### 3.3V Voltage Step Down

#### USB Connector

#### IR LEDs

#### External RGB LED

#### Buzzer

#### Status LEDs



#### 

------

### Software

### Unreal Prototype
