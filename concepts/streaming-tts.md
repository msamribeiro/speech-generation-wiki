---
slug: streaming-tts
title: Streaming TTS
aliases: [real-time TTS, low-latency TTS, incremental TTS, chunk-based synthesis, online voice conversion, streaming VC]
status: established
related_concepts: [spoken-language-model, autoregressive-codec-tts, neural-codec, voice-conversion, gan-vocoder, flow-matching]
last_updated: 2026-06-13
---

## Executive Summary

> [!abstract]
> Streaming (or real-time) TTS produces output audio incrementally as input arrives, operating causally without accessing future input frames. The key metrics are time-to-first-audio (TTFA), real-time factor (RTF < 1), and quality degradation relative to non-streaming baselines. As of 2026, streaming operation has been demonstrated across all major TTS paradigms — autoregressive codec LMs, flow-matching decoders, and streaming VC — with leading systems achieving TTFA under 100 ms. The bottleneck has shifted from token generation speed to acoustic vocoder latency, and the quality gap between streaming and non-streaming has largely closed for speaker similarity; small WER penalties from limited right context remain.

## Current Status

established — Streaming TTS is now a standard deployment requirement, not a research novelty. Multiple production systems (Qwen3-TTS [[2601.15621]], Fish Audio S2 [[2603.08823]], CosyVoice 2 [[2412.10117]]) ship streaming as a core feature. The primary remaining research challenges are sub-20 ms latency, quality under network jitter, and the vocoder latency floor. An end-to-end non-AR streaming system ([[2604.12438]]) has demonstrated RTF 0.0033 and TTFA 48.99 ms — approaching the perceptual transparency threshold.

## Why This Matters

Streaming operation is essential for real-time communication applications (teleconferencing, gaming avatars, virtual humans), interactive voice assistants, live broadcasting, and real-time speech privacy. Systems that cannot stream produce unacceptable latency for any interactive use case. As autoregressive TTS systems become more powerful (VALL-E family, CosyVoice), adapting them for streaming without quality loss is a central engineering challenge.

## Core Idea

A streaming TTS system enforces causality throughout every stage of the pipeline: the content extractor or text encoder, the acoustic model, and the vocoder must all operate on bounded input windows without accessing future frames. Two key constraints: (1) RTF < 1 — the system must process audio faster than it is played; (2) latency — from receipt of first input tokens to first audio output, measured in milliseconds.

For streaming VC, the same principles apply to the content extractor, style encoder, and vocoder. For streaming TTS from text, autoregressive models are inherently streaming at the token level; the bottleneck is the acoustic decoder and vocoder latency.

## Methods and Variants

**Chunk-based streaming.** Source audio or tokens are buffered into fixed-size chunks and processed one chunk at a time. Each output chunk is generated before the next input chunk arrives. A sliding context window provides temporal continuity across chunk boundaries.

**Causal convolutions.** Standard convolutions are replaced with left-padded causal convolutions throughout the pipeline. GAN vocoders like HiFi-GAN require full replacement of transposed convolutions to avoid spectral artifacts from zero-padding; Conan [[2507.14534]] resolves this with pixel shuffle (Causal Shuffle Vocoder) instead.

**Autoregressive streaming.** VALL-E-family models generate tokens autoregressively and are inherently streaming at the token level. The bottleneck is token generation speed (AR RTF). Lower frame rate codecs ([[2510.00981]]) reduce sequence length and thus AR RTF. FlexiCodec-TTS [[2510.00981]] demonstrates that 6.25 Hz AR frame rate achieves RTF 0.07 with competitive quality.

**Causal attention masking.** Transformer-based decoders use causal attention masks to prevent attending to future positions. Emformer [[2507.14534]] caches previous chunk keys/values and summary vectors as a memory bank for streaming. Read/write scheduling (LLaMA-Omni 2 [[2025.acl-long.912]]: R=3, W=10) controls lookahead for spoken chatbot generation.

**Multi-token prediction (MTP).** MTP modules that predict multiple future tokens per step reduce the number of AR decode steps. Qwen3-TTS [[2601.15621]] uses MTP with a fully causal 12.5 Hz RVQ tokenizer for 97 ms first-packet latency. Llasa+ [[2508.06262]] adds frozen MTP modules with token verification to an existing Llasa backbone for 1.48× speedup without quality degradation.

