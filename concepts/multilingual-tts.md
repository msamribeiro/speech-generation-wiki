---
slug: multilingual-tts
title: Multilingual TTS
aliases: [cross-lingual TTS, polyglot TTS, multilingual speech synthesis, cross-lingual voice cloning]
status: dominant
related_concepts: [zero-shot-tts, self-supervised-speech, speaker-adaptation, neural-codec, flow-matching]
last_updated: 2026-06-02
---

## Executive Summary

> [!abstract]
> Multilingual TTS systems synthesize speech in two or more languages from a single model, sharing parameters and acoustic representations across languages. As of 2026, the frontier has shifted from tens of languages with English/Chinese focus to systems claiming full FLEURS coverage (600+ languages), driven by LLM initialization, shared subword tokenizers, and massive data curation pipelines. Multilingual TTS is now a dominant paradigm in production TTS — any new large-scale system is expected to cover at least 5–10 major languages.

## Current Status

dominant — Multilingual capability is now table stakes for production-grade TTS systems. The gap between monolingual and multilingual quality has narrowed substantially: leading open-source multilingual systems (OmniVoice [[2604.00688]], Fish Audio S2 [[2603.08823]], Qwen3-TTS [[2601.15621]]) match or exceed the quality of dedicated monolingual systems on widely-studied languages. Coverage breadth has expanded qualitatively, with OmniVoice [[2604.00688]] claiming 600+ languages from a single 0.8B model trained on 581K hours of open-source data.

## Why This Matters

Monolingual TTS requires separate model training for each language, which is expensive and impractical for low-resource languages. Multilingual TTS enables knowledge transfer across languages (especially helpful for under-resourced ones), supports multilingual speakers and code-switching scenarios, and allows zero-shot cross-lingual voice cloning where a speaker's voice in one language is used to synthesize speech in another.

Cross-lingual zero-shot voice cloning — synthesizing in language B with a speaker enrolled in language A — is a capability unavailable in monolingual systems and is now actively evaluated as a standard benchmark (CV3-Eval cross-lingual protocol).

## Core Idea

A multilingual TTS model is trained on speech data from multiple languages simultaneously, learning a shared acoustic space and language-aware text front-end. Language conditioning may be explicit (language ID embeddings, per-language g2p) or implicit (shared subword tokenizers, LLM initialization with multilingual pre-training). The key challenge is preventing language leakage — where the model synthesizes target text in the wrong accent or prosodic pattern from the reference speaker's language — while maintaining cross-lingual voice identity.

## Methods and Variants

**Character/phoneme-unified models.** F5-TTS ([[2025.acl-long.313]]) uses a joint character vocabulary (2546 tokens covering English alphabets, Chinese pinyin, and all language characters in Emilia) with filler token padding, enabling seamless code-switching without explicit language IDs.

**Semantic token conditioning.** CosyVoice [[2407.05407]] and CosyVoice 2 [[2412.10117]] use supervised semantic tokens from a multilingual ASR encoder. Fish Audio S2 [[2603.08823]] extends this with w2v-BERT 2.0 semantic distillation in the first codebook layer, then trains on 80+ languages with a rich-transcription ASR model for data annotation at scale.

**LLM initialization for discrete NAR.** OmniVoice [[2604.00688]] demonstrates that initializing a bidirectional masked generative model from Qwen3-0.6B LLM weights resolves the intelligibility gap in single-stage NAR TTS — with the LLM's pre-trained multilingual linguistic knowledge directly enabling 600+ language coverage through shared subword tokenization.

**Shared cross-lingual phoneme latent space.** For code-switching without bilingual training data, [[2025.ccl-1.80]] maps Lao and English phoneme representations into a shared latent via cross-attention, plus per-token language ID embeddings, achieving CMOS improvement from -0.97 to -0.27 for Lao synthesis.

**Under-resourced language adaptation.** For languages with limited data, frameworks such as [[2025.acl-industry.42]] (Thai) emphasize data curation, normalization, and leveraging cross-lingual transfer from high-resource languages.

