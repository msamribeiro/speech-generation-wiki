---
slug: subjective-evaluation
title: Subjective Evaluation
aliases: [listening tests, MOS studies, perceptual evaluation, crowdsourced evaluation, human evaluation]
related_concepts: [evaluation-metrics, rlhf-speech]
last_updated: 2026-06-01
status: established
---

## Executive Summary

> [!abstract]
> Subjective evaluation collects human perceptual judgments about synthesized speech naturalness, quality, and speaker similarity, and remains the gold standard for TTS/VC assessment despite the availability of automatic proxies. The field lacks a uniform evaluation protocol — evaluator counts, sample sets, and test conditions vary substantially across papers, making cross-paper CMOS/SMOS comparisons unreliable. A trend toward releasing evaluation prompts and model checkpoints is improving reproducibility.

## Current Status

established — Subjective evaluation (MOS, CMOS, SMOS, MUSHRA) is required for publication at most speech and NLP venues and is universally reported in TTS papers. However, methodology is unstandardized: evaluator counts range from 6 to 53, sample counts vary widely, and test sets differ across papers, limiting the interpretability of reported scores.

## Why This Matters

Objective metrics are proxies for human perception but do not always correlate well with it. UTMOS and DNSMOS are trained to predict MOS but can fail on out-of-distribution systems. WER measures intelligibility but not prosody or naturalness. Subjective evaluation remains the gold standard for assessing naturalness and speaker identity. It is required for publication at most speech and NLP venues.

## Core Idea

Subjective evaluation in TTS refers to the collection of human perceptual judgments about synthesized speech quality, naturalness, and speaker similarity. Unlike objective metrics (WER, SPK-SIM, UTMOS), subjective evaluation directly measures how listeners perceive the output. The most common forms are:

- **MOS (Mean Opinion Score):** Listeners rate naturalness on a 1–5 scale. Reported as an average.
- **CMOS (Comparative MOS):** Listeners compare a test sample to a reference on a differential scale (e.g., -3 to +3), with the ground truth as anchor. Reduces listener calibration effects.
- **SMOS (Similarity MOS):** Listeners rate how similar a synthesized sample sounds to a given reference speaker, on a 1–5 scale with 0.5 intervals.
- **MUSHRA:** Multiple stimuli with hidden reference and anchor; used for codec and bandwidth extension evaluation.

## Methods and Variants

**CMOS.** Differential scoring relative to ground truth. Ground truth is always 0.00; positive scores indicate better-than-reference quality, negative scores indicate lower quality. Well-suited for comparing multiple systems without absolute calibration. Used in F5-TTS ([[2025.acl-long.313]]) with 20 evaluators per language and 30 rounds per evaluator.

**SMOS.** Speaker similarity MOS. Listeners hear a reference clip and a synthesized clip and rate how similar the speaker sounds (1–5, intervals of 0.5). Complementary to SPK-SIM (automatic cosine similarity). F5-TTS ([[2025.acl-long.313]]) reports SMOS 3.89 (EN) and 3.83 (ZH) for the 32 NFE system.

**MOS.** Absolute naturalness score. Less commonly used in recent zero-shot TTS papers because it is harder to calibrate across evaluator pools; CMOS is preferred for comparisons.

## Major Claims

Claims are generalised propositions aggregated from paper evidence. The full claim registry with supporting paper lists is in `wiki/concepts/_evidence/subjective-evaluation.yaml`.

### Strongly Supported

- Subjective evaluation methodology varies substantially across papers in evaluator count, sample count, and test set, making direct cross-paper CMOS/SMOS comparisons unreliable without shared evaluation infrastructure.
  Supporting: [[2025.acl-long.313]], [[2511.12347]], [[2025.coling-main.352]]

- CMOS is preferred over absolute MOS in recent zero-shot TTS papers because it reduces listener calibration effects and is more robust to evaluator pool variation.
  Supporting: [[2025.acl-long.313]], [[2511.12347]], [[2025.acl-long.598]]

