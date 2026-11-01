---
title: MLP
---

A **multi-layer perceptron** — the plain feedforward neural network. For gaze estimation it maps **one frame** of landmark coordinates to a screen point. It is the "tabular" model in our comparison: no memory, no notion of time.

## Core Idea

Stack linear layers with a nonlinearity (ReLU) between them:

$$h_1 = \mathrm{ReLU}(W_1 x + b_1), \quad h_2 = \mathrm{ReLU}(W_2 h_1 + b_2), \quad \hat{y} = W_3 h_2 + b_3$$

Each layer learns increasingly abstract combinations of the inputs. Trained with gradient descent on the squared error between predicted and true gaze point.

## Key Features

- **Per-frame**: frame *t* and frame *t+1* are unrelated to the model — no temporal smoothing.
- **Universal approximator**: can in principle represent any landmark → gaze map, given enough width and data.
- **Needs enough data**: with small datasets it overfits; regularization (dropout, weight decay, early stopping) matters.

## In our stack

In `training/mlp_gru.py` (`MLP`): 72 standardized eye/iris landmark features → 256 → 128 → 2, with ReLU and dropout, trained with Adam + early stopping. On held-out experiment targets: **237 px mean / 191 px median**, clearly better than [[Ridge Regression]] (393 px) but worse than the windowed [[GRU]] (201 px) — showing that temporal context matters more than extra per-frame capacity here. Using all 478 landmarks instead of the 72 eye features made it much worse (~620 px): head-pose noise swamps the signal at our data scale.

## Sources

- [Deep Learning — Goodfellow, Bengio, Courville, ch. 6](https://www.deeplearningbook.org/contents/mlp.html)
