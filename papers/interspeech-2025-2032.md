---
id: "interspeech-2025-2032"
title: "ExagTTS: An Approach Towards Controllable Word Stress Incorporated TTS for Exaggerated Synthesized Speech Aiding Second Language Learners"
authors: ["Anindita Mondal", "Monica Surtani", "Anil Kumar Vuppala", "Parameswari Krishnamurthy", "Chiranjeevi Yarra"]
organization: "IIIT Hyderabad"
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-07-04
task: ["TTS"]
architecture: ["hybrid"]
conditioning: ["text-conditioned", "prosody-conditioned"]
training: ["supervised"]
model_size: "not reported"
codec_used: "none"
datasets_train: ["Tatoeba (900-sample annotated subset)"]
datasets_eval: ["Tatoeba (100-sample validation set)"]
metrics:
  - name: MOS
    value: 4.6
    system: Matcha+Duration (cascaded, exaggeration quality)
    testset: Tatoeba validation (25 sentences)
  - name: MOS
    value: 4.57
    system: Matcha+Duration (cascaded, overall quality)
    testset: Tatoeba validation (25 sentences)
code_available: null
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/mondal25b_interspeech.html"
related_concepts: ["prosody-control", "subjective-evaluation", "evaluation-metrics"]
related_papers: []
field_significance:
  level: "low"
  type: ["engineering-integration"]
generation:
  date: 2026-07-04
  agent: speech-generation-ingest-agent
  model: claude-sonnet-4-6
  commit: "318f1b4"
---

