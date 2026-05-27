---
slug: subjective-evaluation
title: Subjective Evaluation
aliases: [listening tests, MOS studies, perceptual evaluation, crowdsourced evaluation, human evaluation]
related_concepts: [evaluation-metrics, rlhf-speech]
last_updated: 2026-05-27
---

# Subjective Evaluation

## What it is

Subjective evaluation in TTS refers to the collection of human perceptual judgments about synthesized speech quality, naturalness, and speaker similarity. Unlike objective metrics (WER, SPK-SIM, UTMOS), subjective evaluation directly measures how listeners perceive the output. The most common forms are:

- **MOS (Mean Opinion Score):** Listeners rate naturalness on a 1–5 scale. Reported as an average.
- **CMOS (Comparative MOS):** Listeners compare a test sample to a reference on a differential scale (e.g., -3 to +3), with the ground truth as anchor. Reduces listener calibration effects.
- **SMOS (Similarity MOS):** Listeners rate how similar a synthesized sample sounds to a given reference speaker, on a 1–5 scale with 0.5 intervals.
- **MUSHRA:** Multiple stimuli with hidden reference and anchor; used for codec and bandwidth extension evaluation.

## Why it matters

Objective metrics are proxies for human perception but do not always correlate well with it. UTMOS and DNSMOS are trained to predict MOS but can fail on out-of-distribution systems. WER measures intelligibility but not prosody or naturalness. Subjective evaluation remains the gold standard for assessing naturalness and speaker identity. It is required for publication at most speech and NLP venues.

## Current state of the art

Standards for subjective evaluation in TTS are not yet uniform, making cross-paper comparison difficult. Key methodological issues include:

- **Number of evaluators:** DiTTo-TTS used 6 and 12 raters for SMOS and CMOS; NaturalSpeech 3 used 12 natives evaluating 20/10 samples. F5-TTS ([[2025.acl-long.313]]) used 20 native evaluators for both English and Mandarin, rating 30 rounds of randomly selected utterances — a more comprehensive setup.
- **Prompt set release:** Without releasing prompts and generated samples, CMOS/SMOS scores cannot be reproduced or directly compared. F5-TTS advocates for prompt release and open-sourcing model checkpoints for reproducibility.
- **Cross-lingual evaluation:** Evaluating Mandarin speech requires Mandarin-native listeners. F5-TTS ran separate evaluations for EN and ZH with native evaluators for each.

## Key variants and sub-approaches

**CMOS.** Differential scoring relative to ground truth. Ground truth is always 0.00; positive scores indicate better-than-reference quality, negative scores indicate lower quality. Well-suited for comparing multiple systems without absolute calibration. Used in F5-TTS ([[2025.acl-long.313]]) with 20 evaluators per language and 30 rounds per evaluator.

**SMOS.** Speaker similarity MOS. Listeners hear a reference clip and a synthesized clip and rate how similar the speaker sounds (1–5, intervals of 0.5). Complementary to SPK-SIM (automatic cosine similarity). F5-TTS ([[2025.acl-long.313]]) reports SMOS 3.89 (EN) and 3.83 (ZH) for the 32 NFE system.

**MOS.** Absolute naturalness score. Less commonly used in recent zero-shot TTS papers because it is harder to calibrate across evaluator pools; CMOS is preferred for comparisons.

## Comparison to alternatives

Automatic metrics (UTMOS, DNSMOS) are faster and reproducible but can be gamed by systems that overfit to the predictor. WER via Whisper or Paraformer is a strong intelligibility proxy but does not capture naturalness. SPK-SIM via WavLM captures speaker identity but not prosodic naturalness. Subjective evaluation captures holistic quality but is slow, costly, and difficult to reproduce without sample release.

## Year-on-year trajectory

The TTS/SCA field has not converged on a standard subjective evaluation protocol. Recent high-profile papers (F5-TTS [[2025.acl-long.313]], NaturalSpeech 3, Seed-TTS) use different evaluator counts, sample counts, and test sets, making CMOS/SMOS values incomparable across papers. For SCA evaluation, DiVA ([[2025.acl-long.388]]) uses a side-by-side preference study on Prolific (53 participants, 522 judgments) rather than MOS, finding a 72% win rate against Qwen 2 Audio — a different methodology that captures holistic assistant quality rather than speech naturalness alone. The trend toward releasing evaluation samples and model checkpoints is positive for reproducibility.

## Open questions

- Should the community adopt a shared subjective evaluation benchmark (fixed test set, fixed reference samples, centralized annotation)?
- How well do UTMOS and DNSMOS correlate with human CMOS as systems move beyond LJSpeech/LibriSpeech distribution?
- What is the minimum number of evaluators needed for a statistically reliable CMOS comparison at the 0.1-point level?

## Papers

| ID | Title | Venue | Year | Key use of this concept |
|----|-------|-------|------|------------------------|
| [[2025.acl-long.313]] | F5-TTS: A Fairytaler that Fakes Fluent and Faithful Speech with Flow Matching | ACL | 2025 | Runs CMOS and SMOS with 20 native evaluators per language (EN + ZH); advocates for evaluator count transparency and sample release; reports CMOS +0.31 / SMOS 3.89 (EN) and CMOS +0.21 / SMOS 3.83 (ZH) for F5-TTS 32 NFE |
| [[2025.acl-long.388]] | DiVA: Distilling an End-to-End Voice Assistant Without Instruction Training Data | ACL | 2025 | Uses Prolific side-by-side preference study (53 participants, 522 judgments) instead of MOS; reports 72% win rate vs. Qwen 2 Audio; demonstrates preference-based evaluation as holistic SCA quality measure |
| [[2025.acl-long.598]] | Advancing Zero-shot TTS Intelligibility across Diverse Domains via Preference Alignment | ACL | 2025 | Uses N-CMOS (naturalness comparative MOS, -2 to +2 scale), reading accuracy (binary intelligibility judgment by 20 participants, 400 total pairs × 5 systems), and A/B testing for speaker similarity; runs subjective evaluation across all four domain types (regular, articulatory, code-switching, cross-lingual) |
| [[2025.acl-long.1252]] | Finding A Voice: Exploring the Potential of African American Dialect and Voice Generation for Chatbots | ACL | 2025 | Uses 15-metric 5-point Likert scale evaluation by 8–12 AAE-speaking university participants; demonstrates multi-dimensional socially-grounded subjective evaluation for spoken chatbot quality beyond standard naturalness/similarity metrics |
