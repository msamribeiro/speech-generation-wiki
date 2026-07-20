---
id: "interspeech-2025-2573"
title: "SawtArabi: A Benchmark Corpus for Arabic TTS. Standard, Dialectal and Code-Switching"
authors: ["Vasista Sai Lodagala", "Lamya Alkanhal", "Daniel Izham", "Shivam Mehta", "Shammur Absar Chowdhury", "Aqeelah Makki", "Hamdy S. Hussein", "Gustav Eje Henter", "Ahmed Ali"]
organization: null
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: "2025-08-17"
ingested_date: "2026-07-05"
task: ["TTS", "evaluation"]
architecture: ["flow-matching", "GAN"]
conditioning: ["text-conditioned"]
training: ["supervised"]
model_size: "18.2M params (Matcha-TTS baseline)"
codec_used: "none"
datasets_train: ["SawtArabi"]
datasets_eval: ["SawtArabi"]
metrics:
  - name: MOS
    value: 4.11
    system: EGY (modified espeak-ng)
    testset: SawtArabi EGY naturalness
  - name: MOS
    value: 3.15
    system: MSA (modified espeak-ng)
    testset: SawtArabi MSA naturalness
  - name: MOS
    value: 3.78
    system: CS (modified espeak-ng)
    testset: SawtArabi CS naturalness
  - name: MOS
    value: 3.21
    system: MSA vowelized
    testset: SawtArabi MSA vowelization study
code_available: true
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/lodagala25_interspeech.html"
related_concepts: [multilingual-tts, evaluation-metrics, subjective-evaluation, flow-matching, gan-vocoder]
related_papers: ["2301.02111"]
field_significance:
  level: "moderate"
  type: ["dataset-contribution", "evaluation-contribution"]
generation:
  date: "2026-07-05"
  agent: speech-generation-ingest-agent
  model: claude-sonnet-4-6
  commit: "2829243"
---

