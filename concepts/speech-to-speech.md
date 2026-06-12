---
slug: speech-to-speech
title: Speech-to-Speech Systems
aliases: [S2S, end-to-end spoken dialogue, direct speech translation, speech-in speech-out, voice-to-voice]
related_concepts: [spoken-language-model, neural-codec, self-supervised-speech, streaming-tts, voice-conversion]
last_updated: 2026-06-12
status: emerging
---

## Executive Summary

> [!abstract]
> Speech-to-speech (S2S) systems take speech as input and produce speech as output without passing through an explicit text representation, preserving paralinguistic information lost in cascade ASR→LM→TTS pipelines. End-to-end S2S dialogue is emerging rapidly, with Moshi, LLaMA-omni, and GLM-4-Voice establishing real-time full- and half-duplex spoken dialogue in 2024. The key open challenge is the Factor C problem — paralinguistic variability in speech tokens that causes E2E systems to lag behind cascade on semantic coherence.

## Current Status

emerging — End-to-end S2S dialogue is a fast-moving frontier, catalysed by landmark 2024 systems (Moshi, LLaMA-omni, GLM-4-Voice) that demonstrated real-time spoken dialogue for the first time. Corpus coverage is still building: most landmark systems are citation discovery candidates not yet ingested. The 2025 literature focuses on diagnosing fundamental bottlenecks (Factor C), building enabling infrastructure (low-frame-rate codecs, interaction corpora), and streaming building blocks.

## Why This Matters

The cascade approach introduces compounding errors, irreversible information loss (paralinguistics discarded by ASR), and latency from three sequential modules. End-to-end S2S eliminates these: a single model maintains a continuous speech-token context window, can generate overlap and backchannels naturally, and can process and emit audio in real time (full-duplex). [[2412.17048]] identifies that making speech LMs work well is harder than text LMs — paralinguistic variability in speech tokens (Factor C) is the main bottleneck — but this is a fundamental research problem rather than a reason to avoid the paradigm.

For TTS research, S2S systems are relevant because they require the same low-frame-rate, semantically rich codec representations as TTS autoregressive models ([[2510.00981]]), and the same streaming decoding infrastructure as streaming TTS ([[2507.14534]]). The dataset gap is also relevant: [[2025.findings-emnlp.424]] (InteractSpeech) was created specifically because no open corpus covered fine-grained interaction annotations needed for S2S dialogue training.

## Core Idea

Speech-to-speech (S2S) systems take speech as input and produce speech as output, without necessarily passing through an explicit text representation. This distinguishes them from standard TTS (text in, speech out) and from cascade dialogue systems (speech → ASR → LM → TTS). The defining characteristic is that the audio signal is the primary carrier of both the input signal and the output, preserving paralinguistic information — tone, emotion, speaking rate, accents — that would be lost in an ASR/TTS pipeline.

Three main sub-paradigms:

1. **End-to-end spoken dialogue** — a speech language model ingests speech tokens directly and autoregressively generates speech tokens in response (Moshi, LLaMA-omni, Mini-Omni, GLM-4-Voice). No ASR or TTS module required; the LM operates entirely in speech-token space.
2. **Direct S2S translation** — speech in a source language is converted to speech in a target language without intermediate text (SeamlessM4T, Translatotron). Targets low latency and preserves prosody across language boundaries.
3. **Cascade S2S** — ASR → text LM → TTS, connected in a pipeline. Still the production default for most voice assistants; included here for comparison because the same benchmark tasks (latency, naturalness, speaker consistency) apply to all three paradigms.

## Methods and Variants

**Full-duplex E2E dialogue (Moshi family).** A single transformer processes a continuous interleaved stream of user and system speech tokens, enabling the system to listen and speak simultaneously. Requires a codec at sub-10 Hz to make the AR model tractable ([[2510.00981]]). The "inner monologue" trick (generating a text chain-of-thought token stream in parallel with speech tokens) is used to maintain semantic coherence, partially addressing the Factor C problem from [[2412.17048]].