**Multilingual LLM backbone.** VoiceCraft-X [[2511.12347]] and CosyVoice 2 [[2412.10117]] use Qwen3/Qwen2.5 LLM backbones whose pre-trained multilingual knowledge reduces the data requirement per language. T5Gemma-TTS [[2604.01760]] demonstrates that encoder-decoder cross-attention for text conditioning provides better multilingual WER than decoder-only AR TTS.

## Major Claims

Claims are generalised propositions aggregated from paper evidence.

### Strongly Supported

- LLM initialization substantially reduces the data requirement for multilingual TTS and enables coverage of hundreds of languages that would otherwise require language-specific data curation.
  Supporting: [[2604.00688]], [[2511.12347]], [[2412.10117]]

- Supervised semantic tokens derived from multilingual ASR encoders improve zero-shot cross-lingual voice cloning quality compared to unsupervised alternatives.
  Supporting: [[2407.05407]], [[2412.10117]], [[2603.08823]]

- Cross-lingual zero-shot voice cloning (reference in language A, synthesis in language B) is consistently harder than same-language zero-shot synthesis; speaker similarity and intelligibility both degrade.
  Supporting: [[2601.15621]], [[2603.08823]], [[2025.acl-long.598]]

### Emerging

- Preference alignment on English/Chinese pairs generalizes intelligibility improvements to other languages not seen during alignment training, suggesting that DPO targets fundamental articulation capabilities rather than language-specific patterns.
  Supporting: [[2025.acl-long.598]]

- Shared subword tokenization from a multilingual LLM can eliminate per-language g2p preprocessing, and the quality is competitive with phoneme-based front-ends for high-resource languages.
  Supporting: [[2604.00688]], [[2603.18090]]

- Dedicated phoneme-based pipelines with language-specific lexicons still outperform large multilingual models for very low-resource languages with non-Latin scripts.
  Supporting: [[interspeech-2025-0469]]

- Large multilingual LLM-TTS systems fine-tuned with language-specific data and speaker conditioning substantially outperform conventional non-autoregressive models on naturalness for low-resource languages.
  Supporting: [[2508.08715]]

- BPE-based multilingual TTS models systematically underperform on phonologically complex languages where reading ambiguity is high and pitch accent is lexically contrastive; parameter-efficient LoRA with phoneme-tag injection can restore user-controllable pronunciation correction.
  Supporting: [[2508.09767]]

- Two-stage semantic-acoustic decoupling (text-to-semantic LM + conditional flow matching acoustic model) enables independent training and simplifies new language addition to a multilingual system at approximately 200 hours per language.
  Supporting: [[2508.14049]]

- Intelligibility in low-resource languages with limited training data remains markedly worse than high-resource languages within the same multilingual system.
  Supporting: [[2508.14049]]

- Cross-lingual emotion transfer from a high-resource to a low-resource tonal language can be enabled by retrieval-augmented prosody prompting without parallel bilingual emotional data.
  Supporting: [[2508.07302]]

- Non-verbal vocalizations (laughter, sighs) show cross-lingual acoustic generalisability: NV detection models trained on English speech perform comparably on Chinese test sets.
  Supporting: [[2508.05385]]

### Contested

> [!warning]
> Whether scale (more data, more languages) or architectural choice (LLM initialization, semantic tokens) is the primary driver of multilingual quality improvements is not resolved. Large-scale industrial systems trained on millions of hours ([[2601.15621]], [[2603.08823]]) cannot be cleanly compared to smaller open-source systems trained on hundreds of thousands of hours.
> Supporting large-scale data: [[2601.15621]], [[2603.08823]] / Supporting architectural innovation: [[2604.00688]], [[2025.ccl-1.80]]

## Relationship to Other Concepts

### Extends or Builds On
- [[zero-shot-tts]] — cross-lingual voice cloning is zero-shot TTS applied across language boundaries; the same speaker similarity / intelligibility trade-off applies
- [[self-supervised-speech]] — multilingual SSL representations (w2v-BERT 2.0, WavLM) are the primary source of language-agnostic content features
- [[neural-codec]] — shared codec vocabularies (RVQ, FSQ) over multilingual data are the acoustic backbone of most multilingual systems

