---
slug: transformer-enc-dec-tts
title: Transformer Encoder-Decoder TTS
aliases: [non-autoregressive TTS, FastSpeech family, parallel TTS, NAR TTS]
related_concepts: [flow-matching, diffusion-tts, prosody-control, gan-vocoder]
last_updated: 2026-05-29
---

# Transformer Encoder-Decoder TTS

## What it is

# TODO: expand

## Why it matters

# TODO: expand

## Current state of the art

# TODO: expand

## Key variants and sub-approaches

# TODO: expand

## Comparison to alternatives

# TODO: expand

## Year-on-year trajectory

# TODO: expand

## Open questions

# TODO: expand

## Papers

| ID | Title | Venue | Year | Key use of this concept |
|----|-------|-------|------|------------------------|
| [[2025.acl-long.911]] | DNASpeech: A Contextualized and Situated TTS Dataset with Dialogue, Narrative, and Action Prompts | ACL | 2025 | Evaluates FastSpeech2-style and Tacotron-style enc-dec TTS models (TacotronGST, FastSpeech2-GST, FastSpeech2) as baselines for situated TTS; shows that traditional enc-dec models score better on situated naturalness MOS-S than codec TTS despite lower objective scores |
| [[2025.acl-short.81]] | Zero-Shot Text-to-Speech for Vietnamese | ACL | 2025 | Benchmarks VoiceCraft (enc-dec codec LM) and XTTS-v2 (cross-lingual enc-dec with VITS backbone) as zero-shot TTS baselines for Vietnamese; demonstrates XTTS-v2 fine-tuned on domain data outperforms VALL-E-style systems |
| [[2025.coling-main.352]] | DiffStyleTTS: Diffusion-based Hierarchical Prosody Modeling for TTS | workshop | 2025 | Extends FastSpeech2-style enc-dec architecture with a conditional diffusion module replacing the variance adaptor; preserves the text-encoder/decoder structure while adding stochastic prosody generation |
| [[2508.11273]] | EmoSSLSphere: Multilingual Emotional Speech Synthesis with Spherical VAE | arXiv | 2025 | FastSpeech2-style enc-dec with additional AVD emotion embeddings and HuBERT-derived prosody tokens; combines the efficiency of parallel prediction with rich conditioning |
| [[2508.12001]] | FNH-TTS: A Fast, Natural, and Human-Like Speech Synthesis System | arXiv | 2025 | VITS-based enc-dec TTS with MoE duration predictor (16 experts) and VOCOS vocoder replacement; MOS 4.48 on LJSpeech — highest naturalness among evaluated systems including StyleTTS2 and F5-TTS |
| [[2604.01760]] | T5Gemma-TTS Technical Report | arXiv | 2026 | Encoder-decoder codec language model using T5 text encoder with cross-attention to Gemma decoder; solves text dilution in decoder-only AR TTS; best WER across multiple languages |
| [[2604.12438]] | An Ultra-Low Latency End-to-End Streaming Speech Synthesis Architecture | arXiv | 2026 | Non-autoregressive FastSpeech2-style enc-dec with depth-wise sequential codec decoding over 32 Mimi RVQ layers; demonstrates that enc-dec architecture with codec output enables 303× real-time streaming synthesis |
