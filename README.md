# ASL Sign Language Recognition
### Hybrid ResNet50 + Vision Transformer (ViT)

![Python](https://img.shields.io/badge/Python-3.11-blue?style=flat-square&logo=python)
![PyTorch](https://img.shields.io/badge/PyTorch-2.11-EE4C2C?style=flat-square&logo=pytorch)
![Accuracy](https://img.shields.io/badge/Val_Accuracy-99.77%25-brightgreen?style=flat-square)
![Platform](https://img.shields.io/badge/Platform-Mac_M1_|_Colab-lightgrey?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)

Real-time American Sign Language recognition using a hybrid deep learning architecture that combines ResNet50's local feature extraction with Vision Transformer's global context modeling. Trained on 87,000 images across 29 ASL classes


*ASL Alphabet — 26 letters + del, nothing, space*

---

## What this does

Point a webcam at your hand, and the model tells you which ASL sign you're making — in real time. It detects the hand region from the frame, crops it, runs it through the hybrid model, and shows the top 3 predictions with confidence scores.

Works on static images too via the Streamlit web interface.

---

## Model Architecture

The core idea is simple: ResNet50 is really good at picking up local texture and shape details (like finger positions), while ViT is good at understanding the overall spatial arrangement. Combining both gives better results than either alone.

```
Input Image (224×224)
        │
   ┌────┴────┐
   │         │
ResNet50   ViT-Base/16
(pretrained) (pretrained)
2048-d      768-d
   │         │
   └────┬────┘
    concat (2816-d)
        │
   Linear(512) → ReLU → Dropout(0.3)
        │
   Linear(29) → Softmax
        │
   Predicted Sign
```

<img width="1292" height="1482" alt="image" src="https://github.com/user-attachments/assets/1461d563-d54e-4ab7-95e1-e9873cc58e41" />

*Vision Transformer patch-based processing*

Both backbones are pretrained on ImageNet. Early ResNet layers are frozen during training — only the last 2 blocks fine-tune along with the full ViT.

---



## Dataset

**ASL Alphabet** by grassknoted on Kaggle  
- 87,000 training images  
- 29 classes: A–Z + `del`, `nothing`, `space`  
- 200×200 px, RGB

```bash
kaggle datasets download -d grassknoted/asl-alphabet -p ./data --unzip
```

Get your `kaggle.json` from [kaggle.com → Account → API → Create New Token](https://www.kaggle.com/settings/account)

---

## Project Structure

```
Sign_asl/
├── checkpoint_best.pth       # trained model weights
├── predict.py                # real-time webcam inference
├── app.py                    # streamlit web app
├── test_image.py             # single image prediction
├── requirements.txt
└── README.md
```

---

## Setup

**Requirements:** Python 3.11, Mac M1 or Google Colab

```bash
# Clone the repo
git clone https://github.com/yashpratap/asl-sign-language-recognition
cd asl-sign-language-recognition

# Install dependencies
pip install torch torchvision timm streamlit pillow
```

> If you're on Mac M1, use Python 3.11 specifically. Python 3.10 has segfault issues with PyTorch on macOS Tahoe (26.x).

```bash
brew install python@3.11
python3.11 -m pip install torch torchvision timm streamlit pillow
```

---

## Training on Google Colab

Open a new Colab notebook, set runtime to **T4 GPU**, then run:

```python
# Mount Drive (checkpoints save here)
from google.colab import drive
drive.mount('/content/drive')

# Install
!pip install tqdm timm -q

# Download dataset
import os
os.makedirs('/root/.kaggle', exist_ok=True)
!cp kaggle.json /root/.kaggle/
!chmod 600 /root/.kaggle/kaggle.json
!kaggle datasets download -d grassknoted/asl-alphabet -p /content/data --unzip
```

Then paste the full training script from `train_colab.py`. Checkpoints save to `/content/drive/MyDrive/sign_language_checkpoints/` every 2 epochs.

**Resume from checkpoint**: just re-run the training cell — it auto-detects and resumes from the last saved epoch.

Training config:
- Batch size: 64
- Optimizer: AdamW, lr=1e-4
- Scheduler: CosineAnnealingLR
- Epochs: 20 (early stopping if needed)

---

## Running Inference

### Streamlit app (recommended)

```bash
python3.11 -m streamlit run app.py
```

Opens in your browser. Two modes:
- **Webcam** — point your hand at the camera, click to capture, get prediction
- **Upload** — drag any image of a hand sign, get top-5 predictions with confidence bars

![Streamlit](https://streamlit.io/images/brand/streamlit-logo-secondary-colormark-darktext.png)

### Real-time webcam (terminal)

```bash
python3.11 predict.py
```

Shows live feed with bounding box around detected hand, top prediction + confidence, and FPS counter. Press `Q` to quit.

### Single image test

```bash
python3.11 test_image.py path/to/image.jpg
```

---

## Tech Stack

| Component | Tool |
|-----------|------|
| Deep learning | PyTorch 2.11 |
| CNN backbone | ResNet50 (torchvision) |
| Transformer | ViT-Base/16 (timm) |
| Training | Google Colab T4 |
| Checkpointing | Google Drive |
| UI | Streamlit |
| Image ops | Pillow |
| Camera (Mac) | imagesnap (AVFoundation) |

---

## Known Issues

**Segfault on Mac M1 with Python 3.10**  
macOS Tahoe (26.x) has compatibility issues with Python 3.10 + PyTorch. Use Python 3.11.

**OpenCV `imshow` crashes on macOS 26**  
The Streamlit version avoids OpenCV display entirely — use `app.py` instead of `predict.py` if you hit this.

**MediaPipe on M1**  
Removed MediaPipe entirely. Hand detection uses skin color segmentation via numpy — no native library crashes.

---

## Future Work

- Add Indian Sign Language (ISL) support
- Sentence-level continuous sign recognition
- Mobile deployment (CoreML for iOS)
- Two-hand sign detection
- Word-to-sentence translation using an LLM

---

## Author

**Yash Pratap Singh**  
Student, Bennett University  
📧 yashthakur1700@gmail.com  
🔗 [GitHub](https://github.com/yashpratap)

---

## License

MIT License — feel free to use, fork, and build on this.

---

## Acknowledgements

- [grassknoted](https://www.kaggle.com/grassknoted) for the ASL Alphabet dataset on Kaggle
- [rwightman](https://github.com/rwightman/pytorch-image-models) for the timm library (ViT weights)
- [PyTorch](https://pytorch.org) team for torchvision ResNet50 pretrained weights
