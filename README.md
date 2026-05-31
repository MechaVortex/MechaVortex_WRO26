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
├── src/                        # Source code (Arduino sketch)
│   └── mechavortex_rb.ino
├── schemes/                    # Wiring diagrams and schematics
│   └── MechaVortex_RB_Wiring_Diagram.pdf
├── v-photos/                   # Vehicle photos (all sides)
├── t-photos/                   # Team photos
└── README.md
```

---

## 6. Photos and Videos

### Vehicle Photo

📸 [View all vehicle photos on Google Photos](https://photos.app.goo.gl/RxpdFBdBiHLXKT759)

> Additional vehicle photos (all sides: front, back, left, right, top, bottom) are available in the `v-photos/` folder.

### Team Photo

Team photo is available in the `t-photos/` folder.

### Driving Demonstration Videos

| Challenge | Link |
|---|---|
| Open Challenge | [▶ Watch on YouTube](https://youtube.com/shorts/PWFnQNcEeW4?si=brPiWJP_1IaWiaPk) |
| Obstacle Challenge | *(coming soon — in development)* |

---

## 1. Mobility and Mechanical Design

### Chassis

The MechaVortex RB is built on a commercial RC car chassis. Rather than designing and manufacturing a custom chassis from scratch, we made a deliberate decision to use an existing RC car platform. As a young and first-time competing team, we prioritized reliability and simplicity over complexity. Using a proven mechanical base allowed us to focus our engineering effort on electronics, sensors, and software — the areas where autonomous driving is actually achieved.

The RC car chassis provides a standard four-wheeled configuration with front-wheel steering and rear-wheel drive, which fully satisfies the WRO 2026 vehicle regulations (one driving axle and one steering actuator). The vehicle is not a differential wheeled robot — drive and steering are handled by two independent motors.

### Drive System

The vehicle uses two DC motors that were originally part of the RC car:

- **Drive motor** — connected to the rear axle via the original RC car gearbox, responsible for moving the vehicle forward and backward.
- **Steering motor** — controls the front axle direction, turning the vehicle left or right.

Both motors are controlled via the L298N motor driver module, which allows independent speed (PWM) and direction control for each motor.

### Dimensions and Weight

The vehicle fits within the WRO 2026 size limit of 300×200×300 mm. Exact dimensions are visible in the vehicle photos in the `v-photos/` folder.

### Design Iterations

During development, we went through several important iterations:

- **Motor driver module** — We replaced the motor voltage booster/driver module multiple times. The initial module did not provide stable output under load, causing unpredictable motor behaviour. After testing, we settled on the L298N module which gave us reliable and controllable output for both motors.
- **Ultrasonic sensor damage** — During high-speed testing, our HC-SR04 sensors were repeatedly damaged by wall impacts. Instead of adding protective covers (which would add weight and complexity), we addressed the root cause: the vehicle was turning too late. By reducing the drive speed (`snagaPravo = 90`) and adding a short braking impulse (`zakociKratko()` — 80ms brake after obstacle avoidance), we eliminated wall collisions entirely.
- **Kickstart impulse** — We discovered that at lower PWM values, the motors struggled to overcome static friction and start moving from rest. We added a 50ms full-power kickstart pulse in `setup()` to reliably break inertia at the start of each round.
- **Software parameter tuning** — Throughout development, we continuously adjusted `snagaPravo` (cruise speed), `punaSnaga` (full power for turns), obstacle detection threshold (20 cm), and loop delay (30 ms). These values are documented directly in the source code.

---

## 2. Power and Sensor Architecture

### Power System

The vehicle is powered by a single **9V Duracell battery**. This battery supplies power to both the Arduino Uno and the motors through the L298N motor driver module.

```
9V Duracell Battery
        │
        ├──► Arduino Uno Vin pin (onboard 5V regulator handles logic)
        │
        └──► L298N 12V input ──► Drive motor  (ENA / IN1 / IN2)
                             └──► Steering motor (ENB / IN3 / IN4)
