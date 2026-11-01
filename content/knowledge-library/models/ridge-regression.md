---
title: Ridge Regression
---

The simplest learned model: a **linear map** from inputs to outputs, fit by least squares with L2 regularization. The "ridge" penalty (shrinkage of the weights) keeps the solution stable when features are correlated — which real-world features usually are.

## Core Idea

Predict the output as a weighted sum of the inputs:

$$\hat{y} = Wx + b, \qquad W \in \mathbb{R}^{d_{out} \times d_{in}}$$

The weights are chosen to minimize the squared error plus a penalty on their size:

$$\min_W \; \|y - Wx\|_2^2 + \alpha \|W\|_2^2$$

There is a closed-form solution, so training is instant and there is nothing to tune except $\alpha$.

## Key Features

- **No hyperparameters to speak of** — just the regularization strength $\alpha$.
- **Interpretable**: each weight says how much an input feature moves the prediction.
- **Assumes linearity**: fine as a sanity baseline, but real input → output maps are often not truly linear.

## Sources

- [Ridge regression — Wikipedia](https://en.wikipedia.org/wiki/Ridge_regression)
- Hoerl & Kennard (1970), *Ridge Regression: Biased Estimation for Nonorthogonal Problems*
