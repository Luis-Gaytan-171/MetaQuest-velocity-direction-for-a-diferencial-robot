# Meta Quest Cooperative Robot Control 🚗🥽

This project is a cooperative control system for a two-wheel robot car using **two Meta Quest headsets**, **Unity**, and an **ESP32**.

The objective of the game is to move the robot car from **Point A** to **Point B** through a physical track with obstacles.

Each player controls a different motion variable:

- **Player 1 / Meta Quest 1:** controls the linear velocity `v`.
- **Player 2 / Meta Quest 2:** controls the angular velocity `ω`.
- **Unity:** receives, processes, and displays both references.
- **ESP32:** receives the motion references and controls the motors.
- **Robot car:** uses two independent motors in a differential-drive configuration.

---

# 1. General Architecture

```text
             META QUEST 1
               Player 1
           Linear Velocity
                  │
                  │ v
                  ▼
        ┌───────────────────┐
        │                   │
        │       UNITY       │
        │                   │
        │   v       ω       │
        │   │       │       │
        │   └───┬───┘       │
        │       │           │
        │   UI / Route      │
        │ Communication     │
        └────────┬──────────┘
                 │
                 │ v, ω
                 ▼
               ESP32
                 │
          ┌──────┴──────┐
          │             │
          ▼             ▼
      Left Motor     Right Motor


             META QUEST 2
               Player 2
          Angular Velocity
                  │
                  │ ω
                  └──────────────► UNITY
```

---

# 2. Motion Logic

The robot will use a **differential-drive system**.

The two main control inputs are:

```text
v = desired linear velocity
ω = desired angular velocity
```

Examples:

```text
v > 0, ω = 0
→ move forward

v < 0, ω = 0
→ move backward

v > 0, ω > 0
→ move forward while turning

v > 0, ω < 0
→ move forward while turning in the opposite direction

v = 0, ω ≠ 0
→ rotate in place
```

The individual wheel references can later be calculated as:

```text
vLeft  = v - (ω * L / 2)
vRight = v + (ω * L / 2)
```

where:

```text
L = distance between both wheels
```

---

# 3. First Development Stage

For now, the project does **not depend on having the Meta Quest-to-ESP32 communication ready**.

Each module should first be developed independently.

We can temporarily simulate:

```text
Meta Quest 1 → variable v

Meta Quest 2 → variable ω
```

and verify that Unity responds correctly.

Later, the simulated references will be replaced with real Meta Quest inputs.

---

# 4. Team Organization

The project will be divided between three team members.

## Team Member 1 — Unity / Integration

Main responsibilities:

```text
Unity
Meta Quest
User Interface
Final Integration
```

### Tasks

- Create the main Unity project.
- Create a test scene.
- Configure Meta Quest support.
- Create Player 1 input.
- Create Player 2 input.
- Create `linearVelocity`.
- Create `angularVelocity`.
- Create temporary input controls.
- Create a virtual robot simulation.
- Create the user interface.
- Display Point A.
- Display Point B.
- Display the recommended route.
- Display linear velocity.
- Display angular velocity.
- Create the communication module.
- Integrate the ESP32 later.

---

# 5. Team Member 2 — ESP32 / Motor Control

This person can work independently even before the Meta Quest connection is available.

## Objective

Create the firmware responsible for receiving:

```text
v
ω
```

and converting those values into motor commands.

## Tasks

### 5.1 Create ESP32 Structure

Create:

```text
/ESP32
```

with:

```text
ESP32/
│
├── src/
│   ├── main.cpp
│   ├── MotorControl.cpp
│   ├── MotorControl.h
│   ├── DifferentialDrive.cpp
│   └── DifferentialDrive.h
│
├── tests/
│   └── motor_test.cpp
│
└── README.md
```

---

### 5.2 Create Motor Control Class

Create:

```text
MotorControl
```

Suggested functions:

```cpp
setSpeed(float speed);
stop();
forward();
backward();
```

For the first version, use normalized values:

```text
-1.0 = maximum reverse speed
 0.0 = stopped
+1.0 = maximum forward speed
```

---

### 5.3 Create Differential Drive Logic

Input:

```cpp
float linearVelocity;
float angularVelocity;
```

Output:

```cpp
float leftMotor;
float rightMotor;
```

Suggested functions:

```cpp
void setVelocity(float linear, float angular);

float getLeftMotorReference();

float getRightMotorReference();
```

---

### 5.4 Test Without Unity

Test values manually.

Example:

```text
v = 1
ω = 0
```

Expected result:

```text
Left Motor  = forward
Right Motor = forward
```

Example:

```text
v = 0
ω = 1
```

Expected result:

```text
Left Motor  = reverse
Right Motor = forward
```

Example:

```text
v = 0.5
ω = 0.3
```

Expected result:

```text
The robot moves forward while turning.
```

---

# 6. Team Member 3 — Communication / Testing / Protocol

This person will develop the layer used for communication between **Unity and ESP32**.