```

A single power source was chosen for simplicity and to minimise wiring complexity. During testing, a single 9V battery proved sufficient for both logic and motor operation at our chosen speed levels. The Arduino's built-in voltage regulator cleanly supplies 5V to the microcontroller and sensors.

### Sensors

The MechaVortex RB uses **two HC-SR04 ultrasonic distance sensors**:

| Sensor | Position on vehicle | Arduino pins | Purpose |
|---|---|---|---|
| Front sensor | Front of vehicle, facing forward | TRIG: D12 / ECHO: D11 | Detects obstacles ahead, triggers avoidance |
| Right sensor | Right side of vehicle, facing sideways | TRIG: D4 / ECHO: D3 | One-time direction detection at startup |

**Why ultrasonic sensors and not a camera?**  
As a first-year team new to autonomous robotics, we made a conscious engineering decision to use ultrasonic sensors instead of a camera or colour sensors. Camera-based colour detection (needed for the red/green pillars in the Obstacle Challenge) requires image processing libraries, lighting calibration, and significantly more software complexity. We assessed that implementing this reliably within our skill level and timeline was not feasible. A working minimalist robot is more valuable than a complex one that fails unpredictably. We chose reliability over feature completeness.

**Sensor placement rationale:**  
- The **front sensor** is placed centrally at the front to maximise detection range directly ahead.
- The **right sensor** is used only during `setup()` — a single reading at startup determines whether the vehicle should drive clockwise or counter-clockwise. After this single reading, the right sensor is no longer polled during the main loop, simplifying the runtime logic significantly.

### Wiring

A full wiring diagram is available in the `schemes/` folder (`MechaVortex_RB_Wiring_Diagram.pdf`).

**Complete pin map:**

| Arduino Pin | Connected to | Type |
|---|---|---|
| D3 | ECHO — right HC-SR04 | Digital input |
| D4 | TRIG — right HC-SR04 | Digital output |
| D5 (PWM) | ENA — L298N (drive motor speed) | PWM output |
| D6 | IN1 — L298N (drive motor direction) | Digital output |
| D7 | IN2 — L298N (drive motor direction) | Digital output |
| D8 | IN3 — L298N (steering motor direction) | Digital output |
| D9 | IN4 — L298N (steering motor direction) | Digital output |
| D10 (PWM) | ENB — L298N (steering motor speed) | PWM output |
| D11 | ECHO — front HC-SR04 | Digital input |
| D12 | TRIG — front HC-SR04 | Digital output |
| Vin | Battery + (9V) | Power input |
| GND | Common ground bus | Ground |

---

## 3. Software Architecture and Obstacle Strategy

### Programming Environment

All code is written in **C++ using the Arduino IDE**. This is the native environment for the Arduino Uno platform, has excellent documentation, and our team had prior experience with it. No external libraries are required — only built-in Arduino functions (`digitalWrite`, `analogWrite`, `pulseIn`, `delay`) are used.

### File Structure

```
src/
└── mechavortex_rb.ino    # Main Arduino sketch
```

The code is structured into clearly separated sections:

```
mechavortex_rb.ino
├── Pin configuration constants
├── Speed and logic variables
├── readSensor()     — HC-SR04 distance measurement
├── kruziLijevo()    — Counter-clockwise lap driving
├── kruziDesno()     — Clockwise lap driving
├── izbjegniDesnoPunGas()  — Full-speed right avoidance
├── zakociKratko()   — 80ms braking impulse after avoidance
├── stani()          — Full stop (all motors off)
├── setup()          — One-time init + direction detection + kickstart
└── loop()           — Main control loop (runs continuously)
```

### State Machine and Driving Logic

The vehicle operates as a simple two-state machine:

```
POWER ON
    │
    ▼
[SETUP — runs once]
    ├── 2 second delay
    ├── Read RIGHT sensor once → determine rezimKruzenja (1=left, 2=right)
    └── 50ms full-power kickstart impulse
    │
    ▼
[LOOP — runs continuously]
    │
    ├── Read FRONT sensor
    │
    ├── IF distance ≤ 20 cm:
    │       → izbjegniDesnoPunGas()   [full speed right swerve]
    │       → bioUSkretanju = true
    │
    └── ELSE (path clear):
            ├── IF bioUSkretanju == true:
            │       → zakociKratko()  [80ms brake to stabilise]
            │       → bioUSkretanju = false
            │
            └── Resume circular driving:
                    ├── rezimKruzenja == 1 → kruziLijevo()
                    └── rezimKruzenja == 2 → kruziDesno()
```

**Direction detection at startup:**  
The right sensor takes a single reading during `setup()`. If it detects a wall closer than 50 cm, the vehicle drives counter-clockwise (`kruziLijevo`). If no wall is detected, it drives clockwise (`kruziDesno`). This single measurement locks the driving direction for the entire round — the right sensor is not read again during `loop()`.

**Obstacle avoidance:**  
When the front sensor detects an obstacle within 20 cm, the vehicle executes a hard right swerve at full power. After the obstacle is cleared, a short braking impulse stabilises the vehicle before resuming circular driving.

**Obstacle Challenge — note:**  
The vehicle does not differentiate between red and green traffic pillars. All obstacles detected by the front sensor are treated identically (swerve right). This is a known limitation of our distance-only approach.

**Parking:**  
Parallel parking logic is currently under development and will be added in a future commit before the competition.

### Key Tunable Parameters

| Variable | Value | Description |
|---|---|---|
| `snagaPravo` | 90 | PWM cruise speed (0–255) for circular driving |
| `punaSnaga` | 255 | Full PWM power used for turns and kickstart |
| `20` (threshold) | 20 cm | Front sensor distance that triggers avoidance |
| `delay(80)` | 80 ms | Braking duration after avoidance manoeuvre |
| `delay(2000)` | 2000 ms | Startup delay before vehicle begins moving |
| `delay(50)` | 50 ms | Kickstart impulse duration |
| `delay(30)` | 30 ms | Main loop cycle time |

---

## 4. Systems Thinking and Engineering Decisions

### Subsystem Overview

```
[9V Battery]
     │
     ├──► [Arduino Uno] ◄── [HC-SR04 Front] (D12/D11)
     │          │         ◄── [HC-SR04 Right] (D4/D3) — setup only
     │          │
     │          └──► [L298N Motor Driver]
     │                      ├──► [DC Drive Motor]  — ENA/IN1/IN2
     └──────────────────────└──► [DC Steering Motor] — ENB/IN3/IN4
