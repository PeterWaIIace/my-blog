---
title: Cholesky Decomposition
---

Cholesky decomposition rewrites a real symmetric **positive-definite** matrix as:

$$C=LL^T$$

$L$ is **lower triangular**: everything above its diagonal is zero. Think of it as a practical recipe for building correlated values from independent ones.

Positive definite means $v^TCv>0$ for every nonzero vector $v$. For a covariance matrix, this means there is some variance in every direction. Ordinary Cholesky requires this stronger condition; a covariance matrix with a zero-variance direction is only positive semidefinite and may fail the factorization.

## A small example

$$
C=\begin{bmatrix}4&2\\2&3\end{bmatrix},\qquad
L=\begin{bmatrix}2&0\\1&\sqrt{2}\end{bmatrix}
$$

You can find $L$ one entry at a time:

1. First diagonal: $L_{11}=\sqrt{4}=2$.
2. Entry below it: $L_{21}=2/2=1$.
3. Second diagonal: $L_{22}=\sqrt{3-1^2}=\sqrt{2}$.

Multiplying $LL^T$ gives back $C$.

## Turning independent noise into correlated noise

Start with two independent standard normal values $z_1,z_2$. Set $y=Lz$:

$$y_1=2z_1,\qquad y_2=z_1+\sqrt{2}z_2$$

- $y_1$ has variance $4$.
- $y_2$ has variance $1+2=3$.
- Both share $z_1$, giving covariance $2$.

That is exactly the covariance described by $C$.

More generally, if $z\sim\mathcal{N}(0,I)$, then:

$$x=m+\sigma Lz\sim\mathcal{N}(m,\sigma^2C)$$

## Connection to CMA-ES

[[Covariance Matrix Adaptation Evolution Strategy (CMA-ES)|CMA-ES]] needs a way to sample its learned Gaussian search distribution. Cholesky and [[Eigendecomposition]] both provide suitable factors:

| Question                  | Eigendecomposition                       | Cholesky                                  |
| ------------------------- | ---------------------------------------- | ----------------------------------------- |
| Factorization             | $C=Q\Lambda Q^T$                         | $C=LL^T$                                  |
| What does it reveal?      | Principal directions and their variances | A triangular recipe for correlated values |
| Sampling                  | $Q\Lambda^{1/2}z$                        | $Lz$                                      |
| Matrix requirement here   | Symmetric positive semidefinite          | Symmetric positive definite               |
| Typical dense computation | $O(n^3)$                                 | $O(n^3)$, usually cheaper in practice     |

They generate the **same distribution**, but generally different samples from the same noise vector $z$.

Standard CMA-ES commonly uses eigendecomposition, also needed for its symmetric inverse square root. Cholesky-based variants can maintain factors through updates. Replacing the sampling factor alone does not replace every use of the eigendecomposition: standard step-size adaptation also uses $C^{-1/2}$, and $L^{-1}$ is generally a different matrix.

## Another use: solving equations

To solve $Cx=b$, first factor $C=LL^T$, then solve two triangular systems:

1. $Lu=b$ — work from the first row downward.
2. $L^Tx=u$ — work from the last row upward.

This avoids explicitly calculating $C^{-1}$.

## Python example

```python
import numpy as np

C = np.array([[4.0, 2.0], [2.0, 3.0]])
L = np.linalg.cholesky(C)

rng = np.random.default_rng(42)
y = L @ rng.standard_normal(2)

assert np.allclose(L @ L.T, C)

b = np.array([1.0, 2.0])
u = np.linalg.solve(L, b)
x = np.linalg.solve(L.T, u)
assert np.allclose(C @ x, b)
```

## Sources

- [NumPy: `cholesky`](https://numpy.org/doc/stable/reference/generated/numpy.linalg.cholesky.html)
- [CMA-ES reference site: Cholesky-based variants](https://cma-es.github.io/)
