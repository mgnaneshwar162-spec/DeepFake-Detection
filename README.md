# DeepScan — AI Deepfake Detection System

A deepfake and AI-generated image/video detector using a 3-model ensemble.
Colab runs the backend, Render hosts the frontend.

---

## Models Used

| Model | Type | Notes |
|-------|------|-------|
| Your ResNet50 | Custom trained | 97.61% val accuracy |
| umm-maybe/AI-image-detector | HuggingFace | General AI image detector |
| Organika/sdxl-detector | HuggingFace | SDXL-focused detector |

All 3 predictions are averaged equally (33% each) for the final ensemble verdict.

---

## Project Structure

```
backend_colab.ipynb   → Colab notebook (Flask API + your model)
index.html            → Frontend (deploy on Render)
render.yaml           → Render static site config
README.md             → This file
```

---

## Setup Guide

### Part 1 — Colab Backend

#### Step 1 — Get ngrok token
1. Go to https://ngrok.com and sign up free
2. Dashboard → Copy your Authtoken

#### Step 2 — Open notebook
1. Upload `backend_colab.ipynb` to Google Colab
2. Set Runtime → Change runtime type → **T4 GPU**

#### Step 3 — Run cells in order

| Cell | What it does | Edit needed? |
|------|-------------|--------------|
| Cell 1 | Installs all dependencies | Paste your ngrok token |
| Cell 2 | Upload your .pt/.pth model | No |
| Cell 3 | Loads your ResNet50 model | No — preconfigured |
| Cell 4 | Loads 2 HuggingFace models | No |
| Cell 5 | Ensemble detection helpers | No |
| Cell 6 | Starts Flask + ngrok tunnel | No |

#### Step 4 — Copy your API URL
After Cell 6 runs you will see:
```
=======================================================
  API URL:  https://xxxx.ngrok-free.app
=======================================================
```
Copy this URL — you need it for the frontend.

> Keep Cell 6 running at all times. Stopping it kills the API.
> Every time you restart Colab you get a new ngrok URL.

---

### Part 2 — Render Frontend

#### Step 1 — Push to GitHub
Create a new GitHub repo and upload:
- `index.html`
- `render.yaml`

#### Step 2 — Deploy on Render
1. Go to https://render.com → New → Static Site
2. Connect your GitHub repo
3. Settings:
   - Build Command: `echo "No build needed"`
   - Publish Directory: `.`
4. Click Deploy
5. Render gives you a URL like `https://deepscan-xxx.onrender.com`

#### Step 3 — Connect to backend
1. Open your Render URL
2. Paste your ngrok URL into the API URL box
3. Click Connect — status dot turns green

---

## API Endpoints

| Method | Endpoint | Field | Description |
|--------|----------|-------|-------------|
| GET | /health | — | Check server status |
| POST | /detect | `file` | Detect image |
| POST | /detect_video | `video` | Detect video |
| GET | /analytics | — | Get cumulative stats |

---

## Detection Output

### Image
```json
{
  "verdict": "AI GENERATED",
  "confidence": 0.823,
  "ai_probability": 0.823,
  "real_probability": 0.177,
  "per_model": {
    "Your Model":        { "ai_probability": 0.91, "real_probability": 0.09, "verdict": "AI GENERATED" },
    "AI-Image-Detector": { "ai_probability": 0.79, "real_probability": 0.21, "verdict": "AI GENERATED" },
    "SDXL-Detector":     { "ai_probability": 0.77, "real_probability": 0.23, "verdict": "AI GENERATED" }
  }
}
```

### Video
```json
{
  "verdict": "LIKELY DEEPFAKE",
  "ai_frame_ratio": 0.73,
  "avg_ai_prob": 0.81,
  "total_frames_sampled": 24,
  "timeline": [0.82, 0.79, 0.91, ...],
  "per_model_avg": {
    "Your Model":        { "avg_ai_probability": 0.88, "avg_real_probability": 0.12 },
    "AI-Image-Detector": { "avg_ai_probability": 0.76, "avg_real_probability": 0.24 },
    "SDXL-Detector":     { "avg_ai_probability": 0.79, "avg_real_probability": 0.21 }
  }
}
```

---

## Verdict Logic

| Condition | Verdict |
|-----------|---------|
| Ensemble AI prob >= 75% | AI GENERATED |
| Ensemble Real prob >= 75% | REAL IMAGE |
| Neither above 75% | UNCERTAIN |
| Video AI frame ratio > 40% | LIKELY DEEPFAKE |
| Video AI frame ratio <= 40% | LIKELY REAL |

---

## Limits

| Type | Max size |
|------|----------|
| Image | 10 MB |
| Video | 100 MB |

Video is sampled every 30th frame to keep inference fast.

---

## Important Notes

- Free ngrok allows 1 tunnel at a time
- ngrok URL changes every Colab session — update it in the frontend
- GPU (T4) is strongly recommended — CPU inference is very slow
- The Flask development server warning is normal and harmless
