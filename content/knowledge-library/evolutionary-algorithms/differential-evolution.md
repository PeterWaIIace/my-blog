---
title: Differential Evolution
---

A simple yet robust **evolutionary algorithm** for continuous optimization, using **vector differences** to guide the search for optimal solutions.

## Evolutionary Lineage
Differential Evolution (DE) is part of the **evolutionary algorithms (EAs)** family and shares similarities with other continuous optimization methods. Below is a graph showing its connections to other key algorithms:

```mermaid
graph TD
    A[Evolutionary Algorithms] --> B[Genetic Algorithms]
    A --> C[Evolution Strategies]
    C --> D[(1+1)-ES]
    C --> E[CMA-ES]
    A --> F[Differential Evolution]
    D --> E
```

### Key Connections:
1. **Evolution Strategies (ES)**:
   - Like ES, DE is designed for **continuous optimization** but uses **vector differences** instead of adaptive mutation.
2. **CMA-ES**:
   - CMA-ES adapts the **covariance matrix** of a Gaussian distribution, while DE relies on **differential mutation**.
3. **Genetic Algorithms (GAs)**:
   - DE shares GAs' population-based approach but focuses on **real-valued vectors** and **differential mutation** rather than crossover.

## Core Idea
DE optimizes a problem by maintaining a **population of candidate solutions** and iteratively improving them using **vector differences**. It is particularly effective for:
- Continuous optimization problems.
- Noisy or non-linear fitness landscapes.
- Problems where traditional gradient-based methods fail.

### Algorithm Overview
1. **Initialization**: Create a random population of $ N $ candidate solutions.
2. **Mutation**: Generate a mutant vector by adding the weighted difference of two random vectors to a third vector:
   $$\mathbf{v}_i = \mathbf{x}_{r1} + F \cdot (\mathbf{x}_{r2} - \mathbf{x}_{r3})$$
   where $ F $ is the **scaling factor** and $ r1, r2, r3 $ are distinct indices.
3. **Crossover**: Combine the mutant vector with a target vector to create a trial vector:
   $$u_{i,j} = \begin{cases}
      v_{i,j} & \text{if } rand(0,1) \leq CR \text{ or } j = j_{rand} \\
      x_{i,j} & \text{otherwise}
   \end{cases}$$
   where $ CR $ is the **crossover probability**.
4. **Selection**: Replace the target vector if the trial vector has better fitness:
   $$\mathbf{x}_i = \begin{cases}
      \mathbf{u}_i & \text{if } f(\mathbf{u}_i) \leq f(\mathbf{x}_i) \\
      \mathbf{x}_i & \text{otherwise}
   \end{cases}$$

## Key Features
- **Simple and Robust**: Few hyperparameters to tune ($ F $, $ CR $, and population size).
- **Vector Differences**: Exploits the structure of the search space to guide mutations.
- **No Gradient Information**: Works for black-box optimization problems.
- **Parallelizable**: Population-based approach allows for parallel evaluation.

## Applications
- **Hyperparameter Tuning**: Optimizing machine learning models.
- **Engineering Design**: Structural optimization, control systems.
- **Robotics**: Trajectory optimization.
- **Benchmarking**: Widely used in optimization competitions.

## See Also
- [CMA-ES](./cma-es.md): A state-of-the-art evolutionary strategy with adaptive step-size and covariance matrix adaptation.
- [(1+1)-ES](./(1+1)-es.md): The simplest evolution strategy, focusing on mutation and elitist selection.
- [Genetic Algorithm](./genetic-algorithm.md): A population-based approach emphasizing crossover and diversity.
