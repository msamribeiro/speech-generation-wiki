---
title: "arXiv 2024"
venue: arXiv
year: 2024
papers_ingested: 20
last_updated: 2026-06-12
---

# arXiv 2024

## Overview

The 20 arXiv 2024 preprints in this corpus span the full hierarchy from foundational LLM infrastructure to production speech systems. The dominant task theme is zero-shot TTS: [[2403.03100]] (NaturalSpeech 3) introduced disentangled factorized codecs achieving human-parity naturalness; [[2406.02430]] (Seed-TTS) demonstrated foundation-model-scale synthesis with RL post-training; [[2406.05370]] (VALL-E 2) reached human parity on VCTK and LibriSpeech through grouped codec sampling and repetition-aware sampling; [[2409.00750]] (MaskGCT) demonstrated that masked generative transformers can match AR codec LM quality in zero-shot TTS; [[2407.05407]] (CosyVoice) established supervised semantic tokens for multilingual zero-shot; and [[2412.10117]] (CosyVoice 2) added streaming. Production-quality systems also include [[2409.03283]] (FireRedTTS) from Kuaishou, designed for large-scale industrial deployment. The codec design space was actively contested: [[2408.16532]] (WavTokenizer) proposed single-codebook high-compression tokens for audio language models, while [[2407.04051]] (FunAudioLLM) from Alibaba combined CosyVoice TTS with SenseVoice ASR as an open-weight voice foundation suite. The most architecturally significant development was the emergence of end-to-end spoken conversational agents: [[2408.16725]] (Mini-Omni), [[2409.06666]] (LLaMA-Omni), and [[2411.00774]] (Freeze-Omni) each proposed distinct strategies for simultaneous text-audio generation with streaming LLM backbones, establishing the "omni-modal LLM" pattern; [[2410.00037]] (Moshi) established the multi-stream full-duplex speech-text LLM paradigm; and [[2410.21276]] (GPT-4o) demonstrated end-to-end natively multimodal dialogue at production scale. Crucially, [[2410.17196]] (VoiceBench) provided the first systematic evidence that pipeline-based voice assistants substantially outperform all current end-to-end audio LLMs on spoken instruction understanding, creating an important negative result against the omni-modal trend. On the LLM backbone side, [[2407.21783]] (Llama 3), [[2407.10759]] (Qwen2-Audio), and [[2412.15115]] (Qwen2.5) provide the open-weight foundations that TTS and SCA systems build on; [[2412.02612]] (GLM-4-Voice) and [[2411.13577]] (WavChat survey) round out the picture with a bilingual spoken chatbot and a comprehensive taxonomy of the emerging spoken dialogue field. The year is characterized by the convergence of LLM capability and audio generation into unified omni systems, tempered by systematic evaluation evidence that end-to-end audio LLMs still trail cascaded pipelines on semantic tasks.

## Papers

| ID | Title |
|----|-------|
| 2406.02430 | Seed-TTS: A Family of High-Quality Versatile Speech Generation Models |
| 2407.05407 | CosyVoice: A Scalable Multilingual Zero-shot Text-to-speech Synthesizer based on Supervised Semantic Tokens |
| 2412.10117 | CosyVoice 2: Scalable Streaming Speech Synthesis with Large Language Models |
| 2410.00037 | Moshi: a speech-text foundation model for real-time dialogue |
| 2411.13577 | WavChat: A Survey of Spoken Dialogue Models |
| 2407.21783 | The Llama 3 Herd of Models |
| 2407.10759 | Qwen2-Audio Technical Report |
| 2403.03100 | NaturalSpeech 3: Zero-Shot Speech Synthesis with Factorized Codec and Diffusion Models |
| 2412.02612 | GLM-4-Voice: Towards Intelligent and Human-Like End-to-End Spoken Chatbot |
| 2410.21276 | GPT-4o System Card |
| 2412.15115 | Qwen2.5 Technical Report |
| 2409.00750 | MaskGCT: Zero-Shot Text-to-Speech with Masked Generative Codec Transformer |
| 2408.16725 | Mini-Omni: Language Models Can Hear, Talk While Thinking in Streaming |
| 2406.05370 | VALL-E 2: Neural Codec Language Models are Human Parity Zero-Shot Text |
| 2409.06666 | LLaMA-Omni: Seamless Speech Interaction with Large Language Models |
| 2411.00774 | Freeze-Omni: A Smart and Low Latency Speech-to-speech Dialogue Model w |
| 2408.16532 | WavTokenizer: an Efficient Acoustic Discrete Codec Tokenizer for Audio |
| 2407.04051 | FunAudioLLM: Voice Understanding and Generation Foundation Models for  |
| 2410.17196 | VoiceBench: Benchmarking LLM-Based Voice Assistants |
| 2409.03283 | FireRedTTS: A Foundation Text-To-Speech Framework for Industry-Level G |
