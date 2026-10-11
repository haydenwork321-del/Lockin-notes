# SmashBash — Web + Camera (Python) Edition

This keeps the whole game in Python. `handcv_web.py` is your original
`HandCV.py`'s detection engine, effectively unchanged. `app.py` is a port
of `Interface.py`'s game loop, with pygame's `screen.blit(...)` calls
replaced by OpenCV drawing calls, wired up through
[streamlit-webrtc](https://github.com/whitphx/streamlit-webrtc) so a
visitor's browser webcam streams into this Python process and the
annotated frame streams back out — no server GPU, no desktop window, no
JavaScript written by you.

## Run it locally

```bash
pip install -r requirements.txt
streamlit run app.py
```

It'll open in your browser at `http://localhost:8501`. First run downloads
the MediaPipe hand model automatically (same as the desktop version).

## Deploy it so anyone can play

Static hosting (GitHub Pages) can't run this — it needs a live Python
process. Two free options that will:

- **Streamlit Community Cloud** (streamlit.io/cloud) — connect this repo,
  point it at `app.py`, done. Easiest option.
- **Hugging Face Spaces** (choose the "Streamlit" SDK) — same idea, also free.

Either way you get a real shareable URL.

## What's different from the desktop version, and why

- **Rendering**: pygame can only draw to a native desktop window, so all
  drawing now happens with OpenCV directly on the video frame
  (`cv2.circle`, `cv2.putText`, etc.) instead of `screen.blit(...)`. The
  game *logic* underneath — spawn timing, drum zones, hit/miss rules,
  lives, the revive message — is a direct line-for-line port.
- **Sprites**: the original PNGs (hamburger, pizza, memes, hearts, drums)
  weren't provided, so falling objects are colored discs with a 3-letter
  label, and hearts/drums are drawn with OpenCV shapes.
  **To use your real images instead:** load each with
  `cv2.imread("images/hamburger.png", cv2.IMREAD_UNCHANGED)` once at
  startup, then alpha-composite it onto `img` at `(obj["x"], obj["y"])`
  each frame instead of the `cv2.circle`/`cv2.putText` calls in
  `GameProcessor.recv`.
- **Music**: background audio doesn't have a clean way to sync into a
  frame-by-frame video stream like this, so it's left out. If you want
  music, the simplest route is an `<audio autoplay loop>` tag added via
  `st.markdown(..., unsafe_allow_html=True)` playing independently of the
  video stream (won't be frame-synced, but will loop in the background).

## Files

- `handcv_web.py` — hand detection (from your `HandCV.py`)
- `app.py` — game logic + rendering + Streamlit/webrtc wiring (from your `Interface.py`)
- `requirements.txt` — dependencies
