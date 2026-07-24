---
slug: rlhf-speech
concept: rlhf-speech
render_type: overview
title: RLHF for Speech
aliases: [RLAIF speech, preference optimization for TTS, reinforcement learning from human feedback, DPO for speech, direct preference optimization TTS]
status: emerging
last_reviewed: 2026-07-24
source_digest_date: 2026-07-20
paper_count: 29
generation:
  schema_version: 2
  date: "2026-07-24"
  stage: render
  mode: light
  runtime: codex
  provider: openai
  agent: speech-generation-render-agent
  model: "gpt-5"
  commit: "39007c6"
---

> [!abstract]
> RLHF for speech uses preferences or reward signals to improve a pretrained TTS, spoken-language,
> or speech-to-speech model after supervised training. Across 29 reviewed papers, post-training
> reliably improves targeted qualities with relatively little data, but proxy rewards can damage
> prosody, diversity, emotion, or other dimensions that the optimizer does not measure.

## Current State

- **Preference and reward post-training consistently improve over supervised fine-tuning on their
  targeted dimensions.** DPO, KTO, GRPO, policy-gradient methods, and differentiable reward
  optimization report gains in intelligibility, speaker similarity, naturalness, or semantic
  coherence across discrete and continuous speech generators ([[2406.00654]],
  [[2025.acl-long.598]], [[2509.00685|MPO]]).
- **Reward hacking is the central cross-method risk.** Optimizing WER, likelihood, or another proxy
  can produce monotone prosody, lower emotional appropriateness, reduced diversity, or evasive
  dialogue behavior even while the reward improves ([[2509.18531]], [[2508.16332|Vevo2]],
  [[2502.11946|Step-Audio]]).
- **Preference data construction matters as much as the objective.** Pairs need meaningful
  diversity and a reliable ordering; fine-tuning that narrows the generation distribution can
  leave DPO with too little useful contrast ([[2025.icnlsp-1.34]], [[2509.14946]]).
- **Multi-objective rewards are safer than single-metric rewards, but balancing remains
  under-specified.** Combining intelligibility, identity, emotion, and prosody avoids some
  one-dimensional failures, yet most papers do not establish principled weights.
- **Speech alignment no longer requires a discrete autoregressive decoder.** Preference objectives
  and policy gradients have been adapted to flow matching, diffusion, autoregressive-diffusion,
  and CTC decoders, although most techniques remain one-backbone demonstrations
  ([[2504.02407|F5R-TTS]], [[2509.18928]], [[2509.25416]]).
- **Alignment cannot create capability absent from the base model.** Iterative optimization shows
  diminishing returns and rarely reaches oracle best-of-N or strong cascaded toplines.

## Method Landscape

| Family | Practical idea | Main limitation |
|--------|----------------|-----------------|
| **DPO/KTO on discrete speech tokens** | Learn from preferred and rejected generations without online rollouts | Highly sensitive to pair construction and reference regularization |
| **Policy-gradient RL** | Optimize explicit audio or task rewards with GRPO/PPO/REINFORCE | Expensive sampling, reward hacking, and unstable training |
| **Differentiable reward optimization** | Backpropagate through discrete tokens from a frozen critic | Efficient, but so far concentrated in the CosyVoice lineage |
| **Continuous-generative preference optimization** | Reformulate DPO or RL for flow/diffusion trajectories | Each method is validated on a different backbone with little replication |
| **Reward-model infrastructure** | Train reusable audio-quality or dialogue reward evaluators | Few reward models have been validated inside a complete alignment loop |

## Key Trade-offs

- **Target improvement versus unmeasured quality.** A better reward score can conceal worse
  prosody, diversity, or emotional suitability.
- **Offline stability versus online flexibility.** DPO avoids rollouts; policy-gradient methods can
  optimize arbitrary rewards but are more expensive and unstable.
- **External reward quality versus reward-model cost.** Differentiable critics can be efficient but
  may encode narrow ASR or quality biases.
- **Stronger adaptation versus capability preservation.** Long or weakly regularized runs can
  collapse intelligibility, identity, or held-out quality.
- **Component-level alignment versus final audio quality.** Optimizing an LM cannot repair
  attributes controlled by a frozen downstream flow model or vocoder.

## Open Questions

- Is there a general defense against reward hacking, or only task-specific combinations of held-out
  metrics, constraints, and human preference stages?
- Do continuous-generation preference methods transfer across flow and diffusion backbones?
- Can intrinsic rewards from the speech model itself replicate beyond one GFlowNet-based result?
- How should RLHF for conversational behavior differ from acoustic-quality alignment?
- Can reward models trained as standalone evaluators remain reliable when used to optimize a
  generator against them?

## Go Deeper

Read [[concepts/rlhf-speech-in-depth|RLHF for Speech: In Depth]] for the evidence on reward hacking,
preference data, multi-objective optimization, continuous generators, training stability, and the
limits imposed by the base model and downstream pipeline.

## Scope

This assessment covers 29 integrated papers published between June 2024 and September 2025, with
roughly two-thirds appearing in Q3 2025. The area is therefore fast-moving but young. Several
strong-looking results come from a single backbone or proprietary system, and spoken-dialogue RLHF
is much less developed than acoustic TTS alignment. Complete provenance remains in
`wiki/_claims/rlhf-speech.yaml`.

---

_This page is generated from `wiki/_claims/rlhf-speech.yaml` (digest date: 2026-07-20).
Complete structured claims and paper-level provenance remain in the source YAML._