### Emerging

- Preference-based evaluation (side-by-side, ABX, win-rate) is gaining traction as an alternative to MOS for capturing holistic system quality in conversational and style-prompted settings.
  Supporting: [[2025.acl-long.388]], [[2509.00685]]

- Releasing evaluation prompts, generated samples, and model checkpoints is necessary for subjective scores to be reproducible and comparable across papers.
  Supporting: [[2025.acl-long.313]]

- Domain-specific and socially-grounded subjective evaluation (multi-dimensional Likert scales, expert evaluators) is required when evaluating speech beyond standard naturalness and speaker similarity.
  Supporting: [[2025.acl-long.1252]], [[interspeech-2025-0779]]

## Relationship to Other Concepts

### Extends or Builds On
- [[evaluation-metrics]] — subjective evaluation is the ground-truth reference against which automatic objective metrics (UTMOS, DNSMOS, SPK-SIM) are validated and calibrated.

### Competes With
- [[evaluation-metrics]] — automatic metrics (UTMOS, DNSMOS, WER via Whisper) are faster and reproducible but can be gamed by systems that overfit to the predictor and do not always correlate with human judgments on out-of-distribution systems.

### Commonly Paired With
- [[rlhf-speech]] — subjective preference judgments (ABX, win-rate, N-CMOS) are used as the reward signal or evaluation criterion in RLHF-aligned TTS systems.

## Representative Papers

### Foundational
- [[2025.acl-long.313]] — establishes a high-standard subjective evaluation protocol for zero-shot TTS (20 native evaluators per language, 30 rounds, CMOS + SMOS for EN and ZH) and advocates for prompt and checkpoint release.

### Influential
- [[2025.acl-long.388]] — demonstrates preference-based evaluation (Prolific, 53 participants, 522 judgments, win-rate) as an alternative to MOS for holistic SCA quality assessment.
- [[2025.acl-long.598]] — uses N-CMOS, binary reading accuracy, and A/B speaker similarity across four domain types, showing multi-faceted subjective evaluation for preference-aligned TTS.

### Recent Highlights
- [[2025.emnlp-main.180]] — introduces Consistency MOS and Tag Recall as new subjective metrics for style-prompted TTS, revealing that style consistency requires both intrinsic and situational annotation.
- [[interspeech-2025-0762]] — applies psychometric calibration (recursive binary-search sigmoid estimation) and Best-Worst experimental design across 32 listeners, demonstrating rigorous evaluation for accentedness perception.

## Open Questions

- Should the community adopt a shared subjective evaluation benchmark (fixed test set, fixed reference samples, centralized annotation)?
- How well do UTMOS and DNSMOS correlate with human CMOS as systems move beyond LJSpeech/LibriSpeech distribution?
- What is the minimum number of evaluators needed for a statistically reliable CMOS comparison at the 0.1-point level?

## Trend Summary

The TTS/SCA field has not converged on a standard subjective evaluation protocol. Recent high-profile papers (F5-TTS [[2025.acl-long.313]], NaturalSpeech 3, Seed-TTS) use different evaluator counts, sample counts, and test sets, making CMOS/SMOS values incomparable across papers. For SCA evaluation, DiVA ([[2025.acl-long.388]]) uses a side-by-side preference study on Prolific (53 participants, 522 judgments) rather than MOS, finding a 72% win rate against Qwen 2 Audio — a different methodology that captures holistic assistant quality rather than speech naturalness alone. The trend toward releasing evaluation samples and model checkpoints is positive for reproducibility.

## All Papers