### Competes With
- Monolingual model ensembles — higher per-language ceiling quality but much higher total training and deployment cost

### Commonly Paired With
- [[flow-matching]] — FM acoustic decoder conditioned on multilingual semantic tokens is the dominant hybrid architecture
- [[speaker-adaptation]] — cross-lingual voice cloning requires speaker identity to generalize across language boundaries

## Representative Papers

### Foundational
- [[2407.05407]] — CosyVoice established supervised semantic token conditioning as the dominant multilingual paradigm, covering 5 languages at 170K hours

### Influential
- [[2412.10117]] — CosyVoice 2 extended multilingual coverage with an LLM backbone and streaming; documented Japanese CER degradation as a concrete failure mode
- [[2025.acl-long.313]] — F5-TTS demonstrated joint EN+ZH training with a unified pinyin+character vocabulary, enabling implicit code-switching

### Recent Highlights
- [[2604.00688]] — OmniVoice: 600+ languages from 581K hours of open-source data via LLM initialization; qualitative step-change in scope
- [[2601.15621]] — Qwen3-TTS: best multilingual SPK-SIM across 10 languages; 5M hours proprietary training
- [[2603.08823]] — Fish Audio S2: best average multilingual WER on CV3-Eval (3.01), 80+ languages, 10M hours

### Cautionary / Negative Evidence
- [[interspeech-2025-0469]] — Punjabi/Urdu: dedicated phoneme-based Tacotron pipelines (WER 16.1%) substantially outperform Meta's MMS (31.4%) for very low-resource non-Latin-script languages, suggesting multilingual scale does not yet solve the long tail
- [[2412.10117]] — CosyVoice 2 reports CER 18.79% for Japanese due to character-set overlap with Chinese, illustrating a specific failure mode of shared subword tokenization

## Open Questions

- Can the Emilia/Pinyin approach be extended to non-CJK, non-Latin scripts (e.g., Arabic, Devanagari) without explicit phoneme alignment?
- What is the minimum data per language needed to achieve acceptable zero-shot quality in a multilingual model?
- How can code-switching models handle intra-word switches and phonological interference at language boundaries?
- Does preference alignment on EN+ZH pairs generalize to languages beyond the ones tested in [[2025.acl-long.598]] (JA, KO, DE, FR)? And does it hold for tonal or morphologically complex languages?
- Is cross-lingual zero-shot voice cloning fundamentally limited by the absence of shared phoneme inventory, or is it a training data problem?
- Does shared subword tokenization reliably handle tonal languages (Mandarin, Cantonese, Thai, Vietnamese) where tone is phonemically contrastive?
- UtterTune [[2508.09767]] provides targeted pitch accent correction for Japanese via LoRA; can this approach generalise to other tonally complex languages (Mandarin tones, Scandinavian pitch accent) without language-specific phonemic transcription?
- MahaTTS-v2 [[2508.14049]] achieves competitive results on 22 Indic languages but English dominates (58%) the training data; how much of the cross-language quality is genuine transfer vs. data dominance?
- The non-verbal speech pipeline [[2508.05385]] generalises across language in its NV detection model without retraining; does this hold for synthesis, where NV vocalizations may have culture-specific acoustic realizations?

## Trend Summary

