---
title: "arXiv 2024"
venue: arXiv
year: 2024
papers_ingested: 6
last_updated: 2026-06-09
---

# arXiv 2024

## Overview

Three arXiv 2024 preprints are currently ingested, all foundational zero-shot TTS systems that established or advanced the state of the art heading into 2025. All three are from major industry labs (ByteDance, Alibaba) and are landmark systems widely cited in subsequent work.

The dominant theme is the hybrid architecture: an autoregressive language model generates discrete or semi-supervised speech tokens, followed by a continuous diffusion or flow-matching decoder that reconstructs high-quality mel spectrograms. All three systems demonstrate human-parity or near-human-parity performance on zero-shot TTS benchmarks (CMOS within 0.1 of ground truth, or exceeding human on objective metrics).

Key differentiators: [[2406.02430]] (Seed-TTS) operates at the largest scale and introduces self-distillation for timbre disentanglement and REINFORCE post-training; [[2407.05407]] (CosyVoice) introduces supervised S3 tokens from an ASR encoder to improve content consistency; [[2412.10117]] (CosyVoice 2) upgrades CosyVoice with FSQ tokenization and a chunk-aware causal flow matching model for streaming synthesis.

## Papers

| ID | Title |
|----|-------|
| 2406.02430 | Seed-TTS: A Family of High-Quality Versatile Speech Generation Models |
| 2407.05407 | CosyVoice: A Scalable Multilingual Zero-shot Text-to-speech Synthesizer based on Supervised Semantic Tokens |
| 2412.10117 | CosyVoice 2: Scalable Streaming Speech Synthesis with Large Language Models |
| 2410.00037 | Moshi: a speech-text foundation model for real-time dialogue |
| 2411.13577 | WavChat: A Survey of Spoken Dialogue Models |
| 2407.21783 | The Llama 3 Herd of Models |
