---
id: interspeech-2025-2449
title: "Accelerating Flow-Matching-Based Text-to-Speech via Empirically Pruned Step Sampling"
authors: ["Qixi Zheng", "Yushen Chen", "Zhikang Niu", "Ziyang Ma", "Xiaofei Wang", "Kai Yu", "Xie Chen"]
organization: Shanghai Jiao Tong University
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-06-03
task: [TTS]
architecture: [flow-matching]
conditioning: [text-conditioned, zero-shot, prompt-conditioned]
training: [supervised]
model_size: "336M params (F5-TTS Base)"
codec_used: "not reported"
datasets_train: ["Emilia"]
datasets_eval: ["Seed-TTS-eval test-en", "Seed-TTS-eval test-zh", "LibriSpeech-PC test-clean"]
metrics:
  - name: WER
    value: 1.74
    system: Fast F5-TTS (7 NFE)
    testset: Seed-TTS-eval test-en
  - name: SPK-SIM
    value: 0.68
    system: Fast F5-TTS (7 NFE)
    testset: Seed-TTS-eval test-en
  - name: UTMOS
    value: 3.78
    system: Fast F5-TTS (7 NFE)
    testset: Seed-TTS-eval test-en
  - name: WER
    value: 2.45
    system: Fast F5-TTS (7 NFE)
    testset: LibriSpeech-PC test-clean
code_available: true
demo_available: true
url: "https://www.isca-archive.org/interspeech_2025/zheng25d_interspeech.html"
related_concepts: ["flow-matching", "zero-shot-tts", "streaming-tts", "evaluation-metrics"]
related_papers: ["2406.02430", "2412.10117", "2410.06885"]
field_significance:
  level: moderate
  type: [engineering-integration]
generation:
  date: 2026-06-03
  model: claude-sonnet-4-6
  commit: "55d0339"
---