2024: Emilia dataset (100K hours EN+ZH) enabled high-quality open-weight multilingual TTS. CosyVoice [[2407.05407]] demonstrated multilingual at 170K hours; CosyVoice 2 [[2412.10117]] added streaming and an LLM backbone. 2025: F5-TTS ([[2025.acl-long.313]]) showed EN+ZH code-switching; preference alignment ([[2025.acl-long.598]]) extended to cross-lingual domains; under-resourced language TTS ([[2025.acl-industry.42]], [[2025.ccl-1.80]]) attracted attention at NLP venues; LongCat-AudioDiT [[2603.29339]] used UMT5 to support 107 languages. Integration pass 5 adds five new multilingual perspectives: MahaTTS-v2 [[2508.14049]] demonstrates a two-stage semantic-acoustic pipeline for 22 Indic languages from 20K hours of open-source data, with 200 hours sufficient to add a new language; MultiGen [[2508.08715]] demonstrates fine-tuning a multilingual foundation model (CosyVoice-300M) for child-friendly speech in three low-resource Southeast Asian languages with as few as 1,400 utterances; UtterTune [[2508.09767]] demonstrates LoRA-based pitch accent correction for Japanese as a lightweight language-specific corrective layer; XEmoRAG [[2508.07302]] demonstrates retrieval-augmented cross-lingual emotion transfer from Chinese to Thai without parallel bilingual emotional data; the non-verbal speech pipeline [[2508.05385]] provides a bilingual (EN/ZH) NV corpus showing cross-lingual generalisation in NV detection without retraining. The Meitei Mayek TTS paper [[2508.06870]] adds the first existence proof for a TTS system in the Manipuri script, reaffirming that phoneme mapping development remains a prerequisite for any standard pipeline applied to indigenous writing systems. 2026: OmniVoice [[2604.00688]] crossed 600 languages from a single open-source model — a qualitative step change in scope; Qwen3-TTS [[2601.15621]] achieved top SPK-SIM across 10 languages with 5M hours; Fish Audio S2 [[2603.08823]] achieved best multilingual WER across 80+ languages on 10M hours. The frontier has shifted from dozens of languages with EN/ZH focus to systems claiming full FLEURS coverage, driven by LLM initialization, shared subword tokenizers, and massive data curation pipelines.

## All Papers

