---
title: Temporal Transformer
---

A **transformer over a sequence**: instead of a recurrent state, each step in the window becomes a token and **self-attention** lets every step look at all others. Same sequence task as the [[GRU]], different mechanism for using history.

## Core Idea

1. **Tokenize**: project each step's feature vector to a model dimension and add a learned **positional embedding** (the only place order enters — attention itself is order-agnostic):
$$z_t = W_{tok} x_t + p_t$$
2. **Attend** (multi-head self-attention, **causally masked** so step *t* sees only steps ≤ *t*):
$$\mathrm{Attn}(Q, K, V) = \mathrm{softmax}\!\left(\frac{QK^T}{\sqrt{d_k}} + M\right) V$$
3. **Read out** the last step's token through a small linear head: $\hat{y} = W_{out} z_T + b_{out}$.

With causal masking the model is real-time usable: at step *t* it only depends on steps ≤ *t*.

## Key Features

- **Global receptive field within the window**: every step directly attends to every past step — no information has to survive through a single recurrent state.
- **Parallel training**: unlike the GRU, all steps of a window are processed in parallel.
- **Data-hungry**: no recurrent inductive bias, so on small datasets it usually loses to the GRU, though it can win on typical-case error with enough data.

## Sources

- [Vaswani et al. (2017), Attention Is All You Need](https://arxiv.org/abs/1706.03762)
- [PyTorch TransformerEncoder docs](https://pytorch.org/docs/stable/generated/torch.nn.TransformerEncoder.html)
