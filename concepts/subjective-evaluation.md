---
slug: subjective-evaluation
title: Subjective Evaluation
aliases: [listening tests, MOS studies, perceptual evaluation, crowdsourced evaluation, human evaluation]
related_concepts: [evaluation-metrics, rlhf-speech]
last_updated: 2026-06-12
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

- Multi-reference normalisation in LLM-as-judge evaluation (scoring candidates relative to multiple TTS-synthesised references) achieves high rank correlation with human expert judgements, making automated speech role-playing benchmarking feasible at scale.
  Supporting: [[2508.02013]]

- Small-scale MOS panels (10–30 listeners, 30 samples per condition) are pervasive in low-resource and domain-specific TTS evaluation, but lack the statistical power to support fine-grained system comparisons.
  Supporting: [[2508.08715]], [[2508.06870]]

- Human Fooling Rate (HFR) — the fraction of trials where listeners misclassify synthetic speech as human — detects performance gaps that preference-based metrics (CMOS, MUSHRA) conceal, because reference matching inflates preference scores against low-expressivity references.
  Supporting: [[interspeech-2025-2765]]

- Open-source TTS systems remain 20+ percentage points behind commercial systems on HFR for expressive conversational speech, contradicting claims of human parity from CMOS/MUSHRA-based evaluations.
  Supporting: [[interspeech-2025-2765]]

- Shareable evaluation recipes — self-contained bundles of procedure, anonymised data, and configuration — lower the practical barrier to replication without requiring infrastructure replication, making perceptual evaluation more reproducible at the protocol level.
  Supporting: [[interspeech-2025-0401]]

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
- For emotion-conditioned TTS, [[2504.12867]] shows sentence-level emotion similarity correlates only ~40% with human MOS; what evaluation paradigm captures utterance-level emotional appropriateness reliably?
- SpeechRole [[2508.02013]] uses TTS-synthesised references rather than human recordings as benchmarks; does this create a quality ceiling that masks genuine capability differences?
- The VoiceMOS/AudioMOS Challenge survey [[2508.00317]] confirms that current SQA models cannot discriminate among near-human-quality systems; should the community define a dedicated benchmark targeting this regime specifically?

## Trend Summary

