---
concept: streaming-tts
render_type: in-depth
title: "Streaming Text-to-Speech: In Depth"
source_digest_date: 2026-09-26
paper_count: 75
generation:
  schema_version: 2
  date: "2026-11-07"
  stage: render
  mode: full
  runtime: codex
  provider: openai
  agent: speech-generation-render-agent
  model: "gpt-5"
  commit: "0e1aff0"
---

For the concise view, see [[concepts/streaming-tts|Streaming Text-to-Speech]]. Complete provenance remains in `wiki/_claims/streaming-tts.yaml`.

## Findings at a Glance

- Incremental acoustic or speech-token generation can substantially reduce first-packet and response latency. (strongly supported, high confidence.)
- Chunk size and read–write policy create a direct trade-off among latency, alignment, and acoustic quality. (strongly supported, high confidence.)
- Causal constraints and limited lookahead reduce delay but can degrade quality unless the model is adapted explicitly. (strongly supported, high confidence.)
- Teacher distillation can recover quality lost when codecs, acoustic decoders, or vocoders are made causal. (strongly supported, high confidence.)
- Interleaving or jointly generating text and speech tokens enables streaming output while retaining semantic guidance. (strongly supported, high confidence.)
- Parallel, multi-token, and speculative decoding reduce serial speech-token generation cost. (strongly supported, high confidence.)
- Low-frame-rate and compact speech tokenizers shorten sequences and improve streaming efficiency. (strongly supported, high confidence.)
- Shared models can support both streaming and offline speech generation through masks, chunk schedules, or mode conditioning. (strongly supported, high confidence.)
- Full-duplex speech systems require generation to incorporate an incoming audio stream while speaking. (strongly supported, high confidence.)
- Natural streaming dialogue requires explicit modeling of turn boundaries, interruptions, barge-in, and response timing. (strongly supported, high confidence.)
- Compact, preemptible reasoning during the listening window can improve multi-step responses without adding equivalent post-turn delay. (strongly supported, high confidence.)
- The dominant latency bottleneck can move among language-model inference, ASR, acoustic decoding, vocoding, retrieval, and orchestration. (strongly supported, high confidence.)

## Scope

The graph contains 75 papers through Q4 2025 and 19 synthesized clusters. All 19 are strongly supported with high confidence, and none is contested. Direct experiments are separated from infrastructure, history, surveys, and downstream adoption. Numeric outcomes are not pooled across unmatched hardware, data, decoders, prompts, or perceptual protocols.

## Research Landscape

**Autoregressive streaming generation** groups 46 memberships around a shared mechanism. Overlap is expected when systems combine conditioning, alignment, representation, and generation choices. Family size indicates adoption in this graph rather than a quality ranking.

**Hybrid and modular streaming systems** groups 33 memberships around a shared mechanism. Overlap is expected when systems combine conditioning, alignment, representation, and generation choices. Family size indicates adoption in this graph rather than a quality ranking.

**Flow-matching streaming decoders** groups 13 memberships around a shared mechanism. Overlap is expected when systems combine conditioning, alignment, representation, and generation choices. Family size indicates adoption in this graph rather than a quality ranking.

**GAN-based causal streaming vocoders** groups 13 memberships around a shared mechanism. Overlap is expected when systems combine conditioning, alignment, representation, and generation choices. Family size indicates adoption in this graph rather than a quality ranking.

**VAE-based compressed streaming latents** groups 5 memberships around a shared mechanism. Overlap is expected when systems combine conditioning, alignment, representation, and generation choices. Family size indicates adoption in this graph rather than a quality ranking.

## What the Research Shows

### Evidence theme 1

**Current assessment:** Incremental acoustic or speech-token generation can substantially reduce first-packet and response latency. The graph rates this conclusion **strongly supported** with high confidence. Representative support comes from [[2402.08093|BASE TTS: Lessons from building a billion-parameter Text-to-Speech model on 100K hours of data]], [[2409.06666|LLaMA-Omni: Seamless Speech Interaction with Large Language Models]]. Reported latency depends on hardware, buffering policy, network assumptions, and whether upstream processing is included. The design consequence is to test this mechanism under the intended inputs, speakers, languages, hardware, and operating point. Confidence would change most through matched independent comparisons that preserve data, compute, representation, and perceptual protocol.

