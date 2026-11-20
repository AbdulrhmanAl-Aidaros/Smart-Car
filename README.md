# 🚗 Smart Robot Car — Arduino Obstacle Avoidance & Bluetooth Control

A 4-wheel smart car built on **Arduino UNO** that drives itself, **avoids obstacles** using an ultrasonic sensor mounted on a servo, and can be **remote-controlled over Bluetooth** from a phone.

![Arduino](https://img.shields.io/badge/Arduino-UNO-00979D?logo=arduino&logoColor=white)
![Language](https://img.shields.io/badge/language-C%2B%2B%20(Arduino)-blue)
![Control](https://img.shields.io/badge/control-Bluetooth%20HC--05%2FHC--06-informational)
![Status](https://img.shields.io/badge/status-completed-success)

---

## 📌 Overview (the simple version)

The car does two things:

1. **Sees obstacles.** An ultrasonic sensor measures the distance in front of the car. If something is closer than **12 cm**, the car stops, backs up a little, looks left and right with the servo, and turns toward the side with more free space.
2. **Obeys your phone.** A Bluetooth module receives single-letter commands (`F`, `B`, `L`, `R`, `S`) from a phone app and moves the car accordingly.

It was built as a university project to apply what we studied in our lectures: wiring electronic devices, motor control, sensors, and embedded programming.

### Why this project?
- Simple, low-cost components.
- A simple but creative idea.
- It touches many of the subjects we studied.

### Goals
- A smart car controlled by Bluetooth.
- A car that can overcome obstacles on its own.
- A handmade frame built from simple materials.
- Learning how to connect the Arduino, ultrasonic sensor, servo motor, and DC motors together.

---

## 🧰 Components

| Component | Role in the project |
|---|---|
| **Arduino UNO** | The brain that controls all devices |
| **L293D Motor Driver Shield (AFMotor)** | Drives the 4 DC motors and the servo |
| **4 × DC gear motors + wheels** | Move the car |
| **HC-SR04 Ultrasonic sensor** | Measures the distance to obstacles |
| **SG90 Servo motor** | Rotates the ultrasonic sensor left/right |
| **Bluetooth module (HC-05 / HC-06)** | Wireless link between the phone and the car |
| **2 × 18650 Li-ion batteries (3.7 V) + dual holder** | Power supply (≈ 7.4 V) |
| **Switch, jumper wires, foam-board chassis** | Power switching, connections, and the car body |

---

## 🔌 Wiring

![Circuit diagram](images/circuit_diagram.png)

> Circuit diagram by **SriTu Hobby** (from the tutorial this build is based on).

**Pin summary (as defined in the code):**

| Signal | Pin |
|---|---|
| Ultrasonic **Echo** | `A0` |
| Ultrasonic **Trig** | `A1` |
| Servo signal | `10` (SERVO header on the shield) |
| Motors M1–M4 | M1–M4 terminals on the L293D shield |
| Bluetooth | Hardware serial (`RX`/`TX`) at **9600 baud** |

> ⚠️ Disconnect the Bluetooth module's TX/RX wires while uploading code, since they share the USB serial port.

---

## 🧠 How it works (the deeper version)

### Main loop
```cpp
void loop() {
  Obstacle();          // read distance, avoid obstacles
  Bluetoothcontrol();  // read phone command and act on it
  //voicecontrol();    // optional voice-control mode (disabled)
}
```

### Obstacle avoidance — `Obstacle()`
1. `ultrasonic()` triggers the HC-SR04 and converts the echo time to centimetres (`t / 29 / 2`).
2. If `distance <= 12` cm:
   - Stop → back up for 100 ms → stop.
   - `leftsee()` turns the servo to 180° and reads the distance; `rightsee()` turns it to 20° and reads again; the servo then returns to the centre (`spoint = 103`).
   - If **left < right** → turn right 500 ms; if **left > right** → turn left 500 ms; then stop 200 ms.
3. Otherwise the car moves `forward()`.

### Bluetooth control — `Bluetoothcontrol()`

| Command | Action |
|:---:|---|
| `F` | Forward |
| `B` | Backward |
| `L` | Turn left |
| `R` | Turn right |
| `S` | Stop |

### Voice control — `voicecontrol()` *(disabled by default)*
Uncomment `voicecontrol();` in `loop()` to use it with a voice-control app that sends these characters:

| Char | Action |
|:---:|---|
| `^` | Forward |
| `-` | Backward |
| `<` | Look left; turn left if free space ≥ 10 cm, otherwise stop |
| `>` | Look right; turn right if free space ≥ 10 cm, otherwise stop |
| `*` | Stop |

### Tunable parameters

| Constant | Default | Meaning |
|---|:---:|---|
| `Speed` | `170` | Motor speed (0–255) |
| `spoint` | `103` | Servo centre angle (adjust so the sensor faces straight ahead) |
| Obstacle threshold | `12` | Distance in cm that triggers avoidance (inside `Obstacle()`) |

### Notes on the current behavior
- `Obstacle()` calls `forward()` on every loop when the path is clear, so the car keeps driving forward on its own. Bluetooth commands are applied in the same loop, but the next obstacle check can override them. If you want pure remote-control with obstacle protection only, call `forward()` from the `Obstacle()` branch only when no manual command is active.
- The code uses `pulseIn()` without a timeout; adding one (e.g. `pulseIn(Echo, HIGH, 30000)`) avoids long blocking if no echo returns.

---

## 📁 Repository structure

```
.
├── README.md
├── Ropot_._Car.ino                         # Arduino sketch (main source code)
├── hex/                                    # Pre-compiled firmware
│   ├── Ropot_._Car.ino.standard.hex        #   Arduino UNO
│   ├── Ropot_._Car.ino.with_bootloader.standard.hex
│   ├── Ropot_._Car.ino.mega.hex            #   Arduino Mega
│   └── Ropot_._Car.ino.with_bootloader.mega.hex
├── docs/
│   ├── Project Ropot . Car.pdf             # Build guide: parts list, wiring, code explanation
│   └── Presentation.pptx                   # Project presentation (+ .ppsx slideshow)
└── images/
    ├── circuit_diagram.png
    └── arduino_ide.png
```

> Tip: rename `Ropot_._Car.ino` to match its folder name (e.g. `Robot_Car/Robot_Car.ino`), because the Arduino IDE requires the sketch and folder names to match and dislikes dots in names.

---

## 🚀 Getting started

### 1. Requirements
- [Arduino IDE](https://www.arduino.cc/en/software) 2.x
- Libraries (Library Manager):
  - **Adafruit Motor Shield library (V1)** — provides `AFMotor.h`
  - **Servo** (built-in)

### 2. Upload
1. Open `Ropot_._Car.ino` in the Arduino IDE.
2. Select **Board → Arduino Uno** and the correct **Port**.
3. Disconnect the Bluetooth module, click **Upload**, then reconnect it.

*(Or flash the ready-made `.standard.hex` file with a tool such as `avrdude` / XLoader.)*

### 3. Control from your phone
1. Pair the phone with the Bluetooth module (default PIN is usually `1234` or `0000`).
2. Open a Bluetooth serial / RC-car app and map the buttons to `F`, `B`, `L`, `R`, `S`.
3. Power the car with the switch and drive.

---

## 🖼️ Screenshot

![Arduino IDE](images/arduino_ide.png)

---

## 👥 Team

**Prepared by:**
- Bashar Al-Waseai
- Saqer Al-Kholani
- Abdullah AL-Materi
- Abdulrhman Al-idros
- Aymen Al-Saidi
- Haitham Al-Qadsi

**Supervisor:** Eng. Alaa Aldeen Al-Mqtari

---

## 🙏 Credits & References

- Circuit diagram and build guide based on the **"How to make a multi-function Arduino robot car"** tutorial by **SriTu Hobby**.
- Adafruit Motor Shield library (`AFMotor`).

---

## 📄 License

Add a license of your choice (e.g. [MIT](https://choosealicense.com/licenses/mit/)) by creating a `LICENSE` file in the repository root.