**Half-duplex E2E dialogue (LLaMA-omni, GLM-4-Voice).** Speech input is encoded once per turn; response speech tokens are generated autoregressively. Simpler architecture but requires explicit turn-taking logic. Less natural for interruptions and backchannels.

**Direct S2S translation.** Encoder reads source-language speech; decoder generates target-language speech directly. Key challenge: cross-lingual prosody transfer and duration alignment. Not a primary focus of this corpus.

**Cascade.** ASR → LM → TTS. Latency ~1–3 s end-to-end. Each module can be independently improved but the pipeline breaks paralinguistic continuity. Most deployed voice assistants (Siri, Alexa, Google Assistant) use cascade. Still the baseline in most S2S benchmarks.

## Major Claims

Claims are generalised propositions aggregated from paper evidence. The full claim registry with supporting paper lists is in `wiki/concepts/_evidence/speech-to-speech.yaml`.

### Strongly Supported

- Paralinguistic variability in speech tokens (Factor C) is the primary bottleneck limiting E2E speech LM semantic coherence relative to cascade systems, regardless of model scale.
  Supporting: [[2412.17048]]

- Low-frame-rate codecs with semantically rich token representations are necessary infrastructure for tractable autoregressive generation in full-duplex S2S dialogue systems.
  Supporting: [[2510.00981]]

### Emerging

- Empathetic and open-source E2E S2S systems with streaming interleaved decoding can achieve competitive performance on spoken dialogue benchmarks without proprietary training data.
  Supporting: [[2025.emnlp-demos.70]]

- Fine-grained interaction annotations (turn-taking, backchannels, interruptions) are necessary training signal for E2E S2S models, and no open corpus covered these before 2025.
  Supporting: [[2025.findings-emnlp.424]]

## Relationship to Other Concepts

### Competes With
- Cascade (ASR→LM→TTS) — cascade achieves higher semantic quality (text LM reasoning) and lower training complexity at the cost of higher latency (~1–3 s) and irreversible loss of paralinguistic information; E2E S2S trades semantic quality for latency and paralinguistic continuity.

### Extends or Builds On
- [[spoken-language-model]] — E2E S2S dialogue is the application layer built on top of speech LM foundations; the speech token vocabulary, codec choice, and AR decoding infrastructure are shared.
- [[neural-codec]] — low-frame-rate codecs (e.g., FlexiCodec [[2510.00981]]) are enabling infrastructure for tractable AR generation in full-duplex S2S systems.
- [[streaming-tts]] — streaming decoding infrastructure developed for TTS (e.g., [[2507.14534]]) is directly applicable to the synthesis stage of S2S systems.

### Commonly Paired With
- [[self-supervised-speech]] — SSL representations (HuBERT, WavLM) are used as input encoders or to initialize speech token vocabularies in E2E S2S models.
- [[voice-conversion]] — streaming VC building blocks (causal vocoders, chunk-wise encoding) overlap with the synthesis-side requirements of real-time S2S systems.

## Representative Papers

### Foundational
- [[2412.17048]] — diagnoses Factor C (paralinguistic variability) as the primary bottleneck for E2E speech LM semantic coherence; frames the core research challenge for the field.

### Influential
- [[2510.00981]] — demonstrates that low-frame-rate codecs are necessary for tractable AR sequence lengths in S2S dialogue LMs, directly enabling systems like Moshi.
- [[2025.acl-long.682]] — surveys all three S2S sub-paradigms, taxonomizes full-duplex systems, and evaluates their latency, paralinguistic continuity, and training complexity trade-offs.

### Recent Highlights
- [[2025.emnlp-demos.70]] — first fully open-source E2E empathetic speech-in speech-out system with streaming interleaved decoding, competitive on VoiceBench and URO-Bench.
- [[2025.findings-emnlp.424]] — InteractSpeech fills the interaction annotation gap, providing turn-taking, backchannel, and interruption labels needed for training S2S dialogue models.

## Open Questions