| ID | Title | Venue | Year | Key use of this concept |
|----|-------|-------|------|------------------------|
| [[2025.acl-long.313]] | F5-TTS: A Fairytaler that Fakes Fluent and Faithful Speech with Flow Matching | ACL | 2025 | Runs CMOS and SMOS with 20 native evaluators per language (EN + ZH); advocates for evaluator count transparency and sample release; reports CMOS +0.31 / SMOS 3.89 (EN) and CMOS +0.21 / SMOS 3.83 (ZH) for F5-TTS 32 NFE |
| [[2025.acl-long.388]] | DiVA: Distilling an End-to-End Voice Assistant Without Instruction Training Data | ACL | 2025 | Uses Prolific side-by-side preference study (53 participants, 522 judgments) instead of MOS; reports 72% win rate vs. Qwen 2 Audio; demonstrates preference-based evaluation as holistic SCA quality measure |
| [[2025.acl-long.598]] | Advancing Zero-shot TTS Intelligibility across Diverse Domains via Preference Alignment | ACL | 2025 | Uses N-CMOS (naturalness comparative MOS, -2 to +2 scale), reading accuracy (binary intelligibility judgment by 20 participants, 400 total pairs × 5 systems), and A/B testing for speaker similarity; runs subjective evaluation across all four domain types (regular, articulatory, code-switching, cross-lingual) |
| [[2025.acl-long.1252]] | Finding A Voice: Exploring the Potential of African American Dialect and Voice Generation for Chatbots | ACL | 2025 | Uses 15-metric 5-point Likert scale evaluation by 8–12 AAE-speaking university participants; demonstrates multi-dimensional socially-grounded subjective evaluation for spoken chatbot quality beyond standard naturalness/similarity metrics |
| [[2025.coling-main.352]] | DiffStyleTTS: Diffusion-based Hierarchical Prosody Modeling for TTS | workshop | 2025 | Uses MOS listening tests (14 raters, 18 utterances per model) on a private Mandarin dataset; demonstrates that hierarchical prosody diffusion outperforms Grad-TTS, Guided-TTS, and DiffProsody on naturalness (MOS 4.18 vs. 4.08–4.15) |
| [[2509.00685]] | MPO: Multidimensional Preference Optimization for LM-based TTS | arXiv | 2025 | Uses ABX preference test alongside objective metrics for validating multidimensional RL alignment; provides methodological example of combining objective and subjective evaluation for RL-aligned TTS |
| [[2511.12347]] | VoiceCraft-X: Unifying Multilingual Voice-Cloning Speech Synthesis and Editing | EMNLP | 2025 | Uses CMOS as primary naturalness metric (CMOS 0.63 — highest among all compared systems on Seed-TTS test-en); demonstrates preference of CMOS over MOS for capturing naturalness differences in multilingual zero-shot TTS |
| [[interspeech-2025-0779]] | Intelligibility of Text-to-Speech Systems for Mathematical Expressions | Interspeech | 2025 | Two-tier evaluation: L1 MOS+transcription (49 listeners, 600 samples) and MUSHRA-inspired L2 comparison against human expert reference; demonstrates that TTS-ASR cascade metrics do not capture human comprehension of mathematical speech |
| [[interspeech-2025-0406]] | Zero-Shot Mono-to-Binaural Speech Synthesis | Interspeech | 2025 | MOS + MUSHRA for binaural TTS quality; MUSHRA shows no significant preference between ZeroBAS and supervised methods on in-distribution BSD; significant preference for ZeroBAS on out-of-distribution TMB |
| [[2025.emnlp-main.180]] | Scaling Rich Style-Prompted Text-to-Speech Datasets | EMNLP | 2025 | Consistency MOS and Tag Recall as new evaluation metrics for style-prompted TTS; AMT evaluation reveals that style consistency requires both intrinsic (speaker-level) and situational (utterance-level) annotation for reliable training signal |
| [[interspeech-2025-0762]] | Intrasentential English in Swedish TTS: perceived English-accentedness | Interspeech | 2025 | Psychometric calibration via recursive binary-search sigmoid estimation for mapping engineering parameter to perceived accentedness; Best-Worst experimental design for preference evaluation across 32 listeners |
