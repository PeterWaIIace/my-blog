---
title: Ridge Regression
---

The simplest possible model for gaze estimation: a **linear map** from landmark coordinates to screen position, fit by least squares with L2 regularization. The "ridge" penalty (shrinkage of the weights) keeps the solution stable when features are correlated — which face landmarks are, heavily.

## Core Idea

Predict the gaze point as a weighted sum of the inputs:

$$\hat{y} = Wx + b, \qquad W \in \mathbb{R}^{2 \times F}$$  

The weights are chosen to minimize the squared error plus a penalty on their size:

$$\min_W \; \|y - Wx\|_2^2 + \alpha \|W\|_2^2$$

There is a closed-form solution, so training is instant and there is nothing to tune except $\alpha$.

## Key Features

- **No hyperparameters to speak of** — just the regularization strength $\alpha$.
- **Interpretable**: each weight says how much a landmark coordinate moves the predicted gaze.
- **Assumes linearity**: fine as a sanity baseline, but the landmark → gaze map is not truly linear (head pose, eye geometry).

## In our stack

In `training/train.py` it is the first learned baseline (`ridge_all` on all 478 landmarks, `ridge_eyes` on the 72 eye features). On held-out experiment targets it reaches **393 px mean error** (all landmarks) vs 610 px for predicting the screen center and 185 px for the calibrated EyeGestures engine — a useful lower bound for "how far does linear get". We picked $\alpha$ from a few values on a validation split.

## Sources

- [Ridge regression — Wikipedia](https://en.wikipedia.org/wiki/Ridge_regression)
- Hoerl & Kennard (1970), *Ridge Regression: Biased Estimation for Nonorthogonal Problems*
