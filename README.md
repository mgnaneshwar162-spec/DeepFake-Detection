# DeepScan — Setup Guide

## Architecture
```
[Your Browser] → [Render: index.html] → [ngrok tunnel] → [Colab: Flask API + Your Model]
```

---

## PART 1 — Colab Backend

### Step 1 — Get a free ngrok account
1. Go to https://ngrok.com → Sign up free
2. Dashboard → Copy your **Authtoken**

### Step 2 — Open the notebook
1. Upload `backend_colab.ipynb` to Google Colab
2. Set Runtime → **T4 GPU** (Runtime → Change runtime type)

### Step 3 — Run each cell in order

**Cell 1** — Installs Flask, ngrok, torch etc.
- Paste your ngrok token where it says `YOUR_TOKEN_HERE`

**Cell 2** — Upload your `.pt` / `.pth` model file when prompted

**Cell 3** — run cell

**Cell 4** — Detection logic (no edits needed unless your output format differs)

**Cell 5** — Starts the Flask server + ngrok tunnel
- You will see a URL like: `https://xxxx.ngrok-free.app`
- **Copy this URL** — you need it for the frontend

### API Endpoints
| Method | Path | Description |
|--------|------|-------------|
| GET | /health | Check if server is alive |
| POST | /detect | Detect image (field: `file`) |
| POST | /detect_video | Detect video (field: `video`) |
| GET | /analytics | Get cumulative stats |

---

## PART 2 — Render Frontend

### Step 1 — Create a GitHub repo
1. Create a new repo (public or private)
2. Upload these two files:
   - `index.html`
   - `render.yaml`

### Step 2 — Deploy on Render
1. Go to https://render.com → New → Static Site
2. Connect your GitHub repo
3. Settings:
   - **Build Command:** `echo "No build needed"`
   - **Publish Directory:** `.`
4. Click Deploy
5. Render gives you a URL like `https://deepscan-xxx.onrender.com`

### Step 3 — Connect frontend to backend
1. Open your Render URL in the browser
2. Paste your **ngrok URL** into the API URL box at the top
3. Click **Connect** — the status dot turns green ✓

---

## Usage

### Image Detection
1. Click the **Image** tab
2. Drop or select an image (JPG/PNG/WEBP)
3. Click **Analyse Image**
4. See verdict, confidence scores, and probability charts

### Video Detection
1. Click the **Video** tab
2. Drop or select a video (MP4/AVI/MOV)
3. Click **Analyse Video**
4. See verdict, AI frame ratio, and timeline chart

### Analytics
1. Click the **Analytics** tab
2. See cumulative stats for all detections this session

---

## Important Notes

- **ngrok URL changes every session** — each time you restart Colab,
  you get a new URL. Just paste the new one in the frontend.
- **Keep Cell 5 running** — stopping it kills the API server.
- **Free ngrok** allows 1 tunnel and has a request rate limit. For
  production use, upgrade ngrok or use a paid hosting solution.
- **Model cell (Cell 3)** must be edited to match your exact architecture
  before running — otherwise the state_dict load will fail.