The TTS/SCA field has not converged on a standard subjective evaluation protocol. Recent high-profile papers (F5-TTS [[2025.acl-long.313]], NaturalSpeech 3, Seed-TTS) use different evaluator counts, sample counts, and test sets, making CMOS/SMOS values incomparable across papers. For SCA evaluation, DiVA ([[2025.acl-long.388]]) uses a side-by-side preference study on Prolific (53 participants, 522 judgments) rather than MOS, finding a 72% win rate against Qwen 2 Audio — a different methodology that captures holistic assistant quality rather than speech naturalness alone. The trend toward releasing evaluation samples and model checkpoints is positive for reproducibility. 2025: New multi-dimensional evaluation frameworks are emerging for richer subjective assessment: SpeechRole [[2508.02013]] introduces a nine-dimension role-playing evaluation with LLM-as-judge; Maestro-EVC [[2508.06890]] uses four-dimension MOS (naturalness, emotion similarity, speaker similarity, prosody similarity) for VC evaluation. The VoiceMOS/AudioMOS Challenge series [[2508.00317]] provides a four-year view showing that: (1) community infrastructure accelerates progress more than individual modelling advances, and (2) near-human-quality systems defeat all current SQA predictors — a ceiling effect that will require new benchmark design. Small-scale evaluations (30 samples, 10 listeners) remain prevalent in low-resource settings [[2508.08715]], [[2508.06870]], and represent a known weakness in statistical reliability that the field has not systematically addressed. Integration pass 6 adds two important contributions to this picture: [[interspeech-2025-2765]] provides the largest English TTS subjective study in this corpus (135 participants, 30,000+ ratings) and demonstrates via HFR that CMOS scores from standard benchmarks overstate naturalness — specifically because low-expressivity references inflate apparent quality. [[interspeech-2025-0401]] pairs this empirical critique with an infrastructure response: the replikant platform and evaluation recipe schema that operationalise what the field has described as best practices since Wester et al. (2015) but not yet adopted at scale. These two papers together make the case that the field needs both better metrics (HFR, domain-specific evaluation) and better infrastructure (shareable recipes, reviewer checklists) to close the gap between reported results and real-world quality.

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
| [[2503.04721]] | Full-Duplex-Bench | arXiv | 2025 | Automated turn-taking evaluation without human raters; argues that descriptive, reproducible metrics can substitute for subjective judgement in dialogue system benchmarking |
| [[2504.12867]] | EmoVoice | arXiv | 2025 | 30-rater MOS for emotional expressiveness; sentence-level emotion similarity correlates ~40% with human MOS, demonstrating limits of automatic proxies for fine-grained emotion assessment |
| [[2507.20091]] | ProsodyLM | arXiv | 2025 | MOS on 36 audiobook excerpts with separate prosody quality and naturalness dimensions; controlled comparison of codec-token vs. prosody-token LM architectures |
| [[2508.00317]] | Advancing Speech Quality Assessment through Challenges | arXiv | 2025 | Retrospective on VoiceMOS/AudioMOS Challenge series; near-human-quality systems defeat current SQA predictors; community infrastructure drives progress more than individual model contributions |
| [[2508.01796]] | Enhancing Spectrogram Realism in Singing Voice Synthesis | arXiv | 2025 | Combines human visual spectrogram realism judgment with standard audio MOS; dual-axis evaluation captures distinct quality dimensions not jointly addressable by MOS alone |
| [[2508.02013]] | SpeechRole | arXiv | 2025 | Nine-dimension LLM-as-judge evaluation with multi-reference normalisation; Spearman ρ 0.67–0.98 with human experts across English and Chinese |
| [[2508.06870]] | TTS for Meitei Mayek Script | arXiv | 2025 | Native-speaker MOS on 10 samples per condition with no baseline; representative of evaluation constraints in zero-resource script contexts |
| [[2508.07711]] | Is GAN Necessary for Neural Vocoder? | arXiv | 2025 | 100-listener MOS shows no significant difference between GAN and GAN-free vocoder; UTMOS ranking diverges from subjective ranking, adding to evidence of predictor unreliability |
| [[2508.08715]] | MultiGen | arXiv | 2025 | 10-listener 30-sample MOS per language for three low-resource Southeast Asian languages; exemplifies statistical limitations of small-scale evaluation |
| [[interspeech-2025-0762]] | Intrasentential English in Swedish TTS: perceived English-accentedness | Interspeech | 2025 | Psychometric calibration via recursive binary-search sigmoid estimation for mapping engineering parameter to perceived accentedness; Best-Worst experimental design for preference evaluation across 32 listeners |
| [[interspeech-2025-2765]] | The State of TTS: Human Fooling Rates | Interspeech | 2025 | Large-scale HFR study (135 native listeners, 30k+ ratings, 10 TTS systems) on 4 datasets; CMOS/MUSHRA shown to overestimate quality; binary forced-choice more efficient (24s vs. 42s per sample) |
| [[interspeech-2025-0401]] | Enabling Replicability of Speech Synthesis Perceptual Evaluations | Interspeech | 2025 | Proposes shareable recipe schema, replikant platform (Flask+SQLite), and reviewer checklist; distinguishes replicability from reproducibility for subjective evaluations |
| [[interspeech-2025-1066]] | Score-Based Training for Energy-Based TTS | Interspeech | 2025 | 3-listener MOS on 100 samples; UTMOSv2 diverges from human MOS for delta loss EBMs — a concrete instance of automatic predictor failure on non-standard systems |
| [[interspeech-2025-1122]] | BitTTS | Interspeech | 2025 | 15-rater 30-sample MOS for compact TTS evaluation; documents statistical fragility of small-panel evaluations at ±0.11 CI |
| [[interspeech-2025-0739]] | FD-Bench | Interspeech | 2025 | GPT-4o-mini subjective scoring across 6 dimensions as a surrogate for human evaluation of full-duplex dialogue quality |
| [[2508.08957]] | QAMRO | ASRU | 2025 | System-level SRCC as primary subjective alignment metric; ablation shows both adaptive margin and quality-aware weighting improve SRCC over standard regression |
| [[2508.09702]] | M3PDB | arXiv | 2025 | Subjective evaluation absent; all metrics automatic; illustrates the need for subjective validation in prompt selection quality evaluation |
| [[interspeech-2025-0984]] | Benchmarking Neural Speech Codec Intelligibility with SITool | Interspeech | 2025 | SITool crowdsourced DRT/MRT platform; 82 retained Mechanical Turk participants; STOI/WER correlation with DRT scores |
| [[2508.13028]] | Integrating Feedback Loss from Bi-modal Sarcasm Detector for Sarcastic Speech Synthesis | arXiv | 2025 | 13-listener subjective evaluation on sarcasm prosody; exact MOS not reported; preference test showing 53% preference for proposed system |
| [[2508.15931]] | QvTAD | arXiv | 2025 | VCTK-RVA pairwise comparison dataset; human annotation of relative timbre attribute strength as ground truth |
| [[2508.17031]] | RephraseTTS | arXiv | 2025 | 6-listener MOS on 15 LibriTTS utterances for speech insertion quality; MOS 3.93 vs. 4.32 ground truth |
| [[2508.17494]] | Improving French Synthetic Speech Quality via SSML Prosody Control | workshop | 2025 | 18-listener 30-pair AB test on one-minute segments; MOS 3.87 vs. 3.20 baseline; p<0.005 significance |
| [[2508.17623]] | EMO-Reasoning | arXiv | 2025 | 20-annotator human perceptual ERS evaluation across 20 sampled dialogues per system; three-axis subjective protocol |
| [[2509.00675]] | Speaker-Conditioned Phrase Break Prediction | arXiv | 2025 | MOS evaluation on LibriTTS-R with VITS and Matcha-TTS backbones; statistical significance testing between PLM variants |
| [[2509.03292]] | Improving Perceptual Audio Aesthetic Assessment via Triplet Loss | arXiv | 2025 | Human-annotated AudioMOS Challenge 2025 scores as ground truth for multi-axis aesthetic prediction |
| [[2509.03940]] | VoxRole | arXiv | 2025 | 20-annotator human evaluation of 20 sampled dialogues; Pearson correlation 0.762 between LLM judge and human scores; validates acoustically-aware LLM evaluation paradigm |
| [[2509.04072]] | Computational Narrative Understanding for Expressive TTS | arXiv | 2025 | 30-sample CMOS study (2 per speaker), win-rate pairwise, and ContextMOS via Gemini-2.5-Pro for LibriQuote expressivity benchmark |
| [[2410.17196]] | VoiceBench: Benchmarking LLM-Based Voice Assistants | arXiv | 2024 | Automated evaluation framework for voice assistants using LLM-based scoring; demonstrates how multi-domain capability benchmarking complements naturalness-only subjective evaluation |
