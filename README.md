# Sign Language Interpreter AI

![Python](https://img.shields.io/badge/Python-3.8+-blue?style=flat-square&logo=python)
![MediaPipe](https://img.shields.io/badge/MediaPipe-Hands-green?style=flat-square)
![Claude API](https://img.shields.io/badge/Claude-API-blueviolet?style=flat-square)
![FastAPI](https://img.shields.io/badge/FastAPI-Backend-teal?style=flat-square&logo=fastapi)
![License](https://img.shields.io/badge/License-MIT-lightgrey?style=flat-square)

Real-time sign language translation using computer vision and large language models. The system captures hand gestures through a webcam, detects 21 landmark points per hand, classifies the gesture, and then uses Claude to produce grammatically natural output — not just a list of words.

Built as part of a broader GenAI accessibility initiative aimed at bridging communication gaps for the deaf and hard-of-hearing community.

---

![Architecture Overview](https://raw.githubusercontent.com/YASH7110/GENAI-Project/main/assets/architecture.png)

> Pipeline: Camera → MediaPipe landmark detection → LSTM classifier → Claude refinement → Text/Speech output

---

## How it works

The interpreter runs in three stages.

First, MediaPipe Hands extracts 21 key points from each hand frame — fingertips, knuckles, wrist — giving you a `21 × 3` array of normalized coordinates per frame. This runs at ~30fps on a standard laptop without any GPU.

Second, a trained LSTM model receives a sliding window of ~30 frames and predicts the gesture being performed. Static signs like "yes" or "no" need only a single frame; dynamic signs involving motion require the full sequence.

Third, the sequence of predicted words gets sent to Claude, which converts choppy classifier output like `["I", "want", "drink", "water"]` into a fluent, natural sentence. This is what makes the output actually usable in conversation.

---

## Tech stack

| Layer | Tools |
|---|---|
| Hand detection | MediaPipe Hands |
| Frame capture | OpenCV |
| Gesture classification | LSTM / CNN (PyTorch) |
| Language refinement | Claude API (Anthropic) |
| Backend | FastAPI + WebSockets |
| Speech output | gTTS / ElevenLabs |
| Dataset | WLASL, ISL-CSLRT |

---

## Getting started

**Requirements**

- Python 3.8+
- A webcam
- Anthropic API key (get one at console.anthropic.com)

**Installation**

```bash
git clone https://github.com/YASH7110/sign-language-interpreter.git
cd sign-language-interpreter
pip install -r requirements.txt
```

**Set your API key**

```bash
export ANTHROPIC_API_KEY=your_key_here
```

**Run the interpreter**

```bash
python main.py
```

For the web interface:

```bash
uvicorn app:app --reload
```

Then open `http://localhost:8000` in your browser.

---

## Collecting your own gesture data

The project ships with a data collection script. Point your webcam at your hand and run:

```bash
python collect_data.py --gesture "hello" --samples 200
```

This saves 200 samples of the landmark array for that gesture. Repeat for each sign you want to support. Ten gestures with 200 samples each is enough to get started.

```
data/
  hello/
    sample_001.npy
    sample_002.npy
    ...
  thankyou/
    ...
```

---

## Training

Once you have data collected:

```bash
python train.py --epochs 50 --model lstm
```

The training script handles the sequence padding and train/val split automatically. Expect ~90% accuracy on a 10-gesture dataset after 50 epochs.

---

## Project structure

```
sign-language-interpreter/
  main.py               # Entry point, runs the interpreter loop
  collect_data.py       # Webcam-based data collection
  train.py              # Model training script
  app.py                # FastAPI backend
  models/
    lstm_classifier.py  # LSTM model definition
    classifier.pt       # Saved weights (after training)
  utils/
    mediapipe_utils.py  # Landmark extraction helpers
    llm_refiner.py      # Claude API integration
  data/                 # Collected gesture samples
  templates/            # Web UI
  requirements.txt
```

---

## Example output

Raw classifier output:

```
["I", "name", "Yash", "from", "India"]
```

After Claude refinement:

```
"My name is Yash and I'm from India."
```

---

## Supported gestures (default set)

The pretrained weights cover ten basic signs: Hello, Thank you, Yes, No, Please, Help, Water, Food, I, You.

You can extend this to any vocabulary by collecting data and retraining. The WLASL dataset covers over 2000 ASL words if you want to go further.

---

## Roadmap

- [x] MediaPipe landmark extraction
- [x] LSTM classifier on static gestures
- [x] Claude API refinement layer
- [x] FastAPI + WebSocket backend
- [ ] Dynamic gesture support (motion-based signs)
- [ ] ISL (Indian Sign Language) dedicated model
- [ ] Mobile app (Flutter)
- [ ] Bidirectional communication (text-to-sign avatar)
- [ ] Offline mode with on-device model

---

## Contributing

Pull requests are welcome. If you want to add support for a new sign language (ISL, BSL, etc.) or improve the classifier architecture, open an issue first so we can discuss the approach.

For dataset contributions, follow the collection format in `collect_data.py` and submit a PR with your samples included.

---

## License

MIT License. See [LICENSE](LICENSE) for details.

---

## Author

Yash Pratap Singh  
yashthakur1700@gmail.com  
[github.com/YASH7110](https://github.com/YASH7110)

---

*Part of the GenAI Accessibility Project — building AI tools for deaf, hard-of-hearing, and visually impaired communities.*
