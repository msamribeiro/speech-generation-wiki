---
slug: transformer-enc-dec-tts
title: Transformer Encoder-Decoder TTS
aliases: [non-autoregressive TTS, FastSpeech family, parallel TTS, NAR TTS]
related_concepts: [flow-matching, diffusion-tts, prosody-control, gan-vocoder]
last_updated: 2026-06-01
status: declining
---

## Executive Summary

> [!abstract]
> Transformer encoder-decoder TTS (FastSpeech2, Tacotron, VITS) was the dominant paradigm from 2018–2022, offering fast parallel synthesis and controllable prosody. It has been largely superseded by flow-matching and codec LM approaches for high-naturalness applications, but remains relevant for low-resource languages, pathological speech adaptation, and streaming scenarios requiring ultra-low latency.

## Current Status

declining — Transformer enc-dec TTS is no longer the dominant approach for high-quality English TTS; flow-matching (F5-TTS, Voicebox) and autoregressive codec LMs (VALL-E family) have surpassed it on naturalness benchmarks. However, the architecture remains active in low-resource language work, specialized domains (dysarthric TTS, situated dialogue, accent adaptation), and streaming architectures where its parallel decoding enables 303× real-time synthesis.

## Why This Matters

# TODO: expand

## Core Idea

# TODO: expand

## Methods and Variants

# TODO: expand

## Major Claims

Claims are generalised propositions aggregated from paper evidence. The full claim registry with supporting paper lists is in `wiki/concepts/_evidence/transformer-enc-dec-tts.yaml`.

### Strongly Supported

- Transformer enc-dec TTS (FastSpeech2, VITS) remains competitive with or superior to codec LM approaches on situated naturalness and low-resource language benchmarks where training data is limited.
  Supporting: [[2025.acl-long.911]], [[2025.acl-short.81]], [[interspeech-2025-0469]]

### Emerging

- Hybrid enc-dec architectures that replace the variance adaptor with a diffusion or flow-matching module preserve the efficiency of parallel prediction while adding stochastic prosody generation.
  Supporting: [[2025.coling-main.352]], [[2508.11273]]

- Non-autoregressive enc-dec with codec output enables ultra-low-latency streaming synthesis (303× real-time) that purely autoregressive codec LMs cannot match.
  Supporting: [[2604.12438]]

## Relationship to Other Concepts

### Competes With
- [[flow-matching]] — flow-matching TTS (F5-TTS, Voicebox) has largely superseded enc-dec for high-naturalness synthesis on English benchmarks; enc-dec retains an advantage in low-resource settings and streaming latency.
- [[autoregressive-codec-tts]] — codec LMs (VALL-E family) outperform enc-dec on zero-shot speaker generalization; enc-dec remains competitive for situated naturalness and parallel decoding speed.

### Extends or Builds On
- [[prosody-control]] — the variance adaptor in FastSpeech2-style models is the primary mechanism for explicit prosody control (duration, pitch, energy); enc-dec architecture was the original host for this component.

### Commonly Paired With
- [[gan-vocoder]] — GAN vocoders (HiFi-GAN, Vocos) are the standard waveform synthesis backend for FastSpeech2-style and VITS-based enc-dec TTS systems.
- [[diffusion-tts]] — hybrid enc-dec+diffusion architectures replace the variance adaptor with a diffusion module, combining parallel text encoding with stochastic prosody generation.

## Representative Papers

### Foundational
- [[2508.12001]] — VITS-based enc-dec TTS with MoE duration predictor and Vocos vocoder replacement achieves MOS 4.48 on LJSpeech, the highest among evaluated systems including StyleTTS2 and F5-TTS.

### Influential
- [[2025.acl-long.911]] — demonstrates that FastSpeech2-style enc-dec models score better on situated naturalness MOS-S than codec TTS despite lower objective scores, revealing a domain-specific advantage for traditional architectures.
- [[2604.12438]] — non-autoregressive enc-dec with depth-wise sequential codec decoding achieves 303× real-time streaming synthesis, showing the architecture's unique latency advantage.

### Recent Highlights
- [[2604.01760]] — T5Gemma-TTS uses T5 text encoder with cross-attention to Gemma decoder as an encoder-decoder codec LM, solving text dilution in decoder-only AR TTS and achieving best WER across multiple languages.

