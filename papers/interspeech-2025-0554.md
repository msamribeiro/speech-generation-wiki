---
id: interspeech-2025-0554
title: "RapFlow-TTS: Rapid and High-Fidelity Text-to-Speech with Improved Consistency Flow Matching"
authors: [Hyun Joon Park, Jeongmin Liu, Jin Sob Kim, Jeong Yeol Yang, Sung Won Han, Eunwoo Song]
organization: NAVER Cloud
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: "2025-08-17"
ingested_date: "2026-05-30"
task: [TTS]
architecture: [flow-matching, GAN]
conditioning: [text-conditioned]
training: [supervised]
model_size: "18.2M params"
codec_used: "none"
datasets_train: [LJSpeech, VCTK]
datasets_eval: [LJSpeech test set, VCTK test set]
metrics:
  - name: MOS
    value: 4.01
    system: RapFlow-TTS+ (2 NFE, with improved techniques)
    testset: LJSpeech
  - name: MOS
    value: 3.89
    system: RapFlow-TTS (2 NFE, base)
    testset: LJSpeech
  - name: WER
    value: 3.11
    system: RapFlow-TTS+ (2 NFE)
    testset: LJSpeech
  - name: MOS
    value: 4.28
    system: RapFlow-TTS+ (2 NFE)
    testset: VCTK
  - name: MOS
    value: 3.83
    system: Matcha-TTS (10 NFE)
    testset: LJSpeech
  - name: MOS
    value: 3.19
    system: Comospeech (2 NFE)
    testset: LJSpeech
code_available: true
demo_available: true
url: "https://www.isca-archive.org/interspeech_2025/park25b_interspeech.html"
related_concepts: [flow-matching, diffusion-tts, evaluation-metrics, subjective-evaluation]
related_papers: []
---
> [!abstract] Interspeech · 2025 · Conference
> **Hyun Joon Park et al.** (NAVER Cloud) · [→ Paper](https://www.isca-archive.org/interspeech_2025/park25b_interspeech.html) · Demo: ✓ · Code: ✓
>
> RapFlow-TTS is the first TTS system based on consistency flow matching, enforcing velocity-field self-consistency along straight ODE trajectories to achieve high-fidelity synthesis in just 2 function evaluations — matching the speed of FastSpeech2 while substantially exceeding its naturalness.

## Problem

ODE-based TTS models (diffusion and flow matching) require many generation steps to achieve natural-quality speech, creating a trade-off between quality and inference speed. Consistency distillation has been applied to diffusion-based TTS (Comospeech), but diffusion ODE trajectories are complex and curved, limiting the effectiveness of consistency constraints. Flow matching produces straighter ODE trajectories, making it a better candidate for consistency modeling. However, applying consistency constraints to FM for TTS had not been explored prior to this work.

## Method

RapFlow-TTS adopts the Matcha-TTS architecture as its backbone (text encoder + aligner + FM decoder, 18.2M parameters) and replaces the standard FM training objective with a consistency flow matching (CFM) objective.

**Consistency FM:** The time range [0,1] is divided into S=2 segments. Within each segment, the objective combines: (1) a straight flow loss that constrains the estimated trajectory endpoint, and (2) a velocity consistency loss that enforces that the predicted velocity at time t matches the velocity at t+Δt (with stop-gradient on the target). A two-stage training procedure is used: Stage 1 trains straight flow only (optimizing only the straight flow loss with ground-truth segment endpoints), and Stage 2 adds velocity consistency on top of the straight trajectory learned in Stage 1.

**Improved techniques (contributing to RapFlow-TTS+):**
- Encoder freeze: the text encoder is frozen in Stage 2, stabilizing the conditioning signal and speeding up training 1.3×.
- Shared dropout: the same random dropout state is applied to both the online model and the stop-gradient target in Stage 2, improving robustness.
- Pseudo-Huber loss: replaces L2 metric with d(x,y) = sqrt(||x-y||² + c²) - c, reducing gradient variance from outliers.
- Delta scheduling: Δt is linearly annealed from 0.1 to 0.001 over K=8 intervals, reducing bias near end of training.
- Adversarial learning: after Stage 2, a Conv2d discriminator applies MSE-based adversarial and feature-matching losses at the mel-spectrogram level, with multi-segment targets. This is the single highest-impact technique.

At inference, an Euler ODE solver with 2 function evaluations (NFE=2) is used.

## Key Results

On LJSpeech (NFE=2, 20 raters, 20 utterances): RapFlow-TTS base achieves MOS 3.89 vs. Matcha-TTS (10 NFE) at 3.83, Comospeech (2 NFE) at 3.19, FastSpeech2 (1 NFE) at 3.35. RapFlow-TTS+ (with all techniques) achieves MOS 4.01, WER 3.11%. RTF=0.031, matching FastSpeech2's speed. Ablation (Table 2) confirms adversarial learning yields the largest NISQA improvement (3.78→4.19 on LJSpeech), and that all techniques contribute without redundancy. On VCTK multi-speaker: RapFlow-TTS+ achieves MOS 4.28 (2 NFE), surpassing the Stage-1 model at 10 NFE (MOS 3.83). The consistency model's few-step quality on VCTK is also superior to stage-1 at 10 steps, demonstrating effectiveness on multi-speaker settings. Notably, more NFE steps do not consistently improve quality for consistency models (characteristic behavior).

## Novelty Assessment

The first TTS application of consistency flow matching, with a clean two-stage training recipe and a principled set of improvements. Building consistency on a straight trajectory (FM) rather than a curved one (diffusion) is the key insight — it makes consistency constraints more accurate and effective. The multi-segment extension (S=2) prevents mode collapse at trajectory boundaries. The combination of shared dropout, Huber loss, delta scheduling, and adversarial learning is ablated carefully, making each contribution traceable. The result — 2-step TTS at FastSpeech2 speed but Matcha-TTS quality — is practically significant. The model is compact (18.2M), making the approach accessible.

## Limitations and Open Questions

Quality at 2 NFE remains slightly below ground truth (MOS 4.01 vs. GT 4.42 on LJSpeech). The consistency model property where more steps do not improve (or degrade) quality is a known limitation — multi-step inference is essentially wasted after consistency training. The system is not zero-shot; it conditions on speaker identity via training. The VCTK results show a small WER increase from the base consistency model (3.25%) that is corrected by improved techniques (2.01%), suggesting consistency training can hurt intelligibility when not carefully regularized. Extension to large-scale data and zero-shot capability is left to future work.

## Wiki Connections

RapFlow-TTS advances [[flow-matching]] by introducing consistency constraints on straight ODE trajectories, analogous to how Comospeech advanced [[diffusion-tts]] with consistency distillation. The adversarial learning component connects to [[gan-vocoder]] patterns applied at the mel-spectrogram level. The evaluation framework (MOS + WER + RTF) is documented in [[evaluation-metrics]] and [[subjective-evaluation]].

Related few-step FM acceleration work from Interspeech 2025: [[interspeech-2025-0455]] (APTTS) takes a complementary approach via adversarial post-training on a latent flow-matching TTS model, targeting 4-step inference in latent space. Where RapFlow-TTS enforces velocity-field self-consistency during training of a Matcha-TTS backbone, APTTS uses a discriminator to guide the ODE trajectory post-hoc. Both papers reduce FM TTS inference to 2–4 steps with competitive quality, confirming that aggressive step reduction is achievable with different techniques.
