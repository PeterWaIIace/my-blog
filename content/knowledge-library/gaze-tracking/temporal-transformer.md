---
title: Temporal Transformer
---

A **transformer over time**: instead of a recurrent state, each frame in the window becomes a token and **self-attention** lets every frame look at all previous frames. Same task as the [[GRU]], different mechanism for using history.

## Core Idea

1. **Tokenize**: project each frame's feature vector to a model dimension and add a learned **positional embedding** (the only place order enters — attention itself is order-agnostic):
$$z_t = W_{tok} x_t + p_t$$
2. **Attend** (multi-head self-attention, **causally masked** so frame *t* sees only frames ≤ *t*):
$$\mathrm{Attn}(Q, K, V) = \mathrm{softmax}\!\left(\frac{QK^T}{\sqrt{d_k}} + M\right) V$$
3. **Read out** the last frame's token through a small linear head: $\hat{y} = W_{out} z_T + b_{out}$.

With causal masking the model is real-time usable: at frame *t* it only depends on frames ≤ *t*.

## Key Features

- **Global receptive field within the window**: every frame directly attends to every past frame — no information has to survive through a single recurrent state.
- **Parallel training**: unlike the GRU, all frames of a window are processed in parallel.
- **Data-hungry**: no recurrent inductive bias, so on small datasets it usually loses to the GRU on mean error, though it can win on median.

## In our stack

`TemporalTransformer` in `training/mlp_gru.py`: 72 eye features → tokens of dim 128 with learned positional embeddings over the 20-frame window → 2 encoder layers (4 heads, pre-norm) with causal mask → head to (x, y). Same training/evaluation protocol as the GRU. Held-out result: **249 px mean / 147 px median** — best median of all learned models, but a worse tail (p90 572 px vs GRU's 469 px), so the [[GRU]] stays the deployed model for now. With more sessions in the datalake the transformer is the model most likely to overtake it.

## Sources

- [Vaswani et al. (2017), Attention Is All You Need](https://arxiv.org/abs/1706.03762)
- [PyTorch TransformerEncoder docs](https://pytorch.org/docs/stable/generated/torch.nn.TransformerEncoder.html)
