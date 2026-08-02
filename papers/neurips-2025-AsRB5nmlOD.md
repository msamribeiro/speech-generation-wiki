---
id: "neurips-2025-AsRB5nmlOD"
title: "SALMONN-omni: A Standalone Speech LLM without Codec Injection for Full-duplex Conversation"
authors: ["Wenyi Yu", "Siyin Wang", "Xiaoyu Yang", "Xianzhao Chen", "Xiaohai Tian", "Jun Zhang", "Guangzhi Sun", "Lu Lu", "Yuxuan Wang", "Chao Zhang"]
organization: "Tsinghua University / ByteDance / University of Cambridge"
venue: NeurIPS
venue_type: conference
year: 2025
month: 12
published_date: "2025-12-10"
ingested_date: "2026-08-02"
task: [TTS, SCA]
architecture: [hybrid]
conditioning: [text-conditioned]
training: [supervised, RLHF, fine-tuning]
model_size: "Llama-3-8B-Instruct backbone (LoRA rank 32) + Mamba streaming encoder (32 blocks, 2048-dim) + CosyVoice2-0.5B streaming synthesizer"
codec_used: "none (codec-free; interleaves LLM hidden-state embeddings directly with streaming encoder/synthesizer, no discrete audio codec tokens in the LLM vocabulary)"
datasets_train: ["LibriSpeech-960h", "GigaSpeech-M", "Alpaca-52k", "Web Questions", "TriviaQA", "SQuAD", "Natural Questions", "VoiceAssistant-400K", "UltraChat"]
datasets_eval: ["Llama Questions", "Web Questions", "TriviaQA (OpenAudioBench subset)", "AlpacaEval (VoiceBench)"]
metrics:
  - name: UTMOS
    value: 4.261
    system: proposed
    testset: "average across Llama Questions/Web Questions/TriviaQA/AlpacaEval, oracle turn-taking"
code_available: null
demo_available: true
url: "https://openreview.net/forum?id=AsRB5nmlOD"
related_concepts: [spoken-language-model, speech-to-speech, streaming-tts, rlhf-speech]
related_papers: ["2412.10117"]
field_significance:
  level: "high"
  type: [architectural-novelty]
generation:
  schema_version: 2
  date: "2026-08-02"
  runtime: "claude-code"
  provider: "anthropic"
  agent: speech-generation-ingest-agent
  model: "claude-sonnet-5"
  commit: "b6e27f4"
---

