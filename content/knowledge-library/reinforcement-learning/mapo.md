---
title: MAPO
---

Mixed Advantage Policy Optimization — a 2025 variant of [[GRPO]] that changes how response advantages are calculated based on the group's success rate.

## Core Idea

GRPO compares each response's reward with its group using a z-score. This can give small reward differences strong weight and make low-reward and high-reward groups look the same after normalization.

MAPO keeps GRPO's rollout structure and objective, but blends two advantage estimates:

- **Mixed successes and failures**: give more weight to GRPO's z-score, $(r_i-\mu)/\sigma$.
- **Mostly successes or mostly failures**: give more weight to **Advantage Percent Deviation (APD)**:

$$A_i = \frac{r_i - \mu}{\mu}$$

APD measures the reward difference relative to the mean reward, rather than the standard deviation. It distinguishes groups with different average rewards.

The blend is called **Trajectory Certainty Reweight (TCR)**. Let $p$ be the fraction of responses counted as successful:

$$\lambda(p)=1-4p(1-p)$$

$$A_i^{\text{MAPO}}=(1-\lambda(p))\frac{r_i-\mu}{\sigma}+\lambda(p)\frac{r_i-\mu}{\mu}$$

At $p=0.5$, MAPO uses the z-score. As $p$ approaches 0 or 1, it puts more weight on APD. Here, **certainty** means consistent outcomes, including consistent failure — not confidence that an answer is correct.

These are the paper's idealized formulas. Zero denominators need explicit handling: APD is undefined at $\mu=0$, and the z-score is undefined at $\sigma=0$. MAPO does not create a relative reward signal when all rewards are identical.

## Key Features

- Same per-trajectory credit assignment and critic-free design as GRPO — only the advantage estimator changes.
- Adapts the advantage estimator to the group's success rate.
- The paper reports improvements over GRPO in its evaluated settings; this is not a general guarantee.

## Where it is used

Designed for GRPO-style foundation-model post-training. The paper evaluates Qwen2.5-VL-7B on visual mathematics and emotion reasoning, including Geo3K and EmoSet. Not in Stable-Baselines3; see [[Reinforcement Learning]] for the family comparison and [[GRPO]] for the base algorithm.

## Sources

- [MAPO: Mixed Advantage Policy Optimization (2025), §3.2](https://arxiv.org/html/2509.18849v3#S3.SS2)