> [!abstract] Interspeech · 2025 · Conference
> **Mondal et al.** (IIIT Hyderabad) · [→ Paper](https://www.isca-archive.org/interspeech_2025/mondal25b_interspeech.html) · Demo: ? · Code: ?
>
> Proposes two controllable TTS-based strategies for generating exaggerated speech at user-specified word positions, targeting pronunciation feedback in computer-assisted language learning for second-language learners.

## Problem

TTS systems synthesize prosody implicitly from textual context and offer no mechanism for users to designate which words should receive heightened stress or exaggeration. Computer-assisted language learning (CALL) applications require precisely targeted feedback: when a learner mispronounces a word, the system should exaggerate that specific word in the corrective output. Prior CALL approaches rely on recordings by expert speakers, which are expensive, finite, and cannot scale to arbitrary input sentences. The absence of a controllable, scalable method for word-level exaggeration in synthesized speech is the gap this work addresses.

## Method

ExagTTS proposes two complementary strategies.

The **direct strategy** modifies GlowTTS through label-conditioned tokenization. Words tagged with a binary prominence label of 1 are converted to uppercase before text encoding. The character lookup table is expanded to include both uppercase and lowercase versions of each character, giving the text encoder a distinct embedding pathway for stressed words. GlowTTS then learns the mapping from these conditioned representations to mel-spectrograms via its flow-based decoder and monotonic alignment search. Speech waveforms are produced with a pretrained HiFi-GAN vocoder. This modified system is called GlowEx TTS and is trained for 7k iterations using the Adam optimizer with a Noam learning rate schedule on an annotated subset of the Tatoeba corpus (900 training samples).

The **cascaded strategy** leaves the TTS system unchanged and post-processes its output with a WORLD vocoder. After synthesizing speech from any of six SOTA TTS systems (Tacotron2, FastSpeech2, VITS, GlowTTS, Matcha-TTS, Parler TTS), forced alignment identifies word-segment timestamps. The WORLD vocoder then modifies energy, duration, or both at target word positions, using energy-to-duration ratios calibrated against prior CALL work. The approach is architecture-agnostic and requires no retraining.

Evaluation uses an automatic stress detection pipeline (DNN classifier and K-Means clustering on FastSpeech2 variance adaptor prosody embeddings, measuring true positive and false positive rates) and a perceptual MOS study with 20 expert evaluators (Masters/PhD scholars in speech and language processing) rating 25 sentences on exaggeration quality and overall naturalness on a 1-to-5 scale.

## Key Results

Under automatic DNN stress detection, Matcha-TTS with duration modification achieves the highest true positive rate (97.59%, false positive rate 3.2%), followed by FastPitch+Duration (96.93% TP, 4.68% FP). GlowEx TTS reaches 88.76% TP (DNN) and 83.54% (K-Means), outperforming baseline GlowTTS in the unsupervised K-Means condition (66.6%) but performing comparably under supervised DNN classification (89.84% for baseline GlowTTS).

In perceptual evaluation, Matcha-TTS with duration modification achieves the highest exaggeration quality MOS of 4.6 and the highest overall quality MOS of 4.57. Tacotron2 with combined energy and duration modification reaches 4.5 exaggeration quality MOS and 95.7% DNN true positive rate. Energy-only modification consistently underperforms duration-only modification across all TTS systems and slightly reduces overall naturalness scores.

## Novelty Assessment

The label-conditioned tokenization in GlowEx TTS is a minor conditioning adaptation: it uppercase-encodes stressed words and duplicates the character lookup table entries, effectively adding a case-sensitive embedding layer to GlowTTS. This does not constitute a new architecture or training objective. The cascaded strategy is a standard engineering pipeline combining existing TTS synthesis, forced alignment, and WORLD vocoder post-processing with energy and duration scaling ratios drawn from prior CALL work. Neither contribution introduces new model designs, training methods, or data collection techniques beyond annotating a 1000-sentence Tatoeba subset. The primary value lies in the application framing and the comparative evaluation across six TTS systems in a CALL-specific context.

## Field Significance

Low - ExagTTS addresses a specific and narrow application: scalable word-level exaggeration for second-language pronunciation training. Within the CALL domain, the comparative evaluation across six TTS backends provides useful evidence that duration modification is a more reliable exaggeration cue than energy modification. The contribution is engineering-level integration of existing components and does not introduce principles that transfer broadly beyond this application context.

## Claims

- **supports:** Explicit prosody conditioning via label-encoded tokenization enables word-level stress control in flow-based TTS without modifying the core model architecture.
  > *Evidence:* GlowEx TTS, which encodes stressed words as uppercase tokens in an expanded character lookup table within GlowTTS, achieves 88.76% true positive rate under supervised DNN stress detection and outperforms baseline GlowTTS in unsupervised K-Means classification (83.54% vs. 66.6%). *(§4.1, Table 1)*

- **supports:** Duration modification is a more reliable cue for perceptible word-level stress exaggeration than energy modification in cascaded TTS-vocoder systems.
  > *Evidence:* Duration-based WORLD vocoder modifications consistently achieve higher stress detection true positive rates and higher exaggeration quality MOS (Matcha+Duration: 97.59% TP, MOS 4.6) compared to energy-only modifications (Matcha+Energy: 88.76% TP, MOS 4.5) across all six TTS systems evaluated. *(§6.1, §6.2, Table 1)*

- **complicates:** Unsupervised and supervised evaluation methods for prosody detection may disagree on the effectiveness of explicit stress conditioning approaches.
  > *Evidence:* GlowEx TTS outperforms baseline GlowTTS under unsupervised K-Means stress detection (83.54% vs. 66.6%) but shows nearly identical performance under supervised DNN classification (88.76% vs. 89.84%), suggesting the benefit of label conditioning is partially masked when classifiers are trained on the same prosody distribution. *(§6.1, Table 1)*

- **complicates:** Energy modifications in post-hoc vocoder-based exaggeration can reduce perceived naturalness even while boosting stress prominence, creating a trade-off for applications that require both.
  > *Evidence:* Energy modifications consistently lower overall quality MOS relative to duration-only modifications across all cascaded TTS systems (e.g., Matcha overall MOS 4.5 with energy vs. 4.57 with duration), while duration modification preserves the better balance between exaggeration quality and naturalness. *(§6.2, Figure 4)*

## Limitations and Open Questions

The study is restricted to English with a small dataset (1000 sentences from Tatoeba, 900 training/100 validation) manually annotated by the authors, limiting statistical reliability and generalizability to other languages or speaking styles. All six TTS baselines in the cascaded strategy use pretrained models not fine-tuned to the Tatoeba domain. The forced alignment step introduces potential errors that are not independently evaluated. The perceptual study recruits 20 evaluators who are experts in speech and language processing, rather than actual L2 learners, which may not reflect the end-user experience. The authors identify suppression of non-target word prominence as future work.

## Wiki Connections

- [[prosody-control|Prosody Control]] - this paper introduces a word-level stress conditioning mechanism (label-conditioned tokenization in GlowEx TTS) as an explicit prosody control approach for TTS-based pronunciation training.
- [[subjective-evaluation|Subjective Evaluation]] - the paper uses a structured perceptual MOS study with 20 expert evaluators rating both exaggeration quality and overall naturalness across seven speech conditions per sentence.
- [[evaluation-metrics|Evaluation Metrics]] - the paper proposes an automatic stress detection pipeline using FastSpeech2 variance adaptor prosody embeddings evaluated with both supervised DNN classification and unsupervised K-Means clustering.
