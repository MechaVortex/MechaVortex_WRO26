# MechaVortex — WRO 2026 Future Engineers

![WRO 2026](https://img.shields.io/badge/WRO-2026-blue) ![Future Engineers](https://img.shields.io/badge/Category-Future%20Engineers-green) ![Arduino](https://img.shields.io/badge/Controller-Arduino%20Uno-teal)

## Team

**Team name:** MechaVortex  
**Vehicle name:** MechaVortex RB  
**Country:** Bosnia and Herzegovina  
**Institution:** Faculty of Mechanical Engineering, Zenica  
**Season:** WRO 2026 — Self-Driving Cars

| Member | Role |
|---|---|
| Ashar Čičak | Team member |
| Amar Đuhera | Team member |
| Bakir Rebihić | Team member |

*Special thanks to our mentor Dženan for guiding and supporting us throughout the season.*

---

## Repository Structure

```
├── src/          # Source code (Arduino sketch)
├── schemes/      # Wiring diagrams and schematics
├── v-photos/     # Vehicle photos (all sides)
├── t-photos/     # Team photos
└── video/        # Links to YouTube driving demonstration videos
```

---

## 1. Mobility and Mechanical Design

### Chassis

The MechaVortex RB is built on a commercial RC car chassis. Rather than designing and manufacturing a custom chassis from scratch, we made a deliberate decision to use an existing RC car platform. As a young and first-time competing team, we prioritized reliability and simplicity over complexity. Using a proven mechanical base allowed us to focus our engineering effort on electronics, sensors, and software — the areas where autonomous driving is actually achieved.

The RC car chassis provides a standard four-wheeled configuration with front-wheel steering and rear-wheel drive, which satisfies the WRO 2026 vehicle regulations (one driving axle and one steering actuator).

### Drive System

The vehicle uses two DC motors that were originally part of the RC car:

- **Drive motor** — connected to the rear axle, responsible for moving the vehicle forward and backward.
- **Steering motor** — controls the front axle direction, turning the vehicle left or right.

This separation of drive and steering functions follows a conventional car-like kinematic model, as required by the rules (not a differential wheeled robot).

### Dimensions and Weight

The vehicle fits within the WRO 2026 size limit of 300×200×300 mm. The exact dimensions are noted in the vehicle photos included in the `v-photos/` folder.

### Design Iterations

During the development process, we went through several iterations:

- **Motor driver / voltage booster** — We replaced the motor voltage booster module multiple times. The initial module did not provide stable output under load, which caused unpredictable motor behavior. After testing different configurations, we selected a module that provides consistent voltage to both motors.
- **Ultrasonic sensors** — Due to physical impacts during testing (the vehicle would occasionally hit walls at speed), our ultrasonic sensors were repeatedly damaged. After analysing the root cause, we determined that the vehicle was turning too late at high speed. By calculating and setting an appropriate turning speed and timing, we eliminated the wall collisions and the sensor damage that came with them.
- **Software parameters** — Throughout development we continuously tuned values including motor speed (PWM), turning duration, and trigger distances. These are documented in the source code comments.

---

## 2. Power and Sensor Architecture

### Power System

The vehicle is powered by a single **9V Duracell battery**. This battery supplies power to both the Arduino Uno and the motors through the motor driver module.

```
9V Duracell Battery
        │
        ├──► Arduino Uno (logic, sensor reading, control)
        │
        └──► Motor Driver Module ──► Drive motor
                                 └──► Steering motor
```

A single power source was chosen for simplicity and to minimise wiring complexity. The Arduino Uno's built-in voltage regulator handles the 5V logic supply internally. The motor driver module receives the full battery voltage and drives the motors accordingly.

### Sensors

The MechaVortex RB uses **two HC-SR04 ultrasonic distance sensors**:

| Sensor | Position | Purpose |
|---|---|---|
| Front sensor | Mounted on the front of the vehicle | Detects obstacles directly ahead |
| Right sensor | Mounted on the right side of the vehicle | Used for lateral navigation and wall tracking |

**Why ultrasonic sensors?**  
As a first-year team new to autonomous robotics, we made a conscious engineering decision to use ultrasonic sensors instead of a camera or colour sensors. Cameras and colour-based detection (for the red/green pillars) require image processing software, calibration, and significantly more development time. We assessed that implementing a camera-based system reliably within our available time and skill level was not feasible. Instead, we chose a minimalist but functional approach: distance-only navigation using two ultrasonic sensors. This decision allowed us to build a vehicle that works consistently, rather than a more complex vehicle that might fail unpredictably.

**Sensor placement rationale:**  
The right-side sensor is the primary navigation sensor. Placing it on the right side allows the vehicle to track the right wall and maintain a consistent driving path around the track. The front sensor acts as a safety mechanism to prevent collisions with obstacles directly ahead.

### Wiring

A full wiring diagram is available in the `schemes/` folder. Connections follow the standard Arduino Uno pinout with the ultrasonic sensors connected to digital I/O pins and the motor driver connected to PWM-capable pins.

---

## 3. Software Architecture and Obstacle Strategy

### Programming Environment

All code is written in **C++ using the Arduino IDE**. The language was chosen because it is the native environment for the Arduino Uno platform, has extensive documentation, and our team had prior familiarity with it.

### File Structure

```
src/
└── mechavortex_rb.ino    # Main Arduino sketch (single file)
```

The code is contained in a single `.ino` file, organised into clearly commented sections: pin definitions, constants, setup, and the main control loop.

### Driving Logic — Open Challenge

The vehicle's navigation is based entirely on distance readings from the two ultrasonic sensors. The logic operates as follows:

```
START
  │
  ├── 2-second delay (preparation time after power-on)
  │
  └── LOOP:
        │
        ├── Read FRONT sensor distance
        ├── Read RIGHT sensor distance
        │
        ├── If RIGHT sensor detects inner wall (parking area wall):
        │       → Drive in LEFT direction (counter-clockwise laps)
        │
        ├── If RIGHT sensor detects NO obstacle at start:
        │       → Drive in RIGHT direction (clockwise laps)
        │
        ├── If FRONT sensor detects obstacle within 20 cm:
        │       → Steer RIGHT to avoid (right sensor guides wall tracking)
        │
        └── Otherwise:
              → Drive straight, maintain course
```

**Clockwise vs counter-clockwise detection:**  
The driving direction is determined at the start of the round. If the right-side sensor detects the inner wall (the parking lot boundary) at startup, the vehicle drives counter-clockwise. If the right sensor detects no obstacle at startup, the vehicle drives clockwise. This allows the vehicle to adapt to the randomly assigned driving direction without any manual input after the start button is pressed.

**Obstacle avoidance:**  
When the front sensor detects an obstacle within 20 cm, the vehicle steers to the right. Since the right-side sensor is always monitoring the right wall, this steering correction is bounded — the vehicle knows where the right wall is and avoids hitting it. This results in the vehicle swerving slightly around obstacles in its path.

Note: The vehicle does not differentiate between red and green traffic pillars. All pillars are treated as generic obstacles to avoid. This is a known limitation of our sensor-only approach and is a planned area of improvement.

### Obstacle Challenge — Parking

Parallel parking at the end of the Obstacle Challenge round is currently under development. The parking logic will be added in a future commit before the competition.

### Key Software Parameters

The following constants in the code can be tuned for different field conditions:

| Parameter | Description |
|---|---|
| `FRONT_THRESHOLD` | Distance (cm) at which front obstacle avoidance triggers |
| `TURN_DURATION` | Duration (ms) of each steering correction |
| `DRIVE_SPEED` | PWM value for the drive motor |
| `STARTUP_DELAY` | Delay (ms) after power-on before the vehicle starts moving |

---

## 4. Systems Thinking and Engineering Decisions

### Subsystem Overview

The MechaVortex RB is built around four interconnected subsystems:

```
[Power]──►[Arduino Uno (Control)]──►[Motor Driver]──►[Drive Motor]
                    ▲                              └──►[Steering Motor]
                    │
          [Front Ultrasonic]
          [Right Ultrasonic]
```

All sensor data flows into the Arduino, which processes it and sends commands to the motor driver in real time.

### Key Engineering Decisions

**Decision 1 — Ultrasonic over camera**  
We explicitly chose not to use a camera. While a camera would allow colour detection of the red and green pillars (enabling full compliance with the Obstacle Challenge rules), the implementation complexity was beyond our current skill level. A working minimalist robot scores more points than a broken complex one. We chose reliability over feature completeness.

**Decision 2 — Single battery**  
We considered using separate batteries for logic and motors to avoid voltage drops affecting the Arduino when motors draw peak current. However, given the low current draw of our motors at the selected speed, a single 9V battery proved sufficient during testing. This simplified the wiring and reduced weight.

**Decision 3 — Speed calibration to prevent sensor damage**  
Early in testing, our ultrasonic sensors were physically damaged by wall impacts. Instead of adding protective covers (which would add weight and complexity), we addressed the root cause: the vehicle was moving too fast to react in time. By reducing speed and recalculating turn timing, we eliminated the collisions entirely. This was a more elegant engineering solution.

**Decision 4 — Right-side navigation sensor**  
Placing the navigation sensor on the right side (rather than left or front) was a deliberate choice. Since the WRO track is a closed loop, the vehicle always has a wall on its right side (when driving clockwise) or left side (when driving counter-clockwise). By tracking one consistent wall, the vehicle can navigate the full track without needing to understand the full field geometry.

### Constraints and Trade-offs

| Constraint | Impact | How we handled it |
|---|---|---|
| Team experience (first season) | Limited time to learn complex systems | Chose simpler sensor approach |
| Budget | Could not buy expensive sensors or SBC | Used Arduino + basic ultrasonic sensors |
| Sensor fragility | Sensors broke during wall impacts | Reduced speed, tuned timing |
| No colour detection | Cannot obey red/green pillar rules | Treat all pillars as generic obstacles |

---

## 5. How to Build and Run the Code

### Requirements

- Arduino IDE (version 1.8.x or 2.x)
- Arduino Uno board
- USB Type-B cable (for uploading)
- No external libraries required — only standard Arduino functions are used

### Upload Instructions

1. Connect the Arduino Uno to a laptop via USB cable.
2. Open Arduino IDE.
3. Open the file `src/mechavortex_rb.ino`.
4. Select **Tools → Board → Arduino Uno**.
5. Select the correct **Tools → Port** (the COM port where Arduino is connected).
6. Click **Upload** (the right-arrow button).
7. Wait for "Done uploading" message.
8. Disconnect the USB cable.
9. Power the vehicle using the 9V battery.
10. The vehicle will wait 2 seconds after power-on, then begin driving autonomously.

### Starting Procedure (Competition)

1. Place the vehicle in the starting zone, switched OFF.
2. Switch the vehicle ON using the power switch.
3. The vehicle enters a 2-second waiting state.
4. On the judge's "Go" signal — no additional button press is needed; the vehicle starts automatically after the delay.

> Note: The vehicle starts automatically after the 2-second delay from power-on. There is no separate start button — the power switch serves as the only interaction point before the round begins.

---

## 6. Photos and Videos

### Vehicle Photos

Vehicle photos (all six sides: front, back, left, right, top, bottom) are available in the `v-photos/` folder.

### Team Photo

Team photo is available in the `t-photos/` folder.

### Driving Demonstration Videos

| Challenge | Link |
|---|---|
| Open Challenge | *(video link — to be added)* |
| Obstacle Challenge | *(video link — to be added)* |

---

## Acknowledgements

We would like to thank our mentor **Dženan** for his guidance, support, and encouragement throughout the 2026 WRO season. We would also like to thank the Faculty of Mechanical Engineering in Zenica for providing us with the resources and space to develop and test our vehicle.

---

*MechaVortex — Faculty of Mechanical Engineering Zenica, Bosnia and Herzegovina — WRO 2026*
