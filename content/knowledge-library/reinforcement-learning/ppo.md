---
title: PPO
---

Proximal Policy Optimization — a widely used reinforcement learning algorithm for control tasks, and the starting point for [[GRPO]] and [[MAPO]].

## Core Idea

PPO learns from fresh rollouts collected by its current policy (**on-policy** learning). In its usual **actor-critic** setup, the actor chooses actions and the critic estimates how much future reward to expect.

This note describes **PPO-Clip**, the commonly used variant:

- **Advantage**: estimates whether an action turned out better or worse than expected. Common implementations use **GAE (Generalized Advantage Estimation)** to calculate this from rewards and the critic's predictions.
- **Clipping**: limits the incentive to keep increasing or decreasing an action's probability within an update. This discourages large policy changes, but does not guarantee they cannot happen.

## Key Features

- **Per-step feedback**: the usual critic-and-GAE setup gives each action its own advantage estimate.
- **Simple updates**: clipping helps stabilize training without a hard trust-region constraint.
- **Flexible starting states**: rollouts need not share a starting state, but must be collected on-policy, with trajectory information for computing returns and advantages.

## Where it is used

Discrete and continuous control — games, robotics, drones (our swarm environment), and MuJoCo-style benchmarks. Also widely used in LLM reinforcement learning from human feedback (RLHF).

## In our stack

Shipped in Stable-Baselines3, and used by our experiment scripts for the drone tasks. With dense rewards (target distance, coverage/patrol timers) and 1000-step episodes, the critic and GAE provide useful per-step feedback. See [[Reinforcement Learning]] for the family comparison and [[24.09.2025 Building Swarm Environment]] for the environment.

## Sources

- [PPO paper (2017)](https://arxiv.org/abs/1707.06347)
- [OpenAI Spinning Up: PPO](https://spinningup.openai.com/en/latest/algorithms/ppo.html)
