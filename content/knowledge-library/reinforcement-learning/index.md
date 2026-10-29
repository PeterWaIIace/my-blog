---
title: Reinforcement Learning
---

Notes on policy-optimization algorithms I'm exploring, from classic control (PPO) to LLM post-training (GRPO, MAPO).

## PPO, GRPO, MAPO — what is different?

| | PPO | GRPO | MAPO |
| ---- | ---- | ---- | ---- |
| Family | Actor-critic, model-free | PPO's clipped objective, critic removed | GRPO variant (same objective) |
| Advantage source | Learned critic + GAE per step | Group z-score: $A_i = (r_i - \mu)/\sigma$ over G rollouts from the same start | Same, but reweighted by trajectory certainty |
| Credit assignment | Per-step (temporal-difference) | Per-trajectory only — one advantage for the whole episode | Per-trajectory only |
| Critic | Yes (value network) | No — replaced by group baseline | No |
| Rollout structure | Any batch of transitions | Must sample G episodes per identical starting state | Same as GRPO |
| Advantage normalization | Fixed formula (GAE) | Fixed formula (z-score) across all samples | Adaptive: z-score for uncertain groups, APD for certain groups, blended by TCR |
| Failure mode it fixes | — | Advantage reversion (small $\sigma$ gives unstable scores) and advantage mirror ($\mu$ ignored) | Both, via APD + TCR |
| Typical domain | Continuous control (our drones) | LLM/VLM reasoning RL (one reward per response) | Same, math reasoning benchmarks |

## Are GRPO and MAPO RL algorithms?

Yes — both are genuine RL algorithms. They are on-policy **policy-gradient methods**, same family as PPO and SAC:

1. A policy network (the LLM) outputs actions (tokens / a whole response).
2. An environment: prompt in, sampled response out, reward (rule-based correctness, or a reward model).
3. The policy is updated to maximize expected reward via the policy-gradient objective — GRPO even reuses PPO's clipped surrogate loss verbatim; only the advantage estimator differs.

What is special is only the **setting** they are used in — RLHF/RLVR post-training of foundation models, where:

- one "episode" = one response generation,
- the reward is a single terminal verdict (answer right/wrong), not a per-step signal,
- the group baseline replaces the critic.

So "GRPO/MAPO" name algorithms, while "RLVR" (reinforcement learning with verifiable rewards) or "RLHF" names the problem setting they are applied to. A researcher from 2017 would recognize GRPO as "PPO with a Monte-Carlo group baseline instead of GAE" — nothing conceptually new about it being RL.

## Where they are used

- **GRPO** (DeepSeekMath, 2024) was invented because vanilla PPO is impractical at LLM scale: the critic would be a second network the size of the LLM itself, doubling memory. GRPO replaces it with a cheap statistical baseline — sample G answers to the same question, reward them, use the group z-score as the advantage. DeepSeek-R1's reasoning training used exactly this.
- **MAPO** (2025) fixes cases where the group z-score misbehaves — questions the model nearly always solves (or nearly always fails) — by switching the advantage formula based on how certain the group's outcomes are.
- In practice: training reasoning models (math, code, logic), aligning chat models with human-preference rewards (RLHF), and agentic/tool-use training. Every major reasoning model — DeepSeek-R1, Qwen3, Kimi — went through a GRPO-family trainer.

## Why not GRPO for the swarm?

For our drone tasks the LLM-setting advantages flip into disadvantages: with dense per-step shaping rewards and 1000-step episodes, GRPO/MAPO would tell a drone "this whole episode was 0.3σ better than its siblings" but not **which step helped**. A critic with GAE earns its memory cost here — that is why everyone uses PPO/SAC in control and GRPO-family in LLM post-training. More on the environment side in [[24.09.2025 Building Swarm Environment]].

## Availability

Not in Stable-Baselines3. SB3 ships PPO, A2C, SAC, TD3, DDPG (+ HER); sb3-contrib has RecurrentPPO, TQC, QR-DQN, CrossQ, ARS — no GRPO, no MAPO. GRPO implementations exist only in LLM post-training frameworks (TRL's GRPOTrainer, OpenRLHF, verl), built around transformer generate-then-score loops, not gym VecEnvs.

Cheat sheet:

- [[PPO]] — clipped policy optimization with a learned critic and per-step GAE advantages
- [[GRPO]] — PPO's objective without the critic; advantage is the group z-score over G rollouts
- [[MAPO]] — GRPO with certainty-adaptive advantages (APD/TCR) for skewed outcome groups