## Open Questions

# TODO: expand

## Trend Summary

# TODO: expand

## All Papers

| ID | Title | Venue | Year | Key use of this concept |
|----|-------|-------|------|------------------------|
| [[2025.acl-long.911]] | DNASpeech: A Contextualized and Situated TTS Dataset with Dialogue, Narrative, and Action Prompts | ACL | 2025 | Evaluates FastSpeech2-style and Tacotron-style enc-dec TTS models (TacotronGST, FastSpeech2-GST, FastSpeech2) as baselines for situated TTS; shows that traditional enc-dec models score better on situated naturalness MOS-S than codec TTS despite lower objective scores |
| [[2025.acl-short.81]] | Zero-Shot Text-to-Speech for Vietnamese | ACL | 2025 | Benchmarks VoiceCraft (enc-dec codec LM) and XTTS-v2 (cross-lingual enc-dec with VITS backbone) as zero-shot TTS baselines for Vietnamese; demonstrates XTTS-v2 fine-tuned on domain data outperforms VALL-E-style systems |
| [[2025.coling-main.352]] | DiffStyleTTS: Diffusion-based Hierarchical Prosody Modeling for TTS | workshop | 2025 | Extends FastSpeech2-style enc-dec architecture with a conditional diffusion module replacing the variance adaptor; preserves the text-encoder/decoder structure while adding stochastic prosody generation |
| [[2508.11273]] | EmoSSLSphere: Multilingual Emotional Speech Synthesis with Spherical VAE | arXiv | 2025 | FastSpeech2-style enc-dec with additional AVD emotion embeddings and HuBERT-derived prosody tokens; combines the efficiency of parallel prediction with rich conditioning |
| [[2508.12001]] | FNH-TTS: A Fast, Natural, and Human-Like Speech Synthesis System | arXiv | 2025 | VITS-based enc-dec TTS with MoE duration predictor (16 experts) and VOCOS vocoder replacement; MOS 4.48 on LJSpeech — highest naturalness among evaluated systems including StyleTTS2 and F5-TTS |
| [[2604.01760]] | T5Gemma-TTS Technical Report | arXiv | 2026 | Encoder-decoder codec language model using T5 text encoder with cross-attention to Gemma decoder; solves text dilution in decoder-only AR TTS; best WER across multiple languages |
| [[2604.12438]] | An Ultra-Low Latency End-to-End Streaming Speech Synthesis Architecture | arXiv | 2026 | Non-autoregressive FastSpeech2-style enc-dec with depth-wise sequential codec decoding over 32 Mimi RVQ layers; demonstrates that enc-dec architecture with codec output enables 303× real-time streaming synthesis |
| [[interspeech-2025-0596]] | Facilitating Personalized TTS for Dysarthric Speakers Using Knowledge Anchoring and Curriculum Learning | Interspeech | 2025 | FastSpeech2 + HiFi-GAN backbone with teacher-student speaker encoder for dysarthric TTS; demonstrates enc-dec TTS with specialized speaker adaptation curriculum for pathological reference audio |
| [[interspeech-2025-0754]] | EME-TTS: Unlocking the Emphasis and Emotion Link in Speech Synthesis | Interspeech | 2025 | FastSpeech2-style backbone with EPE (Emphasis Perception Enhancement) block adding attention modulation for joint emphasis-emotion control; variance-based emphasis features alongside standard duration/energy predictors |
| [[interspeech-2025-0469]] | Developing High-Quality TTS for Punjabi and Urdu: Benchmarking against MMS Models | Interspeech | 2025 | Tacotron 1 and Tacotron 2+WaveGlow applied to low-resource South Asian languages with phoneme-based lexicons; demonstrates phoneme-level enc-dec TTS outperforms MMS character-based approach |
| [[interspeech-2025-1034]] | Non-Standard Accent TTS Support via Large Multi-Accent Frontend Pronunciation Knowledge Transfer | Interspeech | 2025 | LSTM encoder-decoder Seq2Seq frontend for 14-accent pronunciation knowledge transfer; demonstrates that Tacotron-era architecture components remain relevant for low-resource accent TTS frontends |
