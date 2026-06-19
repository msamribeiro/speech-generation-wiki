---
id: "interspeech-2025-0779"
title: "Intelligibility of Text-to-Speech Systems for Mathematical Expressions"
authors: ["Sujoy Roychowdhury", "Ranjani H.G.", "Sumit Soman", "Nishtha Paul", "Subhadip Bandyopadhyay", "Siddhanth Iyengar"]
organization: Ericsson R&D
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-05-30
task: [TTS, evaluation]
architecture: []
conditioning: [text-conditioned]
training: []
model_size: "not reported"
codec_used: "not reported"
datasets_train: []
datasets_eval: ["HME (Handwritten Mathematical Equations Kaggle dataset), 120 MX across 8 categories"]
metrics:
  - name: MOS
    value: 4.9
    system: Amazon Polly (Numerics category)
    testset: HME (L1 listening test)
  - name: MOS
    value: 2.3
    system: Seamless (Summation category)
    testset: HME (L1 listening test)
  - name: CER
    value: 0.10
    system: TTS-ASR cascade (average across models)
    testset: HME MXText
code_available: null
demo_available: null
url: https://www.isca-archive.org/interspeech_2025/roychowdhury25_interspeech.html
related_concepts: [evaluation-metrics, subjective-evaluation]
related_papers: []
---
> [!abstract] Interspeech · 2025 · Conference
> **Sujoy Roychowdhury et al.** (Ericsson R&D) · [→ Paper](https://www.isca-archive.org/interspeech_2025/roychowdhury25_interspeech.html) · Demo: ? · Code: ?
>
> A listener study evaluating five commercial TTS systems on 120 mathematical expressions across 8 categories reveals that TTS intelligibility for mathematical content varies strongly by category and model, and falls significantly below expert human rendition for most categories — establishing a benchmark for future math-aware TTS development.

## Problem

Existing TTS evaluation focuses on natural speech intelligibility (WER, MOS) that does not capture whether a listener can correctly transcribe a mathematical expression from audio. Mathematical expressions (MX) present special challenges: prosody indicating grouping, operator precedence, and spatial layout (fractions, summations, matrices) is absent from naive TTS. Prior work on TTS for MX assumed TTS systems were error-free; this paper is the first to systematically evaluate the intelligibility of commercial TTS systems for MX via listening experiments.

## Method

**Dataset**: 3,141 MX from the HME (Handwritten Mathematical Equations) Kaggle dataset, manually categorized into 8 types: Advanced Algebraic, Basic Algebraic, Fractions, Calculus, Matrices, Numerics, Square Roots and Trig, Summation. 120 MX sampled across these categories for listening tests.

**LLM pronunciation step**: Since TTS models cannot process LaTeX directly, two LLMs (QWEN2.5-7b open-source; GPT-4 for remainder) generate English pronunciation text (MXText) from LaTeX MX. 87.5% of generated pronunciations are manually verified as correct (85.9% QWEN, 92.9% GPT-4).

**TTS models evaluated**: Amazon Polly (AP), Microsoft Azure AI Speech (AZ), Google TTS AI API (GA), OpenAI TTS (OA), Seamless/Meta (SL, open-source).

**L1 Listening Test (intelligibility)**: 49 listeners rate 600 AudioMX (5 models × 120 MX) for intelligibility on a 1–5 Likert scale (MOS) and transcribe each MX in LaTeX. Each AudioMX rated by 3 listeners (1,800 total ratings). Metrics: MOS, Count-of-Correct (CoC, expert-verified), LaTeX Character Error Rate (LCER), TeXBLEU.

**L2 Listening Test (MUSHRA-inspired comparison)**: 13 listeners compare 5 TTS AudioMX against a hidden reference (RAudioMX) recorded by 4 human experts, rating clarity and intelligibility on [0,100] for 35 MX across 5 categories.

## Key Results

L1 (intelligibility by category):
- **Numerics**: All models near ceiling (MOS 4.7–4.9; CoC ~97–100%)
- **Fractions**: Good performance (MOS 3.9–4.3; CoC 48–67%)
- **Summation**: Poorest overall (MOS 2.3–3.5; CoC 40–59%; Seamless LCER=0.4)
- **Matrices**: Consistently difficult (MOS 3.1–3.6; CoC 17–29%)
- No single TTS model dominates across all categories

ANOVA on LCER: both TTS model (p<0.05) and MX category (p<0.05) are statistically significant; LLM choice is insignificant when pronunciation is correct.

L2 (gap vs. human expert): For all categories except Numerics and Fractions, TTS models show large gaps vs. reference (difference scores of 24–73 points on [0,100] scale). Human expert CoC is 97% vs. TTS average well below that.

TTS-ASR cascade metrics (CER=0.10, BLEU=0.66, ROUGE=0.84) correlate very weakly (<0.15 absolute) with human intelligibility metrics, confirming that ASR-based evaluation is insufficient for MX intelligibility.

## Novelty Assessment

The primary contribution is empirical: this is the first listener study of modern commercial TTS models on mathematical expressions with transcription-based intelligibility measurement. The introduction of LaTeX-aware metrics (LCER, TeXBLEU) alongside MOS is a practical methodological advance. The finding that ASR-based metrics are uncorrelated with human intelligibility for MX is an important negative result for the field. The work is narrowly evaluative — it does not propose a new TTS system or training method — but the benchmark and findings are directly actionable for future math-aware TTS development.

## Limitations and Open Questions

- Only English MX evaluated; mathematical verbalization conventions differ across languages.
- 49 listeners for L1 is a modest panel; each AudioMX has only 3 ratings.
- The LLM-generated pronunciation step introduces errors (12.5% incorrect); a cleaner math-to-speech conversion module would be needed for deployment.
- Expert reference recordings are from 4 individuals; inter-expert variation is not analyzed.
- Fine-tuning TTS for MX is left as future work.

## Wiki Connections

This paper contributes directly to [[evaluation-metrics]] by introducing transcription-based intelligibility metrics (CoC, LCER, TeXBLEU) for a domain where standard WER is insufficient, and demonstrating that TTS-ASR cascade metrics fail to capture human comprehension of mathematical speech. It also informs [[subjective-evaluation]] through the MUSHRA-inspired L2 design with a hidden human reference. The paper is relevant to any work on domain-specific or specialized-content TTS, including prosody modeling for structured content.
