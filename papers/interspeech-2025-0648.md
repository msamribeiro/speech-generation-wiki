---
id: interspeech-2025-0648
title: "MIKU-PAL: An Automated and Standardized Multimodal Method for Speech Paralinguistic and Affect Labeling"
authors: [Yifan Cheng, Ruoyi Zhang, Jiatong Shi]
organization: Fish Audio
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-05-30
task: [TTS, evaluation]
architecture: []
conditioning: [emotion-conditioned]
training: [supervised, fine-tuning]
model_size: "not reported"
codec_used: "none"
datasets_train: [MIKU-EmoBench (released), IEMOCAP (eval), MELD (eval), MSP-Podcast (eval)]
datasets_eval: [MELD, IEMOCAP, MIKU-EmoBench]
metrics:
  - name: MOS
    value: 4.12
    system: MIKU-EmoBench-ft Fish-Speech
    testset: MELD test set
  - name: MOS
    value: 3.91
    system: Fish-Speech (baseline)
    testset: MELD test set
  - name: WER
    value: 2.4
    system: MIKU-EmoBench-ft Fish-Speech
    testset: MELD test set
code_available: true
demo_available: null
url: https://www.isca-archive.org/interspeech_2025/cheng25_interspeech.html
related_concepts: [emotion-synthesis, evaluation-metrics, subjective-evaluation]
related_papers: [2412.10117, 2406.02430, 2301.02111]
---
> [!abstract] Interspeech · 2025 · Conference
> **Yifan Cheng et al.** (Fish Audio) · [→ Paper](https://www.isca-archive.org/interspeech_2025/cheng25_interspeech.html) · Demo: ? · Code: ✓
>
> MIKU-PAL is a fully automated multimodal pipeline using Gemini 2.0 Flash for zero-shot emotion labeling of video-sourced speech that achieves human-level accuracy (68.5% on MELD) with near-perfect consistency (Fleiss κ=0.93) at ~50 cents/hour, enabling the MIKU-EmoBench dataset (131.2h, 26 emotion categories).

## Problem

Emotional speech datasets are severely limited in scale (typically under 300 hours) and emotion granularity (5-8 categories, usually Ekman's six basic emotions). Manual annotation is expensive (~$10/hour) and inconsistent — human annotators on IEMOCAP and MELD achieve Fleiss κ of only ~0.4, meaning there is very low inter-rater agreement. This bottleneck prevents training large-scale emotional TTS systems that need millions of hours of data. Additionally, the basic emotion taxonomy is being challenged by psychological research showing that human emotions form a continuous 27-category space (Cowen & Keltner, 2017), which existing datasets do not capture.

## Method

MIKU-PAL (Multimodal Intelligence Kit for Understanding — Paralinguistic and Affect Labeling) is a three-stage pipeline:

**Stage 1 — Audio preprocessing.** Raw video audio is passed through MDX-Net (dual-stream music source separation) to extract clean vocals, achieving a 36% SNR improvement. Whisper-large-v3 provides word-level timestamps and transcriptions for downstream alignment and MLLM input.

**Stage 2 — Vision preprocessing.** Faces are detected using S3FD or DSFD. Active speaker identification uses TalkNet (92% accuracy on AVA challenge) to ensure correct speaker attribution. Visual preprocessing improves MELD accuracy by 25.6% in ablation.

**Stage 3 — MLLM emotion analysis.** Gemini 2.0 Flash performs zero-shot multimodal emotion analysis using a structured prompt: a task description, a textual description of 26 emotion categories (derived from Cowen's 27-category psychological framework), and an output structure. The user prompt contains only raw video and text. The model outputs emotion intensity scores per category, textual rationale, and a multi-modal judgment. This zero-shot design allows flexible emotion classification without category-specific training.

The pipeline runs at 1:12 speed ratio on 720p 30fps video, retains 42% of raw YouTube content as usable labeled audio, and costs ~$0.50/hour using Gemini 2.0 Flash API.

**MIKU-EmoBench dataset.** The resulting dataset contains 131.2 hours (65,970 utterances, mean 7.16s) from diverse YouTube content (interviews, movies, daily conversations), covering 26 emotion categories with intensity scores and textual rationales per utterance.

**Downstream TTS validation.** Fish-Speech is incrementally fine-tuned with emotion special tokens on each dataset variant (IEMOCAP, MELD, MSP-Podcast, MIKU-EmoBench) and tested on the MELD test set. MIKU-EmoBench fine-tuning achieves MOS 4.12 (vs. 3.91 baseline Fish-Speech) and emotion similarity 0.92 (vs. 0.88 baseline), the best of all fine-tuned variants.

## Key Results

On MELD test set, MIKU-PAL achieves 68.5% accuracy with Fleiss κ=0.95, compared to human annotators at Fleiss κ=0.43. On IEMOCAP, accuracy is 58.6% (humans: 72.9%) with Fleiss κ=0.93 (humans: 0.40). The consistency advantage is the headline result: MIKU-PAL is approximately 2x more consistent than human annotators at roughly 1/20th of the cost.

Downstream TTS: MIKU-EmoBench-ft Fish-Speech achieves MOS 4.12 vs. 3.91 baseline, WER 2.4%, SPK-SIM 0.792, emotion similarity 0.92. CosyVoice 2.0 (zero-shot instruction-conditioned) achieves emotion similarity 0.87 — below the fine-tuned Fish-Speech on MIKU-EmoBench.

A mixed emotion annotation experiment on 10,000 YouTube segments using t-SNE visualization confirms that MIKU-PAL captures psychologically valid emotion transitions (e.g., admiration → love, joy → satisfied → excitement) consistent with Cowen's research.

## Novelty Assessment

The primary contribution is the pipeline design and resulting dataset, not a new model architecture. The insight that consistency matters more than accuracy for downstream TTS training (Fleiss κ 0.93 vs. 0.40, even if accuracy is slightly lower) is practically important. The 26-category taxonomy grounded in psychological research is a principled extension beyond Ekman's basic emotions. Using a multimodal LLM for zero-shot emotion labeling rather than training a dedicated classifier is an effective engineering shortcut that also enables flexibility. The MIKU-EmoBench dataset provides a substantially larger and more emotionally diverse benchmark than existing alternatives.

## Limitations and Open Questions

MIKU-PAL accuracy (58.6–68.5%) remains below human accuracy on IEMOCAP (72.9%). Performance depends on the capabilities and potential biases of Gemini 2.0 Flash, which may change across model versions. YouTube-sourced data introduces demographic and cultural biases from the content distribution. Current SER models cannot reliably classify all 26 fine-grained emotion categories, so full evaluation of MIKU-EmoBench is limited. The pipeline retains only 42% of raw video, which is efficient but still discards a large fraction. Multilingual generalization beyond English is not demonstrated.

## Wiki Connections

MIKU-PAL directly enables [[emotion-synthesis]] by providing large-scale, high-quality labeled emotional speech. The emotion taxonomy discussion connects to [[evaluation-metrics]] and [[subjective-evaluation]] — the paper argues that existing evaluation frameworks rely on flawed emotion categories. It builds on the Fish-Speech system (in-corpus) and fine-tunes using its architecture. CosyVoice 2.0 ([[2412.10117]]) and SeedTTS ([[2406.02430]]) are cited as context for scale of speech LM training data. VALL-E ([[2301.02111]]) is cited as a foundational codec language model for TTS.
