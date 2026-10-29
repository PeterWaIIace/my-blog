---
title: Covariance Matrix Adaptation Evolution Strategy (CMA-ES)
---

A state-of-the-art evolutionary algorithm for continuous black-box optimization, renowned for its adaptive search capabilities and robustness.

## Evolutionary Lineage
CMA-ES is a modern descendant of **evolution strategies (ES)**, which themselves are part of the broader family of **evolutionary algorithms (EAs)**. Below is a graph showing its connections to other key algorithms:

```mermaid
graph TD
    A[Evolutionary Algorithms] --> B[Genetic Algorithms]
    A --> C[Evolution Strategies]
    C --> D[(1+1)-ES]
    C --> E[CMA-ES]
    A --> F[Differential Evolution]
    D --> E
    E --> G[Modern Variants: BIPOP-CMA-ES, IPOP-CMA-ES]
```

### Key Connections:
1. **Evolution Strategies (ES)**:
   - CMA-ES inherits the core principles of ES, such as **mutation** and **selection**, but introduces **covariance matrix adaptation** for improved performance.
2. **(1+1)-ES**:
   - The simplest ES, which inspired the **step-size adaptation** mechanism in CMA-ES.
3. **Genetic Algorithms (GAs)**:
   - While GAs focus on **crossover** and **discrete representations**, CMA-ES emphasizes **adaptive mutation** and **continuous optimization**.
4. **Differential Evolution (DE)**:
   - Another continuous optimization algorithm, but DE relies on **vector differences** rather than covariance adaptation.
5. **Modern Variants**:
   - **BIPOP-CMA-ES** and **IPOP-CMA-ES** extend CMA-ES with **restart strategies** for global optimization.

## Core Idea
CMA-ES is an **evolution strategy (ES)** that adapts the **covariance matrix** of a multivariate Gaussian distribution to guide the search for optimal solutions. Unlike traditional genetic algorithms, CMA-ES:
- Works directly with **real-valued vectors**.
- Self-adapts its **step size** and **covariance matrix** to exploit problem structure.
- Uses **rank-based selection** to handle noisy or non-linear fitness landscapes.

## Key Features
- **Adaptive Step Size**: Automatically adjusts the scale of mutations to balance exploration and exploitation.
- **Covariance Matrix Adaptation**: Learns correlations between variables to navigate complex fitness landscapes (e.g., elongated valleys or ridges).
- **Useful Defaults**: Internal adaptation parameters have standard defaults; the initial guess, initial step size, evaluation budget, and sometimes population size still need choosing.
- **Robustness**: Useful on noisy or non-convex problems. Full covariance adaptation needs $O(n^2)$ storage, so very high-dimensional tasks may need specialized variants.

## A simple picture: learning the shape of the search

Imagine searching for the lowest point in a long, tilted valley:

1. Generate a cloud of candidate solutions around the current center.
2. Evaluate them and move the center toward the better candidates.
3. Stretch and turn the cloud toward directions that have been useful.
4. Adjust the overall search radius and repeat.

The **mean** $m$ is the center, the **step size** $\sigma$ controls the overall radius, and the **covariance matrix** $C$ controls shape and orientation. The actual sampling covariance is $\sigma^2C$.

The diagonal entries of $C$ describe variance in each coordinate; off-diagonal entries describe how coordinates vary together. A positive off-diagonal entry encourages those coordinates to move in the same direction.

### Where matrix decomposition fits

To sample that cloud, start with independent Gaussian noise $z\sim\mathcal{N}(0,I)$ and transform it:

$$x=m+\sigma Az,\qquad AA^T=C$$

Two ways to find such a factor $A$ are:

- **[[Eigendecomposition]]**: $C=Q\Lambda Q^T$, so use $A=Q\Lambda^{1/2}$. The eigenvectors give the cloud's main directions; square roots of eigenvalues give its relative widths.
- **[[Cholesky Decomposition]]**: $C=LL^T$, so use $A=L$. This gives a triangular recipe for generating the same Gaussian distribution when $C$ is positive definite.

Standard CMA-ES commonly uses eigendecomposition because it also supplies the symmetric inverse square root used in step-size adaptation. Cholesky-based variants adapt the factorization and related updates; the two methods are not interchangeable in every part of the algorithm.

## Algorithm Overview
### 1. Initialization
- Start with a **mean vector** $ \mathbf{m} \in \mathbb{R}^n $ (initial guess for the solution).
- Initialize the **covariance matrix** $ \mathbf{C} = \mathbf{I} $ (identity matrix, representing an isotropic search distribution).
- Initialize the **step size** $ \sigma $ (controls the spread of the search distribution).

### 2. Sampling
At each generation $ g $, sample $ \lambda $ candidate solutions from a multivariate Gaussian distribution:
$$\mathbf{x}_i^{(g+1)} \sim \mathbf{m}^{(g)} + \sigma^{(g)} \cdot \mathcal{N}(\mathbf{0}, \mathbf{C}^{(g)}), \quad i = 1, 2, \dots, \lambda$$

### 3. Selection and Recombination
- Evaluate the fitness of all $ \lambda $ candidates.
- Select the **top $ \mu $** candidates (based on fitness rank).
- Update the mean vector $ \mathbf{m}^{(g+1)} $ as a weighted average of the top $ \mu $ candidates:
  $$\mathbf{m}^{(g+1)} = \sum_{i=1}^\mu w_i \mathbf{x}_{i:\lambda}^{(g+1)}$$
  where $ w_i $ are weights (e.g., $ w_1 \geq w_2 \geq \dots \geq w_\mu $).

