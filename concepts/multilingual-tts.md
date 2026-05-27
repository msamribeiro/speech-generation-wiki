---
slug: multilingual-tts
title: Multilingual TTS
aliases: [cross-lingual TTS, polyglot TTS, multilingual speech synthesis, cross-lingual voice cloning]
related_concepts: [zero-shot-tts, self-supervised-speech, speaker-adaptation, neural-codec, flow-matching]
last_updated: 2026-05-27
---

# Multilingual TTS

## What it is

Multilingual TTS systems can synthesize speech in two or more languages from a single model, typically sharing parameters across languages. They may require explicit language conditioning or rely on shared phoneme/character inventories and shared acoustic representations. A subset of multilingual TTS also supports code-switching — generating speech that seamlessly alternates between languages within a single utterance.

## Why it matters

Monolingual TTS systems require separate model training for each language, which is expensive and impractical for low-resource languages. Multilingual TTS enables knowledge transfer across languages (especially helpful for under-resourced ones), supports multilingual speakers and code-switching scenarios, and allows zero-shot cross-lingual voice cloning where a speaker's voice in one language is used to synthesize speech in another.

## Current state of the art

As of 2025, leading open-weight multilingual TTS systems are F5-TTS ([[2025.acl-long.313]], 336M, trained on 95K hours EN+ZH from Emilia) and CosyVoice (300M, 170K hours multilingual). Both demonstrate strong Chinese and English zero-shot generation. F5-TTS uses full pinyin for Chinese and character sequences for English, with a shared filler-token padding approach that handles code-switching naturally.

For under-resourced languages, specialized frameworks exist (e.g., [[2025.acl-industry.42]] for Thai) that combine efficient data curation with advanced acoustic modeling. These typically require adapting pre-trained acoustic models rather than training from scratch.

## Key variants and sub-approaches

**Character/phoneme-unified models.** F5-TTS ([[2025.acl-long.313]]) uses a joint character vocabulary (2546 tokens covering English alphabets, Chinese pinyin, and all language characters in Emilia) with filler token padding, enabling seamless code-switching without explicit language IDs.

**Semantic token conditioning.** CosyVoice uses supervised semantic tokens learned by an LLM, which can leverage multilingual language model pre-training to improve cross-lingual prosody and pronunciation.

**Under-resourced language adaptation.** For languages with limited data, frameworks such as [[2025.acl-industry.42]] (Thai) emphasize data curation, normalization, and leveraging cross-lingual transfer from high-resource languages.

## Comparison to alternatives

Monolingual TTS with language-specific models achieves higher ceiling quality per language but at much higher total cost. Multilingual models trade some per-language quality for breadth, though the gap is narrowing as training data scales up. Cross-lingual zero-shot voice cloning (synthesizing in language B with a speaker from language A) remains harder than same-language zero-shot, and is an open research challenge not fully solved by any current system.

## Year-on-year trajectory

2024: Emilia dataset (100K hours EN+ZH) enabled training of high-quality open-weight multilingual TTS models. 2025: F5-TTS ([[2025.acl-long.313]]) demonstrated multilingual zero-shot TTS with code-switching on Emilia; CosyVoice 2 extended multilingual coverage. Under-resourced language TTS received attention at ACL 2025 ([[2025.acl-industry.42]]). Preference alignment with code-switching and cross-lingual preference pairs ([[2025.acl-long.598]]) emerged as a post-training approach for robustness in code-switching scenarios, with the INTP dataset showing 53% relative WER reduction in code-switching for F5-TTS.

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
