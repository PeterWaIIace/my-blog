---
title: Covariance Matrix Adaptation Evolution Strategy (CMA-ES)
---

CMA-ES is an evolutionary algorithm for **continuous black-box optimization**: you can evaluate a candidate solution, but you cannot compute gradients. Its trick is to learn *the shape of the good region* as it searches.

## The one-paragraph version

CMA-ES keeps a multivariate Gaussian distribution — think of it as a **cloud** floating over the search space. Each generation it:

1. Samples a batch of candidates from the cloud.
2. Ranks them by fitness and keeps the best.
3. Moves the cloud toward the winners and reshapes it to match them.

Over time, the cloud drifts toward the optimum and stretches along whatever valleys and ridges lead there. The name comes from step 3: adapting the cloud's **covariance matrix** is what lets it learn that shape.

## Key Features
- **Adaptive step size**: The cloud's overall radius shrinks near an optimum and grows when progress stalls — automatically.
- **Learns correlations**: The cloud stretches along ridges and elongated valleys that would trap an axis-aligned search.
- **Rank-based selection**: Only the *ordering* of fitness values matters, never their magnitudes — so noisy fitness functions don't confuse it.
- **Few knobs**: Internal learning rates have well-tested defaults. You only pick the initial guess, the initial step size, and the evaluation budget.
- **Cost**: The covariance matrix costs $O(n^2)$ storage — fine for hundreds of dimensions, a concern for thousands.

## A picture: searching a tilted valley

Imagine finding the lowest point in a long, tilted valley:

1. Generate a cloud of candidates around the current center.
2. Evaluate them and move the center toward the better ones.
3. Stretch and turn the cloud toward directions that have been useful.
4. Adjust the overall radius and repeat.

Three quantities control the cloud:

- The **mean** $m$ is its center.
- The **step size** $\sigma$ is its overall radius.
- The **covariance matrix** $C$ is its shape and orientation.

The actual sampling covariance is $\sigma^2C$. The diagonal of $C$ gives the spread along each coordinate; the off-diagonal entries describe how coordinates move *together*. A positive off-diagonal entry means "when this coordinate grows, that one tends to grow too."

## Where matrix decomposition fits

Sampling from $\mathcal{N}(0, C)$ requires shaping raw noise. Start with independent Gaussian noise $z\sim\mathcal{N}(0,I)$ — a round cloud — and transform it:

$$x=m+\sigma Az,\qquad AA^T=C$$

Any $A$ satisfying $AA^T=C$ maps the round cloud onto the correct ellipsoid. Two standard choices:

- **[[Eigendecomposition]]**: $C=Q\Lambda Q^T$, so use $A=Q\Lambda^{1/2}$. The eigenvectors point along the ellipsoid's axes; the square roots of the eigenvalues give each axis's length.
- **[[Cholesky Decomposition]]**: $C=LL^T$, so use $A=L$. The triangular factor $L$ produces the same noise whenever $C$ is positive definite.

Standard CMA-ES uses the eigendecomposition. Step-size adaptation needs $C^{-1/2}$ anyway, and the eigendecomposition provides it at little extra cost. Cholesky-based variants adapt $L$ directly instead — cheaper, but the update rules must be reworked, so the two decompositions don't slot interchangeably into the algorithm.

## Algorithm Overview

One generation of CMA-ES, step by step.

### 1. Initialization
- A **mean vector** $ \mathbf{m} \in \mathbb{R}^n $ — your initial guess.
- A **covariance matrix** $ \mathbf{C} = \mathbf{I} $ — the cloud starts round, with no preferred direction.
- A **step size** $ \sigma $ — the initial search radius.

### 2. Sampling
Draw $ \lambda $ candidates from the current cloud:
$$\mathbf{x}_i^{(g+1)} \sim \mathbf{m}^{(g)} + \sigma^{(g)} \cdot \mathcal{N}(\mathbf{0}, \mathbf{C}^{(g)}), \quad i = 1, 2, \dots, \lambda$$

### 3. Selection and Recombination
Evaluate all $ \lambda $ candidates, keep the **top $ \mu $**, and move the mean toward them:
$$\mathbf{m}^{(g+1)} = \sum_{i=1}^\mu w_i \mathbf{x}_{i:\lambda}^{(g+1)}$$
where $ w_i $ are weights, typically ordered $ w_1 \geq w_2 \geq \dots \geq w_\mu $. The mean is simply a weighted average of the winners.

### 4. Step Size Adaptation
**Idea**: track how far the mean has moved recently. If it travels farther than chance would predict, the search is working — take bigger steps. If it barely moves, the steps are canceling out — take smaller ones.

$$\mathbf{p}_\sigma^{(g+1)} = (1 - c_\sigma) \mathbf{p}_\sigma^{(g)} + \sqrt{c_\sigma (2 - c_\sigma) \mu_{\text{eff}}} \cdot \mathbf{C}^{(g)^{-1/2}} \cdot \frac{\mathbf{m}^{(g+1)} - \mathbf{m}^{(g)}}{\sigma^{(g)}}$$
$$\sigma^{(g+1)} = \sigma^{(g)} \cdot \exp\left(\frac{c_\sigma}{d_\sigma} \left( \frac{\|\mathbf{p}_\sigma^{(g+1)}\|}{\mathbb{E}[\|\mathcal{N}(\mathbf{0}, \mathbf{I})\|]} - 1 \right)\right)$$

