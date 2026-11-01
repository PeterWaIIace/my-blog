---
title: Gaze Tracking
---

Notes on the models I trained to estimate gaze position from webcam face landmarks, in the **EyeGesturesDataAcquisitionSoftware** project. The pipeline: `dac.html` records per-frame MediaPipe face landmarks + the on-screen target the user fixates into CSVs (stored in `datalake/`), and `training/train.py` trains and evaluates models that map landmarks → gaze point.

## The task

Each camera frame (~30 fps) gives **478 face landmarks** (x, y). The models consume only the **36 eye-contour + iris landmarks (72 features)** — full-face features inject head-pose noise and hurt at this data scale. Output is the gaze point (x, y) on screen, supervised by the fixation target. With one 2-minute session (3,766 frames) we evaluate on 5 held-out experiment targets (518 frames), in pixel error on a 2048×1010 screen.

## Model comparison (held-out experiment targets)

| | Ridge | MLP | GRU | Temporal transformer | EyeGestures engine |
| ---- | ---- | ---- | ---- | ---- | ---- |
| Input | 1 frame | 1 frame | 20-frame window | 20-frame window | internal |
| Type | linear (tabular) | feedforward (tabular) | recurrent (time series) | attention (time series) | WASM engine, calibrated |
| Mean error | 393 px | 237 px | **201 px** | 249 px | 185 px |
| Median error | 392 px | 191 px | 157 px | **147 px** | 104 px |

Baselines for scale: predicting screen center always = 610 px mean.

## Cheat sheet

- [[Ridge Regression]] — linear map landmarks → gaze; the sanity baseline
- [[MLP]] — per-frame feedforward network; tabular model, no memory
- [[GRU]] — causal recurrent window model; our best mean error, deployed to the browser
- [[Temporal Transformer]] — causal attention over frame windows; best median, worse tail on small data

## Reading the results

- **Time beats tabular**: windowed models (GRU/transformer) beat per-frame models — gaze is smooth, and past frames correct landmark jitter.
- **Eye features beat all 478 landmarks** (MLP: ~620 px → 237 px): head pose is noise for this task at small data scale. Scale-invariant iris-relative features were *worse* (306 px) — absolute head position carries real signal about screen position.
- **GRU vs transformer**: with ~3k frames the GRU's recurrent bias wins on mean error; the transformer wins on median but has a worse tail. Expect the transformer to pull ahead with more sessions in the datalake.
- The learned models already approach the calibrated EyeGestures engine (185 px) from a single session — more recordings (users, screens, positions) are the main lever.

## In our stack

`training/train.py` (uv project) trains all four + baselines, auto-splits sessions, and writes `runs/convergence.png` + `metrics.json`. `training/export_js.py` + `gaze_model.js` + `verify.html` run the GRU live in the browser next to the engine (green dot = model, blue = EyeGestures) — JS forward pass verified against PyTorch to ~3e-6. `training/evaluate.py` scores checkpoints on any new recording CSV.