- Can the Factor C problem (paralinguistic variability in speech tokens) be solved by better codecs alone, or does it require architectural changes to the LM?
- Is full-duplex strictly necessary for natural conversation, or can well-designed turn-taking logic in half-duplex models suffice?
- How do S2S systems handle code-switching and multilingual input, where the ASR-intermediate approach has a natural advantage?
- What evaluation benchmarks best capture the interactive quality of S2S systems? [[2025.findings-emnlp.424]] provides interaction annotations but no end-to-end S2S benchmark yet exists for open evaluation.
- Full-Duplex-Bench [[2503.04721]] reveals that all current full-duplex models show a consistent tension between aggressiveness and caution in turn-taking; is this fundamental to the E2E paradigm or addressable by training regime?
- SpeechRole [[2508.02013]] shows that speech role-playing quality is bottlenecked by text LM reasoning even in cascaded systems; does this bottleneck affect open-ended S2S dialogue in the same way?

## Trend Summary

2022–2023: S2S translation matured (Translatotron, SeamlessM4T). Spoken dialogue remained cascade-dominated. 2024: Moshi, LLaMA-omni, and GLM-4-Voice demonstrated real-time E2E spoken dialogue for the first time, catalysing the field. 2025: Corpus papers begin studying the fundamental bottlenecks (Factor C, [[2412.17048]]), building enabling infrastructure (low-frame-rate codecs [[2510.00981]], interaction corpora [[2025.findings-emnlp.424]]), and streaming VC building blocks ([[2507.14534]]). The gap between cascade and E2E on semantic quality remains open. Integration pass 5 adds: Full-Duplex-Bench [[2503.04721]] — the first automated benchmark for full-duplex turn-taking behaviour, revealing that all current systems show a consistent aggressiveness-caution tension; SpeechRole [[2508.02013]] — the first large-scale speech-to-speech role-playing benchmark, providing evidence that interaction quality is text-LM bottlenecked regardless of architecture.

## All Papers

