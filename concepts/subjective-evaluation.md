---
slug: subjective-evaluation
concept: subjective-evaluation
render_type: overview
title: Subjective Evaluation of Generated Speech
aliases: [listening tests, perceptual evaluation, MOS evaluation, human evaluation]
status: established
last_reviewed: 2026-11-07
source_digest_date: 2026-10-31
paper_count: 255
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
> Subjective evaluation measures how listeners perceive generated speech across naturalness, intelligibility, identity, prosody, emotion, interaction, and social suitability. Human judgment remains necessary because these dimensions can move independently and automatic judges remain condition- and attribute-dependent.

## Current State

**There is no single perceptual quality axis.** Naturalness, intelligibility, speaker identity, expressiveness, accent, melody, synchronization, and interaction behavior are distinct outcomes. Improvements on one do not guarantee improvements on the others, and aggregate scores can conceal severe dimension-level failures. Q4 evidence strengthens this conclusion across spoken-dialogue benchmarks, codecs, child voice conversion, controllable synthesis, and multimodal speech ([[2025.emnlp-main.1160|C3]], [[2025.emnlp-main.1447|MULTIVOX]], [[2512.12129|Child Voice Conversion Study]]).

**Protocol design can change the conclusion.** MOS remains useful for broad naturalness comparisons when listener recruitment, instructions, anchors, sampling, and scale are reported rigorously. It becomes less stable across listener populations, cultural settings, reference conditions, and near-tied systems. Pairwise preference and ranking often discriminate small differences more reliably, but establish order only within the chosen comparison set.

**Automatic metrics and listeners frequently disagree.** Learned quality predictors often outperform classical signal measures in aggregate, but that superiority is now contested rather than general: codec evaluations show that classical metrics can match or exceed learned predictors on particular distortions ([[2204.02152|UTMOS]], [[2511.19734|Objective Metrics for Neural Audio Codecs]]). Speaker embeddings are similarly confounded by language, channel, duration, architecture, and reference design.

**Automated language-model and audio-language-model judges are useful only within validated scope.** Grounded rubrics, repeated sampling, and task-specific tuning can reproduce some human rankings, especially for clear differences. Reliability degrades for near ties, utterance-level prosody, emotion, non-verbal cues, and unscripted audio. Spoken-agent benchmarks therefore need dimension-level judge calibration rather than treating one model score as listener evidence ([[2025.emnlp-main.1160|C3]], [[2511.00850|MULTI-Bench]]).

**Preference optimization turns evaluation into a training objective and creates new failure modes.** Human or proxy preferences can improve a targeted attribute beyond supervised training ([[2406.00654|Human-Feedback Zero-Shot TTS]], [[2025.acl-long.598|Preference-Aligned Zero-Shot TTS]]). Stronger rewards can also reduce naturalness, identity, or prosodic diversity, saturate, or invite reward hacking. Held-out perceptual dimensions and stopping criteria are therefore part of the evaluation design.

**Fine-grained controls need fine-grained tests.** Holistic naturalness cannot establish whether a model follows word-level emotion, speaking rate, accent, filler prosody, melody, or voice-quality instructions. The late non-verbal emotion study directly separates naturalness, expressiveness, and recognition: finer control improves expressive recognition while slightly reducing naturalness, illustrating a non-monotonic operating point ([[2605.25504|Fine-Grained Non-Verbal Expression Control]]).

**Population and deployment context matter.** High-resource, clean, English-centric panels do not automatically transfer to low-resource languages, in-the-wild audio, or culturally specific uses. Participatory evaluation adds educational, cultural, accessibility, and usability evidence that scalar laboratory scores omit. Community-scaled review can be more valid than a nominally larger non-native panel when native listeners are scarce ([[2025.iwclul-1.3|South Sámi TTS]]).

> [!warning]
> Underspecified scales cannot support absolute rankings. The late PACC conversational-conversion study lacks sufficient listener, test-set, baseline, and score-scale detail; it strengthens the protocol-reporting warning but not its claimed quality ordering ([[2605.18001|Bridging the Gap]]).

## Method Landscape

- **Absolute category ratings** use MOS-style scales for naturalness, quality, identity, or other attributes. They are easy to aggregate but sensitive to scale use and context.
- **Pairwise preference and ranking** improve discrimination and support preference learning, while remaining relative to the selected alternatives.
- **Reference-based attribute tests** compare identity, accent, emotion, prosody, melody, or timing against anchors whose selection can itself bias results.
- **Task and interaction rubrics** separate semantic success, dialogue behavior, multimodal synchronization, and acoustic delivery in spoken systems.
- **Participatory and qualitative methods** capture community, cultural, pedagogical, and usability outcomes that laboratory scores cannot represent.

## Key Trade-offs

- **Scale versus diagnosticity.** One score is convenient, but hides which perceptual dimension changed.
- **Absolute interpretation versus discrimination.** Pairwise tests detect smaller differences while depending on the comparison set.
- **Laboratory control versus ecological validity.** Controlled prompts improve repeatability but may not predict real users, noise, or interaction.
- **Automation versus attribute coverage.** Automatic judges reduce cost but require task-, domain-, and granularity-matched human calibration.
- **Optimization strength versus unmeasured quality.** Improving a target reward can degrade identity, diversity, naturalness, or intelligibility elsewhere.

## Open Questions

- Which protocols best balance ecological validity, diagnostic resolution, cost, and cross-laboratory reproducibility?
- When can learned metrics or automated judges safely reduce human testing, and which attributes must remain listener-rated?
- How should listener language, demographics, hearing profile, and cultural context enter analysis and reporting?
- How can preference optimization improve a target attribute without suppressing unmeasured dimensions?
- Which calibration and transfer tests should be required before an automatic judge is used in a new domain?

## Go Deeper

Read [[concepts/subjective-evaluation-in-depth|Subjective Evaluation of Generated Speech: In Depth]] for protocol choices, automation limits, preference training, population validity, system bottlenecks, and a representative reading path.

## Scope

This synthesis covers 255 papers published from 2016 through the Q4 2025 evidence window across TTS, voice conversion, spoken agents, codecs, singing, and multimodal generation. Many papers use listening tests to support a system rather than study evaluation itself, so protocol conclusions are weighted most strongly where methods are compared directly. Paper count is not treated as independent replication when studies reuse metrics, benchmarks, listener pools, or system lineages.

---

_This page is generated from `wiki/_claims/subjective-evaluation.yaml` (digest date: 2026-10-31). For complete structured claims and paper-level provenance, use the source YAML._
