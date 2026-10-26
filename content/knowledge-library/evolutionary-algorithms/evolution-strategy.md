---
title: Evolution Strategy
---

In contrast to Genetic Algorithms, Evolution Strategies (ES) were initially developed for **parameter optimization**. According to Rechenberg[35], the first Evolution Strategies were developed in 1964 at the Technical University of Berlin (TUB). The idea was to imitate the principles of organic evolution in experimental parameter optimization for applications such as pipe bending or PID control for nonlinear systems. In his words:

> “The method of organic evolution represents an optimal strategy for the adaptation of living things to their environment… [and] … it should therefore be worthwhile to take over the principles of biological evolution for the optimization of technical systems.”[45]

## Evolutionary Lineage
Evolution Strategies (ES) are a cornerstone of the **evolutionary algorithms (EAs)** family and have inspired many modern optimization methods. Below is a graph showing their connections to other key algorithms:

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
1. **(1+1)-ES**:
   - The simplest ES, focusing on **mutation** and **elitist selection**. It serves as a theoretical testbed and inspired modern step-size adaptation techniques.
2. **CMA-ES**:
   - A state-of-the-art ES that extends the (1+1)-ES with **covariance matrix adaptation** for high-dimensional continuous optimization.
3. **Genetic Algorithms (GAs)**:
   - While GAs emphasize **crossover** and **population diversity**, ES focuses on **mutation** and **adaptive step-size control**.
4. **Differential Evolution (DE)**:
   - Another continuous optimization algorithm, but DE relies on **vector differences** rather than adaptive mutation.

## Core Principles
- **Mutation-Centric**: ES relies primarily on **mutation** to explore the search space, with adaptive step sizes to balance exploration and exploitation.
- **Self-Adaptation**: Many ES variants adapt their parameters (e.g., mutation strength) during the search.
- **Continuous Optimization**: ES is designed for **real-valued parameter optimization**, making it ideal for engineering and machine learning applications.

## Modern Descendants
- **CMA-ES**: The most prominent descendant of ES, widely used for black-box optimization.
- **NES (Natural Evolution Strategies)**: A family of ES that uses **natural gradient descent** to adapt the search distribution.

## See Also
- [Evolutionary Algorithms Overview](./evolutionary-algorithms.md): A unified introduction to the family of evolutionary algorithms.
- [CMA-ES](./cma-es.md): A state-of-the-art evolution strategy with adaptive step-size and covariance matrix adaptation.
- [(1+1)-ES](./(1+1)-es.md): The simplest evolution strategy, focusing on mutation and elitist selection.
- [Genetic Algorithm](./genetic-algorithm.md): A population-based approach emphasizing crossover and diversity.
- [Differential Evolution](./differential-evolution.md): A robust optimizer using vector differences for continuous problems.