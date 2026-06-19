---
id: "interspeech-2025-0787"
title: "Gradual modeling of the Lombard effect by modifying speaker embeddings from a Text-To-Speech model"
authors: ["Thiago Henrique Gomes Lobato", "Magnus Schäfer"]
organization: HEAD acoustics GmbH
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-05-30
task: [TTS]
architecture: [autoregressive-LM]
conditioning: [speaker-conditioned, text-conditioned]
training: [supervised, fine-tuning]
model_size: "Metavoice 1B (1B params)"
codec_used: "EnCodec (via Metavoice 1B)"
datasets_train: ["Lombard Grid Corpus (54 speakers, 50 plain-Lombard utterance pairs per speaker)"]
datasets_eval: ["ITU-T P.501 (12 speakers)"]
metrics:
  - name: MOS
    value: 3.31
    system: ITU Full Transform (ITU-FT, proposed)
    testset: ITU-T P.501 (plausibility)
  - name: MOS
    value: 2.19
    system: ITU Level Only (ITU-LO, baseline)
    testset: ITU-T P.501 (plausibility)
  - name: MOS
    value: 3.26
    system: ITU Full Transform (speaker similarity)
    testset: ITU-T P.501
  - name: MOS
    value: 3.81
    system: Lombard Grid Ideal Interpolation (upper bound)
    testset: LG-II (speaker similarity)
code_available: true
demo_available: true
url: https://www.isca-archive.org/interspeech_2025/lobato25_interspeech.html
related_concepts: [speaker-adaptation, voice-conversion, zero-shot-tts]
related_papers: []
---
> [!abstract] Interspeech · 2025 · Conference
> **Thiago Henrique Gomes Lobato et al.** (HEAD acoustics GmbH) · [→ Paper](https://www.isca-archive.org/interspeech_2025/lobato25_interspeech.html) · Demo: ✓ · Code: ✓
>
> A feedforward network trained on the Lombard Grid Corpus learns to transform plain speaker embeddings into Lombard variants, which are fed to Metavoice 1B to generate scalable Lombard speech; spherical interpolation between plain and Lombard embeddings enables gradual noise-level-calibrated Lombard intensity control.

## Problem

Telecommunication and in-car communication system evaluation requires realistic Lombard speech — speech involuntarily modified by speakers in the presence of loud background noise (increased pitch, longer duration, higher level). Recording Lombard speech per speaker is costly and impractical at scale. The standard ITU-T P.1150 approach applies only a level correction, ignoring pitch and duration changes that are central to the Lombard effect. No scalable, data-efficient approach exists for generating Lombard speech with continuously controllable noise-level-dependent intensity.

## Method

**Data**: Lombard Grid Corpus provides 50 paired plain-Lombard utterances for 54 speakers (~5 seconds each), recorded with 80 dB background noise via headphones. ITU-T P.501 standard signals (12 speakers, 6M/6F) are used for evaluation.

**TTS base model**: Metavoice 1B, a GPT-style autoregressive model predicting EnCodec tokens, conditioned on 256-dimensional LSTM-derived speaker embeddings (unit hypersphere projection). The model was chosen because it was trained on emotional speech and can represent expressive voice characteristics including Lombard-like patterns.

**Embedding transformation**: A small feedforward network (two SwiGLU layers: 256→64→256, ReLU, unit-sphere projection) is trained to maximize cosine similarity between predicted and real Lombard speaker embeddings. 5-fold cross-validation on Lombard Grid achieves 0.95 average cosine similarity.

**Post-processing**: (1) Level adjustment per ITU-T P.1150 as a function of background noise level; (2) Speech stretching via Vocos (mel-spectrogram vocoder) to match the average 1.08× duration ratio observed in Lombard Grid.

**Interpolation calibration**: To generate Lombard speech at intermediate noise levels (50–80 dB), plain and Lombard embeddings are spherically interpolated with a Lombard strength factor x ∈ [0,100]. The fundamental frequency change is assumed linear with background noise level, and a cubic fit to ITU-speaker F0 estimates calibrates the mapping from x to dB level. Similarly, duration stretching up to 1.08× is linearly scaled.

## Key Results

Listening test (10 participants, 48 comparisons of ~15s sequences with increasing background noise):

- **Plausibility**: ITU-FT (proposed) 3.31 vs. ITU-LO (level-only baseline) 2.19 — a +1.12 MOS improvement, confirming that pitch and duration changes are essential for perceptual plausibility.
- **Speaker similarity**: ITU-FT 3.26 vs. LG-II (ideal Lombard embeddings) 3.81 — only 0.55 MOS below the ideal upper bound; much better than random speakers (1.88).
- **Naturalness**: ITU-FT 3.47, slightly higher than LG-II (3.19) and ITU-LO (3.08).
- **Overall quality**: All approaches comparable (~3.06–3.37); transformation does not degrade base TTS quality.

The 5-fold cosine similarity of 0.95 on Lombard Grid validation confirms that the plain-to-Lombard embedding transformation is well-learned from limited data.

## Novelty Assessment

The key insight — that Lombard speech characteristics can be captured in the speaker embedding space of a modern TTS model and transferred via a simple feedforward network — is practically useful and novel for the Lombard speech generation problem. The spherical interpolation calibration to physical noise levels is an elegant engineering contribution. The approach is low-data (only ~250 seconds of Lombard speech per speaker), which is realistic given the cost of Lombard recordings. The limitation is that Metavoice 1B is English-only and closed-source; generalization to other languages or more controllable TTS architectures is a clear next step.

## Limitations and Open Questions

- Metavoice 1B is English-only; multilingual or language-adaptive Lombard generation is unaddressed.
- Only 10 listeners in the listening test; statistical significance is limited.
- The level calibration used ITU-T P.1150, which participants judged as too quiet at the highest noise levels — the standard may underestimate real Lombard level increases.
- The feedforward embedding transform was trained on Lombard Grid (slow, scripted utterances) and tested on ITU-P.501 (different style); domain mismatch effects are partially visible.
- Dynamic/time-varying background noise scenarios are left as future work.

## Wiki Connections

This paper contributes to [[speaker-adaptation]] by demonstrating that embedding-space manipulation can transfer a specific speaking style (Lombard) across speakers with very limited training data. The approach conceptually aligns with [[voice-conversion]] — transforming a speaker's voice characteristics while preserving identity — and shows that latent space interpolation for graduated style control, discussed in [[disentanglement]], is effective in the speaker embedding manifold. The use of Metavoice 1B (an autoregressive LM TTS model) connects to [[autoregressive-codec-tts]].
