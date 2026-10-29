---
title: Eigendecomposition
---

**Matrix decomposition** means rewriting a matrix as a product of simpler matrices. Eigendecomposition explains a matrix through its special directions and how much it stretches them.

## The simple idea

A matrix can transform a vector: stretch it, rotate it, or mix its coordinates. An **eigenvector** is a nonzero vector whose line stays unchanged by that transformation:

$$Cv=\lambda v$$

- $v$: an eigenvector — a special direction.
- $\lambda$: its eigenvalue — the scale applied along that direction. A negative value also flips the direction.

For covariance matrices, eigenvalues are nonnegative: they describe **variance**, not standard deviation.

## Breaking down a covariance matrix

A real covariance matrix is symmetric, so it has an orthonormal set of eigenvectors:

$$C=Q\Lambda Q^T$$

- $Q$: eigenvectors as columns — perpendicular unit directions.
- $\Lambda$: eigenvalues on the diagonal — variance along each direction.
- $Q^T$: the transpose of $Q$, which is also its inverse.

Read the product right to left: **express the input in the special directions, scale it, then transform back**.

This form is guaranteed for real symmetric matrices. A general square matrix may not have enough independent eigenvectors for a decomposition of this kind.

## Example: a tilted search cloud

$$C=\begin{bmatrix}2&1\\1&2\end{bmatrix}$$

Its eigenvectors and eigenvalues are:

- $(1,1)/\sqrt{2}$ with eigenvalue $3$: large variance along the upward diagonal.
- $(1,-1)/\sqrt{2}$ with eigenvalue $1$: smaller variance along the downward diagonal.

A Gaussian with this covariance looks like a tilted ellipse. Its standard deviations along these axes are $\sqrt{3}$ and $1$.

## Why CMA-ES uses it

[[Covariance Matrix Adaptation Evolution Strategy (CMA-ES)|CMA-ES]] learns a covariance matrix to shape its search. To turn independent Gaussian noise $z\sim\mathcal{N}(0,I)$ into a candidate, use:

$$x=m+\sigma Q\Lambda^{1/2}z$$

Here $m$ is the search center, $\sigma$ is the overall step size, and $\Lambda^{1/2}$ contains the square roots of the eigenvalues. The candidate covariance is $\sigma^2C$.

**Stretch a round cloud, orient it along the learned directions, then move it to the search center.**

The same decomposition gives the symmetric inverse square root used in standard cumulative step-size adaptation:

$$C^{-1/2}=Q\Lambda^{-1/2}Q^T$$

This removes the learned stretching and correlation before measuring search progress. It requires positive eigenvalues.

## Python example

```python
import numpy as np

C = np.array([[2.0, 1.0], [1.0, 2.0]])
values, Q = np.linalg.eigh(C)  # For real symmetric matrices
A = Q @ np.diag(np.sqrt(values))

rng = np.random.default_rng(42)
z = rng.standard_normal(2)
y = A @ z  # A zero-mean Gaussian sample with covariance C

assert np.allclose(A @ A.T, C)
```

The sampling factor $A$ satisfies $AA^T=C$. It is not the symmetric square root $C^{1/2}=Q\Lambda^{1/2}Q^T$, though both can generate the same Gaussian distribution.

For a triangular sampling factor instead, see [[Cholesky Decomposition]].

## Sources

- [Hansen: The CMA Evolution Strategy — A Tutorial](https://arxiv.org/abs/1604.00772)
- [NumPy: `eigh`](https://numpy.org/doc/stable/reference/generated/numpy.linalg.eigh.html)
