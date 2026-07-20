---
id: "interspeech-2025-gourav25_interspeech"
title: "Code Mix TTS: An Approach to Infer Human Like Speech for Multi-Lingual Input Texts"
authors: ["Vishal Gourav", "Phanindra Mankale"]
organization: "Oracle Corporation"
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: "2025-08-17"
ingested_date: "2026-07-13"
task: ["TTS"]
architecture: ["diffusion", "GAN"]
conditioning: ["text-conditioned", "multilingual", "speaker-conditioned"]
training: ["supervised"]
model_size: "not reported"
codec_used: "none"
datasets_train: ["LibriTTS-R", "CML-TTS", "custom Hindi dataset"]
datasets_eval: ["custom code-mix dataset (250 utterances)"]
metrics:
  - name: MOS
    value: 4.12
    system: "Code-Mix StyleTTS2 (A-MOS via NISQA)"
    testset: "custom code-mix (250 utterances)"
code_available: null
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/gourav25_interspeech.html"
related_concepts: [multilingual-tts, diffusion-tts, evaluation-metrics]
related_papers: []
field_significance:
  level: low
  type: ["engineering-integration"]
generation:
  date: "2026-07-13"
  agent: speech-generation-ingest-agent
  model: claude-sonnet-4-6
  commit: "b035ebb"
---