**Current assessment:** Chunk size and read–write policy create a direct trade-off among latency, alignment, and acoustic quality. The graph rates this conclusion **strongly supported** with high confidence. Representative support comes from [[2505.02625|LLaMA-Omni2: LLM-based Real-time Spoken Chatbot with Autoregressive Streaming Speech Synthesis]], [[2025.findings-acl.1051|LLMVoX: Autoregressive Streaming Text-to-Speech Model for Any LLM]]. Optimal chunks are architecture-, language-, and metric-dependent. The design consequence is to test this mechanism under the intended inputs, speakers, languages, hardware, and operating point. Confidence would change most through matched independent comparisons that preserve data, compute, representation, and perceptual protocol.

**Current assessment:** Causal constraints and limited lookahead reduce delay but can degrade quality unless the model is adapted explicitly. The graph rates this conclusion **strongly supported** with high confidence. Representative support comes from [[2411.18803|TS3-Codec: Transformer-Based Simple Streaming Single Codec]], [[2025.acl-demo.37|RT-VC: Real-Time Zero-Shot Voice Conversion with Speech Articulatory Coding]]. Pitch, phase, and boundary artifacts may respond differently to the same lookahead budget. The design consequence is to test this mechanism under the intended inputs, speakers, languages, hardware, and operating point. Confidence would change most through matched independent comparisons that preserve data, compute, representation, and perceptual protocol.

**Current assessment:** Teacher distillation can recover quality lost when codecs, acoustic decoders, or vocoders are made causal. The graph rates this conclusion **strongly supported** with high confidence. Representative support comes from [[2411.18803|TS3-Codec: Transformer-Based Simple Streaming Single Codec]], [[2509.13667|A Distilled Low-Latency Neural Vocoder with Explicit Amplitude and Phase Prediction]]. The benefit may depend on access to a strong non-causal teacher and matched training data. The design consequence is to test this mechanism under the intended inputs, speakers, languages, hardware, and operating point. Confidence would change most through matched independent comparisons that preserve data, compute, representation, and perceptual protocol.

### Evidence theme 2

**Current assessment:** Interleaving or jointly generating text and speech tokens enables streaming output while retaining semantic guidance. The graph rates this conclusion **strongly supported** with high confidence. Representative support comes from [[2408.16725|Mini-Omni: Language Models Can Hear, Talk While Thinking in Streaming]], [[2409.06666|LLaMA-Omni: Seamless Speech Interaction with Large Language Models]]. Interleaved decoding can inherit exposure errors from both text and acoustic token streams. The design consequence is to test this mechanism under the intended inputs, speakers, languages, hardware, and operating point. Confidence would change most through matched independent comparisons that preserve data, compute, representation, and perceptual protocol.

**Current assessment:** Parallel, multi-token, and speculative decoding reduce serial speech-token generation cost. The graph rates this conclusion **strongly supported** with high confidence. Representative support comes from [[2408.16725|Mini-Omni: Language Models Can Hear, Talk While Thinking in Streaming]], [[2502.11946|Step-Audio: Unified Understanding and Generation in Intelligent Speech Interaction]]. Speedups can shrink when verification, codec decoding, or small-batch deployment is included. The design consequence is to test this mechanism under the intended inputs, speakers, languages, hardware, and operating point. Confidence would change most through matched independent comparisons that preserve data, compute, representation, and perceptual protocol.

**Current assessment:** Low-frame-rate and compact speech tokenizers shorten sequences and improve streaming efficiency. The graph rates this conclusion **strongly supported** with high confidence. Representative support comes from [[2408.16725|Mini-Omni: Language Models Can Hear, Talk While Thinking in Streaming]], [[2411.18803|TS3-Codec: Transformer-Based Simple Streaming Single Codec]]. Aggressive token compression can discard speaker, prosodic, or transient detail. The design consequence is to test this mechanism under the intended inputs, speakers, languages, hardware, and operating point. Confidence would change most through matched independent comparisons that preserve data, compute, representation, and perceptual protocol.

