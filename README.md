# Multi-Incident Accident Detection and Severity Grading Using YOLOv8 and Optical Flow Tracking 

**VisionGuard** is a real-time traffic surveillance and accident detection system that detects road accidents, hazards, and anomalous driving behaviors from CCTV footage using deep learning, multi-object tracking, and computer vision techniques.

---

## ✨ Key Features

- **Dual-Stage YOLO Detection** — YOLO-A for vehicle detection (`car`, `truck`, `bus`, `motorcycle`) and YOLO-B for localized hazard detection (`crashed_vehicle`, `fire`, `smoke`).
- **Real-Time Tracking** — ByteTrack combined with Lucas-Kanade optical flow for persistent trajectory and velocity tracking across camera frames.
- **Multi-Signal Anomaly Detection** — Rule-based scoring engine analyzing sudden deceleration (speed drop), severe trajectory deviations, vehicle bounding box overlaps, flow incoherence, and lost motorcycle tracks.
- **Tiered Severity Classification** — Incident evaluation into 4 distinct severity tiers (`Minor`, `Moderate`, `Severe`, `Critical`) with hysteresis to avoid flickering.
- **Adaptive Preprocessing** — Dynamic environmental compensation including Dark Channel Prior (DCP) defogging, temporal rain filtering, and CLAHE contrast enhancement.
- **Automated Telegram Alerts** — Instant notification dispatch with incident snapshot images and incident metrics via Telegram bot.
- **Streamlit Interactive Dashboard** — Live operational monitoring, incident logs review, and historical accident analysis.
- **Pipeline Benchmarking** — Built-in hardware timing validator to measure stage-by-stage latency and FPS.

---

## 🏗️ Project Structure

```
visionguard/
├── main.py                     # Primary CLI execution entry point
├── test_pipeline.py            # Stage-by-stage performance benchmark
├── requirements.txt            # Python dependencies
├── LICENSE                     # MIT License
├── README.md                   # Project documentation
├── config/
│   └── config.yaml             # Centralized pipeline configuration
├── alerts/
│   ├── __init__.py
│   └── telegram_bot.py         # Telegram notification sender with cooldown
├── anomaly/
│   ├── __init__.py
│   └── detector.py             # Multi-signal anomaly scoring engine
├── dashboard/
│   └── app.py                  # Streamlit incident management dashboard
├── detection/
│   ├── __init__.py
│   ├── vehicle_detector.py     # YOLO-A vehicle detector
│   └── hazard_detector.py      # YOLO-B hazard & collision detector
├── pipeline/
│   ├── __init__.py
│   ├── engine.py               # Core processing loop orchestrator
│   └── frame_reader.py         # Multi-threaded buffered frame reader
├── preprocessing/
│   ├── __init__.py             # Adaptive preprocessor controller
│   ├── dehazer.py              # Dark Channel Prior defogger
│   ├── enhancer.py             # CLAHE contrast enhancer
│   ├── motion_gate.py          # Background subtraction motion gate
│   └── rain_filter.py          # Temporal rain streak filter
├── reporting/
│   ├── __init__.py
│   └── logger.py               # Thread-safe JSON event logger
├── severity/
│   ├── __init__.py
│   └── classifier.py           # Severity tier scoring classifier
├── tests/
│   ├── test_priority1.py       # Unit tests for queue and tracker
│   └── test_synthetic_regression.py # Anomaly detector regression tests
├── models/
│   └── .gitkeep                # Directory for trained YOLO weights
└── output/
    ├── logs/                   # Generated incident event logs
    └── snapshots/              # Captured accident snapshot images
```

---

## 🚀 Getting Started

### Prerequisites

- Python 3.10+
- Webcam or video files (e.g., MP4/AVI)
- CUDA-compatible GPU (recommended for optimal frame rate)

### Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/B-G-A/Accident_Detection.git
   cd Accident_Detection
   ```

2. **Create and activate a virtual environment:**
   ```bash
   python -m venv venv
   # On Windows:
   venv\Scripts\activate
   # On Linux/macOS:
   source venv/bin/activate
   ```

3. **Install dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

4. **Add YOLO Model Weights:**
   Place your trained weights in the `models/` directory:
   - `models/yoloa_best.pt` (Vehicle detection model)
   - `models/yolob_best.pt` (Hazard detection model)

---

## ⚙️ Configuration

Configure pipeline parameters, alert credentials, and detection sensitivities in `config/config.yaml`:

```yaml
alerts:
  enable_telegram: false
  bot_token: YOUR_TELEGRAM_BOT_TOKEN
  chat_id: YOUR_TELEGRAM_CHAT_ID
  camera_location: Main Intersection Camera
  cooldown_seconds: 120

pipeline:
  detect_interval: 2
  target_fps: 15
  use_threading: true

detection:
  iou_threshold: 0.35
  yoloa_confidence: 0.25
  yolob_confidence: 0.50
```

---

## 🎬 Running VisionGuard

### 1. Run Surveillance on Video

Execute accident detection on a recorded video:

```bash
python main.py --video path/to/traffic_video.mp4
```

**Common flags:**
- `--video <path>`: Path to input video file (required)
- `--config <path>`: Custom configuration YAML (default: `config/config.yaml`)
- `--no-display`: Run headless without video display window (ideal for servers)
- `--log-level`: Set logging verbosity (`DEBUG`, `INFO`, `WARNING`, `ERROR`)

### 2. Launch Monitoring Dashboard

Start the interactive Streamlit dashboard to monitor detections and review incident history:

```bash
streamlit run dashboard/app.py
```

### 3. Run Pipeline Benchmark

Verify that each pipeline stage runs within real-time latency budgets on your hardware:

```bash
python test_pipeline.py --video path/to/traffic_video.mp4 --frames 100
```

### 4. Run Automated Unit Tests

```bash
python -m unittest discover tests
```

---

## 🔬 Severity Scoring Framework

VisionGuard calculates an additive anomaly score based on simultaneous physical and visual signals:

| Signal | Description | Score |
|--------|-------------|-------|
| `speed_drop_heavy` | Rapid deceleration (>70% drop) | +3 |
| `speed_drop_moderate` | Noticeable deceleration (40–70% drop) | +1 |
| `trajectory_deviation` | Sharp angle shift (>45°) | +3 |
| `crashed_vehicle_yolo` | YOLO-B detects visible crash damage | +4 |
| `fire_detected` | YOLO-B identifies active fire | +5 |
| `smoke_detected` | YOLO-B identifies smoke plume | +3 |
| `vehicle_overlap` | Bounding box intersection (IoU > 0.3) | +3 |
| `flow_incoherence` | Erratic LK optical flow motion | +1 |
| `motorcycle_track_lost` | Sudden disappearance of high-speed 2-wheeler | +4 |

### Severity Tiers:

- 🟡 **Minor** (`Score ≤ 3`): Minor anomalies, sudden braking or low-impact swerve.
- 🟠 **Moderate** (`Score 4 – 6`): Strong collision indicator or noticeable crash damage.
- 🔴 **Severe** (`Score 7 – 10`): Multi-signal collision, severe crash damage, or rollover.
- 🚨 **Critical** (`Score > 10`): Severe collision with fire or catastrophic impact.

---

## 🛠️ Built With

- **Object Detection**: [Ultralytics YOLO](https://github.com/ultralytics/ultralytics)
- **Computer Vision**: OpenCV
- **Scientific Computing**: NumPy, SciPy
- **Dashboard**: Streamlit
- **Alerts**: Telegram Bot API

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
