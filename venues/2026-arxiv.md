---
title: "arXiv 2026"
venue: arXiv
year: 2026
papers_ingested: 10
last_updated: 2026-05-30
---

# arXiv 2026

## Overview

This page tracks arXiv preprints first published in 2026. Nine integrated papers are currently represented.

The 2026 arXiv TTS papers in this corpus represent the emerging frontier of large-scale multilingual and production-grade speech synthesis. The dominant themes are: (1) ultra-scale multilingual coverage — OmniVoice [[2604.00688]] covers 600+ languages from 581K hours of open-source data; Qwen3-TTS [[2601.15621]] covers 10 languages from 5M hours of proprietary data; Fish Audio S2 [[2603.08823]] covers 80+ languages from 10M+ hours with production-grade streaming; MOSS-TTS [[2603.18090]] offers a fully open-source large-scale alternative with millions of hours of training; T5Gemma-TTS [[2604.01760]] proposes encoder-decoder conditioning to address text dilution in multilingual decoder-only AR TTS; IndexTTS 2.5 [[2601.03888]] extends IndexTTS 2 to four languages with 2.28x faster inference; (2) waveform-latent flow matching — LongCat-AudioDiT [[2603.29339]] bypasses mel-spectrogram intermediates entirely, achieving the best diffusion NAR speaker similarity in the corpus on Seed-ZH (SIM 0.818); (3) GRPO multi-dimensional RL alignment as the standard post-training approach, used by Fish Audio S2 and Qwen3-TTS (with DPO+GRPO in sequence); (4) architectural diversity in codec utilization — LLaDA-TTS [[2603.26364]] replaces the AR LM stage with masked discrete diffusion achieving CER 0.98% (zh); a novel streaming architecture [[2604.12438]] achieves 303× real-time synthesis with 48.99 ms latency using depth-wise sequential codec decoding.

Standout papers: [[2603.08823]] (Fish Audio S2) achieves best open-source WER (0.54% zh / 0.99% en) and 81.88% win rate on EmergentTTS-Eval against GPT-4o-audio — the strongest publicly available instruction-following TTS result; [[2603.29339]] (LongCat-AudioDiT) identifies a previously unreported training-inference mismatch in all Voicebox/F5-TTS-style models and achieves SOTA diffusion NAR speaker similarity on Seed-ZH. MOSS-TTS [[2603.18090]] closes the quality gap with closed-source systems while being fully open-source at 2.04% WER (EN) and 2.07% CER (ZH) on Seed-TTS-Eval.

## Papers

| ID | Title |
|----|-------|
| 2601.15621 | Qwen3-TTS Technical Report |
| 2604.00688 | OmniVoice: Towards Omnilingual Zero-Shot Text-to-Speech with Diffusion Language Models |
| 2603.08823 | Fish Audio S2 Technical Report |
| 2603.29339 | LongCat-AudioDiT: High-Fidelity Diffusion Text-to-Speech in the Waveform Latent Space |
| 2604.12438 | An Ultra-Low Latency, End-to-End Streaming Speech Synthesis Architecture via Block-Wise Generation and Depth-Wise Codec Decoding |
| 2604.01760 | T5Gemma-TTS Technical Report |
| 2603.18090 | MOSS-TTS Technical Report |
| 2603.26364 | LLaDA-TTS: Unifying Speech Synthesis and Zero-Shot Editing via Masked Diffusion Modeling |
| 2601.03888 | IndexTTS 2.5 Technical Report |
| 2507.09318 | ZipVoice-Dialog: Non-Autoregressive Spoken Dialogue Generation with Flow Matching |
