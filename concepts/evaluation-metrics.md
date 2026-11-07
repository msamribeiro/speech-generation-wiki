---
slug: evaluation-metrics
concept: evaluation-metrics
render_type: overview
title: Evaluation Metrics
aliases: [speech quality metrics, TTS evaluation metrics, objective evaluation, automatic evaluation]
status: mature-infrastructure
last_reviewed: 2026-11-07
source_digest_date: 2026-09-20
paper_count: 346
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

> [!abstract]
> Evaluation determines which improvements we can see and which failures remain hidden. Across 346
> reviewed papers, the main finding is consistent: no single automatic score measures all aspects
> of speech quality. A metric is useful only when it matches the target property, domain, level of
> detail, and intended use.

## Current State

- **Automatic metrics are useful tools, but they do not replace targeted judgment.** WER and CER
  can disagree with human intelligibility. Speaker embeddings can rank a synthetic voice above its
  reference. Automatic MOS predictors can reverse human rankings. PESQ, STOI, and MCD can also
  disagree with perceived quality ([[2403.16973|VoiceCraft]], [[2406.18009|E2 TTS]],
  [[2508.00317]], [[2507.01611]]).
- **Speech quality has several separate dimensions.** Naturalness, intelligibility, identity,
  prosody, diversity, emotion, latency, and robustness can change independently. An evaluation
  should use a small set of metrics chosen for the intended use. A long generic leaderboard does
  not solve the problem.
- **Clean, in-domain tests hide real-world failures.** Performance gaps often grow under noise,
  reverberation, spontaneous speech, unfamiliar speakers, or domain shift
  ([[2403.16973|VoiceCraft]], [[2506.21875|WildSpeech-Bench]]).
- **Text-only tests miss important parts of spoken interaction.** Transcripts and text judges do
  not capture timing, emotion, speaker traits, background sound, or non-verbal behavior. This is a
  major limit for spoken agents and full-duplex systems ([[2508.18240|MTalk-Bench]],
  [[2507.23159|Full-Duplex-Bench v1.5]]).
- **Learned judges work well only for some capabilities.** SSL-based MOS predictors are useful and
  mature. LLM and audio-LM judges work better for instruction following and rubric checks than for
  fine prosody or raw-waveform cues ([[2204.02152|UTMOS]],
  [[2506.16381|InstructTTSEval]], [[2509.19928]]).
- **New tests cover behavior and social impact.** Recent work measures multi-turn reasoning,
  full-duplex timing, fairness, clinical suitability, and watermark robustness. Aggregate MOS or
  accuracy cannot expose these failures.
- **Q4 makes evaluation more specific to each intervention.** Full-duplex tests separate fast
  response from false interruption. Fairness audits expose group differences hidden by averages.
  Watermark tests use realistic transformations, not only basic noise
  ([[2507.23159|Full-Duplex-Bench v1.5]],
  [[interspeech-2025-2328|A Watermark for Auto-Regressive Speech Generation Models]]).

## Method Landscape

| Evaluation layer | Best use | Main limit |
|---|---|---|
| **Human listening and interaction tests** | Naturalness, preference, identity, emotion, and complete behavior | Expensive and sensitive to test design; scores can saturate near the quality ceiling |
| **Task proxies** | WER/CER for content, speaker embeddings for identity, and classifiers for emotion | Measures the proxy model as well as the speech |
| **Signal and perceptual metrics** | Fast checks of distortion, reconstruction, or intelligibility | Rankings can change across architectures, sampling rates, and domains |
| **Learned quality and LLM judges** | Large-scale screening and rubric scoring | Sensitive to domain shift, level of detail, and fine acoustic differences |
| **Capability-specific benchmarks** | Dialogue, timing, fairness, clinical use, and codec behavior | Protocols remain fragmented across papers |

## Key Trade-offs

- **Scale versus validity.** Automatic scores support large tests. Human tests remain necessary for
  subtle, perceptual, or poorly modeled properties.
- **Common metrics versus task fit.** MOS or WER helps comparison across papers. A metric designed
  for prosody, accent, identity, timing, or clinical speech can be more informative.
- **System ranking versus sample diagnosis.** A predictor can rank complete systems well but fail
  on individual utterances ([[2507.06116]]).
- **Reconstruction versus downstream use.** A codec can reconstruct speech well and still produce
  poor tokens for a generative model ([[2025.naacl-srw.6]], [[2506.10274]]).
- **Target-score gains versus hidden losses.** Preference optimization, distillation, or alignment
  can improve one metric while reducing prosodic diversity ([[2507.14988]], [[2509.19928]]).
- **Standard protocols versus population validity.** Shared tests improve comparison. Clinical,
  accented, low-resource, and assistive speech still need validation for the target population.

## Open Questions

- Can LLM and audio-LM judges measure fine prosody and non-verbal audio reliably?
- What should complement or replace MOS and MUSHRA near the quality ceiling?
- Can full-duplex and spoken-dialogue evaluation use shared protocols instead of separate lab
  pipelines?
- How should codec tests combine reconstruction, semantic retention, noise robustness, and
  downstream generation?
- Will fairness, accessibility, and multi-turn tests become standard parts of evaluation?

## Go Deeper

Read [[concepts/evaluation-metrics-in-depth|Evaluation Metrics: In Depth]] for the detailed evidence
on metric disagreement, learned judges, subjective-test limits, codec evaluation, spoken-agent
benchmarks, and fit-for-purpose test design.

## Scope

This assessment covers 346 integrated papers through Q4 2025. Many entries are TTS, voice
conversion, codec, or spoken-agent papers that expose one evaluation problem rather than introduce
a metric. The graph is broad, but it is not a formal meta-analysis. Citations are representative;
complete evidence remains in `wiki/_claims/evaluation-metrics.yaml`.

---

_This page is generated from `wiki/_claims/evaluation-metrics.yaml` (digest date: 2026-09-20). For complete structured claims and paper-level provenance, use the source YAML._