- $ \mathbf{p}_\sigma $ is the **evolution path** — the running record of recent mean movement.
- $ c_\sigma $ and $ d_\sigma $ are learning rates.
- $ \mu_{\text{eff}} = (\sum_{i=1}^\mu w_i^2)^{-1} $ is the **effective selection mass**.

This is called **cumulative step-size adaptation (CSA)**.

### 5. Covariance Matrix Adaptation
**Idea**: reshape the cloud along two kinds of evidence. The **rank-one** update stretches it along the path the mean has recently traveled. The **rank-$ \mu $** update stretches it along the directions in which the winners actually spread out.

$$\mathbf{p}_c^{(g+1)} = (1 - c_c) \mathbf{p}_c^{(g)} + \sqrt{c_c (2 - c_c) \mu_{\text{eff}}} \cdot \frac{\mathbf{m}^{(g+1)} - \mathbf{m}^{(g)}}{\sigma^{(g)}}$$
$$\mathbf{C}^{(g+1)} = (1 - c_1 - c_\mu) \mathbf{C}^{(g)} + c_1 \mathbf{p}_c^{(g+1)} \mathbf{p}_c^{(g+1)^T} + c_\mu \sum_{i=1}^\mu w_i \left( \frac{\mathbf{x}_{i:\lambda}^{(g+1)} - \mathbf{m}^{(g)}}{\sigma^{(g)}} \right) \left( \frac{\mathbf{x}_{i:\lambda}^{(g+1)} - \mathbf{m}^{(g)}}{\sigma^{(g)}} \right)^T$$

- $ \mathbf{p}_c $ is the evolution path used for the covariance update.
- $ c_1 $ and $ c_\mu $ are the learning rates for the rank-one and rank-$ \mu $ updates.

The new $ C $ is a weighted mix: mostly the old shape, plus a stretch along the recent travel direction, plus a stretch along the winners' spread.

## Comparison with Other Algorithms

| **Feature**        | **CMA-ES**                                     | **Genetic Algorithms (GAs)**                     | **Differential Evolution (DE)**       |
|--------------------|------------------------------------------------|--------------------------------------------------|--------------------------------------|
| **Representation** | Real-valued vectors                            | Binary strings, integers, or real-valued vectors | Real-valued vectors                  |
| **Mutation**       | Adaptive Gaussian perturbation                 | Random bit flips or small changes                | Vector differences                   |
| **Crossover**      | Not used (replaced by weighted recombination)  | Combines two parents (e.g., single-point)        | Combines mutant and target vectors   |
| **Selection**      | Rank-based (top $ \mu $ candidates)            | Fitness-proportional (e.g., roulette wheel)      | Greedy (keep the better of the two) |
| **Adaptation**     | Self-adapts step size and covariance matrix    | Fixed mutation rate and crossover probability    | Fixed parameters                     |
| **Problem Type**   | Continuous optimization                        | Discrete or continuous problems                  | Continuous optimization              |
| **Convergence**    | Faster for high-dimensional problems           | Slower but more flexible                        | Effective for noisy problems         |

## Applications
- **Hyperparameter tuning** for machine learning models.
- **Engineering design**: aerodynamics, structures, control systems.
- **Robotics**: trajectory optimization and reinforcement learning.
- **Finance**: portfolio optimization and risk management.
- **Benchmarking**: state-of-the-art on the [BBOB benchmark](https://cma-es.github.io/bbob-benchmark/).

## Example: Minimizing the Sphere Function

The **Sphere function** is the "hello world" of optimization:
$$f(\mathbf{x}) = \sum_{i=1}^n x_i^2$$

Its global minimum sits at $ \mathbf{x}^* = \mathbf{0} $ with $ f(\mathbf{x}^*) = 0 $. CMA-ES solves it reliably, but the evaluation count grows with dimension and required accuracy — treat it as a smoke test, not a speed benchmark.

```python
import cma

# Define the Sphere function
def sphere(x):
    return sum(x**2)

# Run CMA-ES
x0 = [1.0, 2.0, 3.0]  # Initial guess
sigma0 = 0.5           # Initial step size
es = cma.CMAEvolutionStrategy(x0, sigma0)
es.optimize(sphere)

# Print the best solution
print("Best solution:", es.result.xbest)
print("Best fitness:", es.result.fbest)
```

## Further Reading
1. **Original paper**: [Hansen, N., & Ostermeier, A. (2001). Completely Derandomized Self-Adaptation in Evolution Strategies. *Evolutionary Computation*, 9(2), 159–195.](https://doi.org/10.1162/106365601750190398)
2. **Tutorial**: [The CMA-ES Tutorial by Nikolaus Hansen](https://arxiv.org/abs/1604.00772)
3. **Python library**: [`cma` (official implementation)](https://github.com/CMA-ES/pycma)
