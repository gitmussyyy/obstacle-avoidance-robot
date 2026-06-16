# Obstacle Avoidance Robot

Arduino-powered autonomous robot with servo-scanning ultrasonic detection and intelligent path selection. Built from scratch during a college robotics internship.

---

## 📖 Overview

Most basic obstacle-avoidance robots stop and blindly turn in one fixed direction. This robot does it smarter.

An **HC-SR04 ultrasonic sensor** is mounted on an **SG90 servo motor** that sweeps left and right. When an obstacle is detected, the robot stops, actively scans both sides, measures free space, and turns toward the **clearer path**. This makes navigation noticeably smoother and less prone to getting cornered.

---

## 🔗 GitHub Repository

> obstacle-avoidance-robot — github.com/YOUR_USERNAME
> 

---

## ✨ What Makes This Different

| Feature | Basic Obstacle Bot | This Robot |
| --- | --- | --- |
| Detection | Fixed forward only | Servo scans left + right |
| Turn decision | Always same direction | Picks side with more space |
| Sensor library | NewPing (abstracted) | Raw pulseIn() — full control |
| Code structure | Single loop blob | Modular functions per behavior |

---

## 🧰 Hardware

| Component | Model | Purpose |
| --- | --- | --- |
| Microcontroller | Arduino UNO / Nano | Main brain |
| Ultrasonic Sensor | HC-SR04 | Distance measurement |
| Servo Motor | SG90 9g | Rotates sensor left/right |
| Motor Driver | L298N or L293D Shield | Controls motor speed & direction |
| Drive Motors | BO DC Gear Motors (×2) | Differential drive |
| Caster Wheel | Front ball caster | Balance & smooth turning |
| Chassis | 2WD Robot Car Kit | Physical frame |
| Power | 9V Battery or 7.4V LiPo | Main supply |
| Switch | SPST toggle | ON/OFF control |
| Hardware | Nuts, bolts, spacers | Mechanical assembly |

---

## 🔊 How the Sensor Works

The HC-SR04 works by timing a sound pulse:

1. Arduino sends a 10µs HIGH pulse to TRIG pin
2. Sensor fires 8 ultrasonic bursts at 40kHz
3. Sound wave travels forward, hits object, bounces back
4. ECHO pin goes HIGH for the duration of the round-trip
5. Arduino reads pulse duration via pulseIn()
6. Distance (cm) = duration × 0.034 / 2

> Speed of sound ≈ 343 m/s = 0.034 cm/µs. Divide by 2 because the pulse travels **to** the obstacle and **back**.
> 

---

## 🧠 Decision Logic

```cpp
[Read distance straight ahead]
        │
  dist < 20cm?
   /         \
  NO          YES
  │            │
[Move Forward]  [Stop Motors]
  │            │
(repeat)  [Servo → 150°] ← look LEFT
          [Read leftDist]
               │
          [Servo → 30°]  ← look RIGHT
          [Read rightDist]
               │
          [Servo → 90°]  ← re-center
               │
      leftDist > rightDist?
        /              \
      YES               NO
       │                 │
  [Turn Left]       [Turn Right]
        \               /
         └──[Move Forward]──► (repeat)
```

---

## 💻 Core Code (Arduino C++)

```cpp
#include <Servo.h>

#define TRIG_PIN  9
#define ECHO_PIN  10
#define SERVO_PIN 11
#define SAFE_DIST 20    // cm

Servo scanServo;

long getDistance() {
  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);
  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);
  long duration = pulseIn(ECHO_PIN, HIGH, 30000);
  return (duration == 0) ? 999 : duration * 0.034 / 2;
}

long scanAt(int angle) {
  scanServo.write(angle);
  delay(350);
  return getDistance();
}

void loop() {
  long dist = getDistance();
  if (dist > SAFE_DIST) {
    moveForward();
  } else {
    stopMotors();
    long leftDist  = scanAt(150);
    long rightDist = scanAt(30);
    scanAt(90);
    leftDist > rightDist ? turnLeft(450) : turnRight(450);
  }
}
```

---

## 🔌 Wiring

### HC-SR04 → Arduino

| Sensor Pin | Arduino Pin | Notes |
| --- | --- | --- |
| VCC | 5V | Do NOT use 3.3V |
| GND | GND | Common ground |
| TRIG | D9 | Output from Arduino |
| ECHO | D10 | Input to Arduino |

### SG90 Servo → Arduino

| Servo Wire | Arduino Pin | Notes |
| --- | --- | --- |
| Signal (Orange) | D11 | PWM pin required |
| VCC (Red) | 5V | Share with sensor |
| GND (Brown) | GND | Common ground |

### L298N Motor Driver → Arduino

| Driver Pin | Arduino Pin / Source |
| --- | --- |
| IN1 | D4 |
| IN2 | D5 |
| IN3 | D6 |
| IN4 | D7 |
| ENA | D3 (PWM) |
| ENB | D8 |
| Vin | 9V Battery (+) |
| GND | Battery (–) + Arduino GND |

---

## 🚀 Getting Started

1. Clone the repo: `git clone https://github.com/YOUR_USERNAME/obstacle-avoidance-robot.git`
2. Open `src/obstacle_avoidance/obstacle_avoidance.ino` in Arduino IDE
3. Select **Board → Arduino UNO** and correct **COM Port**
4. Click **Upload ✓**
5. Disconnect USB, insert 9V battery, flip switch — done!

> No external libraries needed. `Servo.h` is built into Arduino IDE.
> 

---

## 🌍 Real-World Applications

- 🚗 **Self-driving cars** — radar/lidar-based obstacle detection and path planning
- 🏭 **Warehouse robots** — Amazon Kiva robots navigating around shelves and people
- 🏠 **Home cleaning robots** — Roomba's bump-and-scan navigation
- 🛸 **Mars rovers** — Perseverance uses stereo cameras for autonomous hazard detection
- ♿ **Smart wheelchairs** — obstacle detection to assist mobility

---

## 🛠️ Future Improvements

- [ ]  IR sensors for cliff / edge detection
- [ ]  PID-controlled turning for smoother arcs
- [ ]  Bluetooth module for live sensor data on phone
- [ ]  OLED display showing distance readings in real time
- [ ]  Upgrade to ROS2-based navigation stack
- [ ]  Battery voltage monitor with LED indicator

---

## 👤 Author

**Moosa Mubasir** 

> Collaborative college robotics  project — learning embedded systems, sensor integration, and autonomous logic from the ground up.
> 

---

## 📄 License

MIT License — free to use, fork, and build on.
