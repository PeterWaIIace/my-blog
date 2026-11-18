---
title: Models
---

Theory notes on the model families I keep reaching for — from linear baselines to feedforward nets, recurrent units, and attention over time.

## How the families differ

| | Ridge Regression | MLP | GRU | Temporal Transformer |
| ---- | ---- | ---- | ---- | ---- |
| Family | linear | feedforward | recurrent | attention |
| Input | single vector | single vector | window of vectors | window of vectors |
| Memory of the past | none | none | hidden state carried forward | every past token, directly |
| Inductive bias | linearity | none (universal approximator) | locality in time | only via positional embeddings |
| Training | closed-form | parallel over samples | sequential over the window | parallel over the window |
| Data needs | works on tiny datasets | needs enough data | fine on small data | data-hungry |

## Cheat sheet

- [[Ridge Regression]] — linear map with L2 shrinkage; closed form, the sanity baseline
- [[MLP]] — stack of linear layers + nonlinearities; the plain feedforward net, no memory
- [[GRU]] — gated recurrent unit; a causal hidden state carried across a window
- [[Temporal Transformer]] — causal self-attention over a window of steps
- [[Neural Cellular Automata (NCA)]] — a 2D grid of cells running one shared CNN as the update rule; grows and regenerates patterns from a seed

## Reading the families

- **Tabular vs sequential**: ridge and MLP see one input vector at a time — no notion of time. The GRU and the transformer consume a window and exploit history.
- **How history enters**: the GRU compresses the past into a single hidden state — cheap, and biased toward recent steps. The transformer lets every step attend to every past step directly: global receptive field, no recurrent bottleneck, but no recurrent bias either.
- **Causality**: for real-time use, both windowed models can be causally masked — the output at step *t* depends only on inputs ≤ *t*.
- **Grid vs sequence**: the NCA swaps the 1D window for a 2D grid of cells and iterates one shared rule over it — locality in space instead of time, and a process (attractor-seeking) instead of a one-shot function.
- **Small data**: with little data, stronger inductive bias wins — linear first, then recurrent, and attention last. With lots of data the order usually reverses.
