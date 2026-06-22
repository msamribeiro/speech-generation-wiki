---
id: "interspeech-2025-0973"
title: "A Dataset for Automatic Assessment of TTS Quality in Spanish"
authors: ["Alejandro Sosa Welford", "Leonardo Pepino"]
organization: null
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: "2025-08-17"
ingested_date: "2026-05-30"
task: [TTS, evaluation]
architecture: []
conditioning: []
training: [supervised, fine-tuning]
model_size: "NISQA fine-tuned: 304,539 params; DenseMOS: 115,086 params (excluding wav2vec 2.0)"
codec_used: none
datasets_train: ["custom Spanish TTS evaluation dataset (4,326 samples from 52 systems/voices)", "NISQA Corpus (English, for pretraining)"]
datasets_eval: ["custom Spanish TTS evaluation dataset (test split: 392 samples)"]
metrics:
  - name: MOS
    value: 0.73
    system: NISQA fine-tuned (Pearson CC)
    testset: Spanish TTS dataset test split
  - name: MOS
    value: 0.62
    system: DenseMOS (Pearson CC)
    testset: Spanish TTS dataset test split
  - name: MOS
    value: 0.81
    system: NISQA fine-tuned (MAE)
    testset: Spanish TTS dataset test split
  - name: MOS
    value: 0.80
    system: DenseMOS-960h (MAE)
    testset: Spanish TTS dataset test split
code_available: true
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/sosawelford25_interspeech.html"
related_concepts: [evaluation-metrics, subjective-evaluation, self-supervised-speech, multilingual-tts]
related_papers: []
field_significance:
  level: "moderate"
  type: [dataset-contribution, evaluation-contribution]
generation:
  date: 2026-06-22
  agent: speech-generation-review-agent
  model: claude-sonnet-4-6
  commit: "24ea64d"
