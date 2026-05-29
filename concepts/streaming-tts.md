---
slug: streaming-tts
title: Streaming TTS
aliases: [real-time TTS, low-latency TTS, incremental TTS, chunk-based synthesis, online voice conversion, streaming VC]
related_concepts: [spoken-language-model, autoregressive-codec-tts, neural-codec, voice-conversion, gan-vocoder]
last_updated: 2026-05-29
---

# Streaming TTS

## What it is

Streaming (or real-time) TTS refers to speech synthesis systems that produce output audio incrementally as input arrives, without waiting for the full input to be processed. The key properties are low latency (typically measured as time-to-first-audio or chunk processing time relative to chunk duration) and real-time factor (RTF < 1 meaning the system processes audio faster than it is played). In streaming TTS, the model must operate causally — it cannot access future input frames when generating current output frames.

The same principles apply to streaming voice conversion (online VC), where source speech is processed chunk-by-chunk and converted to a target speaker in real time. All components of the pipeline — content extractor, style encoder alignment, decoder, and vocoder — must be redesigned or modified to enforce causality.

## Why it matters

Streaming operation is essential for real-time communication applications (teleconferencing, gaming avatars, virtual humans), interactive voice assistants, live broadcasting, and real-time speech privacy. Systems that cannot stream produce unacceptable latency for any interactive use case. As autoregressive TTS systems become more powerful (e.g., VALL-E family, CosyVoice), adapting them for streaming without quality loss is a central engineering challenge.

## Current state of the art

For streaming VC, [[2507.14534]] (Conan, 2025) achieves 37 ms latency (fast setting) and 140 ms (full setting) with RTF of 0.74 and 0.25 respectively on a single A100 GPU, using Emformer for streaming content extraction (distilled from HuBERT), a causal mel decoder, and the Causal Shuffle Vocoder. Speaker similarity (85.71%) exceeds prior offline baselines. [[2025.acl-demo.37]] (RT-VC, 2025) achieves 61.4 ms CPU latency — 13.3% faster than StreamVC — using articulatory coding (SPARC-based EMA inverter) and a DDSP vocoder without requiring GPU hardware. The two systems represent complementary designs: Conan maximizes speaker similarity on GPU; RT-VC prioritizes CPU deployment and interpretability.

For streaming TTS (text-to-speech), FlexiCodec-TTS ([[2510.00981]]) demonstrates that very low AR frame rate (6.25 Hz) enables a 7.3× speedup in the AR stage (RTF 0.07) while maintaining competitive quality. Qwen3-TTS [[2601.15621]] demonstrates 97 ms first-packet streaming TTS with a 12.5 Hz fully causal RVQ codec and MTP module, across 10 languages at production scale. Llasa+ [[2508.06262]] adds plug-and-play MTP acceleration to a frozen Llasa backbone via a token verification algorithm, achieving 1.48x speedup without quality degradation and streaming via XCodec2-S causal adapter. Fish Audio S2 [[2603.08823]] achieves RTF 0.195 and TTFA <100ms via a Dual-AR architecture on NVIDIA H200.

For streaming speech LLM generation, LLaMA-Omni 2 [[2025.acl-long.912]] achieves 582 ms latency on a single L40 GPU for a 7B spoken chatbot by coupling a read/write-scheduled streaming TTS LM with CosyVoice 2 chunk-aware causal flow matching. VocalNet [[2025.emnlp-main.989]] achieves first-chunk latency of 319–428 ms; OpenS2S [[2025.emnlp-demos.70]] uses streaming interleaved decoding where every 4 LLM states trigger 8 speech tokens.

## Key variants and sub-approaches

**Chunk-based streaming.** Source audio is buffered into fixed-size chunks (20–80 ms for VC; similar for TTS) and processed one chunk at a time. Each output chunk is generated before the next input chunk arrives (RTF < 1 ensures this). A sliding context window provides temporal continuity across chunk boundaries.