### 4. Step Size Adaptation
Adapt the step size $ \sigma $ using the **cumulative path length control (CSA)** method:
$$\mathbf{p}_\sigma^{(g+1)} = (1 - c_\sigma) \mathbf{p}_\sigma^{(g)} + \sqrt{c_\sigma (2 - c_\sigma) \mu_{\text{eff}}} \cdot \mathbf{C}^{(g)^{-1/2}} \cdot \frac{\mathbf{m}^{(g+1)} - \mathbf{m}^{(g)}}{\sigma^{(g)}}$$
$$\sigma^{(g+1)} = \sigma^{(g)} \cdot \exp\left(\frac{c_\sigma}{d_\sigma} \left( \frac{\|\mathbf{p}_\sigma^{(g+1)}\|}{\mathbb{E}[\|\mathcal{N}(\mathbf{0}, \mathbf{I})\|]} - 1 \right)\right)$$
where:
- $ \mathbf{p}_\sigma $ is the **evolution path** for step size adaptation.
- $ c_\sigma $ and $ d_\sigma $ are learning rates.
- $ \mu_{\text{eff}} = (\sum_{i=1}^\mu w_i^2)^{-1} $ is the **effective selection mass**.

### 5. Covariance Matrix Adaptation
Update the covariance matrix $ \mathbf{C} $ to capture correlations between variables:
$$\mathbf{p}_c^{(g+1)} = (1 - c_c) \mathbf{p}_c^{(g)} + \sqrt{c_c (2 - c_c) \mu_{\text{eff}}} \cdot \frac{\mathbf{m}^{(g+1)} - \mathbf{m}^{(g)}}{\sigma^{(g)}}$$
$$\mathbf{C}^{(g+1)} = (1 - c_1 - c_\mu) \mathbf{C}^{(g)} + c_1 \mathbf{p}_c^{(g+1)} \mathbf{p}_c^{(g+1)^T} + c_\mu \sum_{i=1}^\mu w_i \left( \frac{\mathbf{x}_{i:\lambda}^{(g+1)} - \mathbf{m}^{(g)}}{\sigma^{(g)}} \right) \left( \frac{\mathbf{x}_{i:\lambda}^{(g+1)} - \mathbf{m}^{(g)}}{\sigma^{(g)}} \right)^T$$
where:
- $ \mathbf{p}_c $ is the **evolution path** for covariance adaptation.
- $ c_1 $ and $ c_\mu $ are learning rates for the rank-one and rank-$ \mu $ updates.

## Comparison with Other Algorithms
| **Feature**               | **CMA-ES**                                      | **Genetic Algorithms (GAs)**                     | **Differential Evolution (DE)**                  |
|---------------------------|------------------------------------------------|------------------------------------------------|------------------------------------------------|
| **Representation**        | Real-valued vectors                            | Binary strings, integers, or real-valued vectors | Real-valued vectors                            |
| **Mutation**              | Adaptive Gaussian perturbation                 | Random bit flips or small changes              | Vector differences                             |
| **Crossover**             | Not used (replaced by weighted recombination)  | Combines two parents (e.g., single-point)      | Combines mutant and target vectors             |
| **Selection**             | Rank-based (top $ \mu $ candidates)            | Fitness-proportional (e.g., roulette wheel)    | Greedy (keep the better of trial/target)       |
| **Adaptation**            | Self-adapts step size and covariance matrix    | Fixed mutation rate and crossover probability  | Fixed parameters                               |
| **Problem Type**          | Continuous optimization                        | Discrete or continuous problems                | Continuous optimization                        |
| **Convergence**           | Faster for high-dimensional problems           | Slower but more flexible                       | Effective for noisy problems                   |

## Applications
- **Hyperparameter Tuning**: Optimizing machine learning models (e.g., neural networks, support vector machines).
- **Engineering Design**: Aerodynamics, structural optimization, and control systems.
- **Robotics**: Trajectory optimization and reinforcement learning.
- **Finance**: Portfolio optimization and risk management.
- **Benchmarking**: State-of-the-art performance on the [BBOB benchmark](https://cma-es.github.io/bbob-benchmark/).

## Example: Minimizing the Sphere Function
The **Sphere function** is a simple benchmark for optimization algorithms:
$$f(\mathbf{x}) = \sum_{i=1}^n x_i^2$$
- **Global minimum**: $ \mathbf{x}^* = \mathbf{0} $ with $ f(\mathbf{x}^*) = 0 $.
- **CMA-ES Performance**: A useful basic test, but the number of evaluations depends on dimension, initialization, and the requested accuracy; there is no fixed evaluation count to reach the exact minimum.

### Python Implementation
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
1. **Original Paper**:
   - [Hansen, N., & Ostermeier, A. (2001). Completely Derandomized Self-Adaptation in Evolution Strategies. *Evolutionary Computation*, 9(2), 159–195.](https://doi.org/10.1162/106365601750190398)
2. **Tutorial**:
   - [CMA-ES Tutorial by Nikolaus Hansen](https://arxiv.org/abs/1604.00772)
3. **Python Library**:
   - [`cma` (official implementation)](https://github.com/CMA-ES/pycma)