**Current assessment:** Shared models can support both streaming and offline speech generation through masks, chunk schedules, or mode conditioning. The graph rates this conclusion **strongly supported** with high confidence. Representative support comes from [[2412.10117|CosyVoice 2: Scalable Streaming Speech Synthesis with Large Language Models]], [[2509.14784|MELA-TTS: Joint transformer-diffusion model with representation alignment for speech synthesis]]. Joint training may underperform separately optimized models at either extreme. The design consequence is to test this mechanism under the intended inputs, speakers, languages, hardware, and operating point. Confidence would change most through matched independent comparisons that preserve data, compute, representation, and perceptual protocol.

### Evidence theme 3

**Current assessment:** Full-duplex speech systems require generation to incorporate an incoming audio stream while speaking. The graph rates this conclusion **strongly supported** with high confidence. Representative support comes from [[2408.02622|Language Model Can Listen While Speaking]], [[2410.00037|Moshi: a speech-text foundation model for real-time dialogue]]. Many systems demonstrate interruption handling rather than unrestricted simultaneous dialogue. The design consequence is to test this mechanism under the intended inputs, speakers, languages, hardware, and operating point. Confidence would change most through matched independent comparisons that preserve data, compute, representation, and perceptual protocol.

**Current assessment:** Natural streaming dialogue requires explicit modeling of turn boundaries, interruptions, barge-in, and response timing. The graph rates this conclusion **strongly supported** with high confidence. Representative support comes from [[2408.02622|Language Model Can Listen While Speaking]], [[2410.00037|Moshi: a speech-text foundation model for real-time dialogue]]. Turn-taking benchmarks often use constrained speakers, prompts, or synthetic overlap. The design consequence is to test this mechanism under the intended inputs, speakers, languages, hardware, and operating point. Confidence would change most through matched independent comparisons that preserve data, compute, representation, and perceptual protocol.

**Current assessment:** Staged curricula, aligned duplex data, and joint component adaptation improve full-duplex training. The graph rates this conclusion **strongly supported** with high confidence. Representative support comes from [[2408.02622|Language Model Can Listen While Speaking]], [[2408.16725|Mini-Omni: Language Models Can Hear, Talk While Thinking in Streaming]]. Synthetic or weakly aligned interaction data may encode unrealistic timing patterns. The design consequence is to test this mechanism under the intended inputs, speakers, languages, hardware, and operating point. Confidence would change most through matched independent comparisons that preserve data, compute, representation, and perceptual protocol.

**Current assessment:** Modular, cascaded, and end-to-end streaming systems make different trade-offs among latency, semantics, controllability, and paralinguistic fidelity. The graph rates this conclusion **strongly supported** with high confidence. Representative support comes from [[2408.16725|Mini-Omni: Language Models Can Hear, Talk While Thinking in Streaming]], [[2409.06666|LLaMA-Omni: Seamless Speech Interaction with Large Language Models]]. Comparisons frequently confound architecture with model scale, data, and proprietary components. The design consequence is to test this mechanism under the intended inputs, speakers, languages, hardware, and operating point. Confidence would change most through matched independent comparisons that preserve data, compute, representation, and perceptual protocol.

### Evidence theme 4

**Current assessment:** End-to-end speech systems often lag text-mediated or cascaded systems in reasoning and semantic reliability. The graph rates this conclusion **strongly supported** with high confidence. Representative support comes from [[2408.16725|Mini-Omni: Language Models Can Hear, Talk While Thinking in Streaming]], [[2509.26542|Voice Evaluation of Reasoning Ability: Diagnosing the Modality-Induced Performance Gap]]. Reasoning results are sensitive to backbone scale and the amount of speech instruction data. The design consequence is to test this mechanism under the intended inputs, speakers, languages, hardware, and operating point. Confidence would change most through matched independent comparisons that preserve data, compute, representation, and perceptual protocol.