**Causal convolutions.** Standard convolutions are replaced with left-padded causal convolutions throughout the synthesis pipeline. GAN vocoders like HiFi-GAN require full replacement of transposed convolutions to avoid spectral artifacts from zero-padding ([[2507.14534]] resolves this with pixel shuffle instead).

**Autoregressive streaming.** VALL-E-family models generate tokens autoregressively and are inherently streaming at the token level; the bottleneck is token generation speed (AR RTF). Lower frame rate codecs ([[2510.00981]]) reduce sequence length and thus AR RTF.

**Causal attention masking.** Transformer-based decoders use causal attention masks to prevent attending to future positions. Emformer ([[2507.14534]]) caches previous chunk keys/values and summary vectors as a memory bank.

## Comparison to alternatives

Non-streaming TTS achieves higher quality (access to full context) but cannot be used interactively. The quality gap has narrowed substantially: [[2507.14534]] shows streaming VC can match or exceed offline quality in speaker similarity. The primary remaining cost of streaming is in content accuracy (WER slightly higher for streaming systems due to limited right context).

## Year-on-year trajectory

2022–2024: Early streaming VC systems used zero-padding causal vocoders (quality degradation), frame-by-frame autoregressive decoders (slow), and distillation-based causal content encoders (insufficient accuracy). 2025: Conan ([[2507.14534]]) eliminates all three failure modes with Emformer distillation, chunk-level parallel generation, and Causal Shuffle Vocoder. RT-VC ([[2025.acl-demo.37]]) provides an alternative CPU-deployable articulatory-coding route. FireRedTTS-2 ([[2509.02020]]) extends streaming to multi-speaker dialogue TTS with <100 ms first-packet latency. FlexiCodec-TTS ([[2510.00981]]) demonstrates that sub-10 Hz codec tokens allow near-streaming AR deployment. VocalNet [[2025.emnlp-main.989]] and OpenS2S [[2025.emnlp-demos.70]] establish streaming speech LLM architectures with first-chunk latencies of 300–430 ms. Llasa+ [[2508.06262]] demonstrates plug-and-play MTP acceleration with token verification for 1.48x speedup without quality degradation. LLaMA-Omni 2 [[2025.acl-long.912]] achieves 582 ms latency for a 7B spoken chatbot via read/write-scheduled streaming AR TTS. 2026: Qwen3-TTS [[2601.15621]] achieves 97 ms first-packet latency with a fully causal 12.5 Hz RVQ tokenizer — the best reported streaming latency in the corpus for a production-grade multilingual system. Fish Audio S2 [[2603.08823]] achieves RTF 0.195 and TTFA <100ms via Dual-AR with streaming DAC-based codec. The bottleneck has shifted from token generation speed to acoustic vocoder latency; the gap between fully causal (100 ms) and non-streaming (<60 ms RTF) systems has largely closed.

## Open questions

- Can streaming zero-shot TTS (not just VC) achieve comparable quality to non-streaming SOTA at < 100 ms latency?
- How do streaming systems behave under network jitter (variable chunk arrival timing)?
- Is 37 ms latency (Conan fast) perceptually transparent to users in real-time communication? No user study on this threshold is reported.
- Causal vocoders still trade some quality for causality; is pixel shuffle the optimal approach, or are there better alternatives?

## Papers

