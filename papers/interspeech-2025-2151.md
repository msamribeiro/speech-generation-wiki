---
id: "interspeech-2025-2151"
title: "FaVC: A Validated, Transcribed, Parallel Farsi Speech Dataset for Voice Conversion"
authors: ["Mina Serajian", "Saeed Najafzadeh Rahaghi", "Hadi Veisi", "Saman Haratizadeh"]
organization: "University of Tehran"
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-07-04
task: ["VC", "evaluation"]
architecture: ["GAN"]
conditioning: ["speaker-conditioned", "zero-shot"]
training: ["supervised"]
model_size: "not reported"
codec_used: "none"
datasets_train: ["FaVC", "VCC2018", "VCTK"]
datasets_eval: ["FaVC", "VCC2018", "VCTK"]
metrics:
  - name: MOS
    value: 3.94
    system: "Modified StyleTTS-VC"
    testset: FaVC
  - name: SMOS
    value: 3.87
    system: "Modified StyleTTS-VC"
    testset: FaVC
code_available: null
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/serajian25_interspeech.html"
related_concepts: ["voice-conversion", "evaluation-metrics", "subjective-evaluation", "gan-vocoder"]
related_papers: []
field_significance:
  level: "moderate"
  type: ["dataset-contribution"]
generation:
  date: 2026-07-04
  agent: speech-generation-ingest-agent
  model: claude-sonnet-4-6
  commit: "318f1b4"
---