**Chunk-aware causal flow matching.** CosyVoice 2 [[2412.10117]] trains a single FM model with four masking patterns (offline, full-causal, chunk-M, chunk-2M), enabling simultaneous streaming/non-streaming deployment with virtually lossless streaming on standard test sets.

**Non-AR depth-wise streaming.** [[2604.12438]] applies depth-wise sequential decoding over 32 Mimi RVQ layers in a non-autoregressive architecture, achieving RTF 0.0033 (303× real-time) and 48.99 ms average first-byte latency — the fastest streaming TTS in the corpus.

**Multi-queue AR scheduling.** LLMVoX [[2025.findings-acl.1051]] runs two LLMVoX instances concurrently on sentence-partitioned queues for 475 ms end-to-end latency vs. 4200 ms for XTTS — demonstrating that pipeline parallelism at the sentence level provides large latency gains independent of model speed.

**Speculative decoding for AR TTS.** [[interspeech-2025-2447]] adapts speculative decoding from text LLMs to speech, using a lightweight draft model initialised from the upper layers of the target AR model to propose candidate speech tokens for parallel verification. A tolerance factor β relaxes strict token-distribution acceptance, exploiting the many-to-one mapping from speech token sequences to perceptual quality. At β=0.4 and draft length L=3, it achieves 1.4× AR LM speedup (RTF 0.504 → 0.36) on CosyVoice 2 with negligible degradation in naturalness and speaker similarity, and is deployable without modifying the target model.

**Non-uniform step pruning for FM TTS.** [[interspeech-2025-2449]] (EPSS) constructs non-uniform ODE time-step schedules for flow-matching TTS by pruning later, near-linear trajectory steps while densely sampling the high-curvature early phase. Applied to F5-TTS, this reduces NFE from 32 to 7 (4× RTF speedup: 0.123 → 0.030) with near-identical WER, SPK-SIM, and UTMOS on standard benchmarks, and without any retraining.

## Major Claims

Claims are generalised propositions aggregated from paper evidence.

### Strongly Supported

- Streaming zero-shot VC can match or exceed offline VC quality in speaker similarity, eliminating the quality penalty previously assumed to accompany real-time constraints.
  Supporting: [[2507.14534]]

- Lower AR frame rate codecs (6–12.5 Hz) substantially reduce streaming TTS latency without proportional quality loss, because shorter token sequences require fewer decode steps.
  Supporting: [[2510.00981]], [[2601.15621]], [[2509.02020]]

- Chunk-aware causal flow matching with multiple masking patterns can be trained once and deployed in both streaming and non-streaming modes with virtually lossless streaming quality.
  Supporting: [[2412.10117]]

### Emerging

- Depth-wise sequential decoding over RVQ layers (non-AR) can achieve an order-of-magnitude lower RTF than AR streaming TTS while maintaining competitive quality.
  Supporting: [[2604.12438]]

- Multi-token prediction modules can be added to existing frozen AR TTS models as plug-and-play acceleration without quality degradation, removing the need to retrain for streaming.
  Supporting: [[2508.06262]], [[2601.15621]]

- The primary remaining latency bottleneck in streaming TTS is the acoustic vocoder, not the token generation stage; improvements to the AR stage yield diminishing returns once RTF < 0.1.
  Supporting: [[2025.emnlp-main.989]], [[2507.14534]]

- Speculative decoding adapted to speech codec LMs with a tolerance-factor acceptance criterion achieves inference speedup without measurable subjective quality degradation, exploiting the many-to-one mapping from token sequences to perceptual quality.
  Supporting: [[interspeech-2025-2447]]

- Non-uniform ODE step scheduling that preserves the high-curvature early phase of FM trajectories enables 4× speedup from 32 to 7 NFE with near-negligible quality degradation in flow-matching TTS, as a training-free plug-in.
  Supporting: [[interspeech-2025-2449]]

### Contested

> [!warning]
> Whether 37 ms latency (Conan fast setting) is perceptually transparent in real-time communication is not established — no user study on this threshold is reported in the corpus. The perceptual threshold for streaming TTS vs. interactive conversation likely differs by application context.
> Latency reported: [[2507.14534]] / User study: not available

## Relationship to Other Concepts

