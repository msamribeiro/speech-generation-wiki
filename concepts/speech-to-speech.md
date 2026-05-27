---
slug: speech-to-speech
title: Speech-to-Speech Systems
aliases: [S2S, end-to-end spoken dialogue, direct speech translation, speech-in speech-out, voice-to-voice]
related_concepts: [spoken-language-model, neural-codec, self-supervised-speech, streaming-tts, voice-conversion]
last_updated: 2026-05-27
---

# Speech-to-Speech Systems

## What it is

Speech-to-speech (S2S) systems take speech as input and produce speech as output, without necessarily passing through an explicit text representation. This distinguishes them from standard TTS (text in, speech out) and from cascade dialogue systems (speech → ASR → LM → TTS). The defining characteristic is that the audio signal is the primary carrier of both the input signal and the output, preserving paralinguistic information — tone, emotion, speaking rate, accents — that would be lost in an ASR/TTS pipeline.

Three main sub-paradigms:

1. **End-to-end spoken dialogue** — a speech language model ingests speech tokens directly and autoregressively generates speech tokens in response (Moshi, LLaMA-omni, Mini-Omni, GLM-4-Voice). No ASR or TTS module required; the LM operates entirely in speech-token space.
2. **Direct S2S translation** — speech in a source language is converted to speech in a target language without intermediate text (SeamlessM4T, Translatotron). Targets low latency and preserves prosody across language boundaries.
3. **Cascade S2S** — ASR → text LM → TTS, connected in a pipeline. Still the production default for most voice assistants; included here for comparison because the same benchmark tasks (latency, naturalness, speaker consistency) apply to all three paradigms.

## Why it matters

The cascade approach introduces compounding errors, irreversible information loss (paralinguistics discarded by ASR), and latency from three sequential modules. End-to-end S2S eliminates these: a single model maintains a continuous speech-token context window, can generate overlap and backchannels naturally, and can process and emit audio in real time (full-duplex). [[2412.17048]] identifies that making speech LMs work well is harder than text LMs — paralinguistic variability in speech tokens (Factor C) is the main bottleneck — but this is a fundamental research problem rather than a reason to avoid the paradigm.

For TTS research, S2S systems are relevant because they require the same low-frame-rate, semantically rich codec representations as TTS autoregressive models ([[2510.00981]]), and the same streaming decoding infrastructure as streaming TTS ([[2507.14534]]). The dataset gap is also relevant: [[2025.findings-emnlp.424]] (InteractSpeech) was created specifically because no open corpus covered fine-grained interaction annotations needed for S2S dialogue training.

## Current state of the art

Most landmark S2S systems are not yet in corpus (see citation discovery candidates below). Based on external knowledge and cross-references in corpus papers:

- **Moshi** (Kyutai, 2410.00037, cited 53× in corpus) — full-duplex speech LM; inner monologue text stream + speech stream; real-time on a single GPU. Currently the most-cited S2S system in the corpus.
- **LLaMA-omni** (2409.06666, cited 28× in corpus) — speech input + speech output LLaMA adaptation; low latency.
- **GLM-4-Voice** (2412.02612, cited 35× in corpus) — bilingual (EN/ZH) end-to-end spoken dialogue.
- **Mini-Omni** — lightweight E2E spoken dialogue targeting on-device deployment.

All four are citation discovery candidates and should be fetched and ingested.

## Key variants and sub-approaches

**Full-duplex E2E dialogue (Moshi family).** A single transformer processes a continuous interleaved stream of user and system speech tokens, enabling the system to listen and speak simultaneously. Requires a codec at sub-10 Hz to make the AR model tractable ([[2510.00981]]). The "inner monologue" trick (generating a text chain-of-thought token stream in parallel with speech tokens) is used to maintain semantic coherence, partially addressing the Factor C problem from [[2412.17048]].

**Half-duplex E2E dialogue (LLaMA-omni, GLM-4-Voice).** Speech input is encoded once per turn; response speech tokens are generated autoregressively. Simpler architecture but requires explicit turn-taking logic. Less natural for interruptions and backchannels.

**Direct S2S translation.** Encoder reads source-language speech; decoder generates target-language speech directly. Key challenge: cross-lingual prosody transfer and duration alignment. Not a primary focus of this corpus.

