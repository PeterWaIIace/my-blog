---
title: Evolutionary Algorithms
---

Overview of evolutionary algorithms I'm exploring.

The origins of Evolution Computing can be traced to early
work by Computer Scientists in the 1950s and 1960s with
the idea that evolutionary processes could be applied to
engineering problems of optimization. This led to three
major independent implementations of Evolutionary
Computing of which two are Evolution Strategies and
Genetic Algorithms.

Questions:

[x] Are Genetic Algorithms and Evolution Strategy basically same algorithm?

Papers: 

https://www.ceas3.uc.edu/ret/archive/2018/ret/docs/readings/Project%203/2018RET_ReadingMaterial_Introduction%20to%20Genetic%20Algorithms.pdf

Cheat sheet table tiwth algorithms:

- [[Genetic Algorithm]] - population of solutions evolved via selection, crossover, and mutation
- [[Evolution Strategy]] - optimization driven by mutation strength adaptation
- [[Genetic Programming]] - evolves programs or expressions, not fixed-length vectors
- [[Differential Evolution]] - simple, robust optimizer using vector differences
- [[Neuroevolution (NEAT)]] - evolves neural networks through topology and weight mutations
- [[Particle Swarm Optimization]] - swarm of particles guided by personal and global bests

| Algorithm                         | Representation                                       | Main Search Mechanism                  | Best For                                    | Weaknesses                                         |
| --------------------------------- | ---------------------------------------------------- | -------------------------------------- | ------------------------------------------- | -------------------------------------------------- |
| Genetic Algorithm (GA)            | Fixed-length vectors (binary, integers, real values) | Selection + crossover + mutation       | General optimization                        | Lots of parameters; crossover isn't always helpful |
| Evolution Strategy (ES)           | Real-valued vectors                                  | Mutation + adaptive mutation size      | Continuous optimization                     | Less suitable for discrete/combinatorial problems  |
| Genetic Programming (GP)          | Trees (programs, formulas)                           | Subtree crossover + mutation           | Symbolic regression, program synthesis      | Code bloat, computationally expensive              |
| Differential Evolution (DE)       | Real-valued vectors                                  | Vector differences between individuals | Continuous optimization                     | Doesn't naturally handle discrete variables        |
| Neuroevolution (NEAT)             | Neural network topology + weights                    | Structural mutations + crossover       | Reinforcement learning, architecture search | Can become computationally expensive               |
| Particle Swarm Optimization (PSO) | Particle positions                                   | Velocity updates using best solutions  | Fast continuous optimization                | Can converge prematurely                           |
