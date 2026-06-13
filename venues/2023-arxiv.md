---
title: "arXiv 2023"
venue: arXiv
year: 2023
papers_ingested: 17
last_updated: 2026-06-13
---

## Overview

The 17 arXiv 2023 papers in this corpus fall into four clusters. The foundation and infrastructure layer is established by [[2302.13971]] (LLaMA) and [[2309.15505]] (FSQ), providing the LLM backbone and simpler quantization alternative that subsequent TTS systems build on. The neural codec and tokenization cluster is anchored by [[2308.16692]] (SpeechTokenizer), which demonstrates that SSL-distilled semantic-first RVQ produces substantially better speech LM tokens than undifferentiated acoustic codecs; [[2305.02765]] (HiFi-Codec) introduces group-RVQ as an open-source alternative to EnCodec; and [[2306.00814]] (Vocos) advances GAN vocoder design into the Fourier domain, achieving 10x inference speedup at matched quality. The speech language model cluster expands substantially in this batch: [[2305.11000]] (SpeechGPT) establishes the chain-of-modality pretraining curriculum; [[2303.03926]] (VALL-E X) extends codec LM generation to cross-lingual voice transfer without paired bilingual data; [[2306.12925]] (AudioPaLM) demonstrates that LLM backbone initialization substantially outperforms training from scratch; [[2305.09636]] (SoundStorm) shows that parallel masked decoding over RVQ achieves 100x speedup at matched quality; [[2310.00704]] (UniAudio) proves multi-task audio LM training generalises across TTS, VC, and singing; and [[2311.07919]] (Qwen-Audio) addresses audio-text multimodal understanding. The TTS quality and scaling cluster features [[2304.09116]] (NaturalSpeech 2) for diffusion-based zero-shot synthesis, [[2305.07243]] (Tortoise TTS) for scaling self-supervised techniques from image generation to speech, and [[2312.05187]] (Seamless) for multilingual streaming translation. The central tension of 2023 is between semantic tokens (SpeechGPT/SpeechTokenizer/Spirit LM lineage) and acoustic codec tokens (VALL-E lineage), with SpeechTokenizer proposing a synthesis that imports semantic grounding while preserving the codec structure.

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
| 2306.00814 | Vocos: Closing the gap between time-domain and Fourier-based neural vo |
| 2305.02765 | HiFi-Codec: Group-residual Vector quantization for High Fidelity Audio |
| 2303.03926 | Speak Foreign Languages with Your Own Voice: Cross-Lingual Neural Code |
| 2305.09636 | SoundStorm: Efficient Parallel Audio Generation |
| 2306.12925 | AudioPaLM: A Large Language Model That Can Speak and Listen |
| 2305.07243 | Better speech synthesis through scaling |
| 2310.00704 | UniAudio: An Audio Foundation Model Toward Universal Audio Generation |
