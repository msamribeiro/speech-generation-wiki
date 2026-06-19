---
id: "interspeech-2025-0854"
title: "Bridging the Training–Inference Gap in TTS: Training Strategies for Robust Generative Postprocessing for Low-Resource Speakers"
authors: ["Frank Zalkow", "Paolo Sani", "Kishor Kayyar Lakshminarayana", "Emanuël A. P. Habets", "Nicola Pia", "Christian Dittmar"]
organization: "Fraunhofer IIS"
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: "2025-08-17"
ingested_date: "2026-05-30"
task: [TTS]
architecture: [GAN, flow-matching, transformer-enc-dec]
conditioning: [text-conditioned, speaker-conditioned]
training: [supervised]
model_size: "not reported (acoustic model: ForwardTacotron; postprocessing: GAN U-Net or CFM Transformer U-Net)"
codec_used: none
datasets_train: ["Hi-Fi TTS (3 speakers, ~15 hrs each)", "PTDB-TUG (2 speakers, ~13–14 min each)"]
datasets_eval: ["PTDB-TUG (15 hold-out samples per speaker)"]
metrics:
  - name: MUSHRA
    value: 74.8
    system: GAN Proposed
    testset: PTDB-TUG combined (F05+M03)
  - name: MUSHRA
    value: 72.3
    system: GAN Standard
    testset: PTDB-TUG combined (F05+M03)
  - name: MUSHRA
    value: 79.8
    system: CFM Proposed
    testset: PTDB-TUG combined (F05+M03)
  - name: MUSHRA
    value: 79.0
    system: CFM Standard
    testset: PTDB-TUG combined (F05+M03)
  - name: MUSHRA
    value: 50.9
    system: ForwardTacotron baseline
    testset: PTDB-TUG combined (F05+M03)
