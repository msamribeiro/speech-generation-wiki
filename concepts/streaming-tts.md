---
slug: streaming-tts
aliases: [incremental TTS, real-time speech generation, full-duplex speech]
status: established
last_reviewed: 2026-08-02
concept: streaming-tts
render_type: overview
title: "Streaming Text-to-Speech"
source_digest_date: 2026-07-30
paper_count: 54
generation:
  schema_version: 2
  date: "2026-08-02"
  stage: render
  mode: full
  runtime: codex
  provider: openai
  agent: speech-generation-render-agent
  model: "gpt-5"
  commit: "cbed7c1"
---

> [!abstract]
> Streaming TTS produces speech incrementally to reduce response delay and support interactive systems. Causal components, compact tokens, parallel decoding, and duplex training make low latency practical, while chunking, lookahead, reasoning, turn-taking, and inconsistent measurement remain central constraints.

## Current State

**Incremental acoustic or speech-token generation can substantially reduce first-packet and response latency.** This is strongly supported within the reviewed graph; representative evidence includes [[2402.08093|BASE TTS: Lessons from building a billion-parameter Text-to-Speech model on 100K hours of data]], [[2409.06666|LLaMA-Omni: Seamless Speech Interaction with Large Language Models]]. Reported latency depends on hardware, buffering policy, network assumptions, and whether upstream processing is included.

**Chunk size and read–write policy create a direct trade-off among latency, alignment, and acoustic quality.** This is emerging within the reviewed graph; representative evidence includes [[2505.02625|LLaMA-Omni2: LLM-based Real-time Spoken Chatbot with Autoregressive Streaming Speech Synthesis]], [[2025.findings-acl.1051|LLMVoX: Autoregressive Streaming Text-to-Speech Model for Any LLM]]. Optimal chunks are architecture-, language-, and metric-dependent.

**Causal constraints and limited lookahead reduce delay but can degrade quality unless the model is adapted explicitly.** This is strongly supported within the reviewed graph; representative evidence includes [[2411.18803|TS3-Codec: Transformer-Based Simple Streaming Single Codec]], [[2025.acl-demo.37|RT-VC: Real-Time Zero-Shot Voice Conversion with Speech Articulatory Coding]]. Pitch, phase, and boundary artifacts may respond differently to the same lookahead budget.

**Teacher distillation can recover quality lost when codecs, acoustic decoders, or vocoders are made causal.** This is strongly supported within the reviewed graph; representative evidence includes [[2411.18803|TS3-Codec: Transformer-Based Simple Streaming Single Codec]], [[2509.13667|A Distilled Low-Latency Neural Vocoder with Explicit Amplitude and Phase Prediction]]. The benefit may depend on access to a strong non-causal teacher and matched training data.

**Interleaving or jointly generating text and speech tokens enables streaming output while retaining semantic guidance.** This is strongly supported within the reviewed graph; representative evidence includes [[2408.16725|Mini-Omni: Language Models Can Hear, Talk While Thinking in Streaming]], [[2409.06666|LLaMA-Omni: Seamless Speech Interaction with Large Language Models]]. Interleaved decoding can inherit exposure errors from both text and acoustic token streams.

**Parallel, multi-token, and speculative decoding reduce serial speech-token generation cost.** This is strongly supported within the reviewed graph; representative evidence includes [[2408.16725|Mini-Omni: Language Models Can Hear, Talk While Thinking in Streaming]], [[2502.11946|Step-Audio: Unified Understanding and Generation in Intelligent Speech Interaction]]. Speedups can shrink when verification, codec decoding, or small-batch deployment is included.

## Method Landscape

- **Autoregressive streaming generation.** 32 memberships demonstrate a shared practical pattern, not matched superiority.
- **Hybrid and modular streaming systems.** 22 memberships demonstrate a shared practical pattern, not matched superiority.
- **Flow-matching streaming decoders.** 10 memberships demonstrate a shared practical pattern, not matched superiority.
- **GAN-based causal streaming vocoders.** 8 memberships demonstrate a shared practical pattern, not matched superiority.
- **VAE-based compressed streaming latents.** 2 memberships demonstrate a shared practical pattern, not matched superiority.

## Key Trade-offs

- **Low-frame-rate and compact speech tokenizers shorten sequences and improve streaming efficiency.** Aggressive token compression can discard speaker, prosodic, or transient detail.
- **Shared models can support both streaming and offline speech generation through masks, chunk schedules, or mode conditioning.** Joint training may underperform separately optimized models at either extreme.
- **Full-duplex speech systems require generation to incorporate an incoming audio stream while speaking.** Many systems demonstrate interruption handling rather than unrestricted simultaneous dialogue.
- **Natural streaming dialogue requires explicit modeling of turn boundaries, interruptions, barge-in, and response timing.** Turn-taking benchmarks often use constrained speakers, prompts, or synthetic overlap.

## Open Questions

- Which latency definition and measurement boundary should be standard for streaming speech generation?
- How small can chunks and lookahead become before intelligibility, prosody, or speaker identity fails?
- Can native speech systems match text-mediated reasoning while preserving paralinguistic information?
- What training data and objectives support genuine simultaneous listening, speaking, interruption, and recovery?
- How should token rate, codebook structure, and parallel decoding be co-designed for interactive generation?

## Go Deeper

Read [[concepts/streaming-tts-in-depth|Streaming Text-to-Speech: In Depth]] for detailed evidence, method relationships, disagreements, implications, and a representative reading path.

## Scope

This synthesis covers 54 papers through Q3 2025. Evidence spans multiple architectures, datasets, tasks, and evaluation protocols. Strong status applies within the encoded graph; shared lineages, infrastructure evidence, emerging findings, and unmatched speed or quality comparisons retain narrower interpretations.

---

_This page is generated from `wiki/_claims/streaming-tts.yaml` (digest date: 2026-07-30). For complete structured claims and paper-level provenance, use the source YAML._