> [!abstract] Interspeech · 2025 · Conference
> **Lodagala et al.** · [→ Paper](https://www.isca-archive.org/interspeech_2025/lodagala25_interspeech.html) · Demo: ? · Code: ✓
>
> SawtArabi introduces a 4-hour single-speaker Arabic TTS benchmark covering Modern Standard Arabic, Egyptian dialect, English, and Egyptian-English code-switching, accompanied by a modified espeak-ng phonemizer that reduces Arabic mispronunciations and a set of baseline Matcha-TTS systems establishing the first dialectal and code-switching TTS benchmarks for Arabic.

## Problem

Arabic TTS research has historically focused on Modern Standard Arabic (MSA) in high-resource, clean recording conditions, leaving dialectal Arabic and code-switching speech without standardized corpora or evaluation benchmarks. Prior corpora such as ClArTTS cover classical Arabic, and NatiQ covers MSA only; none address the Egyptian dialect or mixed-language speech that characterises everyday conversation in Arabic-speaking urban environments. Additionally, existing tools for Arabic text processing, including the widely used espeak-ng phonemizer, contain systematic errors that degrade TTS quality regardless of the modeling approach chosen.

## Method

SawtArabi is a 4-hour single-speaker corpus recorded at 22 kHz by a native male Egyptian speaker (aged 34), organised into four equal subsets: Egyptian dialect (EGY), Modern Standard Arabic (MSA), English (ENG), and Egyptian-English code-switching (CS). Each subset is split approximately 45 min training, 10 min development, and 5 min test. MSA text derives from the NatiQ dataset; English sentences are drawn from LJSpeech; Egyptian and code-switching sentences were generated with GPT-4. Transcription follows phonetic rather than orthographic conventions for dialectal subsets to ensure text-audio consistency.

A modified version of espeak-ng version 1.52 addresses four systematic issues in Arabic phonemization: (1) the shared phonetic symbol for two phonetically distinct Arabic letters (za and thal); (2) incorrect retention of /t/ for Ta' Marbuta carrying Sukun, which should surface as /h/ in natural speech; (3) erroneous insertion of a glottal stop for the definite article in medial sentence positions; and (4) inconsistent gemination representation, normalised to the doubled-consonant form via post-processing.

Baseline TTS systems pair Matcha-TTS (conditional flow matching, 18.2M parameters) as the acoustic model with HiFi-GAN (v1 configuration) as the vocoder. Separate systems are trained for each dialect and language subset using either the standard or modified espeak-ng, enabling controlled comparisons. Code-switching sentences are handled by splitting at phrase boundaries, phonemizing each monolingual phrase separately, and concatenating the phoneme sequences. Training uses a single A100 (80 GB GPU), batch size 256, for up to 16,399 epochs per system.

## Key Results

The modified phonemizer reduces mispronounced words from 157 (70 unique cases) with the standard espeak-ng to 97 (44 unique cases) across all Arabic subsets (§4.2). In subjective evaluation, the modified phonemizer consistently improves naturalness and pronunciation MOS across Egyptian (naturalness: 3.94 vs. 4.11), MSA (3.03 vs. 3.15), and code-switching (3.76 vs. 3.78) conditions, while objective MCD scores remain nearly unchanged, confirming that the phonemizer improvement is perceptible but not captured by mel-cepstral distance (§4.2, Table 2).

The vowelization study on MSA shows a large impact: MSA naturalness drops from 3.21 (vowelized) to 2.67 (unvowelized), with pronunciation falling from 3.95 to 2.53. Intelligibility also degrades substantially from 4.22 to 3.20 (§4.3, Table 3). The English subset trained on the same speaker achieves naturalness MOS 4.23, demonstrating that the speaker's L2 English performance is captured adequately for cross-dialect analysis.

Code-switching remains the hardest condition: pronunciation MOS with the modified phonemizer is 4.23, compared to 4.57 for Egyptian and 4.27 for MSA (Table 2), reflecting the compounded difficulty of phrase-level language switching and mixed phoneme inventories.

## Novelty Assessment

The primary contribution is the corpus itself. SawtArabi is, to the authors' knowledge, the first dataset to address both dialectal Arabic TTS and Arabic-English code-switching TTS from the same speaker, enabling controlled L2-speaker and cross-dialect analysis. The phonemizer modifications are incremental engineering fixes to an existing tool rather than a new phonemization system; the four identified issues are well-defined and the fixes are rule-based. The TTS baselines apply standard Matcha-TTS and HiFi-GAN without modification. The listening test design (25 samples, 25 raters, 275 ratings per evaluator) is conventional. The main limitation is scale: at 4 hours total (1 hour per condition), SawtArabi is small relative to corpora used to train state-of-the-art TTS systems, which constrains baseline model quality and generalization claims.

## Field Significance

Moderate - SawtArabi fills a concrete gap in publicly available Arabic TTS resources by providing the first dialectal and code-switching benchmark with clean professional recordings and accompanying phonemizer tooling. It provides a standardised evaluation framework that prior Arabic TTS work lacks, enabling fair cross-system comparison for Egyptian dialect and code-switching conditions. The dataset's small scale (4 hours) limits its direct use as a training corpus for large-scale systems, but its value as an evaluation benchmark and as a resource for studying dialectal and L2 TTS characteristics is clear.

## Claims

- **supports:** Accurate grapheme-to-phoneme conversion is a critical prerequisite for TTS quality in morphologically complex languages, with phonemizer errors directly degrading perceived naturalness and pronunciation.
  > *Evidence:* The modified espeak-ng phonemizer reduced mispronounced words by 38% (157 to 97) and improved naturalness MOS by 0.17 points for Egyptian dialect; objective MCD scores were unchanged, confirming the perceptual significance of phonemizer accuracy beyond spectrogram-level metrics. *(§4.2, Table 2)*

- **supports:** Diacritization (vowelization) of Arabic text is essential for intelligible TTS; training without vowels causes consonant clustering and robotic-sounding output.
  > *Evidence:* MSA naturalness MOS dropped from 3.21 (vowelized) to 2.67 (unvowelized) and pronunciation from 3.95 to 2.53; listening revealed that missing vowels caused consonant clustering that reduced intelligibility and produced unnatural speech. *(§4.3, Table 3)*

- **complicates:** Code-switching TTS is more difficult than monolingual TTS even when both use the same acoustic model and vocoder trained on equivalent data quantities.
  > *Evidence:* CS pronunciation MOS (4.23 with modified phonemizer) is lower than Egyptian (4.57) and MSA (4.27) despite identical training duration and model architecture, with the phrase-level language-switching strategy introducing boundary artefacts not present in monolingual conditions. *(§4.2, Table 2)*

- **supports:** Recording the same speaker across multiple dialects and an L2 language enables controlled disentanglement of speaker identity from dialect-specific acoustic characteristics in TTS datasets.
  > *Evidence:* SawtArabi records a native Egyptian speaker in EGY, MSA, English, and CS conditions, allowing comparison of L2-speaking characteristics against native dialect production within a constant speaker identity. *(§2.2)*

## Limitations and Open Questions

> [!warning]
> The corpus is single-speaker (one male voice talent), meaning baseline systems cannot generalise to multi-speaker Arabic TTS and MOS comparisons are specific to this speaker's voice characteristics. Reproducibility of baseline MOS scores by external labs depends on access to similar listener pools proficient in Egyptian Arabic.

The corpus size (4 hours total, 1 hour per condition) is small relative to modern TTS training norms. Matcha-TTS baselines trained on 45 minutes of speech per dialect are unlikely to achieve the quality attainable with larger corpora, which limits the benchmark's utility for measuring state-of-the-art systems. The paper does not include multi-speaker baselines or zero-shot TTS evaluation, leaving open how well the corpus scales to speaker-adaptive or large-model settings. Future work plans to expand to additional Arabic dialects but does not address multi-speaker coverage.

## Wiki Connections

- [[multilingual-tts|Multilingual TTS]] - SawtArabi provides a benchmark covering Arabic (MSA and Egyptian dialect) and English, with code-switching data that directly tests cross-lingual TTS capabilities.
- [[evaluation-metrics|Evaluation Metrics]] - the paper employs MOS (intelligibility, naturalness, pronunciation), MCD, and F0-RMSE to evaluate Arabic TTS baselines, demonstrating that objective metrics like MCD miss phonemizer-related improvements that are perceptible to listeners.
- [[subjective-evaluation|Subjective Evaluation]] - human listening tests with 25 raters proficient in Arabic dialects are used to assess intelligibility, naturalness, and pronunciation across all conditions and dialect-phonemizer combinations.
- [[flow-matching|Flow Matching]] - Matcha-TTS (conditional flow matching) is used as the acoustic model baseline to validate the SawtArabi corpus across all four language/dialect conditions.
- [[gan-vocoder|GAN Vocoder]] - HiFi-GAN (v1) is trained separately for each dialect condition and paired with Matcha-TTS to produce the benchmark baseline speech outputs.
- [[2301.02111|VALL-E]] - cited as an example of how English TTS has scaled to 60,000 hours of training data, contrasting with the scarcity of large-scale Arabic TTS resources that SawtArabi begins to address.