> [!abstract] Interspeech · 2025 · Conference
> **Gourav and Mankale** (Oracle Corporation) · [→ Paper](https://www.isca-archive.org/interspeech_2025/gourav25_interspeech.html) · Demo: ? · Code: ?
>
> A code-mixed TTS system that modifies the G2P text preprocessing layer of a multilingual StyleTTS2 model to phonemize language-specific text chunks separately, enabling mixed-language speech synthesis over English, Spanish, Portuguese, and Hindi without retraining the acoustic model.

## Problem

TTS systems are conventionally designed to accept single-language input, but natural human speech in multilingual societies routinely mixes languages within a single utterance. Providing code-mixed text to a monolingual TTS pipeline causes misphonemization and unnatural output because the G2P module is calibrated for one language's phoneme inventory. Prior work on multilingual TTS handles multiple languages by training separate models or selecting a language mode at inference time; none directly addresses inference over mixed-language input without additional fine-tuning.

## Method

The approach builds on StyleTTS2, a style-diffusion-based TTS model that generates mel-spectrograms conditioned on style vectors encoding speaker and prosodic characteristics, with an iSTFTNet vocoder for waveform synthesis. The authors first train a multilingual StyleTTS2 instance on approximately 1,000 hours of audio across four languages: English (LibriTTS-R), Spanish and Portuguese (CML-TTS), and Hindi (internal dataset), totaling 370k utterances standardized to 24 kHz. Training runs in two stages (50 epochs then 20 epochs) on A100 GPUs over roughly 3.5 weeks.

The core contribution is an inference-time modification to the G2P layer, implemented as a script that wraps the existing espeak-ng phonemizer. Given a mixed-language input string (at most two languages, one of which must be English), the script detects language boundaries, splits the text into monolingual chunks, phonemizes each chunk with the corresponding espeak-ng language model, concatenates the resulting phoneme sequences, and passes the unified token sequence to the unchanged acoustic model and vocoder. No new training data, additional model parameters, or fine-tuning steps are required beyond the initial multilingual training.

## Key Results

On a custom 250-utterance code-mix evaluation set, the proposed system achieves an automated MOS (A-MOS via NISQA) of 4.12 ± 0.09, matching the monolingual StyleTTS2 baseline trained on LJSpeech at 4.12 ± 0.08. Both outperform VITS (3.69 ± 0.12) and YourTTS (2.35 ± 0.07) by substantial margins. Average inference time on CPU is 2.88 ± 0.43 seconds, comparable to StyleTTS2 (2.93 ± 0.22 sec) and slightly faster than YourTTS (3.25 sec). No statistically significant drop in automated quality is observed relative to the monolingual system, suggesting the G2P modification does not degrade acoustic model performance on the metrics used.

## Novelty Assessment

The contribution is narrow engineering integration: the acoustic model (StyleTTS2) is unchanged and multilingual TTS training itself is not novel. The specific innovation is the language-chunking wrapper at the G2P stage, which is a lightweight inference-time script rather than a learned component. The "without additional training" framing applies only to the G2P wrapper; the model does require a full multilingual training run. Evaluation is limited to automated MOS on a proprietary 250-utterance test set with CPU inference, and no human perceptual study is conducted. Results are not compared against dedicated code-switching TTS systems from prior literature.

## Field Significance

Low — This paper describes a pragmatic inference-time text preprocessing fix for code-mixed TTS, requiring no architectural change to the underlying synthesis model. It provides a working proof of concept that language-chunk phonemization enables mixed-language output from a multilingual model, but the evaluation is narrow (automated MOS only, 250 utterances, one custom dataset, no human raters) and the G2P chunking idea is straightforward. The contribution is primarily confirmatory evidence that standard multilingual TTS training plus language-aware G2P routing can produce acceptable code-mixed output on automated metrics.

## Claims

- **supports:** Inference-time G2P modification can enable code-mixed speech synthesis from a multilingual TTS model without retraining or fine-tuning the acoustic model.
  > *Evidence:* The authors add a language-detection and chunking layer at the espeak-ng phonemizer; the StyleTTS2 acoustic model and iSTFTNet vocoder are unchanged, and the code-mixed system achieves A-MOS 4.12 matching the monolingual baseline. *(§2)*

- **supports:** Multilingual TTS models trained on diverse language data can maintain automated naturalness scores comparable to single-language models when handling code-mixed input.
  > *Evidence:* Code-Mix StyleTTS2 achieves A-MOS 4.12 ± 0.09 on code-mixed English-Hindi/Spanish/Portuguese inputs, matching monolingual StyleTTS2 at 4.12 ± 0.08 and outperforming VITS (3.69) and YourTTS (2.35). *(§3, Table 1)*

- **complicates:** Code-mixed TTS evaluation lacks standardized benchmarks and relies primarily on automated quality metrics rather than human perceptual studies, limiting comparability across systems.
  > *Evidence:* The evaluation uses only NISQA A-MOS on a custom 250-utterance dataset with no human listener panel; no comparison to dedicated code-switching TTS baselines from prior work is provided. *(§3)*

- **complicates:** Practical code-mixed TTS systems are constrained to language pairs involving a dominant pivot language, limiting generalization to non-English language combinations.
  > *Evidence:* The proposed chunking and phonemization pipeline requires English as one of the two input languages; the paper identifies extension to non-English pairs and romanized mixed scripts (Hinglish, Portunish) as explicit future work. *(§4)*

## Limitations and Open Questions

> [!warning]
> Evaluation uses automated MOS (NISQA) on a custom 250-utterance proprietary test set with CPU inference only. No human listening panel was conducted, and no comparison to existing code-switching TTS systems is made. Results cannot be directly compared to benchmarks in the multilingual TTS literature.

The language-chunking approach is limited to at most two simultaneous languages with English as a mandatory component, ruling out all-native-language pairs. The system does not handle romanized code-mixing (e.g., Hindi written in Latin script) or text that mixes scripts without explicit language boundaries. The extent to which inference time degrades at GPU throughput versus CPU is not reported, and the model's code and weights are not released, making independent replication difficult.

## Wiki Connections

- [[multilingual-tts|Multilingual TTS]] — this paper trains a multilingual StyleTTS2 model and extends it to code-mixed inference via language-aware G2P routing across English, Spanish, Portuguese, and Hindi.
- [[diffusion-tts|Diffusion TTS]] — the backbone acoustic model (StyleTTS2) uses a style diffusion component for style vector generation, which this paper inherits without modification.
- [[evaluation-metrics|Evaluation Metrics]] — the paper relies exclusively on NISQA automated MOS (A-MOS) for quality assessment, illustrating both the utility and the limitations of automated metrics as a substitute for human listening panels in multilingual evaluation.
