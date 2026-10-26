---
title: Genetic Algorithm
---

A genetic algorithm (GA) is a biologically inspired optimization method that mimics the process of natural evolution. It is widely used to solve complex problems where traditional optimization techniques struggle, such as combinatorial optimization, machine learning, and engineering design.

## Core Concepts

### 1. Initialization
The algorithm begins by generating an initial population of $ N $ candidate solutions (genomes). Each genome $ \mathbf{x}_i $ is a vector representing a potential solution to the problem:
$$\mathbf{x}_i = [x_{i1}, x_{i2}, \dots, x_{in}], \quad i = 1, 2, \dots, N$$
where $ n $ is the dimensionality of the problem. The initial population is typically created randomly to ensure diversity:
$$\mathbf{x}_i \sim \mathcal{U}(\text{lower bound}, \text{upper bound})$$

### 2. Fitness Evaluation
Each genome $ \mathbf{x}_i $ is evaluated using a **fitness function** $ f(\mathbf{x}_i) $, which measures how well the solution performs with respect to the problem objective. For a **maximization problem**, the goal is to maximize $ f(\mathbf{x}_i) $:
$$\text{Fitness}(\mathbf{x}_i) = f(\mathbf{x}_i)$$
For a **minimization problem**, the fitness function is often defined as the negative of the objective function:
$$\text{Fitness}(\mathbf{x}_i) = -f(\mathbf{x}_i)$$

### 3. Selection
Selection determines which genomes are chosen as parents for the next generation. Common selection methods include:

#### a. Roulette Wheel Selection
Genomes are selected with probability proportional to their fitness:
$$P(\mathbf{x}_i) = \frac{f(\mathbf{x}_i)}{\sum_{j=1}^N f(\mathbf{x}_j)}$$

#### b. Tournament Selection
A subset of $ k $ genomes is randomly selected, and the best genome in the subset is chosen as a parent. This process is repeated until enough parents are selected.

#### c. Rank-Based Selection
Genomes are ranked by fitness, and selection probability is assigned based on rank rather than raw fitness values.

### 4. Crossover
Crossover combines genetic information from two parent genomes $ \mathbf{x}_1 $ and $ \mathbf{x}_2 $ to create offspring. Common crossover methods include:

#### a. Single-Point Crossover
A random crossover point $ c $ is chosen, and the offspring $ \mathbf{y} $ is created by combining the first $ c $ genes of $ \mathbf{x}_1 $ with the remaining genes of $ \mathbf{x}_2 $:
$$\mathbf{y} = [x_{11}, x_{12}, \dots, x_{1c}, x_{2(c+1)}, \dots, x_{2n}]$$

#### b. Uniform Crossover
Each gene in the offspring is randomly selected from one of the parents with equal probability:
$$y_j = \begin{cases}
    x_{1j} & \text{with probability } 0.5 \\
    x_{2j} & \text{with probability } 0.5
\end{cases}$$

### 5. Mutation
Mutation introduces small random changes to the offspring to maintain diversity. For a genome $ \mathbf{y} $, each gene $ y_j $ is mutated with probability $ p_m $ (mutation rate):
$$y_j' = \begin{cases}
    y_j + \delta & \text{with probability } p_m \\
    y_j & \text{otherwise}
\end{cases}$$
where $ \delta $ is a small random perturbation (e.g., sampled from a Gaussian distribution $ \mathcal{N}(0, \sigma) $).

### 6. Replacement
The newly generated offspring replace some or all of the existing population to form the next generation. Common replacement strategies include:

#### a. Generational Replacement
The entire population is replaced by the offspring.

#### b. Steady-State Replacement
Only a subset of the population is replaced, often the least fit individuals.

#### c. Elitism
The best $ k $ individuals from the current population are preserved in the next generation to ensure that high-quality solutions are not lost.

## Algorithm Pseudocode
```python
population = initialize_population(size=N)
for generation in range(max_generations):
    fitness_scores = evaluate_fitness(population)
    parents = select_parents(population, fitness_scores)
    offspring = crossover(parents)
    offspring = mutate(offspring)
    population = replace(population, offspring)
best_solution = get_best_solution(population)
```

## Mathematical Formulation of Convergence
The convergence of a genetic algorithm can be analyzed using **Markov chains** or **schema theory**. A **schema** $ H $ is a template that matches a subset of genomes with similarities at certain positions. The **Schema Theorem** (Holland, 1975) states that short, low-order schemas with above-average fitness increase exponentially in frequency over generations:
$$m(H, t+1) \geq m(H, t) \cdot \frac{f(H)}{ \bar{f} } \cdot \left(1 - p_c \cdot \frac{\delta(H)}{n-1} \right) \cdot (1 - p_m)^{o(H)}$$
where:
- $ m(H, t) $: Number of genomes matching schema $ H $ at generation $ t $.
- $ f(H) $: Average fitness of genomes matching schema $ H $.
- $ \bar{f} $: Average fitness of the population.
- $ p_c $: Crossover probability.
- $ p_m $: Mutation probability.
- $ \delta(H) $: Defining length of schema $ H $ (distance between first and last fixed positions).
- $ o(H) $: Order of schema $ H $ (number of fixed positions).

## Evolutionary Lineage
Genetic Algorithms (GAs) are a foundational method in the **evolutionary algorithms (EAs)** family and have inspired many variants and extensions. Here’s how they connect to other key algorithms:

### Key Connections:
1. **Evolution Strategies (ES)**:
   - While GAs emphasize **crossover** and **population diversity**, ES focuses on **mutation** and **adaptive step-size control**.
2. **CMA-ES**:
   - A modern descendant of ES, CMA-ES adapts the **covariance matrix** of a Gaussian distribution for continuous optimization.
3. **(1+1)-ES**:
   - The simplest ES, which inspired the **step-size adaptation** mechanism in CMA-ES.
4. **Differential Evolution (DE)**:
   - Another continuous optimization algorithm, but DE relies on **vector differences** rather than crossover.

## See Also
- [Evolutionary Algorithms Overview](./evolutionary-algorithms.md): A unified introduction to the family of evolutionary algorithms.
- [CMA-ES](./cma-es.md): A state-of-the-art evolution strategy for continuous optimization.
- [(1+1)-ES](./(1+1)-es.md): The simplest evolution strategy, focusing on mutation and elitist selection.
- [Differential Evolution](./differential-evolution.md): A robust optimizer using vector differences for continuous problems.

## Paper
[Introduction to Genetic Algorithms (UC Paper)](https://www.ceas3.uc.edu/ret/archive/2018/ret/docs/readings/Project%203/2018RET_ReadingMaterial_Introduction%20to%20Genetic%20Algorithms.pdf)