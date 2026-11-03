# Vision-Based Fatigue Detection System

A real-time computer vision application that monitors facial cues through a webcam to identify potential signs of fatigue and drowsiness. The system analyzes eye closure, yawning, and head pose, displays detection information on the video feed, and plays audio alerts when configured thresholds are crossed.

## Overview

Driver fatigue and reduced alertness can affect reaction time and decision-making. This project demonstrates a camera-based approach to monitoring visible indicators of drowsiness using Python, OpenCV, and facial landmark analysis.

The application processes webcam frames in real time and uses facial geometry to estimate whether the eyes are closed, whether the person is yawning, and whether their head is tilted downward.

> **Note:** This is an academic/prototype project. It is not a certified driver-monitoring or medical device, and its alerts should not be treated as a substitute for rest or other safety measures.

## Features

- **Real-time webcam monitoring** using OpenCV.
- **Face detection** with Dlib's HOG-based frontal face detector.
- **Facial landmark detection** using a 68-point landmark predictor.
- **Eye-closure detection** using the Eye Aspect Ratio (EAR).
- **Yawning detection** using the Mouth Aspect Ratio (MAR).
- **Head-pose estimation** using OpenCV's `solvePnP()` and Euler angles.
- **On-screen feedback**, including facial landmarks, head-pose angles, EAR, MAR, and detection labels.
- **Audio alerts** for prolonged eye closure, yawning, and downward head pose.

## Technology Stack

| Technology / Library | Purpose |
| --- | --- |
| Python | Main programming language |
| OpenCV (`cv2`) | Webcam capture, image processing, drawing, and head-pose calculations |
| Dlib | Face detection and 68-point facial landmark prediction |
| imutils | Image resizing and facial landmark utilities |
| NumPy | Numerical operations and coordinate arrays |
| SciPy | Euclidean-distance calculations for EAR and MAR |
| Pygame | Audio playback for alerts |
| Threading | Running alert playback in separate threads |

## How It Works

The application follows this processing pipeline:

```text
Webcam Video
     |
     v
Capture Frame with OpenCV
     |
     v
Resize Frame and Convert to Grayscale
     |
     v
Detect Face with Dlib
     |
     v
Predict 68 Facial Landmarks
     |
     +----------------------+----------------------+
     |                      |                      |
     v                      v                      v
 Eye Landmarks         Mouth Landmarks        Facial Landmarks
     |                      |                      |
     v                      v                      v
 Calculate EAR         Calculate MAR          Estimate Head Pose
     |                      |                  (solvePnP / Euler angles)
     v                      v                      v
 Check Eye Closure     Check Yawning           Check Head-Down Pose
     |                      |                      |
     +----------------------+----------------------+
                            |
                            v
                 Display Status and Measurements
                            |
                            v
                     Trigger Audio Alerts
```

### 1. Face and landmark detection

Each webcam frame is converted to grayscale and passed to Dlib's frontal face detector. For every detected face, the 68-point facial landmark predictor estimates key facial coordinates. The eye and mouth landmark groups are then extracted for further calculations.

### 2. Eye closure detection — Eye Aspect Ratio (EAR)

EAR measures the relationship between the vertical and horizontal distances between eye landmarks. A lower EAR indicates that the eye is more closed.

The implementation calculates EAR for both eyes and averages the results:

\[
EAR = \frac{||p_2-p_6|| + ||p_3-p_5||}{2||p_1-p_4||}
\]

The code uses these configured values:

- `EYE_AR_THRESH = 0.25`
- `EYE_AR_CONSEC_FRAMES = 20`

When the average EAR falls below the threshold, a consecutive-frame counter increases. When the counter reaches the configured frame limit, the application displays a **DROWSINESS ALERT!** message and plays the eye-closure alert.

### 3. Yawning detection — Mouth Aspect Ratio (MAR)

MAR measures mouth opening by comparing vertical mouth-landmark distances with the horizontal mouth width.

The implementation uses:

\[
MAR = \frac{d(p_3,p_{11}) + d(p_5,p_9)}{2d(p_1,p_7)}
\]

The configured threshold is:

- `MOU_AR_THRESH = 0.50`

When MAR exceeds the threshold, the application displays a yawning indicator. It counts a yawn when the mouth returns below the threshold. After more than one detected yawn, it displays a drowsiness alert and plays the yawning audio.

### 4. Head-pose estimation

The project estimates head orientation using facial landmark coordinates and OpenCV's `solvePnP()` function. It uses camera calibration values, 3D reference points, and corresponding 2D facial landmarks to estimate the head's rotation.

The resulting Euler angles are displayed on the video feed. The current implementation displays **Head Down** and triggers its associated audio alert when the X Euler angle (`xx`) is greater than `10`.

### 5. Audio alerts and visual feedback

