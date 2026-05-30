---
slug: multilingual-tts
title: Multilingual TTS
aliases: [cross-lingual TTS, polyglot TTS, multilingual speech synthesis, cross-lingual voice cloning]
related_concepts: [zero-shot-tts, self-supervised-speech, speaker-adaptation, neural-codec, flow-matching]
last_updated: 2026-05-30
---

# Multilingual TTS

## What it is

Multilingual TTS systems can synthesize speech in two or more languages from a single model, typically sharing parameters across languages. They may require explicit language conditioning or rely on shared phoneme/character inventories and shared acoustic representations. A subset of multilingual TTS also supports code-switching — generating speech that seamlessly alternates between languages within a single utterance.

## Why it matters

Monolingual TTS systems require separate model training for each language, which is expensive and impractical for low-resource languages. Multilingual TTS enables knowledge transfer across languages (especially helpful for under-resourced ones), supports multilingual speakers and code-switching scenarios, and allows zero-shot cross-lingual voice cloning where a speaker's voice in one language is used to synthesize speech in another.

## Current state of the art

As of early 2026, the multilingual frontier has expanded dramatically. The key systems:
- OmniVoice [[2604.00688]]: 600+ languages from a single 0.8B NAR model trained on 581K hours of open-source data. Achieves average WER 2.85 on MiniMax-Multilingual-24 benchmark, outperforming MiniMax-Speech (3.77) and ElevenLabs Multilingual v2 (10.95). WER ≤5% for 82/102 FLEURS languages with only upsampling for low-resource. LLM initialization and full-codebook masking are key enablers.
- Qwen3-TTS [[2601.15621]]: 10 languages with best SPK-SIM in all evaluated languages; trained on 5M hours proprietary. Cross-lingual voice cloning (CV3-Eval) achieves 4.82 zh-to-ko WER vs. 14.4 for CosyVoice 3.
- Fish Audio S2 [[2603.08823]]: 80+ languages on 10M+ hours. Best open-source multilingual zero-shot WER on CV3-Eval (3.01 average, 23.9% relative improvement over Fish Audio S1).
- CosyVoice [[2407.05407]] / CosyVoice 2 [[2412.10117]]: established AR+FM multilingual baseline at 170K/170K hours with Japanese, Korean, Mandarin, English, Cantonese coverage.
- Lao-English code-switching [[2025.ccl-1.80]]: extends VALL-E X to extremely low-resource cross-lingual synthesis without bilingual training data.

## Key variants and sub-approaches

**Character/phoneme-unified models.** F5-TTS ([[2025.acl-long.313]]) uses a joint character vocabulary (2546 tokens covering English alphabets, Chinese pinyin, and all language characters in Emilia) with filler token padding, enabling seamless code-switching without explicit language IDs.

**Semantic token conditioning.** CosyVoice [[2407.05407]] and CosyVoice 2 [[2412.10117]] use supervised semantic tokens from a multilingual ASR encoder. Fish Audio S2 [[2603.08823]] extends this with w2v-BERT 2.0 semantic distillation in the first codebook layer, then trains on 80+ languages with a rich-transcription ASR model for data annotation at scale.

**LLM initialization for discrete NAR.** OmniVoice [[2604.00688]] demonstrates that initializing a bidirectional masked generative model from Qwen3-0.6B LLM weights resolves the intelligibility gap in single-stage NAR TTS — with the LLM's pre-trained multilingual linguistic knowledge directly enabling 600+ language coverage through shared subword tokenization.

**Shared cross-lingual phoneme latent space.** For code-switching without bilingual training data, [[2025.ccl-1.80]] maps Lao and English phoneme representations into a shared latent via cross-attention, plus per-token language ID embeddings, achieving CMOS improvement from -0.97 to -0.27 for Lao synthesis.

**Under-resourced language adaptation.** For languages with limited data, frameworks such as [[2025.acl-industry.42]] (Thai) emphasize data curation, normalization, and leveraging cross-lingual transfer from high-resource languages.

## Comparison to alternatives

Monolingual TTS with language-specific models achieves higher ceiling quality per language but at much higher total cost. Multilingual models trade some per-language quality for breadth, though the gap is narrowing as training data scales up. Cross-lingual zero-shot voice cloning (synthesizing in language B with a speaker from language A) remains harder than same-language zero-shot, and is an open research challenge not fully solved by any current system.

## Year-on-year trajectory