> [!abstract] NeurIPS · 2025 · Conference
> **Yu & Wang et al.** (Tsinghua University / ByteDance / University of Cambridge) · [→ Paper](https://openreview.net/forum?id=AsRB5nmlOD) · Demo: ✓ · Code: ?
>
> Introduces the first standalone full-duplex speech LLM that supports simultaneous listening and speaking without injecting discrete audio codec tokens into the LLM's vocabulary, using a single autoregressive backbone that learns dialogue-state transitions as part of its normal token generation.

## Problem

Full-duplex spoken dialogue, listening and speaking simultaneously, is essential for natural conversational behaviors like turn-taking, backchanneling, and barge-ins, but existing systems struggle to support it cleanly. Modular architectures with separate voice-activity detectors, interrupters, and state predictors suffer from error accumulation across components. Codec-injection approaches (most notably Moshi), which fold discrete audio codec tokens into a single LLM's vocabulary, avoid needing conversational-dynamics annotations but require large-scale training data to bridge the text/speech modality gap and still degrade substantially when operating on speech rather than text. A third family (VITA, Freeze-Omni, MinMo) connects an LLM to streaming speech components via embeddings without hurting text capability, but runs two interdependent LLM processes, one listening, one speaking, since a single instance cannot do both, adding computational overhead and preventing genuinely standalone full-duplex operation.

## Method

SALMONN-omni integrates a streaming speech encoder, a single LLM backbone (Llama-3-8B-Instruct with LoRA), and a streaming speech synthesizer, without injecting any discrete audio codec into the LLM's vocabulary; speech is passed as continuous hidden embeddings rather than codec tokens. The streaming encoder is a stack of Mamba state-space blocks distilled (via L1 loss) from frozen Whisper-large-v3 log-Mel features, chosen for streaming-friendly linear-time sequence modeling. The streaming synthesizer builds on CosyVoice2, using its fixed-length interleaved text/speech-token generation strategy but replacing CosyVoice2's original text input with the LLM backbone's own output embeddings via a linear projection, so text and speech generation stay coupled to the LLM's internal state rather than a separate text pipeline.

To handle full duplex within one autoregressive model, the conversation is split into an assistant stream (the model's own generated speech) and an environment stream (all incoming audio: user speech, background noise, and the assistant's own echo), which are interleaved with text tokens into a single sequence the LLM models jointly; the model also receives its own prior output speech (including echo) as input before generating its next response, to stabilize training against self-interference. A periodic synchronization mechanism processes fixed-duration time blocks (80ms) of input while generating a matching duration of output, giving the model a consistent temporal frame linking the audio and text modalities. For deciding when to start and stop speaking, the paper compares two "thinking" strategies: an implicit variant where predicted listen/speak state tokens are not fed back into the model's own input sequence, and an explicit variant using two special tokens (`<think>`, generated while listening or waiting; `<shift>`, marking a state transition) that are autoregressively fed back like any other generated token. Training proceeds in three stages: (1) connecting the streaming encoder to a frozen LLM via an MLP and LoRA, trained on ASR and QA; (2) connecting the streaming synthesizer end-to-end, trained on ASR, QA, and multi-turn dialogue including barge-in and backchanneling scenarios, with the encoder and LLM backbone still frozen; (3) reinforcement learning via DPO applied specifically to barge-in and backchanneling tasks to correct behavior that emerges after stage 2's supervised fine-tuning.

![The architecture of SALMONN-omni. Two input streams, the environment stream and the assistant stream, are processed by the streaming speech encoder. Speech embeddings from both streams, along with textual embeddings, are fed into the LLM backbone in an interleaved manner. When in the speaking state, the streaming speech synthesizer takes the textual embeddings derived from the LLM backbone as input to produce speech responses.](assets/neurips-2025-AsRB5nmlOD/figure-1.png)

## Key Results

Explicit thinking outperforms implicit thinking across most benchmarks after Stage 1, and turn-taking success rises from roughly 70% to roughly 90% once the assistant stream is introduced in Stage 2, confirming the design choices. Against other full-duplex speech LLMs under a practical "predicted turn-taking" setting (the model itself decides when to start speaking), SALMONN-omni achieves a 35.9% average relative improvement in accuracy over prior open-source full-duplex models (Moshi, Freeze-Omni) across four spoken QA/dialogue benchmarks. Under an "oracle turn-taking" setting (forced to answer at the end of the question), it remains competitive with, and often exceeds, larger turn-based systems (GLM-4-Voice, Qwen2.5-Omni, VITA-1.5, miniCPM-o, Kimi-Audio, Baichuan-Audio) despite using substantially less training data than some baselines (one uses up to 13M hours of audio), with average UTMOS of 4.261 indicating high output speech quality. On turn-taking timing prediction, SALMONN-omni achieves the highest success rate among full-duplex models (e.g., 92.8% on TriviaQA vs. 72.0% for Freeze-Omni and 37.1% for Moshi), maintaining performance on short, fast-paced inputs where baselines degrade. On distinguishing genuine barge-ins from backchanneling/false positives while hearing its own full-volume echo, SALMONN-omni reaches an F1 of 0.88, matching Moshi and far exceeding Freeze-Omni (0.17 under the same echo condition), which the paper attributes to Freeze-Omni's non-standalone, VAD-dependent design. DPO applied to barge-in/backchanneling behavior recovers precision (0.68 to 0.84-0.88) that supervised fine-tuning alone leaves too permissive, after an initial period of DPO-induced over-conservatism. In live human-machine deployment (60 real dialogue sessions), the model shows average time-to-first-audio of 405ms for normal responses but only 128-144ms for barge-in reactions.

## Novelty Assessment

The core novelty is architectural and training-procedural rather than relying on a new generative backbone: treating dialogue-state transitions as ordinary tokens an LLM learns to generate (rather than adding a separate classifier or codec-injected silence-vs-speech inference), combined with jointly modeling environment and assistant streams in one sequence, is what makes the system genuinely standalone (a single LLM instance handles both listening and speaking) where prior non-codec approaches needed two coupled LLM processes. The explicit-vs-implicit thinking ablation and the RL stage targeting dialogue dynamics specifically (rather than general response quality) are both concretely validated contributions, and the paper explicitly frames itself as the first application of RL to full-duplex dialogue-dynamics modeling.

## Field Significance

high — SALMONN-omni resolves a genuine architectural tension in full-duplex speech LLM design (codec-injection systems need large-scale data and lag on speech; embedding-based systems need two LLM processes) with a single mechanism, and validates it thoroughly: against eight strong open-source baselines, with live human-machine deployment testing (not just held-out benchmark evaluation), and with the first demonstrated use of RL for full-duplex dialogue-dynamics modeling specifically. The substantial relative gains (30%+) over prior full-duplex systems, achieved with less training data than several baselines, indicate the architectural choices are doing real work rather than being compensated for by scale.

## Claims

- **supports:** A single LLM backbone can perform standalone full-duplex spoken dialogue, modeling turn-taking and barge-ins without a separate listening process or external voice-activity-detection module, by interleaving environment-stream and assistant-stream speech embeddings with text into one autoregressive sequence.
  > *Evidence:* SALMONN-omni jointly models environment and assistant speech streams within a single Llama-3-8B backbone and achieves the highest turn-taking success rate among compared full-duplex systems (e.g., 92.8% on TriviaQA vs. 72.0% for VAD-dependent Freeze-Omni and 37.1% for Moshi), without a separate listening LLM process or VAD module. *(§3.3, Table 4)*
- **supports:** Training an LLM to generate explicit dialogue state-transition tokens that are fed back into its own input, rather than withholding them from the input sequence, produces better dialogue-timing behavior.
  > *Evidence:* The "explicit thinking" strategy, where predicted state tokens are autoregressively fed back as input, outperforms the "implicit thinking" strategy that withholds this feedback, across most evaluated datasets. *(§5.1, Table 2)*
- **supports:** Applying direct preference optimization specifically to barge-in and backchanneling scenarios can correct a full-duplex speech LLM's tendency to be too easily interrupted after supervised fine-tuning, without redesigning the underlying architecture.
  > *Evidence:* After SFT alone, contextual-independent barge-in precision is 0.68, reflecting over-readiness to be interrupted; after DPO training, overall F1 for distinguishing genuine barge-ins improves from 0.86 to 0.90, following an initial period of DPO-induced over-conservatism that later recovers. *(§5.3.2, Table 6)*
- **complicates:** A full-duplex speech LLM's real-time responsiveness is not uniform across interaction types; barge-in response latency can be substantially lower than the latency for initiating a normal spoken response, since the two are governed by different architectural constraints.
  > *Evidence:* SALMONN-omni's average time-to-first-audio for a normal spoken response is 405ms, driven by needing four time blocks of context for the synthesizer, while its response latency to barge-in interruptions is only about 128-144ms. *(§5.4)*

## Limitations and Open Questions

The authors identify the 405ms time-to-first-audio for normal responses as a consequence of the design (waiting for four time blocks before the synthesizer has enough context) rather than a fundamental limit, suggesting synthesizer optimization could reduce it, but this remains unvalidated in the paper. Turn-taking and barge-in evaluation relies substantially on synthetically constructed scenarios (GPT-4o- and Llama-3-generated interrupting sentences and context-dependent questions), which may not fully capture the diversity of real conversational interruptions, though the paper does supplement this with 60 live human-machine dialogue sessions. Training data, while smaller in scale than several baselines, remains English-only across all listed corpora, leaving multilingual full-duplex behavior unassessed.

## Wiki Connections

- [[spoken-language-model|Spoken Language Model]] — adapts an LLM (Llama-3-8B) to consume real external streaming speech input (environment stream) while simultaneously generating spoken output in a genuine full-duplex dialogue setting.
- [[speech-to-speech|Speech-to-Speech]] — implements end-to-end spoken dialogue with simultaneous listening and speaking, matching the concept's end-to-end dialogue sub-paradigm.
- [[streaming-tts|Streaming TTS]] — generates speech in real time via a streaming synthesizer coupled to fixed-duration time blocks, with reported sub-second response latency.
- [[rlhf-speech|RLHF Speech]] — applies direct preference optimization specifically to barge-in and backchanneling behavior, the first reported use of RL for full-duplex dialogue-dynamics modeling.
- [[2412.10117|CosyVoice 2]] — the streaming speech synthesizer is built directly on and fine-tuned from CosyVoice2-0.5B, replacing its text input with the LLM backbone's output embeddings.
