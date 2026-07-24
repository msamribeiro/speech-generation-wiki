---
slug: evaluation-metrics
concept: evaluation-metrics
render_type: overview
title: Evaluation Metrics
aliases: [speech quality metrics, TTS evaluation metrics, objective evaluation, automatic evaluation]
status: mature-infrastructure
last_reviewed: 2026-07-24
source_digest_date: 2026-07-21
paper_count: 285
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
> Evaluation determines which speech-generation improvements the field can see—and which failures
> remain hidden. Across 285 reviewed papers, the central conclusion is unusually consistent: no
> single automatic score captures speech quality, and metric validity depends on the perceptual
> dimension, domain, granularity, and downstream use being evaluated.

## Current State

- **Common automatic metrics are useful proxies, not substitutes for targeted judgment.** WER and
  CER can disagree with perceived intelligibility; speaker embeddings can rank synthetic voices
  above their references; automatic MOS predictors can reverse human rankings; and PESQ, STOI, or
  MCD can disagree with perceived quality across architectures ([[2403.16973|VoiceCraft]],
  [[2406.18009|E2 TTS]], [[2508.00317]], [[2507.01611]]).
- **Speech quality is irreducibly multidimensional.** Naturalness, intelligibility, speaker
  identity, prosody, diversity, emotion, latency, and robustness frequently move independently or
  trade off. A credible evaluation therefore needs a small suite chosen for the system's intended
  use rather than a long generic leaderboard.
- **Evaluation on clean, in-domain speech systematically understates real-world failures.**
  Differences widen under noise, reverberation, spontaneous speech, unfamiliar speakers, and
  distribution shift ([[2403.16973|VoiceCraft]], [[2506.21875|WildSpeech-Bench]]).
- **Text-only evaluation misses much of what makes spoken systems worth building.** ASR transcripts
  and text judges omit timing, emotion, speaker traits, ambient sound, and non-verbal behavior.
  This is especially consequential for spoken conversational agents and full-duplex systems
  ([[2508.18240|MTalk-Bench]], [[2507.23159|Full-Duplex-Bench v1.5]]).
- **Learned judges are improving, but their reliability is capability-scoped.** SSL-based MOS
  predictors are a mature and useful family, while LLM/audio-LM judges work better for
  instruction-following and rubric checks than for fine prosody or raw-waveform cues
  ([[2204.02152|UTMOS]], [[2506.16381|InstructTTSEval]], [[2509.19928]]).
- **Evaluation methodology is expanding from acoustic quality to behavior and social impact.**
  Recent work adds multi-turn reasoning, full-duplex timing, fairness, clinical suitability, and
  provenance robustness—dimensions that aggregate MOS or accuracy cannot expose.

## Method Landscape

| Evaluation layer | Best use | Main limitation |
|------------------|----------|-----------------|
| **Human listening and interaction tests** | Naturalness, preference, identity, emotion, and holistic behavior | Expensive, protocol-sensitive, and increasingly saturated near the quality ceiling |
| **Task proxies** | WER/CER for content, ASV embeddings for identity, classifiers for emotion | Measures the proxy model as well as the synthesized speech |
| **Signal and perceptual metrics** | Fast diagnostics for distortion, reconstruction, or intelligibility | Rankings may not transfer across codecs, vocoders, sampling rates, or domains |
| **Learned quality and LLM judges** | Scalable prediction, rubric scoring, and broad benchmark sweeps | Domain shift, granularity mismatch, and weak fine-grained acoustic sensitivity |
| **Capability-specific benchmarks** | Full-duplex timing, dialogue, fairness, clinical use, and codec downstream behavior | Fragmented protocols and limited cross-paper standardization |

## Key Trade-offs

- **Scale versus validity.** Cheap automatic scores permit large studies; human tests remain
  necessary when the target property is perceptual, subtle, or poorly represented by the proxy.
- **Generic comparability versus task fit.** A common MOS or WER column aids comparison but may be
  less informative than a metric designed for prosody, accent, identity, dialogue timing, or a
  clinical population.
- **System-level ranking versus utterance-level diagnosis.** A predictor can correlate well across
  systems while being unreliable for individual samples ([[2507.06116]]).
- **Reconstruction versus downstream utility.** A codec that reconstructs speech well can still
  produce worse tokens for a generative model ([[2025.naacl-srw.6]], [[2506.10274]]).
- **Optimizing a score versus preserving unmeasured qualities.** Preference optimization,
  distillation, or explicit alignment can improve a target metric while reducing prosodic diversity
  ([[2507.14988]], [[2509.19928]]).

## Open Questions

- Can LLM/audio-LM judges become reliable for fine prosody and non-verbal audio, or is that a
  structural limitation?
- What should replace or augment MOS/MUSHRA when modern systems cluster near the quality ceiling?
- Can full-duplex and spoken-dialogue evaluation converge on interoperable protocols rather than
  bespoke pipelines?
- How should codec evaluation combine reconstruction, semantic retention, noise robustness, and
  downstream generation?
- Will fairness, accessibility, and multi-turn testing become standard evaluation dimensions?

## Go Deeper

Read [[concepts/evaluation-metrics-in-depth|Evaluation Metrics: In Depth]] for the evidence behind
metric divergence, learned judges, subjective-test saturation, codec and dialogue evaluation, and
the design of fit-for-purpose evaluation suites.

## Scope

This assessment covers 285 integrated papers through Q3 2025. Evaluation metrics is a cross-cutting
concept: many entries are TTS, VC, codec, or spoken-agent system papers that contribute one
evaluation caution rather than proposing a metric. The reviewed graph is broad but not a formal
meta-analysis; citations here are representative, and complete provenance remains in
`wiki/_claims/evaluation-metrics.yaml`.

---

_This page is generated from `wiki/_claims/evaluation-metrics.yaml` (digest date: 2026-07-21).
Complete structured claims and paper-level provenance remain in the source YAML._
