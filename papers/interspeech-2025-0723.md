---
id: "interspeech-2025-0723"
title: "Counterfactual Activation Editing for Post-hoc Prosody and Mispronunciation Correction in TTS Models"
authors: ["Kyowoon Lee", "Artyom Stitsyuk", "Gunu Jho", "Inchul Hwang", "Jaesik Choi"]
organization: KAIST / Samsung Electronics
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-05-30
task: [TTS]
architecture: [transformer-enc-dec, VAE]
conditioning: [text-conditioned]
training: [supervised]
model_size: "~28M params (Tacotron 2)"
codec_used: none
datasets_train: [LJSpeech]
datasets_eval: [LJSpeech]
metrics:
  - name: WER
    value: 0.056
    system: CAE (after correction)
    testset: LJSpeech (78 mispronunciation targets)
  - name: WER
    value: 0.151
    system: baseline (before correction)
    testset: LJSpeech (78 mispronunciation targets)
  - name: CMOS
    value: 0.764
    system: CAE (after correction vs. before)
    testset: LJSpeech
code_available: null
demo_available: true
url: https://www.isca-archive.org/interspeech_2025/lee25f_interspeech.html
related_concepts: [prosody-control, disentanglement, self-supervised-speech]
related_papers: []
---
> [!abstract] Interspeech · 2025 · Conference
> **Kyowoon Lee et al.** (KAIST / Samsung Electronics) · [→ Paper](https://www.isca-archive.org/interspeech_2025/lee25f_interspeech.html) · Demo: ✓ · Code: ?
>
> Counterfactual Activation Editing (CAE) enables post-hoc control of prosody and mispronunciation correction in a frozen pre-trained TTS encoder by performing gradient-ascent edits in a manifold-preserving VAE latent space, without any retraining.

## Problem

Existing prosody control methods either require specialized modules integrated at training time or depend on latent-space sampling that is difficult to direct precisely at inference. Mispronunciation correction has similarly relied on grapheme-to-phoneme dictionaries, which are unavailable in low-resource settings. Neither class of approach supports post-hoc adjustment of an already-trained encoder-decoder TTS model.

## Method

CAE operates on the intermediate activations of a pre-trained TTS encoder (Tacotron 2, ~28M parameters, trained on LJSpeech). The method has three layers:

1. **Acoustic Correlation Analysis**: A logistic or linear regression classifier is trained to predict prosodic properties (pitch, duration, energy) or semantic tokens (HuBERT layer-6 k-means) from the encoder's last-layer LSTM activations. This locates which neurons carry the relevant information.

2. **Manifold Preserving CAE**: To avoid pushing activations off the data manifold during gradient ascent, activations are first encoded into a beta-VAE latent space (dim=16). Gradient ascent toward the target class is performed in this compressed space, then decoded back.

3. **Prototype Loss**: A VQ-VAE codebook anchors edits to the nearest discrete prototype, preventing large latent shifts from altering unintended speech content (phoneme identity during prosody edits, and vice versa).

For pronunciation correction, the target is to adjust semantic tokens (HuBERT k-means clusters) per grapheme to match a speech-only correction query. Edits can be localised to specific phonemes or words by selecting the corresponding encoder positions.

## Key Results

On a set of 78 mispronounced words drawn from out-of-vocabulary or difficult grapheme-input cases:

- WER drops from 0.151 to 0.056 after CAE correction
- PER drops from 0.069 to 0.017
- Cosine similarity (sentence embedding) improves from 0.954 to 0.969
- Token similarity improves from 0.892 to 0.929
- CMOS improvement of +0.764 points (20 listeners)

For prosody control, Manifold Preserving CAE achieves more accurate pitch/duration ratio changes than direct CAE or the truncation trick baseline, with the prototype loss critical for preserving intelligibility under large prosody shifts.

## Novelty Assessment

The application of counterfactual editing from interpretability research (neuron-level gradient ascent) to TTS control is genuinely novel. The manifold-preserving constraint via beta-VAE latent space, combined with VQ-VAE prototype anchoring, is a well-motivated engineering contribution that directly addresses the known failure mode of off-manifold edits. The scope is deliberately narrow (Tacotron 2 encoder-decoder architecture; single-speaker LJSpeech), so generalization to flow-matching or autoregressive LM TTS is undemonstrated. The method is model-agnostic in principle but validated on one architecture.

## Limitations and Open Questions

- Validated only on Tacotron 2; applicability to flow-matching or codec-based LM TTS is untested.
- Feature entanglement remains: controlling duration slightly shifts pitch (acknowledged in the paper), suggesting that targeting specific neurons rather than full activation vectors is needed.
- Mispronunciation correction requires a speech-only query word as supervision — not purely zero-shot.
- LJSpeech is single-speaker; multi-speaker generalization is not evaluated.

## Wiki Connections

This paper directly informs [[prosody-control]] by providing a post-hoc, retraining-free alternative to all existing explicit-module approaches. It relates to [[disentanglement]] by using VAE latent spaces to separate prosody from pronunciation edits. The use of HuBERT semantic tokens for pronunciation analysis connects to [[self-supervised-speech]]. The work cites the Speech Audio Corrector (SAC) as the closest prior approach to pronunciation correction and presents CAE as a model-agnostic complement.