### Extends or Builds On
- [[autoregressive-codec-tts]] — AR codec LMs are inherently streaming at the token level; streaming TTS engineering adapts their inference loop and vocoder for causal operation
- [[neural-codec]] — lower-frame-rate codecs directly enable streaming by reducing sequence length; causal codec decoders eliminate the waveform reconstruction bottleneck

### Commonly Paired With
- [[voice-conversion]] — streaming VC shares all design constraints with streaming TTS; many systems address both (Conan, RT-VC)
- [[spoken-language-model]] — streaming speech LLMs (LLaMA-Omni 2, VocalNet, OpenS2S) require streaming TTS as their output module
- [[flow-matching]] — chunk-aware causal flow matching (CosyVoice 2) is the dominant approach for streaming in LLM+FM hybrid systems

### Competes With
- Non-streaming (offline) TTS — achieves higher quality (full context) but cannot be used interactively; quality gap has narrowed substantially

## Representative Papers

### Foundational
- [[2412.10117]] — CosyVoice 2: introduced chunk-aware causal flow matching with four masking patterns; virtually lossless streaming — first production-grade streaming FM TTS

### Influential
- [[2507.14534]] — Conan: streaming VC exceeding offline quality; Causal Shuffle Vocoder resolves the GAN vocoder causality problem
- [[2601.15621]] — Qwen3-TTS: 97 ms first-packet latency at production scale across 10 languages; fully causal 12.5 Hz RVQ tokenizer

### Recent Highlights
- [[2604.12438]] — fastest streaming TTS in corpus (RTF 0.0033, TTFA 48.99 ms) via depth-wise sequential non-AR decoding over Mimi RVQ layers
- [[2603.08823]] — Fish Audio S2: RTF 0.195 and TTFA <100 ms via Dual-AR at production scale
- [[2508.06262]] — Llasa+: plug-and-play 1.48× speedup via frozen MTP + token verification without retraining

### Cautionary / Negative Evidence
- [[2025.emnlp-main.989]] — VocalNet identifies the flow-matching vocoder as the dominant latency bottleneck, not the AR LM, suggesting that further AR acceleration has diminishing returns at current streaming speeds

## Open Questions

- Can streaming zero-shot TTS (text-to-speech, not just VC) achieve comparable quality to non-streaming SOTA at under 100 ms latency on diverse speakers?
- How do streaming systems behave under network jitter (variable chunk arrival timing)?
- Is 37 ms latency (Conan fast) perceptually transparent to users in real-time communication? No user study on this threshold is reported.
- Causal vocoders still trade some quality for causality; is pixel shuffle the optimal approach, or are there better alternatives?
- Does depth-wise sequential non-AR decoding ([[2604.12438]]) scale to zero-shot multi-speaker synthesis, or does the RTF advantage diminish with speaker conditioning complexity?
- Dragon-FM [[2507.22746]] demonstrates that chunk-wise AR + within-chunk flow-matching is KV-cache compatible; can this design serve as the basis for a production streaming system with first-byte latency under 100 ms?
- SecoustiCodec [[2508.02849]] is designed for voice-dialogue streaming and achieves 12.08 ms initial latency; how does this compare in practice to other streaming codecs when embedded in a full TTS or VC pipeline?
- SpectroStream [[2508.05207]] demonstrates 80 ms architectural latency for stereo audio codec; is this compatible with real-time streaming requirements for music applications?
- [[interspeech-2025-2447]] reports a WER increase (3.67% → 5.70%) attributed to the draft model's limited training data; would this gap close if the draft model were trained on data matched to the target model's corpus size?
- Can EPSS-style non-uniform step pruning ([[interspeech-2025-2449]]) be combined with FM distillation to push quality-preserving inference below 5 NFE, and does the trajectory linearity assumption hold for different conditioning types?

## Trend Summary