**Cascade.** ASR → LM → TTS. Latency ~1–3 s end-to-end. Each module can be independently improved but the pipeline breaks paralinguistic continuity. Most deployed voice assistants (Siri, Alexa, Google Assistant) use cascade. Still the baseline in most S2S benchmarks.

## Comparison to alternatives

| Paradigm | Latency | Paralinguistic continuity | Training complexity | Semantic quality |
|----------|---------|--------------------------|---------------------|-----------------|
| Cascade (ASR→LM→TTS) | ~1–3 s | Low (ASR discards paralinguistics) | Low (independent modules) | High (text LM) |
| Half-duplex E2E | ~0.5–1 s | Medium (speaker/emotion preserved) | Medium | Medium (Factor C problem) |
| Full-duplex E2E | ~37–100 ms | High | High | Medium (Factor C problem) |

The Factor C problem ([[2412.17048]]) — paralinguistic variability inflating the effective vocabulary of speech tokens — is the main reason E2E systems lag behind cascade on semantic coherence metrics (sBLIMP, Topic-StoryCloze). Low-frame-rate codecs with ASR-supervised tokens ([[2510.00981]]) are the most promising near-term mitigation.

## Year-on-year trajectory

2022–2023: S2S translation matured (Translatotron, SeamlessM4T). Spoken dialogue remained cascade-dominated. 2024: Moshi, LLaMA-omni, and GLM-4-Voice demonstrated real-time E2E spoken dialogue for the first time, catalysing the field. 2025: Corpus papers begin studying the fundamental bottlenecks (Factor C, [[2412.17048]]), building enabling infrastructure (low-frame-rate codecs [[2510.00981]], interaction corpora [[2025.findings-emnlp.424]]), and streaming VC building blocks ([[2507.14534]]). The gap between cascade and E2E on semantic quality remains open.

## Open questions

- Can the Factor C problem (paralinguistic variability in speech tokens) be solved by better codecs alone, or does it require architectural changes to the LM?
- Is full-duplex strictly necessary for natural conversation, or can well-designed turn-taking logic in half-duplex models suffice?
- How do S2S systems handle code-switching and multilingual input, where the ASR-intermediate approach has a natural advantage?
- What evaluation benchmarks best capture the interactive quality of S2S systems? [[2025.findings-emnlp.424]] provides interaction annotations but no end-to-end S2S benchmark yet exists for open evaluation.

## Papers

| ID | Title | Venue | Year | Key use of this concept |
|----|-------|-------|------|------------------------|
| [[2412.17048]] | Why Do Speech Language Models Fail? | arXiv (ICASSP 2026) | 2026 | Diagnoses Factor C (paralinguistic variability) as the primary bottleneck for E2E speech LM semantic coherence |
| [[2510.00981]] | FlexiCodec | arXiv (ICLR 2026) | 2025 | Low-frame-rate codec enabling tractable AR sequence lengths for S2S dialogue LMs |
| [[2025.findings-emnlp.424]] | InteractSpeech | EMNLP 2025 | 2025 | Interaction corpus for training S2S dialogue models on turn-taking, backchannels, interruptions |
| [[2509.02020]] | FireRedTTS-2 | arXiv | 2025 | Cascade S2S dialogue (TTS + dialogue planning); sentence-by-sentence streaming multi-speaker generation |
| [[2025.acl-long.388]] | DiVA: Distilling an End-to-End Voice Assistant Without Instruction Training Data | ACL | 2025 | E2E speech-in text-out voice assistant trained via cross-modal distillation; demonstrates that cascade-style capability forgetting in SFT can be avoided by distilling from text LLM responses to transcripts |
| [[2025.emnlp-demos.70]] | OpenS2S: Advancing Fully Open-Source End-to-End Empathetic Large Speech Language Model | EMNLP | 2025 | Full end-to-end speech-in speech-out system with empathetic response generation; streaming interleaved decoding; open-source training data and code; competitive on VoiceBench and URO-Bench |
| [[2025.acl-long.682]] | Recent Advances in Speech Language Models: A Survey | ACL | 2025 | Survey covering all three S2S sub-paradigms (E2E, direct translation, cascade); taxonomizes full-duplex systems (Moshi, Parrot, LSLM) and evaluates their latency, paralinguistic continuity, and training complexity |
