---
title: GRPO
---

Group Relative Policy Optimization — a [[PPO]]-style algorithm that compares responses to the same prompt instead of using a learned critic. Introduced in DeepSeekMath (2024) and used in DeepSeek-R1's reasoning training.

## Core Idea

In LLM training, PPO's critic can be another large network. GRPO saves that cost by using a group of responses as the baseline. In the **outcome-supervised** version:

1. For one prompt (one identical starting state), sample **G responses**.
2. Score each with the reward (rule-based correctness, or a reward model).
3. Advantage of response $i$ is the **group z-score**:

$$A_i = \frac{r_i - \mu}{\sigma}$$

where $\mu$ and $\sigma$ are the mean and standard deviation of the group's rewards. A response above the group average gets a positive advantage; one below it gets a negative advantage.

Every token in a response shares that advantage. The update uses PPO-style clipping; the original GRPO objective also includes a KL penalty to discourage drifting too far from a reference model. Equal-reward groups need zero-denominator handling and provide no relative reward signal.

## Key Features

- **No learned critic**: saves value-model memory, though reference and reward models may still be needed.
- **Response-level feedback**: with outcome supervision, one advantage is shared across the response. DeepSeekMath also describes a process-supervised version with finer-grained advantages.
- **Structured rollouts**: requires G samples per identical start state, not an arbitrary transition batch.

## Limits of group normalization

- **Small differences can get strong weight**: dividing by the group's standard deviation can give a small reward difference a substantial advantage. A small $\sigma$ alone does not make within-group z-scores unbounded.
- **Absolute reward level is lost**: groups such as $[0, 0.1, 0.1, 0.1]$ and $[0.9, 1, 1, 1]$ have the same z-scores, despite very different average rewards.
- **Identical rewards give no comparison**: if every response receives the same reward, there is no relative reward signal to learn from.

The [[MAPO]] paper calls the first two issues **advantage reversion** and **advantage mirror**, and proposes a different advantage estimator to address them.

## Where it is used

LLM/VLM reasoning RL (RLVR/RLHF post-training of foundation models), agentic and tool-use training. Implemented in TRL's GRPOTrainer, OpenRLHF, verl — not in Stable-Baselines3, since it assumes a generate-then-score loop rather than a gym VecEnv. See [[Reinforcement Learning]] for the family comparison.

## Sources

- [DeepSeekMath (2024), §4.1](https://arxiv.org/html/2402.03300v2#S4.SS1)
- [DeepSeek-R1 (2025)](https://arxiv.org/abs/2501.12948)