---
> [!abstract] Interspeech · 2025 · Conference
> **Alejandro Sosa Welford et al.** · [→ Paper](https://www.isca-archive.org/interspeech_2025/sosawelford25_interspeech.html) · Demo: ? · Code: ✓
>
> This paper introduces the first Spanish-language TTS quality evaluation dataset (4,326 samples from 52 systems and speakers), collected with a standardized subjective listening test, and validates it by training automatic MOS predictors that improve on out-of-the-box NISQA.

## Problem

TTS quality evaluation predominantly relies on subjective listening tests using datasets in English or Mandarin Chinese. Automatic MOS predictors — tools that replace costly human evaluations — are correspondingly trained primarily on English data. No dedicated Spanish dataset existed for training or evaluating such automatic naturalness predictors, limiting cross-lingual generalization and the development of Spanish-specific evaluation tools. Existing non-intrusive quality metrics like NISQA (trained on English POLQA-based data) are claimed to be language-independent but lacked validation on Spanish.

## Method

**Dataset construction:** The authors collected audio from 12 TTS systems covering a range of synthesis paradigms — commercial neural APIs (Amazon Polly, Microsoft Azure, Speechelo, Neurasound), concatenative (Loquendo), parametric (Dewitte), and neural (FastPitch, DC-TTS, Tacotron 2) — generating 40 unique voices (varying speaker gender, accent, regional dialect across es-AR, es-ES, es-MX, es-US). Six human speakers from an Argentinian Spanish corpus served as quality anchors. Two data augmentation techniques were applied to increase variability: Vocal Tract Length Perturbation (VTLP, deformation factor 0.9–1.1) and Griffin-Lim phase alteration (simulating vocoder artifacts). Total: 4,326 rated samples.

**Subjective test design:** Followed ITU-T Rec. P.807 standard. Participants (N=92 native Spanish speakers, mean age 29.9) rated speech naturalness on a 1–5 scale. Samples were presented in batches of 5 spanning diverse quality levels to mitigate range equalization bias. Validity checks excluded ratings submitted too quickly and participants who mis-rated human speech equivalently to heavily degraded synthetic speech. Final dataset: 4,326 ratings (50 excluded). Inter-rater agreement: Krippendorff's Alpha = 0.56, ICC(2,1) = 0.68.

**Automatic predictors:**
1. NISQA fine-tuned: The existing NISQA v1.0 model (CNN + LSTM on mel-spectrograms, pretrained on English NISQA Corpus) was fine-tuned on 3,139 training samples using Adam optimizer, MSELoss, lr=0.001.
2. DenseMOS: A new lightweight downstream model trained on wav2vec 2.0 base representations. The 13 layer outputs (CNN encoder + 12 transformer blocks) are time-averaged and combined via a learned weighted average (α weights), then passed through two FC layers (128 neurons, ReLU, dropout 0.6) and a linear output head scaled to MOS range. DenseMOS-960h uses wav2vec2-base-960h (fine-tuned for ASR) embeddings. Both use MSELoss, Adam, with separate learning rates for the averaging weights (0.001) and MLP layers (0.0001).

## Key Results

NISQA v1.0 (out-of-box, trained on English) achieves PCC=0.71, MAE=0.99, RMSE=1.19 on the Spanish test set. Fine-tuning on the new Spanish dataset improves this to PCC=0.73, MAE=0.81, RMSE=1.06 — consistent with improvements reported by Mittag (2020) on Blizzard/VCC datasets. DenseMOS achieves PCC=0.62, MAE=0.81, and DenseMOS-960h achieves PCC=0.60, MAE=0.80. All automatic models struggle to predict MOS scores near 4.0 (underrepresented in the dataset). The CNN encoder output of wav2vec 2.0 receives the highest weight in DenseMOS's averaging layer, suggesting local, low-level acoustic features (25 ms resolution) are most informative for naturalness prediction, more so than the higher-level contextual representations from deeper transformer blocks.

## Novelty Assessment

The primary contribution is the dataset itself — the first Spanish MOS corpus for automatic TTS evaluation — rather than the models. The NISQA fine-tuning is a straightforward transfer learning exercise. DenseMOS is a simple, effective architecture drawing on prior work using wav2vec 2.0 for speech tasks; its comparable performance to fine-tuned NISQA with far fewer parameters (115k vs. 305k) is interesting. The finding that CNN-based local features dominate over contextual transformer representations for MOS prediction adds to growing understanding of what representations matter for perceptual quality estimation.

The dataset is released publicly alongside source code, making it a durable community resource. Its relatively small scale (4,326 samples, mostly single ratings) and geographic skew toward Rioplatense Spanish (23/52 voices) are acknowledged limitations.

## Field Significance

Moderate — This paper fills a concrete gap in the evaluation infrastructure for Spanish TTS: no labelled naturalness corpus existed for training or benchmarking automatic MOS predictors in that language. The automatic predictors introduced (fine-tuned NISQA, DenseMOS) are lightweight rather than state-of-the-art, and the dataset's scale and dialect coverage are limited. Its value lies primarily in providing a community resource and a methodology that can be replicated for other under-resourced languages.

## Claims

- Automatic MOS predictors trained primarily on English data underperform on Spanish TTS, and language-specific fine-tuning provides meaningful improvement even with small datasets. *(§4.2, Table 2)*
- Low-level local acoustic features from SSL encoder layers are more informative for naturalness prediction than higher-level contextual representations from deeper transformer blocks. *(§4.2, Figure 2)*
- The scarcity of samples rated near MOS 4.0 in evaluation datasets introduces a systematic prediction bias toward the dataset mean, suggesting label distribution matters as much as dataset size for MOS predictor quality. *(§4.2)*
- Lightweight downstream models trained on frozen SSL representations can achieve MOS prediction performance comparable to fine-tuned specialist models, despite having fewer than half the parameters. *(§4.2, Table 2)*

## Limitations and Open Questions

Most audio samples received only a single rating, making inter-rater reliability estimates noisy at the individual-item level. MOS scores near 4.0 are underrepresented, creating a systematic bias in model predictions toward the dataset mean. All models exhibit this bias toward predicting mean MOS, suggesting alternative loss functions (e.g., focal loss, distribution-matching losses) may help. The geographic distribution is skewed toward Argentine Spanish, limiting generalization to other Spanish dialects. The dataset does not include modern large-scale zero-shot or LLM-based TTS systems (VALL-E, Voicebox, etc.).

## Wiki Connections

This paper contributes directly to [[evaluation-metrics]] and [[subjective-evaluation]] by providing the first Spanish-language benchmark for automatic naturalness prediction. The use of wav2vec 2.0 representations places it in the [[self-supervised-speech]] literature. The finding about which wav2vec 2.0 layers are most informative for MOS prediction complements work on using SSL representations for speech quality assessment. The dataset could serve as a resource for future [[multilingual-tts]] evaluation research in Spanish.