```

All sensor data flows into the Arduino Uno, which processes it and sends PWM and direction signals to the L298N. The L298N handles the higher current needed to drive both motors.

### Key Engineering Decisions

**Decision 1 — Ultrasonic over camera**  
We explicitly chose not to use a camera. While a camera would allow colour detection of red and green pillars, the implementation complexity exceeded our current skill level and available development time. We chose reliability over feature completeness — a consistently performing simple robot scores more total points than an unreliable complex one.

**Decision 2 — Single battery**  
We considered separate batteries for logic and motors to prevent voltage sag during peak motor current. However, at our chosen speed (`snagaPravo = 90`, well below maximum), the current draw is moderate and a single 9V Duracell battery proved stable throughout testing. This reduced weight and wiring complexity.

**Decision 3 — Single startup reading for direction**  
Instead of continuously reading the right sensor in every loop cycle, we read it exactly once at startup and lock the driving direction as a variable (`rezimKruzenja`). This simplifies the main loop logic, reduces sensor noise effects, and makes the vehicle's behaviour more predictable and stable during the round.

**Decision 4 — Braking impulse after avoidance**  
After testing, we found that the vehicle would overshoot when returning to circular driving after a full-power swerve. Adding a short 80ms brake (`zakociKratko()`) between avoidance and normal driving stabilised the trajectory significantly.

**Decision 5 — Kickstart impulse**  
At lower PWM values (`snagaPravo = 90`), the motors sometimes failed to overcome static friction and start from rest. A 50ms full-power pulse at startup reliably breaks inertia without affecting the driving speed during the round.

### Constraints and Trade-offs

| Constraint | Impact | How we handled it |
|---|---|---|
| First-year team | Limited development time | Chose simple, proven sensor approach |
| Budget | No camera or SBC available | Arduino + HC-SR04 only |
| Sensor fragility | Sensors broke on wall impact | Reduced speed, added braking logic |
| No colour detection | Cannot obey red/green pillar rules | All pillars treated as generic obstacles |
| Single battery | Possible voltage sag under load | Kept motor speed moderate (PWM 90/255) |

---

## 5. How to Build and Run the Code

### Requirements

- Arduino IDE (version 1.8.x or 2.x) — [download here](https://www.arduino.cc/en/software)
- Arduino Uno board
- USB Type-B cable (for uploading)
- **No external libraries required** — only standard Arduino built-in functions are used

### Upload Instructions

1. Connect the Arduino Uno to a laptop via USB Type-B cable.
2. Open Arduino IDE.
3. Open `src/mechavortex_rb.ino`.
4. Go to **Tools → Board → Arduino AVR Boards → Arduino Uno**.
5. Go to **Tools → Port** and select the correct COM port.
6. Click the **Upload** button (→).
7. Wait for `Done uploading.` message in the status bar.
8. Disconnect USB cable.
9. Connect the 9V battery to power the vehicle.

### Starting Procedure (Competition Day)

1. Place the vehicle in the starting zone, fully **switched OFF**.
2. Switch the vehicle **ON** using the power switch (battery connection).
3. The vehicle will automatically wait **2 seconds** (startup delay).
4. During the 2-second delay, the right sensor takes one reading to determine driving direction.
5. After the delay, the vehicle executes a 50ms kickstart pulse and begins driving autonomously.

> ⚠️ **Important for judges:** The vehicle starts automatically after power-on with a fixed 2-second delay. The power switch acts as the start trigger. No additional button press is required after power-on.

---

## Acknowledgements

We would like to thank our mentor **Dženan** for his guidance, support, and encouragement throughout the 2026 WRO season. We would also like to thank the Faculty of Mechanical Engineering in Zenica for providing us with the resources and space to develop and test our vehicle.

---

*MechaVortex — Faculty of Mechanical Engineering Zenica, Bosnia and Herzegovina — WRO 2026*