2024: Emilia dataset (100K hours EN+ZH) enabled high-quality open-weight multilingual TTS. CosyVoice [[2407.05407]] demonstrated multilingual at 170K hours; CosyVoice 2 [[2412.10117]] added streaming. 2025: F5-TTS ([[2025.acl-long.313]]) demonstrated EN+ZH code-switching; preference alignment on code-switching pairs ([[2025.acl-long.598]]) emerged; under-resourced language TTS ([[2025.acl-industry.42]] for Thai, [[2025.ccl-1.80]] for Lao) received attention; LongCat-AudioDiT [[2603.29339]] used UMT5 to support 107 languages. 2026: OmniVoice [[2604.00688]] crossed 600 languages from a single open-source model — a qualitative step change in scope; Qwen3-TTS [[2601.15621]] achieved top SPK-SIM across 10 languages with 5M hours of proprietary training; Fish Audio S2 [[2603.08823]] achieved best multilingual WER with 80+ languages on 10M hours. The multilingual frontier has shifted from dozens of languages with EN/ZH focus to systems claiming full FLEURS coverage, driven by LLM initialization, shared subword tokenizers, and massive data curation pipelines.

## Open questions

- Can the Emilia/Pinyin approach be extended to non-CJK, non-Latin scripts (e.g., Arabic, Devanagari) without explicit phoneme alignment?
- What is the minimum data per language needed to achieve acceptable zero-shot quality in a multilingual model?
- How can code-switching models handle intra-word switches and phonological interference at language boundaries?
- Does preference alignment on EN+ZH preference pairs generalize to other languages? [[2025.acl-long.598]] shows WER improvements for Japanese, Korean, German, and French after INTP alignment trained only on EN+ZH, suggesting that intelligibility-focused DPO enhances fundamental articulation capabilities across languages.

## Papers