**Current assessment:** Causal codecs and vocoders can approach offline reconstruction quality with suitable architecture and training. The graph rates this conclusion **strongly supported** with high confidence. Representative support comes from [[2025.acl-demo.37|RT-VC: Real-Time Zero-Shot Voice Conversion with Speech Articulatory Coding]], [[2508.06262|Llasa+: Free Lunch for Accelerated and Streaming Llama-Based Speech Synthesis]]. Near-offline averages can conceal boundary, pitch, and long-context failures. The design consequence is to test this mechanism under the intended inputs, speakers, languages, hardware, and operating point. Confidence would change most through matched independent comparisons that preserve data, compute, representation, and perceptual protocol.

**Current assessment:** Flow and diffusion streaming decoders trade integration steps, caching, and algorithmic delay against generation quality. The graph rates this conclusion **strongly supported** with high confidence. Representative support comes from [[2402.08093|BASE TTS: Lessons from building a billion-parameter Text-to-Speech model on 100K hours of data]], [[2408.16725|Mini-Omni: Language Models Can Hear, Talk While Thinking in Streaming]]. Nominal step counts do not directly determine end-to-end latency across implementations. The design consequence is to test this mechanism under the intended inputs, speakers, languages, hardware, and operating point. Confidence would change most through matched independent comparisons that preserve data, compute, representation, and perceptual protocol.

**Current assessment:** Streaming evaluation should measure first-packet delay, interaction timing, chunk throughput, and cache behavior in addition to batch real-time factor. The graph rates this conclusion **strongly supported** with high confidence. Representative support comes from [[2509.02020|FireRedTTS-2: Towards Long Conversational Speech Generation for Podcast and Chatbot]], [[2509.17765|Qwen3-Omni Technical Report]]. Latency definitions and measurement boundaries remain inconsistent across papers. The design consequence is to test this mechanism under the intended inputs, speakers, languages, hardware, and operating point. Confidence would change most through matched independent comparisons that preserve data, compute, representation, and perceptual protocol.

### Evidence theme 5

**Current assessment:** Multilingual, noisy, unseen-speaker, and cross-domain streaming performance remains uneven. The graph rates this conclusion **strongly supported** with high confidence. Representative support comes from [[2408.02622|Language Model Can Listen While Speaking]], [[2408.16725|Mini-Omni: Language Models Can Hear, Talk While Thinking in Streaming]]. Coverage is sparse outside high-resource languages and read or assistant-style speech. The design consequence is to test this mechanism under the intended inputs, speakers, languages, hardware, and operating point. Confidence would change most through matched independent comparisons that preserve data, compute, representation, and perceptual protocol.

### Concurrent work shifts bottlenecks

Q4 systems treat the listening interval as useful compute rather than dead time. [[2510.02066|Chain-of-Thought Reasoning in Streaming Full-Duplex Spoken Dialogue]] and related concurrent designs perform compact, interruptible reasoning while audio is still arriving. The evidence supports better multi-step responses without simply moving all reasoning after end of turn, but the gain depends on block size, synchronization, and whether the task actually benefits from explicit reasoning. This is a scheduling result as much as a model result: useful computation is overlapped with unavoidable listening time.

That overlap changes where optimization pays off. [[2512.18706|X-Talk]] and [[2510.02044|Stream RAG]] illustrate systems in which language-model inference, retrieval, ASR, acoustic decoding, vocoding, or orchestration can dominate under different workloads. A fast speech decoder does not ensure a responsive agent if retrieval or the language model owns the critical path. End-to-end reporting therefore needs component traces as well as a single latency number, including first-packet delay, steady-state chunk throughput, interruption response, and tool or retrieval overhead.

## Where Findings Disagree

The Q4 graph contains no formally contested cluster. Strong support does not imply a universal operating point: optimal chunk size and read–write policy remain architecture-, language-, hardware-, and metric-dependent.

Shared streaming/offline models are feasible, but joint training can still underperform separately optimized models at either extreme. The open issue is the size of that compromise, not whether unified models can work.

The end-to-end reasoning gap is strongly supported, while its magnitude remains sensitive to backbone scale, speech instruction data, and whether concurrent reasoning is allowed during listening.

Multi-dimensional streaming evaluation is strongly supported, but latency definitions and measurement boundaries remain inconsistent enough to block clean system rankings.

## How the Field Is Changing

Streaming research is moving from component-level causal TTS toward full speech-in/speech-out interactive systems.

