# Thundergeddon

## What is Thundergeddon?

**Thundergeddon** is a hybrid tabletop/RC tank battle game: real, small robotic tanks fight in the physical world, while a **Unity companion app** runs the match logic and controls the robots over the local network.

* **Physical side:**
  
  * Custom ESP32 electronics, motor + turret control, and an IR “shooting / hit detection” system plus FPV camera streaming
  
  * Robots measure about 13cm long, 8cm wide and 8cm high
  * Robots body is 3d printed PLA, created in OnShape
  * Electronics designed in KiCad, manufactured by JLC PCB
  
* **Digital side (Unity app):** discovers/hosts robots on Wi-Fi (UDP discovery + WebSocket control), lets you select and drive a tank with an on-screen joystick, shows a live video feed per robot, and runs **turn-based rules** like player/alliance turns, action points, and a coordinated “shoot” sequence.

## To-do

- [ ] Test reverse polarity protection
- [ ] Add hull board LED on spare GPIO



## Hardware

I have designed two boards: hull and turret.  Both boards are standard 2-sided, default JLC PCB boards.  The hull will have two sided assembly, while the turret will only be assembled on the top side.  Both boards have a GND pour on the bottom layer.  

Each board is a separate KiCad file.  

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
  * 100kΩ (to +3V3_HULL with 22pF in parallel (C14 C1653)
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
| 1_0  | Left Motor Control 1   | n/a                  |
| 1_1  | Left Motor Control 2   | n/a                  |
| 1_2  | Turret Motor Control 1 | n/a                  |
| 1_3  | Turret Motor Control 2 | n/a                  |
| 1_4  | Motors Sleep           | n/a                  |
| 1_5  | Right Motor Control 2  | n/a                  |
| 1_6  | Right Motor Control 1  | n/a                  |
| 1_7  | No connection          | n/a                  |

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

Right Drive Motor 1 and 2 connect to 2-pin JST PH (J3 C131337) which connects to a 050 motor with a 118:1 right-angle gearbox for driving the right track.  

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

Left Drive Motor 1 and 2 connect to 2-pin JST PH (J5 C131337) which connects to a 050 motor with a 118:1 right-angle gearbox for driving the right track.  

Turret Motor 1 and 2 connect to 2-pin JST PH (J4 C131337) which connects to a N20 motor with a 1030:1 right-angle gearbox for driving the turret.  

#### IR Receivers

There are eight infrared receivers IRM-V838M3-C/TR1 (C499566) at 45 degree increments, facing outwards along the edge of the board.  

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

The Slip Ring Connector is a JST XH 4-pin connector(J1 C2908602) which provides I2C and power to the turret board, through a 4-wire slip ring.  

* SCL (Yellow wire)
* SDA (Green wire)
* GND (Black wire)
* +BATT (Red wire)

#### Other

* Power Status LED
  * Red Power Indicator LED (D9 C2286)
  * Connected to through inline 1k2Ω resistor (R17 C22765)
* Expansion
  * Five holes for expansion (J7 C50950): SCL, SDA, BAT, 3V3, GND
  * Can also be used as test points
  * Won't have the connector on it, just using it for the footprint/hole layout
* Test Points
  * IR N (TP8)
  * SLEEP (TP5)
  * L CTRL 1 (TP3)
  * L CTRL 2 (TP4)
  * L DRV 1 (TP6)
  * L DRV 2 (TP7)
* Mounting Holes
  * 4 × 2.2mm holes for mounting M2 bolts

------

### Turret Board

The Turret Board contains the ESP32, 3.3V step down, USB connector and connectors for external IR LEDs, buzzer and RGB LEDs.  

#### ESP32

I'm using an ESP32-S3-WROOM-1U-N16R8 (U1 C3013946) which has an IPX connector for an external antenna, and no on-board antenna.  

| PIN    | Connections          |
| ------ | -------------------- |
| EN     | EN Button            |
| GPIO00 | BOOT Button          |
| GPIO01 | Piezo Buzzer         |
| GPIO02 | N/C                  |
| GPIO03 | N/C                  |
| GPIO04 | Camera: SIOD         |
| GPIO05 | Camera: SIOC         |
| GPIO06 | Camera: VSYNC        |
| GPIO07 | Camera: HREF         |
| GPIO08 | Camera: PCLK         |
| GPIO09 | Camera: Y6           |
| GPIO10 | Camera: Y2           |
| GPIO11 | Camera: Y5           |
| GPIO12 | Camera: Y3           |
| GPIO13 | Camera: Y4           |
| GPIO14 | N/C                  |
| GPIO15 | Camera: Y9           |
| GPIO16 | Camera: XCLK         |
| GPIO17 | Camera: Y8           |
| GPIO18 | Camera: Y7           |
| GPIO19 | USB_D-               |
| GPIO20 | USB_D+               |
| GPIO21 | I2C: SDA             |
| GPIO35 | Camera: PSRAM        |
| GPIO36 | Camera: PSRAM        |
| GPIO37 | Camera: PSRAM        |
| GPIO38 | External LEDs WS2812 |
| GPIO39 | IR LED #1            |
| GPIO40 | IR LED #2            |
| GPIO41 | N/C                  |
| GPIO42 | N/C                  |
| GPIO43 | N/C                  |
| GPIO44 | N/C                  |
| GPIO45 | N/C                  |
| GPIO47 | I2C: SCL             |
| GPIO48 | Blue Turret LED      |

The +3V3 connection on the ESP32 is decoupled with two capacitors, 10uF (C28 C15850) and 100nF (C27 C14663).  

##### Buttons

There are buttons on EN  and BOOT.  I chose to include buttons so that I can force the ESP32 into programming mode if required.  I hope that I won't need to press the buttons and just program via USB_D+ and USB_D-.  

###### EN Button

The EN switch (SW2 C318884) is connected on one side to GND and the other side to EN on the ESP32.  Between the switch and EN there is a 1uF capacitor (C23 C15849) to GND and a 10k resistor (R16 C25804) to +3V3.  

###### BOOT button

The BOOT switch (SW1 C318884) is connected on one side to GND and the other side to GPIO00/BOOT on the ESP32.  Between the switch and EN there is a 100nF capacitor (C22 C14663) to GND and a 10k resistor (R15 C25804) to +3V3.  

##### I2C

SDA and SCL are both pulled high with 4.7kΩ resistors (SDA R19, SCL R20 C23162).  They both have 4.7k inline resistors (SDA R21, SCL R22 C23140).  

#### Camera Connector

I am to connect an OV2640 camera module via the camera connector (FPC1 C262643). The camera is fed from 3V3_CLEAN which is converted to 2.8V and 1.3V power rails.  

| Pin  | Camera Pins | Connection   |
| ---- | ----------- | ------------ |
| 1    | NC          | N/C          |
| 2    | AGND        | GND          |
| 3    | SIO_D       | GPIO04       |
| 4    | AVDD        | CAM_AVDD_2V8 |
| 5    | SIO_C       | GPIO05       |
| 6    | RESETB      | CAM_2V8      |
| 7    | VSYNC       | GPIO06       |
| 8    | PWDN        | GND          |
| 9    | HREF        | GPIO07       |
| 10   | DVDD        | CAM_1V2      |
| 11   | DOVDD       | CAM_2V8      |
| 12   | Y9          | GPIO15       |
| 13   | XCLK        | GPIO16       |
| 14   | Y8          | GPIO17       |
| 15   | DGND        | GND          |
| 16   | Y7          | GPIO18       |
| 17   | PCLK        | GPIO08       |
| 18   | Y6          | GPIO09       |
| 19   | Y2          | GPIO10       |
| 20   | Y5          | GPIO11       |
| 21   | Y3          | GPIO12       |
| 22   | Y4          | GPIO13       |
| 23   | Y1          | N/C          |
| 24   | Y0          | N/C          |

* **+3V3_CLEAN**

  * +3V3 goes through a ferrite bead (FB1 C1002)
  * then has 100nF (C8 C14663) and 1uF (C9 C15849) decoupling capacitors 

* **+CAM_2V8**

  * From +3V3_CLEAN there are 100nF (C17 C1525) and 1uF (C20 C15849) decoupling capacitors 

  * it then enters the 2.8V LDO (U3 C53099)

    | U3 2.8V LDO ME6211C28M5G-N |            |
    | -------------------------- | ---------- |
    | VIN                        | +3V3_CLEAN |
    | VSS                        | GND        |
    | CE                         | +3V3_CLEAN |
    | VOUT                       | +CAM_2V8   |

  * After the LDO there are three decoupling capacitors: 100nF (C10 C1525), 1uF (C12 C15849) and 4.7uF (C13 C1779)

* **+CAM_1V3**

  * From +3V3_CLEAN there are 100nF (C19 C1525) and 1uF (C21 C15849) decoupling capacitors 

  * it then enters the 1.3V LDO (U4 C3008051)

  * | U4 1.3V LDO LN1234B132MR-G |            |
    | -------------------------- | ---------- |
    | VIN                        | +3V3_CLEAN |
    | VSS                        | GND        |
    | CE                         | +3V3_CLEAN |
    | VOUT                       | +CAM_1V3   |

  * After the LDO there are two decoupling capacitors: 100nF (C11 C1525) and 1uF (C14 C15849) 

* **+CAM_AVDD_2V8**

  * +CAM_2V8 enters a ferrite bead (FB2 C1002)
  * then decoupling capacitors 100nf (C15 C14663) and 1uF (C16 C15849)

* **SIOD** is pulled up to +CAM_2V8 with a 4.7kΩ resistor (R7 C23162)

* **SIOC** is pulled up to +CAM_2V8 with a 4.7kΩ resistor (R13 C23162)

* **RESETB** is pulled up  to +CAM_2V8 with a 10k resistor (R10 C25804)

* **PWDN** is pulled to GND with a 10k resistor (R12 C25804)

* **XCLK1** has an inline 33Ω resistor (R11 C23140)

#### 3.3V Voltage Step Down

* Step down regulator SY8113BADC (PS1 C78989)
* EN and IN connect to C15 with a 10uF decoupling capacitor (C5 C15850)
* FB connects to:
  * 22kΩ to GND (R4 C31850)
  * 100kΩ (R3 C25803) to +3V3_HULL with 22pF in parallel (C7 C1653)
* GND connected to GND
* BS connects to 100nf (C6 C14663) inline to LX
* LX connects
  * to 4.7uH inductor (L1 C167220)
  * Two decoupling capacitors 100nf (C3 C14663) and 47uF (C4 C16780) after inductor
  * Out to +3V3_HULL

#### USB Connector

#### Slip Ring Connector

#### IR LEDs

#### External RGB LED

#### Buzzer

#### Status LEDs

#### Antenna

#### 

------

### Software

### 
