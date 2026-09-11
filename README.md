# AutoVisionGuardian
### AI-Powered Automatic Sensor Surface Cleaning & Defogging System

![Status](https://img.shields.io/badge/status-in--development-orange)
![Domain](https://img.shields.io/badge/domain-Hardware-red)
![Event](https://img.shields.io/badge/YHACK-26-black)

---

## Team Details

| Field | Detail |
|---|---|
| **Team Name** | Mechaminds |
| **Team ID** | YH403 |
| **PS ID** | Challenge 7 → *When Vision Fails* |
| **Domain** | Hardware |
| **Team Leader** | Harish Chandran |
| **Member 1** | Harish D |
| **Member 2** | Kabilesh S |
| **Member 3** | Nenesh Raj T |

---

## Table of Contents

1. [Problem Statement](#problem-statement)
2. [Proposed Solution](#proposed-solution)
3. [Features](#features)
4. [Tech Stack](#tech-stack)
5. [System Architecture](#system-architecture)
6. [Workflow](#workflow)
7. [Folder Structure](#folder-structure)
8. [Installation & Usage Guide](#installation--usage-guide)
9. [AI/ML Workflow](#aiml-workflow)
10. [Hardware Components & Wiring](#hardware-components--wiring)
11. [Security Measures](#security-measures)
12. [Testing & Performance](#testing--performance)
13. [Challenges Faced & Future Scope](#challenges-faced--future-scope)
14. [Demo](#demo)
15. [References](#references)

---

## Problem Statement

Outdoor autonomous robots rely on cameras and LiDAR sensors for reliable perception and navigation. During outdoor operation, **dust, rain/water, and fog** progressively accumulate on the sensor's protective surface, degrading visibility and sensing quality.

When this contamination is not detected and cleared automatically, the degraded sensor can lead to **unreliable perception** and compromise safe robot operation.

**The Challenge:** Develop a compact and reliable automatic sensor-cleaning and defogging mechanism that can:

- Detect dust, water/rain, and fog contamination on the sensor surface
- Automatically activate the appropriate cleaning or defogging mechanism
- Restore clear sensor visibility with minimal delay
- Minimize interruption to normal robot operation
- Avoid interfering with normal camera/LiDAR sensing
- Operate reliably in outdoor environments

**Core Problem Chain:**

```
Environmental Contamination → Sensor Visibility Degradation → Unreliable Perception → Need for Automatic Recovery
```

---

## Proposed Solution

**AutoVision Guardian AI** — an AI-assisted system that detects contamination on an outdoor robot's camera surface and automatically activates the required cleaning mechanism, then verifies the result before resuming normal operation.

### How It Works

1. **Capture** — Camera captures the sensor surface.
2. **AI Detection** — AI classifies the surface as `CLEAN / DUST / WATER / FOG`.
3. **Decide** — Decision engine selects the required cleaning action.
4. **Clean**
   - `DUST` → Blower
   - `WATER` → Wiper
   - `FOG` → Blower / Defogger
   - `CLEAN` → No action
5. **Verify** — Camera re-checks the surface after cleaning.
6. **Recover** — If clear, resume normal operation; if not, repeat the cleaning cycle.

---

## Features

- Real-time contamination detection (Clean / Dust / Water / Fog) using an on-device AI model
- Automatic, condition-based activation of the correct cleaning mechanism (no manual intervention)
- Closed-loop verification — the system checks its own work and retries if visibility isn't restored
- Lightweight, edge-deployed inference (TensorFlow Lite on Raspberry Pi) for low-latency response
- Confidence-based 3-frame confirmation to prevent false-positive activations
- Modular actuator control (blower, wiper, defogger/heater) via ESP32
- Minimal interruption to the robot's primary sensing and navigation tasks

---

## Tech Stack

| Layer | Technology |
|---|---|
| **AI / ML Model** | MobileNetV3Small (image classification), TensorFlow / TensorFlow Lite |
| **Edge Compute** | Raspberry Pi 4B (2GB) |
| **Microcontroller** | ESP32 |
| **Sensing** | Raspberry Pi Camera Module |
| **Actuation** | Blower motor, wiper mechanism, defogger/heater element |
| **Firmware** | Arduino / ESP-IDF (ESP32 control code) |
| **Languages** | Python (AI/edge inference), C/C++ (ESP32 firmware) |
| **Communication** | Serial / UART or I²C between Raspberry Pi and ESP32 *(confirm and update based on final implementation)* |

> *Update this table with exact library versions (e.g., `tensorflow==x.x`, `opencv-python==x.x`) once your `requirements.txt` is finalized.*

---

## System Architecture

```
┌─────────────────────────┐
│  1. SENSING LAYER        │
│  Raspberry Pi Camera     │
└────────────┬─────────────┘
             ↓
┌─────────────────────────────────────┐
│  2. PROCESSING & AI LAYER            │
│  Raspberry Pi 4B                     │
│  • Data acquisition                  │
│  • Image preprocessing               │
│  • Sensor-quality analysis           │
│  • AI contamination detection        │
│    → CLEAN / DUST / WATER / FOG      │
└────────────┬──────────────────────────┘
             ↓
┌─────────────────────────────────────┐
│  3. DECISION LAYER                   │
│  Decision Engine                     │
│  • Evaluate contamination            │
│  • Select cleaning action            │
│  • Control cleaning cycle            │
└────────────┬──────────────────────────┘
             ↓
┌─────────────────────────────────────┐
│  4. CONTROL & ACTUATION LAYER        │
│  ESP32 → Driver → Cleaning Mechanism │
│  • Blower                            │
│  • Wiper                             │
│  • Defogger / Heater                 │
└────────────┬──────────────────────────┘
             ↓
┌─────────────────────────────────────┐
│  5. RECOVERY & FEEDBACK              │
│  Sensor Recheck → Visibility Restored?│
│  YES → Normal Robot Operation        │
│  NO  → Repeat Cleaning / Defogging   │
└─────────────────────────────────────┘
```

> *Tip: Replace this ASCII diagram with an exported PNG/SVG of your actual architecture diagram (e.g., from draw.io or the slide you already have) and embed it: `![System Architecture](docs/images/system-architecture.png)`.*

---

## Workflow

```
Sense → Analyze → Detect → Decide → Clean/Defog → Verify → Recover
```

| Step | Component | Action |
|---|---|---|
| 1. Sense | Camera | Captures sensor-surface images using the camera and monitor data |
| 2. Analyze | Raspberry Pi | Preprocesses and evaluates sensor condition |
| 3. Detect | AI Model | Classifies contamination as Clean / Dust / Water / Fog |
| 4. Decide | Decision Engine | Selects the suitable cleaning action |
| 5. Clean/Defog | ESP32 | Activates wiper, blower, or defogger |
| 6. Verify | Camera | Rechecks sensor quality after cleaning |
| 7. Recover | System | Restores visibility and resumes normal operation |

**Action mapping:**

| Detected Condition | Action |
|---|---|
| Dust | Blower (+ Wiper) |
| Water / Rain | Wiper (+ Blower) |
| Fog | Defogger / Heater (or Blower) |
| Clean | No action |

**Feedback loop:** If visibility is *not* restored after a cleaning cycle, the system automatically repeats the cycle rather than escalating to manual intervention, until the surface is confirmed clear.

---

## Folder Structure

> *Fill this in to match your actual repository layout. Suggested structure below — update as your repo evolves.*

```
AutoVisionGuardian/
├── ai_model/
│   ├── dataset/                # Clean / Dust / Water / Fog labeled images
│   ├── train_model.py          # MobileNetV3Small training script
│   ├── convert_to_tflite.py    # TFLite conversion
│   └── model/
│       └── autovision_model.tflite
├── raspberry_pi/
│   ├── capture.py              # Camera capture
│   ├── inference.py            # TFLite inference + 3-frame confirmation
│   ├── decision_engine.py      # Maps detection → action
│   └── serial_comm.py          # Communication with ESP32
├── esp32_firmware/
│   ├── main.ino                # Actuator control logic
│   └── pin_config.h
├── hardware/
│   ├── circuit_diagram.png
│   └── wiring_notes.md
├── docs/
│   ├── images/                 # Architecture & workflow diagrams
│   └── demo/                   # Screenshots / video links
├── requirements.txt
├── LICENSE
└── README.md
```

---

## Installation & Usage Guide

### Prerequisites

- Raspberry Pi 4B (2GB or higher) with Raspberry Pi OS
- Raspberry Pi Camera Module (connected and enabled via `raspi-config`)
- ESP32 development board
- Python 3.9+
- Arduino IDE or PlatformIO (for flashing ESP32 firmware)

### 1. Clone the repository

```bash
git clone https://github.com/<your-org>/AutoVisionGuardian.git
cd AutoVisionGuardian
```

### 2. Set up the Raspberry Pi environment

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### 3. Deploy the AI model

```bash
# Model is pre-converted to .tflite and placed in ai_model/model/
cd raspberry_pi
python3 inference.py
```

### 4. Flash the ESP32 firmware

- Open `esp32_firmware/main.ino` in Arduino IDE
- Select the correct board and COM port
- Upload the firmware

### 5. Run the full system

```bash
python3 raspberry_pi/capture.py
python3 raspberry_pi/decision_engine.py
```

> *Update commands/paths above once your actual scripts and entry points are finalized.*

---

## AI/ML Workflow

1. **Data Collection & Labeling** — Collect and label sensor-surface images across four classes: `Clean`, `Dust`, `Water`, `Fog`.
2. **Preprocessing & Augmentation** — Resize, normalize, and augment the dataset (rotation, brightness/contrast shifts, blur) to improve robustness to real outdoor lighting conditions.
3. **Model Training** — Train **MobileNetV3Small**, chosen for its lightweight footprint, for contamination classification.
4. **Edge Conversion** — Convert the trained model to **TensorFlow Lite** for efficient on-device inference.
5. **Deployment** — Deploy the `.tflite` model on the Raspberry Pi 4B (2GB), performing local, low-latency inference (no cloud dependency).
6. **Inference Pipeline** — Live camera frames are classified in real time.
7. **False-Positive Reduction** — A **confidence-based 3-frame confirmation** is used before triggering a cleaning action, avoiding unnecessary actuator cycles from a single noisy frame.

| Class | Trigger Action |
|---|---|
| Dust | Blower + Wiper |
| Water | Wiper + Blower |
| Fog | Defogger / Heater |
| Clean | No action |

> *Add your model's accuracy/precision/recall metrics and confusion matrix here once training is complete.*

---

## Hardware Components & Wiring

| Component | Purpose |
|---|---|
| Raspberry Pi 4B (2GB) | Edge AI processing & decision engine |
| Raspberry Pi Camera Module | Captures sensor-surface images |
| ESP32 | Actuator control (receives commands from Raspberry Pi) |
| Blower / Fan Motor | Clears dust and assists defogging |
| Wiper Mechanism | Clears water/rain droplets |
| Defogger / Heater Element | Clears fog / condensation |
| Motor Driver(s) | Drives blower/wiper motors from ESP32 GPIO |
| Power Supply | *(specify voltage/current ratings used)* |

**Circuit / Wiring Diagram:** *(insert image)*

```
![Circuit Diagram](docs/images/circuit_diagram.png)
```

> *Add your actual fritzing/schematic diagram and a pin-mapping table (ESP32 GPIO → driver → actuator) here.*

---

## Security Measures

- Local, on-device inference — no raw camera data is sent to the cloud, reducing exposure of sensor data
- *(Add: serial/communication authentication between Pi and ESP32, if implemented)*
- *(Add: safe-guard limits, e.g., maximum actuator run-time / retry cap to prevent motor damage or battery drain from a stuck retry loop)*
- *(Add: watchdog/fail-safe behavior if the AI model or ESP32 becomes unresponsive)*

> *This section should be expanded with the actual safeguards implemented in your firmware/software.*

---

## Testing & Performance

| Metric | Result |
|---|---|
| Model accuracy (validation set) | *TBD* |
| Inference latency (per frame, on Pi 4B) | *TBD* |
| Cleaning cycle time (avg.) | *TBD* |
| False-positive rate (before/after 3-frame confirmation) | *TBD* |
| Power consumption | *TBD* |

> *Fill in with actual benchmark numbers from your testing logs. Include test conditions (indoor/outdoor, lighting, contamination type) for credibility.*

---

## Challenges Faced & Future Scope

**Challenges Faced:**
- *(e.g., dataset collection for realistic dust/water/fog conditions)*
- *(e.g., balancing cleaning-cycle speed with actuator power draw)*
- *(e.g., false activation under changing outdoor lighting)*

**Future Scope:**
- Extend classification to detect combined contamination (e.g., dust + water simultaneously)
- Add LiDAR-specific cleaning mechanisms alongside the camera solution
- Predictive/preemptive cleaning based on environmental sensors (humidity, particulate sensors)
- Solar-assisted or low-power actuator design for extended field deployment
- Integration with robot's main navigation stack for cleaning-aware path planning

---

## Demo

- 🎥 Demo Video: *[Add link here]*
- 📸 Screenshots: *[Add links or embed images from `docs/demo/`]*
- 🔗 Live Prototype / Repo Wiki: *[Add link if applicable]*

---

## References

- MobileNetV3: Howard, A. et al., *"Searching for MobileNetV3"*
- TensorFlow Lite documentation — https://www.tensorflow.org/lite
- Raspberry Pi Camera Module documentation — https://www.raspberrypi.com/documentation/
- ESP32 documentation — https://docs.espressif.com/

---

## Event

Built for **YHACK'26**, hosted by **KPR Institute of Engineering and Technology** in association with **IEEE**, **IEEE Robotics & Automation Society (KPRIET SBC)**, **Ausweg**, and **RobotoAI**.

---

## Commit Guidelines (per hackathon rules)

- Commit and push every feature/milestone separately (dataset collection, model training, TFLite conversion, ESP32 firmware, integration, testing, etc.)
- Use clear, descriptive commit messages, e.g.:
  - `feat: add MobileNetV3Small training script`
  - `feat: implement 3-frame confidence confirmation`
  - `fix: correct GPIO mapping for wiper motor`
  - `docs: update README with architecture diagram`
- Avoid a single bulk commit at the end — commit history is part of evaluation.
