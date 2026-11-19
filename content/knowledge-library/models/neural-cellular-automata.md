---
title: Neural Cellular Automata (NCA)
---

A **neural cellular automaton** is a grid of cells that all run the **same tiny neural network** as their update rule. Like a classical cellular automaton (Conway's Game of Life), every cell updates from its local neighborhood — but the transition rule is a _learned_ CNN instead of a hand-coded one, and the grid is run for many steps so that global patterns emerge from purely local interactions.

![[gospers-glider-gun.gif|Gosper's glider gun in Conway's Game of Life — the classical hand-coded cellular automaton that NCAs generalize|250]]

## Core Idea

Each cell carries a hidden state $h \in \mathbb{R}^{16}$; the first 3 channels are the visible RGB image. At every step, each cell perceives its 3×3 neighborhood through Sobel filters and applies a shared convolutional network $f_\theta$:

$$p = [\,h,\; \nabla_x h,\; \nabla_y h\,] \in \mathbb{R}^{80}, \qquad \Delta h = f_\theta(p), \qquad h \leftarrow h + \Delta h$$

Updates are **stochastic**: each cell applies its update only with some probability (typically 0.5), so growth is asynchronous and organic. The same rule is applied to all cells, in parallel, for $T$ steps (typically 64–128).

![[nca-iterative.svg|An NCA grid: one shared update rule applied to every cell for T steps, growing a pattern from a single seed cell|640]]

## NCA vs CNN

An NCA _is_ a CNN — the difference is how it is used, not the architecture. A CNN is a **one-shot function**: one feedforward pass from input to output, with a fixed depth and its own weights per layer, and no state carried between calls.

![[cnn-feedforward.svg|A CNN: a single feedforward pass from input image to output, with different weights at each layer|640]]

An NCA is the **same small network applied to its own output, repeatedly** — a dynamical system rather than a function:

|                | CNN                           | NCA                                                          |
| -------------- | ----------------------------- | ------------------------------------------------------------ |
| Weight sharing | across spatial positions      | across positions **and across iterations**                   |
| Depth          | fixed by the number of layers | emergent — effective depth = number of steps run             |
| Inference      | single feedforward pass       | iterative loop at test time ("test-time compute")            |
| State          | stateless between calls       | each cell carries a hidden state that persists across steps  |
| Training       | input → output pairs          | unroll $T$ steps from a seed, loss on the final state        |
| Behavior       | computes a mapping            | seeks attractors: stable patterns, regeneration after damage |

A useful way to see it: a deep CNN is like an NCA where each step may have its own weights and the step count is fixed in advance. An NCA ties all steps to one shared rule and lets the step count be a runtime choice — which is what makes it a _process_ rather than a _function_.

## Training: growing from a seed

The canonical setup is **growth**: start with a single seed cell in the center of the grid, run the automaton for a random number of steps, and apply a loss between the final visible channels and a target image. Backpropagating through the whole unrolled computation trains the shared rule — the unrolled automaton is just a deep network with tied weights.

Practical tricks that matter: pool the loss over many random step counts, and randomly damage patches of the grid during training so the rule learns to _regenerate_ rather than merely grow.

## Key Features

- **Emergent global structure from local rules** — a computational model of morphogenesis (how organisms develop from a single cell).
- **Regenerative and robust**: damage a patch of the grid and it regrows; the same rule works from different seed positions.
- **Test-time compute**: at inference you choose how many steps to run — more steps, more "thinking".
- **Open problem**: generalizing beyond the trained regime (new goals, new grid sizes) is still active research — see goal-conditioned and "generalizing" NCA variants.

![[nca-regen.gif|An NCA regenerating after damage: a patch of the grid is destroyed and the local rule regrows the target pattern (Mordvintsev et al., 2020)|480]]

## NCAs that compute

Most NCAs are trained to grow and repair shapes. The [ncpu live demo](https://ncpu.pages.dev/) trains one to **compute** instead: one learned rule runs in every cell for many steps, and from inputs drawn on the grid the cells build the answer — Boolean gates (AND, OR, XOR, NAND, NOR, XNOR, MAJ-3, half-adder) and 4-/8-bit binary addition. In the demo you set the input bits (the circles on the left) and the answer forms in the circles on the right, which start blank; each cell's colour is its value (dark = 0, bright = 1).

![[ncpu-gate-xor.gif|XOR computed by the trained rule: all four input combinations side by side, 64 steps each, answer in the right-hand circle|400]]

![[ncpu-gate-nor.gif|NOR, same layout|400]]

![[ncpu-longest.gif|An 8-bit adder rollout (255 + 1): activity spreads from the input columns and the carry settles last — addition is not a local operation, so the carry has to travel|320]]

These gifs are rendered from the demo's own published weights (`data/xor.json`, `data/nor.json`, …) by replaying the exact update rule in numpy; every quadrant is checked against the ground truth before saving.

## Sources

- [Mordvintsev, Randazzo, Niklasson, Levin (2020), Growing Neural Cellular Automata — Distill](https://distill.pub/2020/growing-nca/)
- [Mordvintsev et al. (2020), Growing Neural Cellular Automata — arXiv:2003.02700](https://arxiv.org/abs/2003.02700)

### Figures

- Gosper's glider gun animation — [Lucas Vieira](https://commons.wikimedia.org/wiki/User:LucasVB), [CC BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0/), via [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Gospers_glider_gun.gif)
- NCA regeneration demo — Mordvintsev et al. (2020), Distill, [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/), converted from mp4 to gif
- Gate rollouts (XOR, NOR), adder films, and stills — [ncpu.pages.dev](https://ncpu.pages.dev/) (Zhechev & Walas, ALIFE 2026); gate gifs re-rendered from the site's published model weights, talk films converted from the site's sprite sheets