> [!abstract] Interspeech · 2025 · Conference
> **Qixi Zheng et al.** (Shanghai Jiao Tong University) · [→ Paper](https://www.isca-archive.org/interspeech_2025/zheng25d_interspeech.html) · Demo: ✓ · Code: ✓
>
> A training-free, non-uniform step scheduling strategy (EPSS) that reduces F5-TTS inference from 32 to 7 function evaluations while maintaining near-baseline synthesis quality, achieving a 4x RTF speedup without any retraining.

## Problem

Flow-matching TTS models like F5-TTS and E2 TTS deliver high quality but require many ODE solver steps at inference — typically 32 for F5-TTS. Uniform time-step schedules treat all phases of the trajectory equally, yet the generation dynamics are not uniform: the early portion is highly curved and requires dense sampling, while the later portion becomes nearly linear and tolerates larger step sizes. Simply reducing NFE with a uniform schedule degrades quality significantly: 7-step uniform F5-TTS yields 7.96% WER on Seed-TTS test-zh versus 1.58% for the 32-step baseline.

## Method

The paper proposes **Empirically Pruned Step Sampling (EPSS)**, a training-free, plug-and-play inference modification for flow-matching-based TTS. The approach is grounded in trajectory analysis: by applying PCA to the 100-dimensional Mel spectrogram features during F5-TTS sampling, the authors visualise how individual feature dimensions evolve across the time interval. The analysis reveals two phases — a nonlinear early phase where the trajectory curves substantially (requiring small steps for accurate ODE integration), and a near-linear later phase where the vector field is well-determined (allowing steps to be pruned without accumulating error).

EPSS constructs non-uniform time-step schedules that are dense at the start and sparse toward t=1. The representative 7-NFE configuration uses time-steps {0, 1/16, 1/8, 3/16, 1/4, 1/2, 3/4, 1} in the pre-sway-sampling space. EPSS is applied on top of F5-TTS's existing sway sampling (SS), and the two strategies are complementary: sway sampling redistributes steps along a non-linear curve, while EPSS prunes absolute count. No model weights are modified; the change is purely in the ODE solver schedule at inference time. The baseline model is F5-TTS Base (336M parameters, diffusion transformer architecture, trained on Emilia), with Vocos as vocoder.

## Key Results

With 7 NFE, Fast F5-TTS achieves RTF 0.030 on an NVIDIA RTX 3090, compared to 0.123 for the 32-NFE baseline — a 4x speedup. On Seed-TTS test-en, WER is 1.74% (vs. 1.70% baseline), SPK-SIM is 0.68 (vs. 0.67), and UTMOS is 3.78 (vs. 3.76). On Seed-TTS test-zh, WER is 1.75% (vs. 1.58%). On LibriSpeech-PC test-clean, 7-NFE EPSS scores 2.45% WER and 3.84 UTMOS versus 2.37% and 3.93 for the 32-NFE baseline. The contrast with naive 7-step uniform sampling is stark — uniform 7 NFE collapses to 7.96% WER on test-zh, while EPSS with 7 NFE achieves 1.75%.

Ablation confirms that quality degrades sharply below 6 NFE, and that pruning later steps (schedule 6a) is far less damaging than pruning early steps (schedule 6b) — 2.41% vs. 2.71% WER at 6 steps — which validates the trajectory analysis. EPSS generalises to E2 TTS (333M params), recovering most of the quality lost by naive step reduction at the same NFE.

> [!note]
> Comparisons to Seed-TTS DiT, MaskGCT, and CosyVoice 2 are taken from their original papers and may not use identical test conditions or generation lengths. Direct numerical comparison with these systems should be treated with caution.

## Novelty Assessment

The contribution is primarily engineering-integration: EPSS is a non-uniform ODE time-step schedule derived from visual trajectory inspection, which is a well-established idea in the diffusion/flow-matching literature for image generation. Its novelty within TTS is applying the analysis to Mel spectrogram feature-space trajectories to motivate the pruning. The method requires no training and no architectural change, and can be reproduced in a few lines of code. The 4x RTF gain is real and practically meaningful, and the evaluation is thorough with three test sets and ablations over NFE. The paper falls short of exploring integration with distillation methods, which the authors acknowledge as future work.

## Field Significance

Moderate — EPSS offers a practical, no-cost inference speedup for the F5-TTS family that is immediately deployable without retraining. It contributes solid empirical evidence that the later portion of flow-matching trajectories in TTS is nearly redundant under sway sampling, and that non-uniform step pruning can close most of the quality gap versus uniform reduction. The approach is incremental relative to analogous schedule optimisation work in image generation, but the TTS-specific trajectory analysis and the clean empirical validation strengthen its utility as a reference for future efficient-inference work in flow-matching TTS.

## Claims

- Non-uniform ODE time-step schedules can reduce flow-matching TTS inference to 7 steps with minimal quality degradation when steps are pruned from the later, near-linear portion of the trajectory.
- The early, high-curvature phase of flow-matching inference in TTS is disproportionately critical for quality, and uniform step reduction degrades performance primarily by under-sampling this phase.
- Training-free step scheduling methods generalise across flow-matching TTS architectures without requiring model-specific retraining.
- At very low NFE (below 6 steps), quality degradation in flow-matching TTS becomes abrupt rather than gradual, suggesting a practical floor for inference reduction without distillation.

## Limitations and Open Questions

> [!warning]
> Evaluation uses no subjective listening tests (MOS); quality assessment relies entirely on UTMOS, WER, and speaker cosine similarity — automatic proxies that may not fully capture perceptual degradation at low NFE.

EPSS is validated only on F5-TTS and E2 TTS, both mel-spectrogram-based flow-matching systems sharing similar backbone design. Applicability to token-based or vocoder-free flow-matching systems is unexplored. The time-step configurations are determined empirically by inspection rather than through a principled search procedure, which limits reproducibility on new models. The paper does not explore combining EPSS with consistency distillation or rectified-flow distillation, which could potentially push quality-preserving inference below 7 steps.

## Wiki Connections

Key concepts: [[flow-matching]] · [[zero-shot-tts]] · [[streaming-tts]] · [[evaluation-metrics]]

Related papers: [[2410.06885]] (F5-TTS, the primary baseline) · [[2406.02430]] (Seed-TTS, evaluation benchmark and comparison system) · [[2412.10117]] (CosyVoice 2, comparison system in Table 1)
