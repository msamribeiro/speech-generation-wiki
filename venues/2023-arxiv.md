---
title: "arXiv 2023"
venue: arXiv
year: 2023
papers_ingested: 10
last_updated: 2026-06-12
---

## Overview

The ten arXiv 2023 papers in this corpus fall into three clusters. The foundation layer is established by [[2302.13971]] (LLaMA), which provides the decoder-only transformer backbone that most subsequent speech LM systems adopt, and [[2309.15505]] (FSQ), which offers a simpler alternative to RVQ for discrete audio representation. The neural codec and tokenisation cluster is represented by [[2308.16692]] (SpeechTokenizer), which demonstrates that forcing semantic content into the first RVQ layer via HuBERT distillation produces a substantially better token vocabulary for speech language models than undifferentiated acoustic codecs, and by [[2312.05187]] (Seamless), which addresses multilingual and streaming speech translation. The speech language model cluster features [[2305.11000]] (SpeechGPT), an early unified LLM that expands vocabulary with HuBERT discrete units and introduces the chain-of-modality training curriculum, and [[2311.07919]] (Qwen-Audio), a multimodal audio-text LLM. The diffusion TTS direction is represented by [[2304.09116]] (NaturalSpeech 2), which adapts the RVQ latent codec space for a diffusion model and achieves zero-shot synthesis with singing capability. Emotion representation and evaluation round out the batch with [[2312.15185]] (emotion2vec) and [[2303.08774]] (GPT-4 Technical Report). The central tension of the year: semantic tokens (SpeechGPT/SpeechTokenizer) versus acoustic codec tokens (VALL-E lineage), with SpeechTokenizer proposing a synthesis that preserves the codec structure while importing the semantic grounding.

## Papers

| ID | Title |
|----|-------|
| 2301.02111 | Neural Codec Language Models are Zero-Shot Text to Speech Synthesizers |
| 2312.15185 | emotion2vec: Self-Supervised Pre-Training for Speech Emotion Represent |
| 2303.08774 | GPT-4 Technical Report |
| 2308.16692 | SpeechTokenizer: Unified Speech Tokenizer for Speech Large Language Models |
| 2304.09116 | NaturalSpeech 2: Latent Diffusion Models are Natural and Zero-Shot Spe |
| 2305.11000 | SpeechGPT: Empowering Large Language Models with Intrinsic Cross-Modal |
| 2311.07919 | Qwen-Audio: Advancing Universal Audio Understanding via Unified Large- |
| 2302.13971 | LLaMA: Open and Efficient Foundation Language Models |
| 2312.05187 | Seamless: Multilingual Expressive and Streaming Speech Translation |
| 2309.15505 | Finite Scalar Quantization: VQ-VAE Made Simple |
