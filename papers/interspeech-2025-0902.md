---
id: interspeech-2025-0902
title: "VoiceQualityVC: A Voice Conversion System for Studying the Perceptual Effects of Voice Quality in Speech"
authors: [Harm Lameris, Joakim Gustafsson, Éva Székely]
organization: KTH Royal Institute of Technology
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-05-27
task: [VC]
architecture: [VAE, GAN]
conditioning: [speaker-conditioned, prosody-conditioned]
training: [fine-tuning, supervised]
model_size: "39,351,872 parameters"
codec_used: "none"
datasets_train: [LibriTTS-R]
datasets_eval: [LibriTTS-R (held-out), custom stimuli via ElevenLabs multilingual v2]
metrics:
  - name: MOS
    value: 4.93
    system: Breathy VoiceQualityVC (Intimacy)
    testset: Subjective listening test (25 participants)
  - name: MOS
    value: 4.42
    system: Creaky VoiceQualityVC (Intimacy)
    testset: Subjective listening test (25 participants)
code_available: true
demo_available: false
url: https://www.isca-archive.org/interspeech_2025/lameris25_interspeech.html
related_concepts: [voice-conversion, prosody-control, disentanglement, evaluation-metrics, subjective-evaluation]
related_papers: []
---

# VoiceQualityVC: A Voice Conversion System for Studying the Perceptual Effects of Voice Quality in Speech

**Paper:** [https://www.isca-archive.org/interspeech_2025/lameris25_interspeech.html](https://www.isca-archive.org/interspeech_2025/lameris25_interspeech.html)

**One-sentence contribution:** VoiceQualityVC is an open-source voice conversion tool that explicitly conditions on five acoustic voice quality features (creakiness, CPPS, H1-H2, H1-A3, pitch) to produce controlled modal, creaky, and breathy phonation, enabling perceptual studies of voice quality's paralinguistic effects.

## Problem

Voice quality — the mode of phonation (modal, creaky, breathy) — carries significant paralinguistic information about speaker stance, intimacy, and engagement, yet existing VC systems largely ignore it. Research on the perceptual impact of voice quality requires near-identical stimuli that differ only in voice quality, which natural speech cannot easily provide. Prior work on voice quality synthesis conditioned on features like H1-H2 or creakiness existed only in isolation; no open-source tool combined multiple voice quality features into a controllable VC framework for perceptual experimentation.

## Method

VoiceQualityVC extends the FreeVC architecture (a conditional variational autoencoder with adversarial training for one-shot voice conversion). The base model uses a prior encoder with WavLM representations and a bottleneck extractor for content/prosody, a normalizing flow for distribution learning, and a posterior encoder with an RNN-based speaker encoder. Waveforms are generated directly end-to-end.

The key addition is five voice quality encoders — one per feature (creakiness, CPPS, H1-H2, H1-A3, pitch) — each implemented as an affine layer with the same dimensionality as the WavLM representation. Encoded features are added as conditioning to the WavLM embeddings before the bottleneck extractor. A downward projection of the same features serves as global conditioning before the decoder.

Features are annotated at the utterance or frame level: creakiness via CreaPy (threshold-based, frame-level), CPPS via Praat (utterance-level over voiced segments, speaker-standardized), H1-H2 and H1-A3 via Parselmouth (frame-level, speaker-standardized), and pitch via Wavelet Prosody Toolkit (utterance-level average, corpus-standardized). The model is fine-tuned from a pretrained FreeVC checkpoint on the full LibriTTS-R dataset (pre-split train/validation sets), for 7K iterations on 4 × RTX 3090 24GB GPUs, batch size 32. Voice quality encoders are zero-initialized. Total parameters: ~39.4M (+12K over FreeVC).

At inference, voice quality feature values (expressed in standard deviations from the mean) are set manually to produce target phonation types. For creaky voice: high creakiness (~5 SD), lowered CPPS, lower H1-H2 and H1-A3. For breathy: lowered creak, lowered CPPS, high H1-H2 and H1-A3.

## Key Results

**Objective evaluation**: The system demonstrates controllable modification of all five features across the range [-3, 3] SD from the mean. H1-H2 and H1-A3 show speaker-dependent variation (different patterns for male vs. female). CPPS and pitch show predictable control but with some irregularities at extreme values for some voices, likely due to measurement conditions.

**Subjective evaluation** (25 participants per condition, Prolific, £12/hr, ~7.35 min per test):
- Breathy vs. modal voice: Breathy rated significantly more intimate (p=.01) and more invested (p<.01). No significant valence difference.
- Creaky vs. modal voice: Creaky rated significantly less intimate (p=.04) and less positive (p<.01). No significant investment difference.

Results replicate prior findings from natural speech studies and extend them to synthesized voice quality across multiple speakers.

## Novelty Assessment

The primary contribution is practical: a controllable, open-source, multi-feature voice quality VC tool for use as a research instrument, rather than a system optimized for speech naturalness per se. The perceptual findings (breathy = intimate/invested; creaky = detached/negative) replicate and generalize prior work using more ecologically valid multi-speaker synthesized stimuli. The architecture extends FreeVC with minimal parameter additions. The novelty is incremental on the modeling side but genuinely useful for paralinguistic and UX voice design research.

## Limitations and Open Questions

- CPPS is measured over voiced segments rather than sustained vowels, introducing minor irregularities.
- Pitch control generalizes imperfectly across genders due to corpus-level standardization combining both genders.
- The system models acoustic correlates of voice quality rather than voice quality per se; phonetic validation by trained phoneticians would strengthen claims.
- No MUSHRA or absolute MOS for speech naturalness — evaluation focused on paralinguistic perception rather than synthesis fidelity.
- Integration into live dialogue systems for adaptive voice quality in conversation is proposed as future work.

## Wiki Connections

VoiceQualityVC is most directly relevant to [[voice-conversion]] (VC architecture) and [[prosody-control]] (explicit control over phonation features). The use of WavLM representations connects to [[self-supervised-speech]]. The subjective listening design and use of MOS-like scales is relevant to [[subjective-evaluation]]. The work opens a connection to the broader question of whether TTS/VC systems can express paralinguistic stance — a gap also noted in [[spoken-language-model]] research.