Even if the final wireless method has not been selected yet, the communication protocol can be defined now.

Create:

```text
/Communication
```

Suggested structure:

```text
Communication/
│
├── protocol/
│   ├── protocol.md
│   └── examples.md
│
├── simulator/
│   ├── command_generator.py
│   └── esp32_simulator.py
│
└── tests/
    └── communication_tests.md
```

---

# 7. Communication Protocol

Unity and ESP32 must use the same message format.

A first version could use JSON:

```json
{
  "linearVelocity": 0.6,
  "angularVelocity": -0.2
}
```

A more compact format could also be used:

```text
V:0.60,W:-0.20
```

For early development, JSON is recommended because it is easier to read and debug.

---

# 8. Standard Input Ranges

The entire project should initially use normalized values.

## Linear Velocity

```text
-1 ≤ v ≤ 1
```

Meaning:

```text
-1 = maximum reverse speed
 0 = stopped
+1 = maximum forward speed
```

## Angular Velocity

```text
-1 ≤ ω ≤ 1
```

Meaning:

```text
-1 = maximum turn in one direction
 0 = no rotation
+1 = maximum turn in the opposite direction
```

The sign convention can be adjusted later depending on motor orientation.

---

# 9. Temporary Input Simulator

Before using the Meta Quest headsets, Unity should allow manual control of:

```text
Linear Velocity
Angular Velocity
```

using:

```text
sliders

or

keyboard input
```

Example:

```text
W / S → linear velocity

A / D → angular velocity
```

This allows most of the software architecture to be developed before connecting the real hardware.

---

# 10. Unity Scripts

Create:

```text
UnityProject/
└── Assets/
    └── Scripts/
```

Suggested structure:

```text
Scripts/
│
├── Input/
│   ├── LinearVelocityInput.cs
│   ├── AngularVelocityInput.cs
│   └── InputManager.cs
│
├── Vehicle/
│   ├── VehicleController.cs
│   └── DifferentialDrive.cs
│
├── Communication/
│   ├── ESP32Communication.cs
│   └── RobotCommand.cs
│
├── UI/
│   ├── VelocityUI.cs
│   ├── RouteUI.cs
│   └── GameManager.cs
│
├── MetaQuest/
│   ├── Player1Input.cs
│   └── Player2Input.cs
│
└── Debug/
    └── RobotDebug.cs
```

---

# 11. Script Responsibilities

## LinearVelocityInput.cs

Responsible for:

```text
v
```

Initially:

```text
keyboard / slider
```

Later:

```text
Meta Quest 1
```

---

## AngularVelocityInput.cs

Responsible for:

```text
ω
```

Initially:

```text
keyboard / slider
```

Later:

```text
Meta Quest 2
```

---

## InputManager.cs

Combines:

```text
Player 1 → v
Player 2 → ω
```

and generates:

```text
RobotCommand
```

---

## RobotCommand.cs

Represents one robot command.

Example:

```csharp
public class RobotCommand
{
    public float linearVelocity;
    public float angularVelocity;
}
```

---

## DifferentialDrive.cs

Converts:

```text
v + ω
```

into:

```text
Left Motor
Right Motor
```

This also allows the robot behavior to be tested inside Unity without using the physical robot.

---

## ESP32Communication.cs

This should be the only Unity class responsible for sending commands to the hardware.

The rest of the Unity project should not need to know whether communication is done using:

```text
Bluetooth
Wi-Fi
UDP
TCP
Serial
```

This allows the communication technology to change without rewriting the whole project.

---

# 12. Meta Quest Interfaces

The project will use two different POVs.

## Player 1 POV

Main control:

```text
LINEAR VELOCITY
```

Example UI:

```text
SPEED

████████░░

0.72
```

Additional information:

```text
Point A
Point B
Recommended Route
Progress
```

---

## Player 2 POV

Main control:

```text
TURN / ANGULAR VELOCITY
```

Example UI:

```text
TURN

LEFT ←────●────→ RIGHT
```

Additional information:

```text
Point A
Point B
Recommended Route
Progress
```

---

# 13. Safety Controller

A safety system should be added later.

Create:

```text
SafetyController
```

The robot should stop if:

```text
Communication is lost.

No new commands are received.

Emergency Stop is pressed.

Invalid commands are received.

Maximum speed is exceeded.
```

Example behavior:

```text
If Unity stops sending commands for 500 ms:

v = 0
ω = 0

STOP
```

---

# 14. Complete Repository Structure

```text
MetaQuest-Robot-Control/
│
├── README.md
│
├── docs/
│   ├── architecture.md
│   ├── controls.md
│   ├── communication.md
│   ├── hardware.md
│   └── testing.md
│
├── UnityProject/
│   └── Assets/
│       └── Scripts/
│           ├── Input/
│           ├── Vehicle/
│           ├── Communication/
│           ├── MetaQuest/
│           ├── UI/
│           └── Debug/
│
├── ESP32/
│   ├── src/
│   ├── tests/
│   └── README.md
│
├── Communication/
│   ├── protocol/
│   ├── simulator/
│   └── tests/
│
├── Hardware/
│   ├── wiring/
│   ├── datasheets/
│   └── CAD/
│
└── Tests/
    ├── motor_tests/
    ├── communication_tests/
    └── integration_tests/
```

