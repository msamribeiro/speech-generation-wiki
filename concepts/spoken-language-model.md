---
slug: spoken-language-model
concept: spoken-language-model
render_type: overview
title: Spoken Language Models
aliases: [speech language model, spoken conversational model, speech-native language model, audio language model]
status: established
last_reviewed: 2026-08-02
source_digest_date: 2026-07-27
paper_count: 127
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
> Spoken language models process and generate speech within a language-model framework, aiming to
> combine reasoning, voice, timing, and conversation without reducing every interaction to an
> offline transcript. Shared speech–text backbones now support streaming and duplex systems, but
> acoustic representation, paralinguistic reasoning, memory, latency, and safety remain material
> gaps relative to fluent demonstrations.

## Current State

**Discrete tokens make speech-scale language modeling tractable.** Semantic and codec tokens turn
waveforms into compact sequences, while low or variable frame rates reduce context and latency.
Aggressive compression eventually harms intelligibility, timing, identity, or prosody
([[2209.03143|AudioLM]], [[2408.16725|Mini-Omni]]).

**Speech–text alignment preserves much of a text LM's intelligence.** Modality connectors and
interleaved training add speech input and output to pretrained language models. They transfer
transcript-accessible reasoning efficiently, but underrepresent acoustic-only information and can
inherit errors from synthetic interleaving ([[2305.11000|SpeechGPT]], [[2409.06666|LLaMA-Omni]]).

**Interactive latency and full duplex are feasible, not solved.** Incremental tokenization,
generation, and decoding allow responsive systems. Natural overlap requires listening while
speaking, interruption handling, backchannels, and turn transitions; many results use constrained or
simulated conversations ([[2408.02622|Language Model Can Listen While Speaking]],
[[2410.00037|Moshi]]).

**Semantic competence exceeds paralinguistic competence.** Models perform better on transcript-level
content than prosody, emotion, speaker traits, and non-verbal cues. Replacing text with speech also
introduces measurable reasoning and instruction-following gaps through representation, alignment,
and robustness errors.

**Conversation requires memory, evaluation, and alignment beyond audio quality.** Multi-turn systems
need explicit state retrieval and long-horizon planning. Preference optimization improves helpfulness
or naturalness but can exploit incomplete rewards. Benchmarks must separate reasoning, voice,
turn-taking, memory, latency, and safety.

**Data breadth is capability and risk.** Large instruction and conversational corpora expand task
coverage, while synthetic labels, demographic imbalance, and scripted interaction patterns can
create fluent behavior that transfers poorly to live users.

## Method Landscape

- **Autoregressive speech language models** predict semantic or codec tokens causally and form the
  largest family.
- **Hybrid multistage systems** combine text reasoning, speech tokens, and specialized acoustic
  decoders or dynamically route through cascaded components.
- **Flow and latent speech decoders** render continuous acoustics from a language-model plan,
  reducing fully autoregressive audio prediction.
- **Transformer encoder–decoder speech models** retain explicit input–output structure for
  translation or conversation.
- **Adversarial codec and variational front ends** define the speech interface rather than the
  reasoning backbone; their information loss sets the downstream ceiling.

## Key Trade-offs

- **Semantic abstraction versus acoustic richness:** compact tokens aid reasoning while losing voice
  and paralinguistic detail.
- **End-to-end immediacy versus cascaded controllability:** direct speech paths preserve timing;
  cascades retain strong text reasoning and inspectable components.
- **Low latency versus complete context:** early response improves interaction while increasing
  endpoint and interruption errors.
- **Unified breadth versus specialized quality:** one backbone supports many tasks without matching
  the best component on every dimension.
- **Alignment gains versus reward blind spots:** post-training improves selected behavior while
  risking unmeasured acoustic or interaction regressions.

## Open Questions

- Which representation best balances semantics, acoustics, sequence length, and latency?
- How can duplex systems learn overlap and interruption from natural rather than scripted dialogue?
- Which alignment objectives add acoustic competence without eroding text-LM reasoning?
- When should deployment use end-to-end, cascaded, or dynamically routed hybrid systems?
- What live multilingual protocol can jointly measure reasoning, voice, timing, memory, and safety?

## Go Deeper

Read [[concepts/spoken-language-model-in-depth|Spoken Language Models: In Depth]] for the evidence on
tokenization, speech–text transfer, streaming, duplex interaction, memory, paralinguistics,
evaluation, alignment, and deployment choices.

## Scope

This synthesis covers 127 papers from 2022 through September 2025, including 101 from 2025. Spoken
agents provide most direct evidence, with TTS, codecs, and evaluation supplying infrastructure and
controls. Sixteen clusters are strongly supported; bias, privacy, misuse, and provenance evaluation
remain emerging. System comparisons frequently change the base LM, data, tokenizer, and decoder
together. Live multilingual and full-duplex evidence remains much narrower than offline benchmark
coverage, so deployment conclusions require additional qualification. Infrastructure and historical
papers establish interfaces and lineage, while fluent demonstrations support feasibility rather
than equal competence across reasoning, acoustics, interaction, and safety. The assessment is
strongest for recurring architectural capabilities and narrower for real-world user outcomes.
That distinction governs the deployment guidance here.

---

_This page is generated from `wiki/_claims/spoken-language-model.yaml` (digest date: 2026-07-27).
For complete structured claims and paper-level provenance, use the source YAML._
