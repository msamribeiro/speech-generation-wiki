---
slug: speech-to-speech
concept: speech-to-speech
render_type: overview
title: Speech-to-Speech Systems
aliases: [speech-to-speech, S2S, spoken dialogue systems, speech language models, end-to-end spoken agents]
status: established
last_reviewed: 2026-07-24
source_digest_date: 2026-07-19
paper_count: 60
generation:
  schema_version: 2
  date: "2026-07-24"
  stage: render
  mode: light
  runtime: codex
  provider: openai
  agent: speech-generation-render-agent
  model: "gpt-5"
  commit: "39007c6"
---

> [!abstract]
> Speech-to-speech systems accept speech and produce spoken responses, spanning conversational
> agents, full-duplex interaction, and speech-to-speech translation. Across the 60 papers reviewed
> here, real-time interaction is increasingly practical, but the strongest systems still make
> substantial trade-offs between reasoning, latency, turn-taking, and sensitivity to how something
> is said rather than only what was said.

## Current State

- **Cascaded ASR–language-model–TTS systems remain the strongest general-purpose baseline.**
  Mature components still lead many spoken-instruction, reasoning, and role-playing comparisons
  ([[2410.17196|VoiceBench]], [[2506.23049|AURA]], [[2508.02013|SpeechRole]]). This is not a clean
  proof that cascades are intrinsically better: comparisons often also vary model scale, data, and
  component maturity.
- **Modular speech output currently preserves language ability more reliably than joint speech
  training.** Keeping a text-language-model backbone frozen or lightly adapted and attaching a
  streaming speech decoder reduces the reasoning and instruction-following loss seen in several
  jointly trained systems ([[2411.00774|Freeze-Omni]], [[2025.findings-acl.1051|LLMVoX]],
  [[interspeech-2025-0874|NVDuplex]]).
- **Full-duplex operation has not solved conversational timing.** Systems can listen while speaking
  and stop quickly when interrupted, but fast response often comes with false yielding, premature
  takeovers, weak backchannels, or poorer semantic continuation ([[2507.23159|Full-Duplex-Bench
  v1.5]], [[2509.22243|FLEXI]]).
- **Semantic competence is ahead of paralinguistic understanding.** Emotion, speaker traits,
  ambient sound, and non-verbal behavior remain weaker than content understanding across recent
  diagnostic benchmarks ([[2508.18240|MTalk-Bench]], [[2509.03940|VoxRole]],
  [[2508.17623|EMO-Reasoning]]).
- **Single-turn scores overstate conversational ability.** Context recall, emotional coherence,
  information density, and bias can worsen as an interaction continues
  ([[2025.findings-acl.470]], [[2508.18240|MTalk-Bench]]).

## Method Landscape

| Approach | Why use it | Principal limitation |
|----------|------------|----------------------|
| **Cascaded ASR + LM + TTS** | Strong task accuracy, debuggable intermediate text, replaceable components | Pipeline latency and loss of acoustic information during transcription |
| **Modular LM + speech decoder** | Preserves a strong text backbone while enabling streaming spoken output | Often restricted in voice, language, or expressive control |
| **Native joint speech-text model** | Direct token generation and a natural route to full-duplex interaction | Current systems often lose reasoning or knowledge relative to their text backbones |
| **Dual-stream full duplex** | Separates user and agent audio so the system can listen while speaking | Evidence remains concentrated in smaller systems and restricted interaction tests |
| **Simultaneous speech translation** | Optimizes cross-lingual content, voice transfer, and streaming together | Results depend strongly on language pair, latency policy, and proprietary evidence |

These approaches increasingly overlap. The important distinction is not whether a system is called
“end to end,” but where recognition, reasoning, timing, and speech generation are coupled and which
capabilities remain independently trainable.

## Key Trade-offs

- **Task accuracy versus direct acoustic modeling.** A cascade can use a stronger text reasoner,
  while a direct audio model can retain emotion, timing, and other information discarded by a
  transcript.
- **Fast response versus stable floor control.** Lower latency does not guarantee correct decisions
  about interruption, pauses, background speech, or backchannels.
- **Semantic tokens versus acoustic fidelity.** Text-aligned representations work well with
  language models; acoustic codec representations preserve more voice and paralinguistic detail.
- **Joint adaptation versus capability preservation.** Training more of the backbone may improve
  speech integration while risking forgetting of text reasoning and instruction-following.
- **Evaluation scale versus evaluation validity.** Automated judges enable broad comparisons but
  are least reliable for subtle, paralinguistic, or near-tied speech outputs.

## Open Questions

- Will end-to-end systems close the cascade quality gap when backbone scale, data, and components
  are matched?
- Can a full-duplex model respond quickly while also handling pauses, interruptions, and
  backchannels robustly?
- Which training strategy best preserves language ability while learning speech-specific
  representations?
- Can retrieval systems be designed for conversational memory rather than only external factual
  knowledge?
- Which evaluation protocol best combines semantic correctness, acoustic appropriateness,
  interaction quality, and multi-turn behavior?

## Go Deeper

Read [[concepts/speech-to-speech-in-depth|Speech-to-Speech Systems: In Depth]] for the detailed
research landscape, the evidence behind these assessments, disagreements about staged pretraining
and retrieval, and a representative reading path.

## Scope

This assessment covers 60 integrated papers through Q3 2025 and combines spoken dialogue,
full-duplex interaction, and speech-to-speech translation. These tasks share a speech-in/speech-out
interface but not always objectives or metrics. Surveys support taxonomy rather than independent
replication, known duplicate publication records are treated as one evidence point, and several
frontier systems rely on proprietary data or unmatched commercial comparisons.

---

_This page is generated from `wiki/_claims/speech-to-speech.yaml` (digest date: 2026-07-19).
Complete structured claims and paper-level provenance remain in the source YAML._