2022–2024: Early streaming VC systems used zero-padding causal vocoders (quality degradation), frame-by-frame AR decoders (slow), and distillation-based causal content encoders (insufficient accuracy). 2025: Conan [[2507.14534]] eliminated all three failure modes. RT-VC [[2025.acl-demo.37]] demonstrated a CPU-deployable articulatory-coding route. FireRedTTS-2 [[2509.02020]] extended streaming to multi-speaker dialogue TTS with <100 ms first-packet latency. FlexiCodec-TTS [[2510.00981]] demonstrated that sub-10 Hz codec tokens allow near-streaming AR deployment. VocalNet [[2025.emnlp-main.989]] and OpenS2S [[2025.emnlp-demos.70]] established streaming speech LLM architectures with first-chunk latencies of 300–430 ms. Llasa+ [[2508.06262]] demonstrated plug-and-play MTP acceleration with 1.48× speedup without quality degradation. LLaMA-Omni 2 [[2025.acl-long.912]] achieved 582 ms latency for a 7B spoken chatbot. 2026: Qwen3-TTS [[2601.15621]] achieved 97 ms first-packet latency with a fully causal 12.5 Hz RVQ tokenizer — the best reported streaming latency for a production-grade multilingual system. Fish Audio S2 [[2603.08823]] achieved RTF 0.195 via Dual-AR with streaming DAC codec. [[2604.12438]] demonstrated RTF 0.0033 via non-AR depth-wise streaming — 303× real-time. The bottleneck has shifted from token generation speed to acoustic vocoder latency; the gap between fully causal and non-streaming systems has largely closed.

## All Papers