The application uses Pygame's mixer to play separate audio files for the configured alert conditions. OpenCV overlays status messages, facial landmarks, head-pose axes, and EAR/MAR measurements on the webcam feed.

## Repository Structure

```text
Vision-Based-Fatigue-System/
│
├── main.py
├── shape_predictor_68_face_landmarks.dat
├── siren-alert-96052.mp3
├── sound.wav
├── music.wav
└── README.md
```

- `main.py` — Main application containing webcam processing, face and landmark detection, EAR/MAR calculations, head-pose estimation, and alert logic.
- `shape_predictor_68_face_landmarks.dat` — Pre-trained Dlib facial landmark predictor required by the application.
- `siren-alert-96052.mp3` — Audio file used for the head-down alert.
- `sound.wav` — Audio file used for the eye-closure alert.
- `music.wav` — Audio file used for the yawning alert.

## Getting Started

### Prerequisites

- Python installed on your computer.
- A working webcam.
- A microphone is **not** required.
- Speakers or headphones are recommended to hear the alerts.
- The facial landmark model file must be present in the project directory.

### 1. Clone the repository

```bash
git clone https://github.com/adityahargunani/Vision-Based-Fatigue-System.git
cd Vision-Based-Fatigue-System
```

### 2. Create and activate a virtual environment (recommended)

**Windows:**

```bash
python -m venv venv
venv\Scripts\activate
```

**macOS / Linux:**

```bash
python3 -m venv venv
source venv/bin/activate
```

### 3. Install dependencies

```bash
pip install numpy scipy imutils opencv-python pygame dlib
```

Dlib installation can vary by operating system and Python version. If `pip install dlib` fails, follow the installation instructions for Dlib for your platform before continuing.

### 4. Verify the required model file

Make sure this file exists in the project root:

```text
shape_predictor_68_face_landmarks.dat
```

The repository includes this model file. If you obtain it separately, use the official Dlib 68-point facial landmark predictor and place the extracted `.dat` file in the project root.

### 5. Run the application

```bash
python main.py
```

The webcam window should open and begin processing frames. Press **`q`** while the video window is active to exit.

## Configuration

The following detection thresholds are defined in `main.py` and can be adjusted for experimentation:

| Constant | Current value | Description |
| --- | ---: | --- |
| `EYE_AR_THRESH` | `0.25` | EAR value below which the eyes are considered closed |
| `EYE_AR_CONSEC_FRAMES` | `20` | Number of consecutive frames required for the eye-closure alert |
| `MOU_AR_THRESH` | `0.50` | MAR value above which yawning is detected |
| Head-down threshold | `xx > 10` | Euler-angle condition used for the head-down indicator and alert |

These are implementation thresholds, not universal values. Their behavior can vary with camera position, lighting, face orientation, frame rate, and individual facial characteristics.

## Troubleshooting

| Issue | What to check |
| --- | --- |
| `Could not open camera` | Check webcam permissions, camera connection, and whether another application is using it. |
| Dlib model file error | Confirm that `shape_predictor_68_face_landmarks.dat` is in the project root and that the filename matches the path in `main.py`. |
| `ModuleNotFoundError` | Activate the intended virtual environment and install the missing dependency. |
| No audio alert | Check system volume, output device, audio-file paths, and Pygame installation. |
| Inconsistent detection | Improve lighting, keep the face visible, position the camera near eye level, and experiment with thresholds. |

## Limitations

- Detection quality depends on lighting, camera placement, image quality, and face visibility.
- Facial landmark tracking may become less reliable with large head rotations, occlusions, or partially visible faces.
- Fixed EAR, MAR, and head-pose thresholds may not work equally well for every person or environment.
- The current implementation uses geometric thresholds and does not train or run a separate deep-learning fatigue-classification model.
- The head-pose calibration values may need adjustment for a different camera setup.
- The system is a prototype and has not been validated for safety-critical use.

## Future Enhancements

- Add blink-rate and eye-closure-duration analysis over longer time windows.
- Improve alert logic to reduce repeated or unnecessary alarms.
- Add configurable thresholds and camera calibration.
- Introduce a dashboard for fatigue-event summaries and session statistics.
- Evaluate the system under different lighting and camera conditions.
- Explore machine-learning approaches for more robust fatigue classification.

## Usage

This repository is intended for learning, experimentation, and further development in computer vision and real-time monitoring. If you build on this project, please review the licenses and usage terms of the third-party libraries, pre-trained model, and audio assets.

## Author

**Aditya Hargunani**

- GitHub: [@adityahargunani](https://github.com/adityahargunani)
- Project Repository: [Vision-Based-Fatigue-System](https://github.com/adityahargunani/Vision-Based-Fatigue-System)

---

If you find this project useful, consider giving the repository a ⭐ on GitHub.
