---
id: interspeech-2025-0063
title: "Fine-Tuning Text-to-Speech Diffusion Models Using Reinforcement Learning with Human Feedback"
authors: ["Jingyi Chen", "Ju Seung Byun", "Micha Elsner", "Pichao Wang", "Andrew Perrault"]
organization: null
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: "2025-08-17"
ingested_date: "2026-05-30"
task: [TTS]
architecture: [diffusion]
conditioning: [text-conditioned]
training: [RLHF, fine-tuning]
model_size: "not reported"
codec_used: none
datasets_train: ["LJSpeech (pre-training + RLHF fine-tuning; 12388 train / 512 val / 200 test)"]
datasets_eval: ["LJSpeech (200 unseen samples)"]
metrics:
  - name: UTMOS
    value: 3.65
    system: DLPO (proposed)
    testset: LJSpeech
  - name: UTMOS
    value: 2.90
    system: WaveGrad 2R (baseline)
    testset: LJSpeech
  - name: WER
    value: 1.2
    system: DLPO (proposed)
    testset: LJSpeech
  - name: WER
    value: 1.5
    system: WaveGrad 2R (baseline)
    testset: LJSpeech
code_available: true
demo_available: true
url: "https://www.isca-archive.org/interspeech_2025/chen25b_interspeech.html"
related_concepts: [rlhf-speech, diffusion-tts, evaluation-metrics]
related_papers: []
---

# Fine-Tuning Text-to-Speech Diffusion Models Using Reinforcement Learning with Human Feedback

**Paper:** [https://www.isca-archive.org/interspeech_2025/chen25b_interspeech.html](https://www.isca-archive.org/interspeech_2025/chen25b_interspeech.html)

**One-sentence contribution:** DLPO (Diffusion Loss-Guided Policy Optimization) is an RLHF framework that improves TTS diffusion model quality by incorporating the original diffusion training loss directly into the RL reward function, outperforming four competing RL fine-tuning strategies on a WaveGrad 2 baseline.

## Problem

TTS diffusion models produce high-fidelity speech but are difficult to align with human naturalness preferences. Prior RLHF methods developed for text-to-image diffusion (RWR, DDPO, DPOK, KLinR) transfer poorly to TTS because audio waveform generation demands stronger temporal coherence and acoustic consistency than image synthesis. KL-based regularizers (DPOK, KLinR) stabilize training but limit task-specific improvements, while score-function estimators (DDPO) cannot handle the sequential complexity of audio. The result is that applying existing image-domain RL recipes to TTS either degrades quality or yields only marginal improvements.

## Method

DLPO frames denoising as a finite-horizon Markov decision process. The policy is the pre-trained WaveGrad 2R (a PyTorch reproduction of WaveGrad 2, a non-autoregressive DDPM TTS model trained on LJSpeech). The reward model is UTMOS, a MOS predictor trained on VoiceMOS Challenge 2022 data.

The key innovation is the DLPO objective, which adds the original DDPM noise-prediction loss as a penalty term alongside the reward:

```
J_DLPO = E[ -α·r(x₀, c) - β·||ε̃(xₜ,t) - εθ(xₜ,c,t)||₂ ]
```

The diffusion loss term serves as a task-specific regularizer that: (1) prevents the policy from drifting too far from the base model's probabilistic structure; (2) maintains temporal coherence in sequential waveform generation; (3) avoids the over-optimization instabilities seen in KL-based approaches. Fine-tuning runs for 5.5 hours on 8 A100-SXM-80GB GPUs with batch size 64 and 10 denoising steps.

Evaluation uses two held-out MOS predictors (UTMOS and NISQA — the latter is separate from the reward model to guard against overfitting) plus Whisper-based WER and a human preference study (11 raters, 20 pairs).

## Key Results

| Method | UTMOS | NISQA | WER |
|--------|-------|-------|-----|
| Ground truth | 4.20 | 4.37 | 0.99% |
| WaveGrad 2R (base) | 2.90 | 3.74 | 1.5% |
| RWR | 2.18 | 3.00 | 8.9% |
| DDPO | 2.69 | 2.96 | 2.1% |
| DPOK | 3.18 | 3.76 | 1.1% |
| KLinR | 3.02 | 3.73 | 1.3% |
| OnlyDL (diffusion loss only) | 3.16 | 3.45 | 1.4% |
| **DLPO** | **3.65** | **4.02** | **1.2%** |

DLPO preferred by humans in 67% of pairwise comparisons vs. base WaveGrad 2R (binomial test p < 10⁻¹⁶). NISQA score of 4.02 approaches the ground truth ceiling of 4.37. WER is held at 1.2%, close to the base model's 1.5% and the ground truth 0.99%, demonstrating intelligibility is not compromised.

## Novelty Assessment

The specific mechanism — using the original training loss as a reward penalty in RL fine-tuning — is simple but empirically effective and directly motivated by the failure modes of existing approaches. The idea of mixing pretraining gradients with RL gradients has precedent in NLP (cited by the authors), but applying it via reward shaping in the diffusion denoising MDP is new to TTS. The evaluation is rigorous (dual MOS predictors, human study, WER) and the ablations against five competing methods are thorough. The limitation is scope: evaluation is single-speaker (LJSpeech), and the base model (WaveGrad 2R) is relatively simple. It is not clear how DLPO scales to more capable multi-speaker diffusion TTS systems.

## Limitations and Open Questions

- Single-speaker dataset (LJSpeech); generalization to multi-speaker and zero-shot diffusion TTS is unverified.
- WaveGrad 2R is a relatively simple baseline; DLPO's benefit on state-of-the-art flow-matching or score-based TTS systems remains to be shown.
- UTMOS as reward model introduces reward hacking risk; the NISQA and human study help but do not fully eliminate it.
- Prosody and intonation improvement are claimed but not measured with dedicated metrics (F0-RMSE, MUSHRA).
- Training compute cost (5.5h on 8 A100s) is not trivial for academic use cases.

## Wiki Connections

This paper is a direct contribution to [[rlhf-speech]], demonstrating RLHF applied to [[diffusion-tts]] rather than the more commonly studied autoregressive codec models. The choice of UTMOS as reward model connects to [[evaluation-metrics]] and the VoiceMOS challenge literature. The problem of temporal coherence in sequential diffusion generation is structurally related to challenges discussed in [[autoregressive-codec-tts]] papers. The DLPO objective's penalty term is analogous in spirit to KL regularization used in DPOK but derived from task structure rather than distributional divergence.

Related RLHF-for-TTS work from Interspeech 2025: [[interspeech-2025-0704]] (DiffRO) applies RLHF to codec LM-based TTS (CosyVoice 2.0) using Gumbel-Softmax differentiable token sampling and a multi-task reward model that operates directly on codec tokens. DiffRO addresses the AR+FM paradigm while DLPO addresses the diffusion paradigm — together they demonstrate that RLHF is applicable across the major TTS architectures of 2025.
