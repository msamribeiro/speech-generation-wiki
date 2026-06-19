---
id: "interspeech-2025-0143"
title: "Multimodal Prosody Modeling: A Use Case for Multilingual Sentence Mode Prediction"
authors: ["Bogdan Vlasenko", "Mathew Magimai Doss"]
organization: Idiap Research Institute
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: "2025-08-17"
ingested_date: "2026-05-30"
task: [TTS, evaluation]
architecture: []
conditioning: [multilingual]
training: [supervised]
model_size: "not reported"
codec_used: none
datasets_train: ["CML-TTS (Italian, French, German audiobook subsets)", "EMO-DB (German emotional speech)"]
datasets_eval: ["CML-TTS (Italian, French, German)", "EMO-DB"]
metrics:
  - name: UAR
    value: 74.22
    system: Early fusion (data-driven acoustic + word-level linguistic), intra-language German
    testset: CML-TTS German
  - name: UAR
    value: 67.10
    system: Early fusion (data-driven acoustic + word-level linguistic), intra-language French
    testset: CML-TTS French
  - name: UAR
    value: 64.14
    system: Early fusion (data-driven acoustic + word-level linguistic), intra-language Italian
    testset: CML-TTS Italian
code_available: true
demo_available: false
url: "https://www.isca-archive.org/interspeech_2025/vlasenko25_interspeech.html"
related_concepts: [prosody-control, multilingual-tts, self-supervised-speech, evaluation-metrics]
related_papers: []
---
> [!abstract] Interspeech · 2025 · Conference
> **Bogdan Vlasenko et al.** (Idiap Research Institute) · [→ Paper](https://www.isca-archive.org/interspeech_2025/vlasenko25_interspeech.html) · Demo: ✗ · Code: ✓
>
> An empirical study showing that early fusion of multilingual SSL acoustic embeddings (WavLM Large) with word-level multilingual text embeddings (XLM-RoBERTa) significantly outperforms unimodal and knowledge-based features for sentence mode prediction (declarative/interrogative/exclamatory) across Italian, French, and German, with implications for multilingual prosody modeling in TTS.

## Problem

Sentence mode (declarative, interrogative, exclamatory) is a suprasegmental prosodic property that TTS systems must model to generate contextually appropriate speech. TOBI-based prosodic annotation is language-specific and costly, limiting scalable multilingual prosody modeling. Prior work relies on knowledge-based features (F0 modeling) or unimodal SSL representations. It remains unclear how well multilingual latent spaces preserve sentence-mode information, and whether acoustic or linguistic features are more discriminative for cross-lingual transfer.

## Method

The study evaluates three systems for sentence mode prediction (3-class: '.', '!', '?') on Italian, French, and German:

**System 1 (main comparison):** An MLP classifier (100 hidden units, ReLU) applied to a variety of feature representations:
- Acoustic: (a) knowledge-based (COMPARE 2016, 6373 features, openSMILE) and (b) data-driven (WavLM Large final-layer embeddings, mean-pooled to sentence level)
- Linguistic: (a) phoneme-level (XPhoneBERT, multilingual phoneme BERT) and (b) word-level (XLM-RoBERTa, trained on 2.5TB of 100-language CommonCrawl)
- Early fusion (EF): (a) data-driven acoustic + word-level linguistic, and (b) all four feature types combined

Both intra-language (5-fold cross-validation) and inter-language (train on one, test on another) protocols are evaluated.

**System 2 (ASR baseline):** WHISPER Large V3 transcripts with punctuation detection.

**System 3 (cross-corpora):** MLP trained on CML-TTS German acoustic features, evaluated on EMO-DB (German emotional speech, neutral vs. emotional conditions).

Data: CML-TTS audiobook recordings (Italian: 6226, French: 7160, German: 45718 samples). Labels derived automatically from terminal punctuation marks.

## Key Results

Best intra-language UAR (System 1, early fusion of all features):
- German: 74.22% (data-driven EF) / 73.40% (all EF)
- French: 67.10% (data-driven EF)
- Italian: 64.14% (data-driven EF)

Key findings:
- Linguistic FRS (word-level XLM-RoBERTa) consistently outperform acoustic FRS for interrogative sentence discrimination.
- Acoustic FRS (knowledge-based) capture exclamatory sentences better.
- Early fusion outperforms unimodal features in intra-language settings.
- Cross-lingual transfer is feasible: models trained on German achieve UAR up to 63.62% on French (inter-language).
- WHISPER (System 2) performs poorly on exclamatory sentences (recall well below chance for all languages), despite strong declarative/interrogative recall, suggesting current ASR systems cannot reliably generate prosodically informative punctuation.
- Emotional speech (EMO-DB) causes significant degradation: most emotional samples are misclassified as exclamatory, reflecting the definitional overlap between exclamatory tone and emotional speech.

## Novelty Assessment

This is primarily an empirical study rather than a new model. Its contribution is a systematic, multilingual evaluation of multimodal feature fusion for sentence mode prediction, using recent SSL embeddings not previously benchmarked for this task. The finding that WHISPER transcripts fail to capture exclamatory sentence prosody is practically important for TTS pipelines that rely on ASR-derived text labels. The cross-corpora emotional speech experiment is a useful stress test. The lack of a dedicated TTS synthesis component means this paper's direct application to TTS output quality is indirect — it informs feature engineering for prosody prediction models that could be integrated into TTS frontends.

## Limitations and Open Questions

- No end-to-end TTS experiment; the study measures sentence mode prediction accuracy, not synthesized prosody quality.
- Labels derived from punctuation marks are a proxy for sentence mode; they may not reflect actual prosodic realization (especially for audiobooks where speakers may monotonize exclamatory passages).
- Only three languages evaluated; coverage of typologically diverse languages is absent.
- MLP classifier is simple; more powerful sequence models may perform better.
- UAR is used rather than accuracy due to class imbalance, but class sizes differ substantially across languages.

## Wiki Connections

This paper informs [[prosody-control]] by establishing that multilingual SSL embeddings encode sentence-mode information useful for TTS prosody frontends. The finding on WavLM and XLM-RoBERTa embedding utility connects to [[self-supervised-speech]] and [[multilingual-tts]] research. The WHISPER ASR baseline result is relevant to [[evaluation-metrics]], specifically concerning the reliability of text-derived prosody labels. The cross-lingual transfer experiment connects to broader [[multilingual-tts]] questions about shared prosodic representations across language families.