| ID | Title | Venue | Year | Role in this concept |
|----|-------|-------|------|---------------------|
| [[2025.acl-long.313]] | F5-TTS: A Fairytaler that Fakes Fluent and Faithful Speech with Flow Matching | ACL | 2025 | Joint EN+ZH training with unified pinyin+character vocabulary; EN+ZH code-switching |
| [[2025.acl-industry.42]] | Scaling Under-Resourced TTS: A Data-Optimized Framework for Thai | ACL | 2025 | Under-resourced language TTS via data curation and cross-lingual transfer |
| [[2025.acl-long.598]] | Advancing Zero-shot TTS Intelligibility across Diverse Domains via Preference Alignment | ACL | 2025 | DPO alignment on EN+ZH pairs generalizes to unseen languages (JA, KO, DE, FR) |
| [[2025.acl-long.388]] | Distilling an End-to-End Voice Assistant Without Instruction Training Data | ACL | 2025 | Cross-modal distillation from text LLM generalizes to multilingual speech translation |
| [[2407.05407]] | CosyVoice: A Scalable Multilingual Zero-shot TTS based on Supervised Semantic Tokens | arXiv | 2024 | Foundational multilingual zero-shot TTS; supervised semantic tokens from multilingual ASR encoder |
| [[2412.10117]] | CosyVoice 2: Scalable Streaming Speech Synthesis with Large Language Models | arXiv | 2024 | Extended multilingual coverage with Qwen2.5-0.5B backbone; documents Japanese CER failure mode |
| [[2603.08823]] | Fish Audio S2 Technical Report | arXiv | 2026 | 80+ languages on 10M+ hours; best average WER on CV3-Eval multilingual voice cloning |
| [[2601.15621]] | Qwen3-TTS Technical Report | arXiv | 2026 | Best SPK-SIM across 10 languages; top cross-lingual voice cloning on CV3-Eval |
| [[2604.00688]] | OmniVoice: Towards Omnilingual Zero-Shot TTS with Diffusion Language Models | arXiv | 2026 | 600+ languages from open-source data; LLM initialization resolves NAR intelligibility gap |
| [[2603.29339]] | LongCat-AudioDiT: High-Fidelity Diffusion TTS in the Waveform Latent Space | arXiv | 2026 | UMT5-base text encoder supporting 107 languages |
| [[2512.04720]] | M3-TTS: Multi-modal DiT Alignment and Mel-latent for Zero-shot TTS | arXiv | 2025 | Bilingual EN+ZH flow-matching NAR trained on 95K hours Emilia |
| [[2025.ccl-1.80]] | Lao-English Code-Switched Speech Synthesis Via Neural Codec Language Modeling | workshop | 2025 | Extends VALL-E X to Lao-English code-switching without bilingual training data |
| [[2025.acl-short.81]] | Zero-Shot Text-to-Speech for Vietnamese | ACL | 2025 | PhoAudiobook dataset; challenges of adapting English-trained codec LMs to a tonal low-resource language |
| [[2511.12347]] | VoiceCraft-X: Unifying Multilingual Voice-Cloning Speech Synthesis and Editing | EMNLP | 2025 | 11-language TTS and editing via Qwen3 LLM backbone |
| [[2601.03888]] | IndexTTS 2.5 Technical Report | arXiv | 2026 | 4-language zero-shot TTS; token-level concatenation for multilingual voice cloning |
| [[2603.18090]] | MOSS-TTS Technical Report | arXiv | 2026 | Large-scale AR TTS; causal tokenizer for unified multilingual speech representation |
| [[2604.01760]] | T5Gemma-TTS Technical Report | arXiv | 2026 | Encoder-decoder codec LM; best WER for EN, IT, PT, ZH among compared systems |
| [[interspeech-2025-0469]] | Developing High-Quality TTS for Punjabi and Urdu: Benchmarking against MMS Models | Interspeech | 2025 | Dedicated phoneme-based pipelines outperform Meta MMS for very low-resource non-Latin scripts |
| [[interspeech-2025-1034]] | Non-Standard Accent TTS Support via Large Multi-Accent Frontend Pronunciation Knowledge Transfer | Interspeech | 2025 | 14-accent neural Seq2Seq frontend; 95% data reduction for new accent via cross-accent transfer |
| [[interspeech-2025-0762]] | Intrasentential English in Swedish TTS: perceived English-accentedness | Interspeech | 2025 | Per-phoneme accentedness parameter for intrasentential code-switching in Swedish TTS |
| [[interspeech-2025-0143]] | Multimodal Prosody Modeling: A Use Case for Multilingual Sentence Mode Prediction | Interspeech | 2025 | Cross-lingual transfer for sentence mode prediction across Italian, French, and German |
| [[2508.05385]] | Non-Verbal Speech Generation Pipeline | arXiv | 2025 | Bilingual EN/ZH NV dataset; cross-lingual NV detection generalisation without retraining; demonstrates language-agnostic acoustic features of non-verbal sounds |
| [[2508.06870]] | TTS for Meitei Mayek Script | arXiv | 2025 | First TTS system for Manipuri (Meitei Mayek script); ARPAbet phoneme mapping for novel script; demonstrates existence proof for standard pipeline on indigenous writing system |
| [[2508.07302]] | XEmoRAG | arXiv | 2025 | Cross-lingual emotion transfer from Chinese to Thai via Emo2Vec retrieval; two-stage fine-tuning on 1K hours non-emotional + 60 hours emotional Thai data; no parallel bilingual emotional data required |
| [[2508.07426]] | Scalable Controllable Accented TTS | ASRU | 2025 | Geolocation-based accent label discovery from CommonVoice; kNN-VC timbre augmentation; 11-accent English coverage including underrepresented varieties |
| [[2508.08715]] | MultiGen | arXiv | 2025 | CosyVoice-300M fine-tuning for child-friendly speech in Singaporean-accented Mandarin, Malay, and Tamil; x-vector conditioning; effective from 1,400 utterances |
| [[2508.09767]] | UtterTune | arXiv | 2025 | LoRA pitch accent correction for Japanese on CosyVoice 2; phoneme-tag tokens; accent correctness 0.499→0.899; language-specific corrective layer for BPE-input multilingual TTS |
| [[2508.14049]] | MahaTTS | arXiv | 2025 | 22-language Indic TTS from 20K hours; Gemma-based two-stage pipeline; 200 hours per new language; WER-based evaluation across 17 languages |
