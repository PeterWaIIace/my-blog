---
title: GRU
---

A **gated recurrent unit** — a recurrent neural network that processes a **window of frames** and carries a hidden state across them. In our gaze tracker it is the best-performing learned model and the one deployed to the browser.

## Core Idea

At each frame the GRU updates a hidden state $h_t$ using three gates that decide what to keep and what to forget:

$$r_t = \sigma(W_r x_t + U_r h_{t-1} + b_r) \quad \text{(reset gate)}$$
$$z_t = \sigma(W_z x_t + U_z h_{t-1} + b_z) \quad \text{(update gate)}$$
$$n_t = \tanh(W_n x_t + r_t \odot (U_n h_{t-1})) \quad \text{(candidate)}$$
$$h_t = (1 - z_t) \odot n_t + z_t \odot h_{t-1}$$

The update gate $z_t$ blends the old state with the new candidate — that is the "memory". Because each step only reads the past, the model is **causal** and usable in real time. We predict the gaze from the last frame's state: $\hat{y} = W_{out} h_T + b_{out}$.

## Key Features

- **Temporal smoothing for free**: gaze is smooth in time, so the hidden state averages out per-frame landmark jitter.
- **Cheap**: one small state vector; much lighter than attention over long histories.
- **Vanishing-gradient resistant** compared to vanilla RNNs, thanks to the gates (though LSTMs go further on very long sequences).

## In our stack

`GRUNet` in `training/mlp_gru.py`: 72 eye features → linear projection to 128 → 1-layer GRU (hidden 128) over a **20-frame window (~0.7 s at 30 fps)** → linear head to (x, y). Windows never cross target changes or face-tracking gaps. Trained on calibration + 20 experiment targets, early-stopped on held-out calibration spots. Held-out result: **201 px mean / 157 px median**, vs 237 px for the per-frame [[MLP]] and 185 px for the calibrated EyeGestures engine — from a single 2-minute session. Exported to JSON (`training/export_js.py`) and run live in `verify.html` via `gaze_model.js`, verified against PyTorch to ~3e-6.

## Sources

- [Cho et al. (2014), Learning Phrase Representations using RNN Encoder–Decoder](https://arxiv.org/abs/1406.1078)
- [PyTorch GRU docs](https://pytorch.org/docs/stable/generated/torch.nn.GRU.html)
