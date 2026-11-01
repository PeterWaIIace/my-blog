---
title: MLP
---

A **multi-layer perceptron** — the plain feedforward neural network. It maps **one input vector** to an output vector: no memory, no notion of time. For sequence or tabular data where each sample is independent, it is the default "tabular" model.

## Core Idea

Stack linear layers with a nonlinearity (ReLU) between them:

$$h_1 = \mathrm{ReLU}(W_1 x + b_1), \quad h_2 = \mathrm{ReLU}(W_2 h_1 + b_2), \quad \hat{y} = W_3 h_2 + b_3$$

Each layer learns increasingly abstract combinations of the inputs. Trained with gradient descent on a loss between the predicted and true output (e.g. squared error for regression).

## Key Features

- **Sample-independent**: input *t* and input *t+1* are unrelated to the model — no temporal smoothing.
- **Universal approximator**: can in principle represent any input → output map, given enough width and data.
- **Needs enough data**: with small datasets it overfits; regularization (dropout, weight decay, early stopping) matters.

## Sources

- [Deep Learning — Goodfellow, Bengio, Courville, ch. 6](https://www.deeplearningbook.org/contents/mlp.html)