| ID | Title | Venue | Year | Key use of this concept |
|----|-------|-------|------|------------------------|
| [[2025.acl-long.313]] | F5-TTS: A Fairytaler that Fakes Fluent and Faithful Speech with Flow Matching | ACL | 2025 | Trains on 95K hours EN+ZH Emilia dataset; uses joint pinyin+character vocabulary for code-switching; evaluates on Seed-TTS test-zh as well as English benchmarks |
| [[2025.acl-industry.42]] | Scaling Under-Resourced TTS: A Data-Optimized Framework with Advanced Acoustic Modeling for Thai | ACL | 2025 | Framework for Thai TTS with data curation and acoustic modeling for under-resourced language |
| [[2025.acl-long.598]] | Advancing Zero-shot TTS Intelligibility across Diverse Domains via Preference Alignment | ACL | 2025 | Targets code-switching (EN+ZH mixed text) and cross-lingual synthesis (zh2en, en2zh) as primary evaluation domains; shows preference alignment on INTP generalizes to unseen languages (JA, KO, DE, FR) |
| [[2025.acl-long.388]] | Distilling an End-to-End Voice Assistant Without Instruction Training Data | ACL | 2025 | Demonstrates that cross-modal distillation from a text LLM using only ASR-paired English data generalizes to multilingual speech translation (CoVoST 2) — multilingual coverage from monolingual training without explicit multilingual instruction data |
| [[2407.05407]] | CosyVoice: A Scalable Multilingual Zero-shot TTS based on Supervised Semantic Tokens | arXiv | 2024 | Supervised semantic tokens from a multilingual ASR encoder enable joint coverage of ZH, EN, Yue, JP, KO; cross-lingual cloning drops prompt prosody to prevent language leakage |
| [[2412.10117]] | CosyVoice 2: Scalable Streaming Speech Synthesis with Large Language Models | arXiv | 2024 | Extends multilingual coverage with Qwen2.5-0.5B backbone + instruction fine-tuning; reports degradation on Japanese (CER 18.79%) due to character-set overlap with Chinese |
| [[2603.08823]] | Fish Audio S2 Technical Report | arXiv | 2026 | 80+ languages on 10M+ hours with rich-transcription ASR annotation pipeline; best average WER on CV3-Eval multilingual voice cloning (3.01 vs. 3.96 for Fish Audio S1) |
| [[2601.15621]] | Qwen3-TTS Technical Report | arXiv | 2026 | Best SPK-SIM across 10 languages vs. MiniMax-Speech and ElevenLabs; cross-lingual voice cloning (CV3-Eval) zh-to-ko WER 4.82 vs. 14.4 for CosyVoice 3 |
| [[2604.00688]] | OmniVoice: Towards Omnilingual Zero-Shot TTS with Diffusion Language Models | arXiv | 2026 | 600+ languages from 581K hours of open-source data; LLM initialization resolves NAR intelligibility gap; WER ≤5% for 82/102 FLEURS languages; shared LLM subword tokenizer eliminates per-language g2p |
| [[2603.29339]] | LongCat-AudioDiT: High-Fidelity Diffusion TTS in the Waveform Latent Space | arXiv | 2026 | Uses UMT5-base text encoder (107 languages) with dual embedding (final hidden state + raw word embeddings) for multilingual support; Chinese+English primary evaluation on Seed benchmark |
| [[2512.04720]] | M3-TTS: Multi-modal DiT Alignment and Mel-latent for Zero-shot High-fidelity Speech Synthesis | arXiv | 2025 | Bilingual (EN+ZH) flow-matching NAR TTS trained on 95K hours Emilia; MMDiT cross-modal attention for alignment-free synthesis |
| [[2025.ccl-1.80]] | Lao-English Code-Switched Speech Synthesis Via Neural Codec Language Modeling | workshop | 2025 | Extends VALL-E X to Lao-English code-switching without bilingual training data via shared phoneme latent space and language ID embeddings |
| [[2025.acl-short.81]] | Zero-Shot Text-to-Speech for Vietnamese | ACL | 2025 | Creates PhoAudiobook, a 941-hour Vietnamese dataset, and benchmarks three zero-shot TTS systems; highlights challenges of adapting English-trained codec LMs to a tonal low-resource language |
| [[2511.12347]] | VoiceCraft-X: Unifying Multilingual Voice-Cloning Speech Synthesis and Editing | EMNLP | 2025 | Extends VoiceCraft to 11 languages via Qwen3 LLM backbone and text-speech token reordering; joint TTS and speech editing across Chinese, English, French, German, Japanese, Korean, Portuguese, Russian, Spanish, Ukrainian |
| [[2601.03888]] | IndexTTS 2.5 Technical Report | arXiv | 2026 | Extends IndexTTS 2 to Chinese, English, Japanese, and Spanish; evaluates three multilingual strategy variants (language-specific, token concatenation, text transliteration); token-level concatenation provides best naturalness |
| [[2603.18090]] | MOSS-TTS Technical Report | arXiv | 2026 | Multilingual AR TTS trained on millions of hours; achieves WER 2.04% (EN) and CER 2.07% (ZH) on Seed-TTS-Eval; causal tokenizer for unified multilingual speech representation |
| [[2604.01760]] | T5Gemma-TTS Technical Report | arXiv | 2026 | Encoder-decoder codec LM with cross-attention text conditioning evaluated on 6 languages (English, Chinese, Italian, German, Portuguese, Spanish); best WER for EN, IT, PT, ZH among compared systems |
| [[interspeech-2025-0469]] | Developing High-Quality TTS for Punjabi and Urdu: Benchmarking against MMS Models | Interspeech | 2025 | Demonstrates that dedicated phoneme-based Tacotron pipelines with language-specific lexicons substantially outperform Meta's MMS for Punjabi Shahmukhi and Urdu; WER 16.1% vs. 31.4% for MMS on Punjabi |
| [[interspeech-2025-1034]] | Non-Standard Accent TTS Support via Large Multi-Accent Frontend Pronunciation Knowledge Transfer | Interspeech | 2025 | 14-accent neural Seq2Seq frontend reduces pronunciation data for a new non-standard English accent by 95% (from 20k to 1k sentences) via cross-accent knowledge transfer; accent similarity weakly mediates transfer strength |
| [[interspeech-2025-0762]] | Intrasentential English in Swedish TTS: perceived English-accentedness | Interspeech | 2025 | Matcha-TTS extended with per-phoneme accentedness parameter and psychometric calibration; demonstrates listener preferences for intrasentential English vary by insertion type; no single accentedness level universally preferred |
| [[interspeech-2025-0143]] | Multimodal Prosody Modeling: A Use Case for Multilingual Sentence Mode Prediction | Interspeech | 2025 | Evaluates multilingual sentence mode prediction for TTS prosody across Italian, French, and German; cross-lingual transfer (German model applied to French) achieves UAR 63.62%, supporting multilingual prosody modeling via shared SSL representations |