---

# 15. Git Workflow

Do not work directly on:

```text
main
```

The `main` branch should contain only stable and working versions.

Suggested development branches:

```text
main
│
├── unity-dev
│
├── esp32-dev
│
└── communication-dev
```

For specific tasks:

```text
feature/meta-input
feature/linear-velocity
feature/angular-velocity

feature/motor-control
feature/differential-drive

feature/esp32-communication
feature/command-protocol

feature/route-ui
feature/player1-ui
feature/player2-ui
```

---

# 16. Commit Rules

Avoid commit messages such as:

```text
changes

test

new stuff

final final 2
```

Use descriptive messages:

```text
feat: add differential drive controller

feat: add linear velocity input

feat: add angular velocity input

feat: add ESP32 command parser

fix: correct left motor direction

test: add motor control test

docs: add communication protocol
```

---

# 17. Development Workflow

## PHASE 1 — Repository Setup

- [ ] Create GitHub repository.
- [ ] Add README.
- [ ] Create folder structure.
- [ ] Add Unity `.gitignore`.
- [ ] Create development branches.
- [ ] Assign responsibilities.

---

## PHASE 2 — Control Variables

- [ ] Define `linearVelocity`.
- [ ] Define `angularVelocity`.
- [ ] Define ranges.
- [ ] Define units.
- [ ] Define update frequency.
- [ ] Create `RobotCommand`.

---

## PHASE 3 — Unity Without Hardware

- [ ] Create test scene.
- [ ] Create virtual robot.
- [ ] Implement differential-drive motion.
- [ ] Control `v` using keyboard.
- [ ] Control `ω` using keyboard.
- [ ] Display variables in UI.
- [ ] Create Point A.
- [ ] Create Point B.
- [ ] Create route visualization.

---

## PHASE 4 — ESP32

- [ ] Control Left Motor.
- [ ] Control Right Motor.
- [ ] Implement differential drive.
- [ ] Receive `v`.
- [ ] Receive `ω`.
- [ ] Limit commands.
- [ ] Add Emergency Stop.
- [ ] Test robot without Meta Quest.

---

## PHASE 5 — Communication

- [ ] Define protocol.
- [ ] Create simulator.
- [ ] Send data from PC.
- [ ] Receive data on ESP32.
- [ ] Test latency.
- [ ] Detect lost connection.

---

## PHASE 6 — Meta Quest 1

- [ ] Create Player 1 input.
- [ ] Read controller or hand movement.
- [ ] Convert input into `v`.
- [ ] Clamp value between `-1` and `1`.
- [ ] Display speed in POV.

---

## PHASE 7 — Meta Quest 2

- [ ] Create Player 2 input.
- [ ] Read controller or hand movement.
- [ ] Convert input into `ω`.
- [ ] Clamp value between `-1` and `1`.
- [ ] Display turning reference in POV.

---

## PHASE 8 — Integration

```text
Quest 1 ──► v ───┐
                 │
                 ▼
               UNITY ──► ESP32 ──► ROBOT
                 ▲
                 │
Quest 2 ──► ω ───┘
```

- [ ] Quest 1 modifies `v`.
- [ ] Quest 2 modifies `ω`.
- [ ] Unity receives both references.
- [ ] Unity displays both references.
- [ ] Unity sends a robot command.
- [ ] ESP32 receives the command.
- [ ] Motors respond correctly.
- [ ] Test real movement.

---

# 18. Final Tests

Perform incremental tests.

### Test 1

```text
v = 0
ω = 0

Expected:
Robot remains stopped.
```

### Test 2

```text
v = 0.5
ω = 0

Expected:
Robot moves straight forward.
```

### Test 3

```text
v = -0.5
ω = 0

Expected:
Robot moves backward.
```

### Test 4

```text
v = 0
ω = 0.5

Expected:
Robot rotates.
```

### Test 5

```text
v = 0.5
ω = 0.3

Expected:
Robot follows a curved trajectory.
```

### Test 6

```text
Quest 1 controls v.
Quest 2 controls ω.

Expected:
The robot can only be driven successfully through cooperation.
```

---

# 19. Final Objective

The complete system should work as follows:

```text
Player 1
   │
   └──► controls LINEAR VELOCITY
                     │
                     ▼
                   UNITY
                     ▲
                     │
   ┌──► controls ANGULAR VELOCITY
   │
Player 2

                     │
                     ▼
                   ESP32
                     │
                     ▼
           Left Motor + Right Motor
                     │
                     ▼
                  ROBOT
                     │
                     ▼
              POINT A → POINT B
```

The key mechanic of the project is cooperation:

**one player controls how fast the robot moves, while the other controls where it turns.**
