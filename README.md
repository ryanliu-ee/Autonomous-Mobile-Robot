# Autonomous Mobile Navigation Robot

A from-scratch **autonomous mobile robot** built to navigate point-to-point across an environment while avoiding obstacles. Built from individually selected components rather than a kit, prioritizing **precision and repeatable navigation**.

> **Status:** In progress · Started October 2025
> Hardware selected and purchased · Core subsystems bench-tested · Integration underway

---

## Table of Contents
- [Goals](#goals)
- [Current Status](#current-status)
- [Project Phases](#project-phases)
- [Hardware](#hardware)
- [Component Selection & Tradeoffs](#component-selection--tradeoffs)
- [Engineering Challenges](#engineering-challenges)
- [Software](#software)
- [Progress Log](#progress-log)
- [Testing Parts](#phase-1--testing-parts)
- [Project Reframe](#project-reframe-jan-2026)
- [Resources](#resources)
- [Team](#team)

---

## Goals

- Build a working autonomous robot that navigates point-to-point while avoiding obstacles.
- Prioritize **precision and accuracy** over raw speed.
- Build it **from scratch** (no kit, DIY parts) to own the full hardware/firmware/controls stack.
- Reuse the platform for future projects — vision- or voice-controlled robots, or a robotic arm.
- Longer term: extend the navigation stack toward maze-solving and mapping.

---

## Current Status

| Milestone | Status |
|---|---|
| Research goals & parts | ✅ Done |
| Select & purchase components | ✅ Done |
| Bench-test motor control | ✅ Done |
| Bench-test IR obstacle detection | ✅ Done |
| IMU integration | 🔧 In progress |
| Encoder odometry | ⬜ Planned |
| Chassis design & assembly | ⬜ Planned |
| Point-to-point navigation loop | ⬜ Planned |
| Custom carrier PCB | ⬜ Planned |

---

## Project Phases

A broad roadmap — not an exact sequence.

### Phase 1 — Hardware
1. Research goals and parts
2. Pick and buy parts
3. Test parts

### Phase 2 — Design
1. Design the robot
2. CAD the chassis
3. Assembly
4. Loop back to Phase 1 as needed

### Phase 3 — Software
1. Simulation (optional, mainly for ROS2)
2. Code the robot

### Phase 4 — Testing
1. Set up a test environment
2. Test navigation and obstacle avoidance
3. Iterate back through earlier phases as needed

---

## Hardware

| Component | Part | Role |
|---|---|---|
| **Microcontroller** | Arduino Nano 33 BLE Rev2 | Brain; runs firmware, has a built-in IMU |
| **Drive** | 12V 50 RPM N20 encoder motors | Propulsion + built-in encoders for odometry |
| **Wheels** | N20 gear-motor wheels | — |
| **Obstacle sensing** | Active IR sensors (×6) | Detect obstacles via reflected IR |
| **Orientation** | Onboard IMU (in the Nano) | Precise heading for turns |
| **Distance** | Motor encoders | Distance traveled → odometry & PID |
| **Power** | 2S LiPo (<1000 mAh) | Powers the system |
| **Regulation** | Adjustable step-down voltage regulators | Stable voltage to MCU and motors |
| **Motor control** | Motor driver | Controls motor speed, direction, current |
| **Prototyping** | Small breadboard | Wire components without soldering (for now) |

---

## Component Selection & Tradeoffs

The interesting engineering here was the **decision-making** behind each part — weighing options instead of buying a kit.

### Microcontroller — Arduino Nano 33 BLE Rev2
Chosen for strong clock speed, flash, and SRAM in a very small footprint, plus a **built-in IMU** (no separate sensor needed for turns). A Raspberry Pi was the fallback if more compute proved necessary.
**Tradeoff learned the hard way:** it doesn't ship pre-soldered — added an assembly/soldering step.

### Drive — 12V 50 RPM N20 Encoder Motors
Brushed DC motors chosen because they're easy to control with **PWM** and need no ESC. The N20s have **encoders built in**, saving cost and enabling odometry and PID.
**Key decision — 35 vs. 50 RPM (torque vs. speed):** chose **50 RPM** to keep the platform fast enough and reusable, planning to recover precision through PID tuning.

### Sensing — Active IR
The emitter shines an IR beam; the receiver detects the reflection. A reflection means an obstacle is ahead, and reflection strength/timing gives a rough distance.
**Ruled out GPS:** it's outdoor-scale (satellite-based lat/long) and far too coarse for indoor navigation.

### Power — 2S LiPo + Regulators
A 2S LiPo sized under ~1000 mAh to the robot's current draw, fed through adjustable step-down regulators — important both to protect the MCU and to give the motors a stable voltage. (Regulators reportedly burn out easily, so spares are worth having.)

---

## Engineering Challenges

Sourcing individual parts surfaced real compatibility problems to reason through:

- **Motor/battery mismatch** — the initial motor+encoder combo would have drained the originally-spec'd battery too quickly; had to reconcile motor current draw against battery capacity.
- **Voltage vs. controllability** — lower motor voltage is generally easier to control precisely with PID, which fed back into part selection.
- **Drive configuration** — weighed 2-motor vs. 4-motor drive; chose **2-wheel drive** to keep the system simpler and easier to control.
- **Assembly reality** — the Nano arriving un-soldered added a hands-on soldering step before bring-up.

---

## Software

- Firmware developed in **Arduino (C/C++)**, with a planned transition to **ROS2** for navigation and optional simulation.
- Subsystems are written and **bench-tested independently** — motor control and IR detection first — before integration into a full sense–plan–move loop.
- Navigation goal: reliable point-to-point movement with obstacle avoidance, extensible toward mapping later.

*(Test code lives in [`/code`](./code) — see the [Progress Log](#progress-log) for what's working.)*

---

## Progress Log

**Oct 17 – Dec 10, 2025 — Hardware research**
Worked through a beginner's guide to building an autonomous robot, listing required and recommended parts (motors/wheels, chassis, microcontroller, sensors, encoders, batteries, regulators, breadboard, motor driver) and the reasoning behind each category.

**Dec 2025 – Jan 2026 — Part selection & purchasing**
Team compiled a parts spreadsheet by category. Reconciled compatibility (motor current vs. battery), debated 35 vs. 50 RPM motors and 2- vs. 4-motor drive. Bought the Arduino Nano 33 BLE Rev2; selected motors, wheels, IR sensors, regulators, and breadboard. Estimated cost ~\$120 across the team.

**Jan 17, 2026 — Parts arriving**
Components began arriving; planned to bench-test with an Arduino Uno / Raspberry Pi while waiting on the Nano and motors. Next: verify each subsystem, then move to chassis design and initial programming.

*(More entries as the build progresses.)*

---

## Phase 1 — Testing Parts

Verifying each component works and is compatible before integration.
*Started Jan 18, 2026 · Ongoing*

### Testing the IR Sensors (Arduino Uno)

To learn the sensor's basics and confirm functionality, I wired an LED, an Arduino Uno, and one IR sensor on a breadboard so the LED lights when the sensor detects an obstacle.

**Connections:**

| From | To |
|------|-----|
| Arduino 5V | IR Sensor VCC |
| Arduino GND | IR Sensor GND |
| Arduino Pin 12 | IR Sensor OUT |
| Arduino Pin 2 → resistor | Green LED (+) |
| Green LED (−) | Arduino GND |

**Test code (Arduino IDE):**

```cpp
const int IR_input  = 2;    // reads the sensor's OUT
const int IR_output = 13;   // drives the indicator LED

void setup() {
  pinMode(IR_input, INPUT);
  pinMode(IR_output, OUTPUT);
  Serial.begin(9600);       // start Serial Monitor
}

void loop() {
  int sensorState = digitalRead(IR_input);

  // LOW means there's an obstacle
  if (sensorState == LOW) {
    digitalWrite(IR_output, HIGH);  // obstacle detected -> LED on
  } else {
    digitalWrite(IR_output, LOW);   // no obstacle -> LED off
  }

  delay(100);
}
```

**Notes:**
- Sensor sensitivity is adjustable via the small screw/potentiometer on the module.
- *(Build photo of the test setup to be added.)*

### Testing the Motors & Motor Driver (Arduino Uno)

Using the Arduino Uno, a DC encoder motor, and a breadboard to verify I can:
- spin the motor,
- change direction,
- change speed (PWM),
- and eventually drive two motors at once.

Also reading the **encoder** values on the Serial Monitor to confirm they work for later odometry/PID.

**Note:** the motor driver we bought already includes a 5V regulator on-board — meaning the separate voltage regulators we purchased won't be needed after all.

*(Motor, encoder, and sensor-driven-motor test code and results to be added as testing continues.)*

**Testing is still in progress** — subsystems are being verified individually before moving fully into design and integration.

---

## Phase 2 — Design: Chassis

Designing the robot around the parts purchased and verified in Phase 1.
*In progress.*

### Rough Draft

The initial layout uses a **2-motor drive with a ball caster** for support. The three main components — motor driver, Arduino Nano, and LiPo — sit in the **center of the chassis** for balance, with the **IR sensors lined up across the front** (some angled outward) to detect obstacles.

Still deciding between a **round** and a **rectangular** chassis design.

*(Layout sketch to be added.)*

### CAD

The chassis needs internal spaces sized to hold each component. Parts will most likely be held in place with double-sided tape; the trickiest part is fitting the **ball caster**.

**Process:**
Working in **Fusion 360**, I started from our base dimensions (**150 mm × 120 mm**) and curved the edges so the robot can't catch on corners. For the rear support wheel, a custom CAD solution (a sphere in a housing) is possible, but a pre-made caster lets us work immediately — and there's a pre-made socket to fit either option.

**Top view — component placement** (top-down; symmetrical pieces marked **S**):
1. L298N motor driver bay
2. Wheel housing
3. (S) Motor wire holes
4. Breadboard housing
5. Battery housing

**Side view (left):**
1. Wheel housing
2. Motor housing

*(CAD renders to be added.)*

---

## Project Reframe (Jan 2026)

**The project scope was intentionally refocused.** Rather than solving a full maze, the robot now targets **point-to-point navigation to a location in a room using its onboard sensors** — the priority is getting a reliably *functional* robot first, with more advanced behavior layered on afterward.

**Longer-term goals for the platform:**
- Convert to a **PCB project** — design custom boards for the sensors and motor drivers.
- Implement **PID** control, then **odometry**, for precise navigation.

The underlying engineering process stays the same; the target behavior is just more focused and achievable.

---

## Resources

**General**
- [The Fastest Maze-Solving Competition On Earth](https://www.youtube.com/watch?v=ZMQbHMgK2rw)
- [A Beginner's Guide to Building a Micromouse](https://micromouseguideforbeginners.wordpress.com/1-about/)
- [Micromouse 2022 Lecture Series (playlist)](https://www.youtube.com/watch?v=UHWE3d_au30&list=PLAWsHzw_h0iiPIaGyXAr44G0XfHfyjOe7)

**Hardware**
- [Infrared Sensors Explanation](https://youtube.com/shorts/epcZA5XsS20)
- [How to Choose a Proper Battery — Micromouse USA](http://micromouseusa.com/?p=1002)
- [DC Motor Connection Video](https://youtu.be/Ey4xoG970Go)

**Software**
- [IR Sensor Arduino Tutorial — Complete Guide with Code](https://circuitdigest.com/microcontroller-projects/interfacing-ir-sensor-module-with-arduino)

---

## Team

A small team project. I (**Ryan**) lead the **electronics and programming** — component selection, subsystem bring-up, and firmware — with teammates contributing to parts research and design.

---

*Part of my [portfolio](https://ryanliu-ee.github.io). Questions? [Email me](mailto:ryanliu50@yahoo.com).*