| ID | Title | Venue | Year | Key use of this concept |
|----|-------|-------|------|------------------------|
| [[2507.14534]] | Conan: A Chunkwise Online Network for Zero-Shot Adaptive Voice Conversion | arXiv (ASRU 2025) | 2025 | Streaming zero-shot VC at 37–140 ms latency using Emformer content extraction and Causal Shuffle Vocoder; demonstrates streaming can match offline VC quality |
| [[2025.emnlp-main.989]] | VocalNet: Speech LLMs with Multi-Token Prediction for Faster and High-Quality Generation | EMNLP | 2025 | Chunk-based streaming attention masking for speech LLM; first-chunk latency 319–428 ms on L20 GPU; identifies flow-matching vocoder as dominant latency bottleneck |
| [[2025.emnlp-demos.70]] | OpenS2S: Advancing Fully Open-Source End-to-End Empathetic Large Speech Language Model | EMNLP | 2025 | Streaming interleaved decoding (4 LLM states → 8 speech tokens per step); chunk-aware causal flow matching + HiFi-GAN vocoder for real-time empathetic speech response generation |
| [[2025.acl-long.682]] | Recent Advances in Speech Language Models: A Survey | ACL | 2025 | Surveys full-duplex and real-time SpeechLM architectures; covers streaming tokenizers, simultaneous bidirectional communication, and systems like Moshi and LSLM |
| [[2509.02020]] | FireRedTTS-2: Towards Long Conversational Speech Generation for Podcast and Chatbot | arXiv | 2025 | Extends streaming TTS to multi-speaker dialogue via dual-transformer architecture and 12.5 Hz RVQ tokenizer; achieves <100 ms first-packet latency with sentence-by-sentence generation |
| [[2025.acl-demo.37]] | RT-VC: Real-Time Zero-Shot Voice Conversion with Speech Articulatory Coding | ACL | 2025 | Achieves 61.4 ms CPU latency via articulatory-space (SPARC EMA inverter) content extraction and causal DDSP vocoder — demonstrates that streaming VC is achievable without GPU hardware |
| [[2601.15621]] | Qwen3-TTS Technical Report | arXiv | 2026 | Fully causal 12.5 Hz 16-layer RVQ tokenizer + MTP enables 97 ms first-packet latency for production-grade multilingual streaming TTS (10 languages) |
| [[2508.06262]] | Llasa+: Free Lunch for Accelerated and Streaming Llama-Based Speech Synthesis | arXiv | 2025 | Adds frozen MTP modules + token verification to existing Llasa model for 1.48x speedup; XCodec2-S causal adapter enables streaming waveform reconstruction |
| [[2603.08823]] | Fish Audio S2 Technical Report | arXiv | 2026 | Dual-AR (slow semantic + fast acoustic) with 21 Hz streaming DAC codec; RTF 0.195, TTFA <100ms at production scale; SGLang-based serving with 86.4% prefix cache hit rate |
| [[2412.10117]] | CosyVoice 2: Scalable Streaming Speech Synthesis with Large Language Models | arXiv | 2024 | Chunk-aware causal flow matching with 4 mask types in one model; streaming LM via text-speech interleaved tokens; virtually lossless streaming on standard test sets |
| [[2025.acl-long.912]] | LLaMA-Omni 2: LLM-based Real-time Spoken Chatbot with Autoregressive Streaming Speech Synthesis | ACL | 2025 | 582 ms latency for 7B spoken chatbot via read/write-scheduled streaming TTS LM (R=3, W=10) with CosyVoice 2 chunk-aware causal FM decoder |
| [[2509.15969]] | VoXtream: Full-Stream Text-to-Speech with Extremely Low Latency | arXiv | 2025 | Fully AR streaming zero-shot TTS that begins from the first received word; GPU first-packet latency 102 ms — among the lowest reported for AR zero-shot streaming TTS; 441M model, 9K hours training |
| [[2603.18090]] | MOSS-TTS Technical Report | arXiv | 2026 | Large-scale AR TTS with causal Transformer tokenizer enabling streaming synthesis; natural language instruction control for speaking rate and style at production scale |
| [[2604.12438]] | An Ultra-Low Latency End-to-End Streaming Speech Synthesis Architecture | arXiv | 2026 | Non-autoregressive end-to-end streaming TTS using depth-wise sequential decoding over 32 Mimi RVQ layers; RTF ~0.0033 (303× real-time), 48.99 ms average first-byte latency — fastest streaming TTS in corpus |