Autoregressive systems increasingly interleave semantic and acoustic streams or generate multiple codec tokens in parallel.

Flow-matching decoders are adopting causal masks, caching, distillation, and few-step inference for bounded-latency synthesis.

Full-duplex work increasingly separates interruption detection, turn timing, and semantic response generation as distinct capabilities.

Evaluation is broadening from batch real-time factor toward first-packet latency, chunk behavior, and interactive timing.

Q4 pushes this evolution toward concurrent system scheduling: reasoning can occur while listening, and the critical latency path can move between model inference, retrieval, speech recognition, acoustic generation, and orchestration. Streaming is therefore becoming an end-to-end systems problem rather than a decoder-only property.

## Implications

- Which latency definition and measurement boundary should be standard for streaming speech generation; design matched evaluation that can resolve this choice.
- How small can chunks and lookahead become before intelligibility, prosody, or speaker identity fails; design matched evaluation that can resolve this choice.
- Can native speech systems match text-mediated reasoning while preserving paralinguistic information; design matched evaluation that can resolve this choice.
- What training data and objectives support genuine simultaneous listening, speaking, interruption, and recovery; design matched evaluation that can resolve this choice.
- How should token rate, codebook structure, and parallel decoding be co-designed for interactive generation; design matched evaluation that can resolve this choice.
- Which architectures provide the best quality–latency–energy trade-off across multilingual and noisy deployment; design matched evaluation that can resolve this choice.

## Representative Reading Path

1. **Investigate incremental generation reduces first packet latency.** Read [[2402.08093|BASE TTS: Lessons from building a billion-parameter Text-to-Speech model on 100K hours of data]], [[2409.06666|LLaMA-Omni: Seamless Speech Interaction with Large Language Models]]; use the encoded caveat to frame replication and deployment decisions.
2. **Investigate chunk size controls latency quality tradeoff.** Read [[2505.02625|LLaMA-Omni2: LLM-based Real-time Spoken Chatbot with Autoregressive Streaming Speech Synthesis]], [[2025.findings-acl.1051|LLMVoX: Autoregressive Streaming Text-to-Speech Model for Any LLM]]; use the encoded caveat to frame replication and deployment decisions.
3. **Investigate causality and lookahead trade latency for quality.** Read [[2411.18803|TS3-Codec: Transformer-Based Simple Streaming Single Codec]], [[2025.acl-demo.37|RT-VC: Real-Time Zero-Shot Voice Conversion with Speech Articulatory Coding]]; use the encoded caveat to frame replication and deployment decisions.
4. **Investigate distillation recovers causal decoder quality.** Read [[2411.18803|TS3-Codec: Transformer-Based Simple Streaming Single Codec]], [[2509.13667|A Distilled Low-Latency Neural Vocoder with Explicit Amplitude and Phase Prediction]]; use the encoded caveat to frame replication and deployment decisions.
5. **Investigate interleaved text speech generation supports streaming.** Read [[2408.16725|Mini-Omni: Language Models Can Hear, Talk While Thinking in Streaming]], [[2409.06666|LLaMA-Omni: Seamless Speech Interaction with Large Language Models]]; use the encoded caveat to frame replication and deployment decisions.
6. **Investigate parallel and speculative decoding accelerate streaming.** Read [[2408.16725|Mini-Omni: Language Models Can Hear, Talk While Thinking in Streaming]], [[2502.11946|Step-Audio: Unified Understanding and Generation in Intelligent Speech Interaction]]; use the encoded caveat to frame replication and deployment decisions.
7. **Investigate Q4 concurrent reasoning and pipeline latency.** Read [[2510.02066|Chain-of-Thought Reasoning in Streaming Full-Duplex Spoken Dialogue]], [[2512.18706|X-Talk]], and [[2510.02044|Stream RAG]]; trace when reasoning occurs and which component controls end-to-end response time.

## Structured Source

Complete structured claims and provenance are in [`wiki/_claims/streaming-tts.yaml`](../_claims/streaming-tts.yaml).

---

_This page is generated from `wiki/_claims/streaming-tts.yaml` (digest date: 2026-09-26)._