| ID | Title | Venue | Year | Role in this concept |
|----|-------|-------|------|---------------------|
| [[2507.14534]] | Conan: A Chunkwise Online Network for Zero-Shot Adaptive Voice Conversion | arXiv (ASRU 2025) | 2025 | Streaming zero-shot VC at 37–140 ms; Emformer content extraction + Causal Shuffle Vocoder |
| [[2025.emnlp-main.989]] | VocalNet: Speech LLMs with Multi-Token Prediction for Faster and High-Quality Generation | EMNLP | 2025 | Chunk-based streaming speech LLM; first-chunk latency 319–428 ms; FM vocoder identified as dominant bottleneck |
| [[2025.emnlp-demos.70]] | OpenS2S: Advancing Fully Open-Source End-to-End Empathetic Large Speech Language Model | EMNLP | 2025 | Streaming interleaved decoding (4 LLM states → 8 speech tokens); chunk-aware causal FM + HiFi-GAN |
| [[2025.acl-long.682]] | Recent Advances in Speech Language Models: A Survey | ACL | 2025 | Surveys full-duplex and real-time SpeechLM architectures; streaming tokenizers and simultaneous communication |
| [[2509.02020]] | FireRedTTS-2: Towards Long Conversational Speech Generation for Podcast and Chatbot | arXiv | 2025 | Streaming multi-speaker dialogue TTS; dual-transformer + 12.5 Hz RVQ; <100 ms first-packet latency |
| [[2025.acl-demo.37]] | RT-VC: Real-Time Zero-Shot Voice Conversion with Speech Articulatory Coding | ACL | 2025 | 61.4 ms CPU latency; articulatory-space content extraction + causal DDSP vocoder |
| [[2601.15621]] | Qwen3-TTS Technical Report | arXiv | 2026 | Fully causal 12.5 Hz 16-layer RVQ + MTP; 97 ms first-packet latency; 10-language production |
| [[2508.06262]] | Llasa+: Free Lunch for Accelerated and Streaming Llama-Based Speech Synthesis | arXiv | 2025 | Frozen MTP + token verification; 1.48× speedup; XCodec2-S causal adapter for streaming |
| [[2603.08823]] | Fish Audio S2 Technical Report | arXiv | 2026 | Dual-AR (slow semantic + fast acoustic); RTF 0.195, TTFA <100 ms; SGLang serving at production scale |
| [[2412.10117]] | CosyVoice 2: Scalable Streaming Speech Synthesis with Large Language Models | arXiv | 2024 | Chunk-aware causal FM with 4 mask types; streaming LM via text-speech interleaved tokens; virtually lossless streaming |
| [[2025.acl-long.912]] | LLaMA-Omni 2: LLM-based Real-time Spoken Chatbot with Autoregressive Streaming Speech Synthesis | ACL | 2025 | 582 ms latency for 7B spoken chatbot; read/write-scheduled streaming TTS LM (R=3, W=10) |
| [[2509.15969]] | VoXtream: Full-Stream Text-to-Speech with Extremely Low Latency | arXiv | 2025 | Fully AR streaming zero-shot TTS; 102 ms GPU first-packet latency — among lowest for AR zero-shot streaming TTS |
| [[2603.18090]] | MOSS-TTS Technical Report | arXiv | 2026 | Large-scale AR TTS with causal Transformer tokenizer; streaming synthesis at production scale |
| [[2604.12438]] | An Ultra-Low Latency End-to-End Streaming Speech Synthesis Architecture | arXiv | 2026 | Depth-wise sequential decoding over 32 Mimi RVQ layers; RTF 0.0033 (303× real-time), TTFA 48.99 ms — fastest in corpus |
| [[2507.22746]] | Next Tokens Denoising for Speech Synthesis (Dragon-FM) | arXiv | 2025 | Causal FSQ codec decoder (12.5 Hz) with chunk-wise AR generation; within-chunk FM denoising at 2 NFE; KV-cache compatible streaming design |
| [[2508.02849]] | SecoustiCodec | arXiv | 2025 | Fully causal single-codebook streaming codec (20 tokens/s); initial latency 12.08 ms; encoder RTF 0.002; designed for voice-dialogue downstream tasks |
| [[2508.05207]] | SpectroStream | arXiv | 2025 | Causal 2D-TF neural codec for 48 kHz stereo general audio; 80 ms architectural latency (1-embedding look-ahead in decoder); real-time CPU inference via KWS streaming framework |
| [[2025.findings-acl.1051]] | LLMVoX: Autoregressive Streaming Text-to-Speech Model for Any LLM | ACL | 2025 | Multi-queue scheduler with two concurrent LLMVoX instances; 475 ms vs. 4200 ms for XTTS; plug-and-play with any LLM |
| [[2510.00981]] | FlexiCodec-TTS: Flexible Frame-Rate Neural Codec for Efficient Streaming TTS | arXiv | 2025 | 6.25 Hz AR frame rate; 7.3× AR speedup (RTF 0.07); competitive quality at very low token rate |
| [[interspeech-2025-2447]] | Accelerating Autoregressive Speech Synthesis Inference With Speech Speculative Decoding | Interspeech | 2025 | Speculative decoding with tolerance-factor acceptance; 1.4× AR LM speedup (RTF 0.504→0.36) on CosyVoice 2 without modifying the target model |
| [[interspeech-2025-2449]] | Accelerating Flow-Matching-Based Text-to-Speech via Empirically Pruned Step Sampling | Interspeech | 2025 | Non-uniform ODE step pruning (32→7 NFE); 4× RTF speedup (0.123→0.030) for F5-TTS; training-free, immediately deployable |
| [[2508.07375]] | TurnGuide | arXiv | 2025 | Turn-level text-speech interleaving in full-duplex SLM; 1.05s first-package latency; streaming design limited by vocoder cost |
| [[interspeech-2025-1289]] | Unlocking Temporal Flexibility: Neural Speech Codec with Variable Frame Rate | Interspeech | 2025 | Entropy-based VFR reduces token sequence length; shorter sequences benefit streaming AR TTS systems directly |
| [[2408.16725]] | Mini-Omni: Language Models Can Hear, Talk While Thinking in Streaming | arXiv | 2024 | Simultaneous text and audio streaming generation from a single AR model; batch-based parallel generation strategy for streaming speech without sequential decode bottleneck |
| [[2409.06666]] | LLaMA-Omni: Seamless Speech Interaction with Large Language Models | arXiv | 2024 | CTC-based non-autoregressive speech decoding from LLM hidden states enables streaming synthesis robust to chunk size variation; demonstrates that CTC decoding suits streaming better than AR for speech rate naturalness |
| [[2411.00774]] | Freeze-Omni: A Smart and Low Latency Speech-to-speech Dialogue Model with Frozen LLM | arXiv | 2024 | Chunk-level state classification integrated into LLM prefill enables duplex interruption detection without a separate streaming component; three-stage training for low-latency speech I/O |
| [[2502.11946]] | Step-Audio: Unified Understanding and Generation in Intelligent Speech Interaction | arXiv | 2025 | Speculative response generation reduces per-response latency 500ms in streaming spoken dialogue; 40% of pre-generated responses usable in real-time |
| [[2402.08093]] | BASE TTS: Lessons from building a billion-parameter TTS model on 100K hours | arXiv | 2024 | Streamable convolutional decoder matches diffusion decoder in naturalness at 3× lower compute; demonstrates streaming TTS at billion-parameter scale |