code_available: null
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/zalkow25_interspeech.html"
related_concepts: [gan-vocoder, flow-matching, speaker-adaptation, evaluation-metrics, subjective-evaluation]
related_papers: []
---
> [!abstract] Interspeech · 2025 · Conference
> **Frank Zalkow et al.** (Fraunhofer IIS) · [→ Paper](https://www.isca-archive.org/interspeech_2025/zalkow25_interspeech.html) · Demo: ? · Code: ?
>
> A data augmentation strategy for training generative spectrogram postprocessing models that simulates low-resource acoustic model artifacts using subsampled high-resource speakers, consistently improving naturalness for low-resource TTS speakers across both GAN and flow-matching postprocessors.

## Problem

Traditional TTS pipelines separate the acoustic model (predicting mel spectrograms from text) from the vocoder (converting spectrograms to waveforms). Acoustic models trained with reconstruction losses produce "oversmoothed" spectrograms lacking fine-grained noise-like texture found in real speech. Since vocoders are trained on ground-truth spectrograms but receive synthetic (oversmoothed) spectrograms at inference, there is a training-inference mismatch that produces audible artifacts.

Generative postprocessing models (GANs, normalizing flows) placed between the acoustic model and vocoder can compensate for this mismatch — but they require paired (synthetic, real) spectrogram training data. For low-resource speakers (< 15 minutes of audio), the acoustic model produces stronger artifacts, and there are too few training pairs for the postprocessor to learn to compensate. The combination of stronger artifacts and fewer examples means existing methods fail to generalize to low-resource speakers.

## Method

The key insight is that the training data scarcity for the postprocessor can be addressed by simulating low-resource artifacts from high-resource speakers. The proposed procedure:

1. From high-resource datasets (H1, H2, H3 — three Hi-Fi TTS speakers, ~15 hrs each), create subsampled "simulated low-resource" variants (H1', H2', H3') matched in duration to the actual low-resource target (L1, ~13–14 min).
2. Train multiple multi-speaker acoustic models: A1 uses full high-resource data + L1; A2 uses H1' + full H2, H3 + L1; A3 uses full H1 + H2' + H3 + L1; A4 uses full H1, H2 + H3' + L1. Each acoustic model sees L1 in its low-resource form.
3. For each high-resource speaker Hi, use the acoustic model that was trained with the subsampled version of Hi to generate synthetic spectrograms. This ensures the synthetic spectrograms exhibit low-resource-like artifacts. Pair these with ground-truth spectrograms.
4. Train a single postprocessing model on the compound dataset (L1 pairs from A1, H1 pairs from A2, H2 pairs from A3, H3 pairs from A4).

The subsampling uses a kernel-density estimate of the low-resource set's duration distribution to weight sampling, so simulated sets match the real duration distribution.

Two postprocessing architectures were tested: (a) a GAN with a convolutional U-Net generator (pix2pix-style), conditioned on the acoustic model's spectrogram; (b) a flow-matching (CFM) Transformer U-Net (adapted from FlowMAC), conditioned similarly, using 32 ODE solver steps (Euler method) with classifier-free guidance (scale 1.0) and temperature τ = 0.7.

The acoustic model is an extended ForwardTacotron with explicit pitch, energy, voicing, and duration predictors. Ground-truth prosody features are provided at inference in all evaluations to isolate naturalness from prosody variation. StyleMelGAN is used as the vocoder throughout.

Evaluation used SCOREQ (objective MOS-distance predictor), MUSHRA-style multi-stimulus listening tests (28 retained participants for F05, 23 for M03), and ranking-by-elimination (RBE) tests with Plackett-Luce analysis.

## Key Results

Both proposed and standard approaches substantially outperform the ForwardTacotron baseline (combined MUSHRA: 50.9) and the noise-augmentation baseline (45.9 — which actually degrades naturalness here because it primarily improves prosody prediction, not spectrogram texture).

GAN with proposed data generation: MUSHRA 74.8 vs. standard 72.3 (statistically significant improvement, Wilcoxon p ≤ 0.01). CFM with proposed data generation: MUSHRA 79.8 vs. standard 79.0 (not statistically significant). CFM is the stronger postprocessor overall. The CFM-proposed system was preferred over the reference in 26.6% of RBE rankings, indicating it sometimes surpasses vocoded ground-truth quality.

SCOREQ results: CFM Proposed 0.27, CFM Standard 0.28, GAN Proposed 0.32, GAN Standard 0.34, ForwardTacotron 0.39 (lower = better). Consistent with listening test ordering.

## Novelty Assessment

The core contribution is a practical training-data generation recipe, not a new architecture. The idea of simulating low-resource conditions from high-resource data via subsampling is straightforward but effective. The careful duration-distribution matching via KDE is a useful detail. The paper thoroughly tests on two generative architectures (GAN and CFM) and two speakers (male/female), giving the results credibility. The RBE/Plackett-Luce evaluation methodology is more rigorous than standard MUSHRA and the finding that CFM-proposed sometimes outperforms vocoded reference is noteworthy.

The limitation that the improvement for CFM is not statistically significant means the method's benefit is more clearly demonstrated for GAN-based postprocessors.

## Limitations and Open Questions

The method requires training multiple acoustic models (4 in the experiments), increasing compute cost proportional to the number of high-resource speakers used. Evaluation uses ground-truth prosody, making it unclear how the approach fares in full end-to-end synthesis with predicted prosody. Only two low-resource speakers (one male, one female) are tested; generalization to different accent or language scenarios is not validated. The authors themselves acknowledge the noise-augmentation baseline performs worse here because it improves prosody — the interactions between prosody control and spectrogram naturalness are not fully resolved.

## Wiki Connections

This paper directly addresses the training-inference mismatch problem in [[gan-vocoder]] and [[flow-matching]] postprocessing pipelines for [[speaker-adaptation]] to low-resource voices. The data simulation strategy is conceptually related to data augmentation in low-resource TTS, which intersects with [[multilingual-tts]] challenges. The evaluation methodology (MUSHRA/RBE with Plackett-Luce) is discussed further in [[subjective-evaluation]] and [[evaluation-metrics]]. The CFM postprocessing module relates to [[flow-matching]] as a spectrogram enhancement technique rather than a full end-to-end TTS system.
