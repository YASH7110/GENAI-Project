# Sign Language Recognition using Bidirectional LSTM

Real-time sign language detector that reads hand gestures from webcam and converts them to text and speech. Built without CNN — uses only hand geometry, which keeps the model small and fast.

---

## What it does

- Detects 10 ASL signs live from webcam
- Shows confidence score and which frames the model focused on (attention heatmap)
- Displays a skeleton mirror — the AI's geometric view of your hand
- Speaks the detected word out loud

---
<img width="1194" height="1647" alt="image" src="https://github.com/user-attachments/assets/22ffabf0-268a-455a-9f8e-c6c933109136" />



## Architecture

```
Webcam (30 frames)
     │
     ▼
MediaPipe Hand Tracking
     │  extracts 21 landmarks per frame
     │  each landmark = x, y, z
     │  21 × 3 = 63 features per frame
     ▼
StandardScaler (normalize)
     │
     ▼
┌─────────────────────────────────┐
│  Bidirectional LSTM  (128 units)│  ← reads gesture forward + backward
│  BatchNorm + Dropout 0.3        │
├─────────────────────────────────┤
│  Bidirectional LSTM  (64 units) │  ← refines temporal pattern
│  BatchNorm + Dropout 0.3        │  ← attention tapped here
├─────────────────────────────────┤
│  LSTM  (64 units)               │  ← collapses 30 frames → 1 vector
│  BatchNorm + Dropout 0.3        │
├─────────────────────────────────┤
│  Dense 128 → Dense 64           │
│  Dropout 0.4 / 0.3              │
├─────────────────────────────────┤
│  Dense 10  →  Softmax           │  ← one of 10 signs
└─────────────────────────────────┘
     │
     ▼
Prediction + Confidence Score
     │
     ├──► Attention Heatmap (30 frames, red=ignored / green=focused)
     ├──► Skeleton Mirror (live geometric hand view)
     └──► Text-to-Speech
```

---
<img width="1192" height="1878" alt="image" src="https://github.com/user-attachments/assets/78a974ac-a8d9-4a7d-b318-4618c15c305b" />




## Why no CNN

Most vision projects use CNN on raw pixels — that makes models 50–200MB and sensitive to lighting and background. Here, MediaPipe extracts the hand's 3D skeleton first, so the LSTM only sees 63 clean numbers per frame instead of 921,600 pixels. Result: **5MB model, 95% accuracy, runs on CPU.**

---

## Signs supported

| # | Sign | # | Sign |
|---|------|---|------|
| 1 | hello | 6 | peace |
| 2 | thanks | 7 | ok |
| 3 | yes | 8 | fist |
| 4 | no | 9 | call |
| 5 | stop | 10 | point |

---

## Project structure

```
sign-language-project/
├── predict.py               ← main file, run this
├── hand_landmarker.task     ← MediaPipe model
├── model/
│   ├── model_improved.weights.h5
│   ├── best_model.keras
│   └── scaler.pkl           ← must match training normalization
└── data/                    ← self-collected dataset
```

---

## Setup

```bash
pip install tensorflow mediapipe opencv-python scikit-learn matplotlib pyttsx3
python3 predict.py
```

Press `Q` to quit.

---

## Training details

| Thing | Value |
|---|---|
| Dataset | Self-collected via webcam |
| Sequences per sign | 30+ |
| Frames per sequence | 30 |
| Features per frame | 63 |
| Train/test split | 80/20 stratified |
| Augmentation | Noise, scale, time warp, hand flip |
| Epochs | Up to 150 (early stopping) |
| Best val accuracy | **95%** |

---

## Features

- **Attention heatmap** — shows which of the 30 frames the LSTM focused on most. Green = model paid attention here, red = ignored.
- **Skeleton mirror** — right panel shows the raw 21-point hand geometry the AI actually uses. No camera, no background, just dots and lines.
- **Confidence score** — color coded bar (green > 85%, yellow > 75%, red below).
- **Buffer indicator** — orange bar shows progress toward the 30-frame window.

---
<img width="962" height="538" alt="Screenshot 2026-04-19 at 2 59 50 PM" src="https://github.com/user-attachments/assets/fda895e7-63da-44d5-8836-9747f42d6f36" />
<img width="960" height="543" alt="Screenshot 2026-04-19 at 2 59 38 PM" src="https://github.com/user-attachments/assets/b8cd8cbb-164d-440e-8b0a-02d91375fbeb" />

## Built with

- TensorFlow / Keras — model training and inference
- MediaPipe — hand landmark extraction
- OpenCV — camera feed and UI
- scikit-learn — data normalization
- pyttsx3 — text to speech
- matplotlib — heatmap rendering


Owner: Yash Pratap singh 
Bennett University##Contact
yashthakur1700@gmail.com 

  ->->->

## 📂 Drive Link (Model Weights & Info)
[Open Google Drive Folder](https://drive.google.com/drive/folders/15Kk_C98q0QB72D8VBTil8e7fRoVM2YlX?usp=sharing)
