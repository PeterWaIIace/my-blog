---
title: GRU
---

A **gated recurrent unit** — a recurrent neural network that processes a sequence step by step, carrying a hidden state across steps. Because each step only reads the past, the model is **causal** and usable in real time.

## Core Idea

At each step the GRU updates a hidden state $h_t$ using gates that decide what to keep and what to forget:

$$r_t = \sigma(W_r x_t + U_r h_{t-1} + b_r) \quad \text{(reset gate)}$$
$$z_t = \sigma(W_z x_t + U_z h_{t-1} + b_z) \quad \text{(update gate)}$$
$$n_t = \tanh(W_n x_t + r_t \odot (U_n h_{t-1})) \quad \text{(candidate)}$$
$$h_t = (1 - z_t) \odot n_t + z_t \odot h_{t-1}$$

The update gate $z_t$ blends the old state with the new candidate — that is the "memory". The output is typically read from the last state: $\hat{y} = W_{out} h_T + b_{out}$.

## Key Features

- **Temporal smoothing for free**: real-world sequences are smooth in time, so the hidden state averages out per-step input noise.
- **Cheap**: one small state vector; much lighter than attention over long histories.
- **Vanishing-gradient resistant** compared to vanilla RNNs, thanks to the gates (though LSTMs go further on very long sequences).

## Sources

- [Cho et al. (2014), Learning Phrase Representations using RNN Encoder–Decoder](https://arxiv.org/abs/1406.1078)
- [PyTorch GRU docs](https://pytorch.org/docs/stable/generated/torch.nn.GRU.html)