| ID | Title | Venue | Year | Key use of this concept |
|----|-------|-------|------|------------------------|
| [[2412.17048]] | Why Do Speech Language Models Fail? | arXiv (ICASSP 2026) | 2026 | Diagnoses Factor C (paralinguistic variability) as the primary bottleneck for E2E speech LM semantic coherence |
| [[2510.00981]] | FlexiCodec | arXiv (ICLR 2026) | 2025 | Low-frame-rate codec enabling tractable AR sequence lengths for S2S dialogue LMs |
| [[2025.findings-emnlp.424]] | InteractSpeech | EMNLP 2025 | 2025 | Interaction corpus for training S2S dialogue models on turn-taking, backchannels, interruptions |
| [[2509.02020]] | FireRedTTS-2 | arXiv | 2025 | Cascade S2S dialogue (TTS + dialogue planning); sentence-by-sentence streaming multi-speaker generation |
| [[2025.acl-long.388]] | DiVA: Distilling an End-to-End Voice Assistant Without Instruction Training Data | ACL | 2025 | E2E speech-in text-out voice assistant trained via cross-modal distillation; demonstrates that cascade-style capability forgetting in SFT can be avoided by distilling from text LLM responses to transcripts |
| [[2025.emnlp-demos.70]] | OpenS2S: Advancing Fully Open-Source End-to-End Empathetic Large Speech Language Model | EMNLP | 2025 | Full end-to-end speech-in speech-out system with empathetic response generation; streaming interleaved decoding; open-source training data and code; competitive on VoiceBench and URO-Bench |
| [[2025.acl-long.682]] | Recent Advances in Speech Language Models: A Survey | ACL | 2025 | Survey covering all three S2S sub-paradigms (E2E, direct translation, cascade); taxonomizes full-duplex systems (Moshi, Parrot, LSLM) and evaluates their latency, paralinguistic continuity, and training complexity |
| [[2503.04721]] | Full-Duplex-Bench | arXiv | 2025 | First automated benchmark for full-duplex S2S dialogue systems; four turn-taking scenarios; reveals aggressiveness-caution trade-off across all tested models; finds commercial systems better calibrated than open-source |
| [[2508.02013]] | SpeechRole | arXiv | 2025 | First large-scale S2S speech role-playing benchmark (111K dialogues, 98 roles); text LM quality is the primary bottleneck for both cascaded and E2E systems |
| [[interspeech-2025-0874]] | Efficient and Direct Duplex Modeling for Speech-to-Speech LLMs | Interspeech | 2025 | Full-duplex S2S via continuous encoder + codec decoder without speech pretraining; open-source first; barge-in 83% vs. 56% for Moshi; NVIDIA NanoCodec at 0.6 kbps for agent output |
| [[interspeech-2025-2660]] | Triadic Multi-party VAP for Turn-Taking | Interspeech | 2025 | Extends VAP from dyadic to 3-party conversation; acoustic-only next-speaker prediction above baseline; relevant to multi-party S2S system turn management |
| [[interspeech-2025-1595]] | Scheduled ILT for Speech-to-Speech Translation | Interspeech | 2025 | LLaMA 3.2-1B fine-tuned for S2ST via scheduled speech-text token interleaving; large BLEU gains for low-resource language pairs |
| [[2508.15827]] | Mini-Omni-Reasoner: Token-Level Thinking-in-Speaking | arXiv | 2025 | Full S2S system with interleaved reasoning; spoken response generation during chain-of-thought computation at 3B scale |
| [[interspeech-2025-0739]] | FD-Bench: Full-Duplex Benchmarking Pipeline | Interspeech | 2025 | Automated evaluation harness for full-duplex S2S systems; TTS-synthesised interruptions at three difficulty levels; Moshi vs. Freeze-Omni vs. VITA-1.5 |
| [[2508.07375]] | TurnGuide | arXiv | 2025 | Turn-level text-speech interleaving for full-duplex S2S; 24% semantic coherence improvement over baseline; evaluated on Full-Duplex-Bench |
| [[2508.09600]] | OSUM-EChat | arXiv | 2025 | Native multimodal end-to-end empathetic spoken chatbot; CosyVoice token-to-wav for synthesis; EChat-eval benchmark for paralinguistic responsiveness |
| [[2508.16188]] | AVLM (Seeing is Believing) | EMNLP | 2025 | SpiritLM-based audio-visual expressive speech generation; Q-Former prefix fusion of full-face visual features |
| [[2508.17623]] | EMO-Reasoning | arXiv | 2025 | Evaluation benchmark for emotional coherence in spoken dialogue systems; documents failure of all tested S2S systems on multi-turn emotional consistency |
| [[2509.03940]] | VoxRole | arXiv | 2025 | Role-playing spoken dialogue benchmark; paralinguistic appropriateness as the weakest dimension; acoustic-aware LLM evaluation for S2S character fidelity |
| [[2305.11000]] | SpeechGPT: Empowering Large Language Models with Intrinsic Cross-Modal Conversational Abilities | arXiv | 2023 | Foundational speech-to-speech system using LLM vocabulary expansion with discrete speech tokens; introduces chain-of-modality pattern (text intermediate before speech response) |
| [[2408.16725]] | Mini-Omni: Language Models Can Hear, Talk While Thinking in Streaming | arXiv | 2024 | First system demonstrating simultaneous speech-in/speech-out with parallel text generation; three-stage adapter curriculum for speech I/O integration |
| [[2409.06666]] | LLaMA-Omni: Seamless Speech Interaction with Large Language Models | arXiv | 2024 | End-to-end speech input/output with CTC streaming decoder; demonstrates E2E S2S with lower latency than cascaded ASR-LLM-TTS pipelines |
| [[2411.00774]] | Freeze-Omni: A Smart and Low Latency Speech-to-speech Dialogue Model with Frozen LLM | arXiv | 2024 | Frozen LLM speech-to-speech dialogue with chunk-level duplex state detection; demonstrates freezing LLM backbone preserves intelligence in S2S alignment |
| [[2410.17196]] | VoiceBench: Benchmarking LLM-Based Voice Assistants | arXiv | 2024 | Evaluation framework exposing gap between pipeline and E2E S2S systems on instruction understanding and robustness; demonstrates E2E S2S trails pipeline under all tested conditions |