> [!abstract] Interspeech · 2025 · Conference
> **Serajian et al.** (University of Tehran) · [→ Paper](https://www.isca-archive.org/interspeech_2025/serajian25_interspeech.html) · Demo: ? · Code: ?
>
> Introduces FaVC, the first validated, parallel, phonemically balanced Farsi speech corpus designed for voice conversion, and establishes baseline results with five existing GAN-based and TTS-derived VC systems.

## Problem

Voice conversion research in Farsi has been constrained by the complete absence of a parallel, phonemically balanced, transcribed dataset. Existing Farsi speech corpora (DeepMine, Farsi ESD, FarsDAT) are non-parallel and serve ASR, TTS, or emotion recognition tasks, making objective VC evaluation metrics such as MCD, F0-RMSE, FFE, and GPE unfeasible because they require ground-truth utterances matching the source content. The gap is particularly significant given that Farsi is spoken by over 100 million people worldwide, yet no public parallel resource existed for converting or benchmarking speech in this language.

## Method

FaVC consists of 405 paired utterances spoken by 11 native Farsi speakers (5 women, 6 men) covering all Farsi phonemes in all possible phonetic combinations. Sentences were drawn from the FarsDAT corpus, which guarantees phonemic balance. Speakers were recorded in home environments using a medium-quality microphone at 22.05 kHz with SNR of at least 36 dB, producing 4,455 audio files ranging from 1 to 8 seconds. A metadata file accompanies the corpus with Farsi text transcriptions, English translations, and IPA phoneme annotations per sentence.

To validate the corpus, five VC systems are trained and evaluated. CycleGAN-VC handles one-to-one conversion; StarGAN-VC, StarGAN-VC2, and StarGANv2-VC cover many-to-many scenarios. A modified version of StyleTTS-VC is also proposed, which places FiLM and AdaIN conditioning layers in the TTS decoder and extends the standard cycle-consistency loss with a second reconstruction term to improve data augmentation during the VC training phase, enabling one-shot conversion from any source to any target speaker using a brief reference clip. HiFi-GAN, fine-tuned on five randomly selected FaVC speakers, serves as the vocoder for all mel-spectrogram-outputting models.

Training uses eight speakers with 350 sentences each; 55 sentences form the validation set and the remaining speakers constitute the test set. Each model is trained twice under identical configuration: once on an English benchmark (VCC2018 or VCTK) and once on FaVC, allowing direct cross-language comparison.

## Key Results

All five models trained on FaVC achieve naturalness (MOS-N) and speaker similarity (MOS-P) scores comparable to those obtained when the same models are trained on established English benchmarks (VCC2018, VCTK), validating the corpus for VC research (§4.3, Table 2).

The modified StyleTTS-VC reaches MOS-N 3.94 and MOS-P 3.87 on FaVC (MCD 5.58), competitive with its VCTK performance (MOS-N 3.89, MOS-P 3.66). StarGANv2-VC achieves MOS-P 4.1 and MOS-N 3.8 on FaVC (MCD 6.05), marginally exceeding its VCC2018 MOS-P of 3.8. Older architectures fare worse on speaker similarity: StarGAN-VC and StarGAN-VC2 obtain MOS-P of 2.9 and 2.6 on FaVC, reflecting a known limitation of those models rather than a dataset artefact. Numerical comparisons across English and Farsi conditions involve different speaker populations and recording environments, so the cross-language comparisons should be read as rough parity rather than precise equivalence.

## Novelty Assessment

The contribution is a dataset creation and validation exercise. No new VC architecture or training methodology is introduced at the core; all five baseline models are pre-existing systems applied with standard configurations. The modified StyleTTS-VC (FiLM+AdaIN in the decoder plus an extended cycle-consistency loss) represents a small engineering adjustment rather than a methodological advance. The paper's value lies in filling a documented resource gap: providing the first parallel, balanced, phonemically transcribed Farsi corpus suited for objective VC evaluation. The validation strategy of re-training each model on both an English dataset and FaVC under identical settings is methodologically sensible given that published results for these models often use different evaluation metrics or test splits.

## Field Significance

Moderate — FaVC addresses a concrete resource gap for VC research in Farsi, a major language that lacked any parallel speech corpus for conversion tasks. The dataset enables objective VC evaluation metrics in Farsi for the first time and provides a foundation for future cross-lingual and low-resource VC work. The paper does not advance VC methodology but provides necessary infrastructure for the field to do so in an underserved language.

## Claims

- **supports:** Objective voice conversion evaluation metrics that require ground-truth utterances (MCD, F0-RMSE, FFE, GPE) are only feasible when a parallel corpus exists, making dataset parallelism a prerequisite for rigorous VC benchmarking.
  > *Evidence:* The authors identify the absence of parallel Farsi data as the direct reason these objective metrics cannot be applied to existing Farsi corpora; FaVC is designed specifically to make them feasible. *(§1.3)*

- **supports:** GAN-based voice conversion systems trained on parallel corpora in low-resource languages can reach naturalness and speaker similarity scores comparable to training on established English benchmarks.
  > *Evidence:* CycleGAN-VC, StarGAN-VC, StarGAN-VC2, and StarGANv2-VC trained on FaVC achieve MOS-N scores of 3.3–3.94 and MOS-P of 2.5–4.1, within the range of the same models trained on VCC2018 under identical configuration. *(§4.3, Table 2)*

- **complicates:** Non-English voice conversion research is limited by the near-absence of parallel, phonemically balanced corpora outside English.
  > *Evidence:* Despite Farsi being spoken by over 100 million people, no publicly available parallel Farsi VC dataset existed prior to FaVC; existing Farsi datasets (DeepMine, Farsi ESD, FarsDAT) are non-parallel and target ASR, TTS, or emotion recognition. *(§1.2, §2.2)*

- **supports:** Incorporating FiLM and AdaIN conditioning into a TTS decoder supports effective one-shot voice conversion with results competitive with many-to-many GAN baselines.
  > *Evidence:* Modified StyleTTS-VC with FiLM+AdaIN achieves MOS-N 3.94 and MOS-P 3.87 on FaVC (MCD 5.58), the highest naturalness score among the five evaluated models and competitive with StarGANv2-VC on speaker similarity (MOS-P 4.1). *(§4.1, §4.3, Table 2)*

## Limitations and Open Questions

The dataset is small (11 speakers, 405 utterances each), which limits the training capacity for data-hungry approaches. Evaluations across FaVC and English datasets conflate language, speaker population, and recording condition differences; the cross-language parity finding should be treated as indicative rather than controlled. No statistical significance testing is reported for MOS comparisons. Cross-lingual VC scenarios (Farsi to English or vice versa) are not evaluated, despite being flagged as a future direction. The dataset is currently distributed upon request rather than through an open public repository, which constrains reproducibility.

## Wiki Connections

- [[voice-conversion|Voice Conversion]] — FaVC is the first parallel Farsi corpus designed specifically for VC, enabling both parallel and non-parallel voice conversion research in the language.
- [[evaluation-metrics|Evaluation Metrics]] — the paper uses MCD, MOS-N, and MOS-P to validate FaVC quality, noting that MCD and pitch-based metrics are only computable with parallel ground-truth utterances.
- [[subjective-evaluation|Subjective Evaluation]] — MOS-N (naturalness) and MOS-P (speaker similarity) are collected from 20 native Farsi speakers rating on a 1–5 scale.
- [[gan-vocoder|GAN Vocoder]] — HiFi-GAN is fine-tuned on FaVC speakers for vocalization, and all primary VC models evaluated are GAN-based (CycleGAN-VC, StarGAN-VC family).
