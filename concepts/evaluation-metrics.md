---
slug: evaluation-metrics
title: Evaluation Metrics
aliases: [speech quality metrics, TTS evaluation metrics, objective evaluation, automatic evaluation]
related_concepts: [subjective-evaluation, rlhf-speech, spoken-language-model]
last_updated: 2026-05-26
---

# Evaluation Metrics

## What it is

Evaluation metrics for speech synthesis quantify system performance along several orthogonal dimensions: naturalness (how human-like the speech sounds), intelligibility (whether the content can be understood), speaker similarity (how closely the voice matches a reference), prosody accuracy, and overall quality. Metrics divide into subjective (requiring human listeners) and objective (computed automatically). Objective metrics are preferred for reproducibility and cost, but their correlation with human perception is imperfect.

## Why it matters

Consistent use of evaluation metrics allows fair comparison across systems. However, the field faces persistent issues: different papers use different test sets, different ASR models for WER, and different speaker verification models for SPK-SIM, making comparisons across papers unreliable. [[2025.findings-emnlp.424]] introduces a novel interaction dynamics evaluation task (binary classification of backchannel/interruption/gap/pause events) with its own benchmark, extending the field beyond standard TTS metrics toward dialogue-specific evaluation.

## Current state of the art

**Dominant metrics in 2025 TTS:**
- WER (using HuBERT-Large-LS960-ft or Whisper ASR): primary intelligibility metric in flow-matching TTS papers [[2509.19668]], VC papers [[2507.14534]], and codec papers [[2510.00981]].
- SPK-SIM (WavLM-Large cosine similarity): primary speaker similarity metric for zero-shot TTS [[2509.19668]].
- MOS (1–5 scale, listening test): NMOS (naturalness) and QMOS (quality) are reported with confidence intervals in TTS papers [[2510.00981]]. Small listener panels (6 experts) are common in recent papers.
- UTMOS: automatic MOS predictor used alongside human MOS in codec evaluation [[2510.00981]].
- PESQ: signal-level quality metric used in codec evaluation [[2510.00981]].

**Dialogue / interaction metrics:**
- [[2025.findings-emnlp.424]] establishes binary classification accuracy on 4 interaction event types (backchannel, interruption, gap, pause) as an evaluation framework for spoken dialogue models. GPT-4o scores 54.2% (near-random for a binary task), highlighting that current models lack fine-grained interaction understanding.

## Key variants and sub-approaches

Canonical metrics tracked in this wiki:

| Metric | Full name | Measures | Direction |
|--------|-----------|----------|-----------|
| MOS | Mean Opinion Score | Naturalness | higher is better |
| SMOS | Speaker Similarity MOS | Speaker likeness | higher is better |
| WER | Word Error Rate | Intelligibility (via ASR) | lower is better |
| CER | Character Error Rate | Intelligibility (via ASR) | lower is better |
| SPK-SIM | Speaker Cosine Similarity | Automatic speaker similarity | higher is better |
| UTMOS | Saeki et al. automatic MOS | Predicted naturalness | higher is better |
| DNSMOS | Microsoft DNS MOS | Predicted naturalness | higher is better |
| EER | Equal Error Rate | Anti-spoofing / speaker verification | lower is better |
| MUSHRA | Multiple Stimuli with Hidden Reference and Anchor | Perceptual quality | higher is better |
| PESQ | Perceptual Evaluation of Speech Quality | Signal-level quality | higher is better |
| STOI | Short-Time Objective Intelligibility | Intelligibility | higher is better |
| F0-RMSE | Pitch tracking error | Prosody accuracy | lower is better |

## Comparison to alternatives

Human listening studies (MOS, MUSHRA) remain the gold standard but are expensive and slow. Automatic metrics (UTMOS, DNSMOS) predict human scores and enable rapid iteration. SPK-SIM via WavLM-Large is now accepted as a proxy for speaker similarity MOS (SMOS), though the two do not perfectly correlate. WER via ASR is a reliable intelligibility proxy for English; cross-lingual WER requires language-specific ASR models, complicating multilingual evaluation ([[2509.19668]] uses the same F5-TTS model for English and Mandarin but observes language-specific CFG behavior).

## Year-on-year trajectory

Pre-2023: MOS and WER were the standard; speaker verification cosine similarity emerged as a faster alternative to SMOS. 2024–2025: UTMOS becomes standard in codec evaluation [[2510.00981]]; Seed-TTS-eval (English and Mandarin cross-sentence prompts) becomes a reference benchmark for zero-shot TTS comparison [[2509.19668]]. 2025: Interaction-specific evaluation emerges [[2025.findings-emnlp.424]], extending beyond audio quality to dialogue dynamics classification.

## Open questions

- Is WavLM-Large-based SPK-SIM a sufficient proxy for human speaker similarity MOS? The two are correlated but not interchangeable.
- How should the WER/SIM trade-off be optimally balanced for zero-shot TTS? Both metrics are reported independently; no composite metric is standardized.
- What are the right metrics for evaluating spoken dialogue system naturalness beyond event classification accuracy [[2025.findings-emnlp.424]]?

## Papers

| ID | Title | Venue | Year | Key use of this concept |
|----|-------|-------|------|------------------------|
| [[2025.findings-emnlp.424]] | InteractSpeech: A Speech Dialogue Interaction Corpus for Spoken Dialogue Model | EMNLP 2025 | 2025 | Introduces interaction event classification (backchannel/interruption/gap/pause binary accuracy) as a new evaluation paradigm for spoken dialogue models |
| [[2025.acl-long.1498]] | Analyzing and Mitigating Inconsistency in Discrete Speech Tokens for Neural Codec Language Models | ACL 2025 | 2025 | Introduces consistency accuracy — a novel codec-level metric measuring token agreement between context-free and context-inclusive encodings of the same audio segment — used alongside PESQ, ViSQOL, WER, SPK-SIM, and UTMOS |
| [[2025.acl-long.598]] | Advancing Zero-shot TTS Intelligibility across Diverse Domains via Preference Alignment | ACL | 2025 | Uses WER (Whisper-large-v3 for EN, Paraformer-zh for ZH/code-switching), SPK-SIM (WavLM TDNN cosine), UTMOS, N-CMOS, and reading accuracy (binary human intelligibility judgment) across four domain types; introduces reading accuracy as a targeted intelligibility evaluation complementary to WER |
