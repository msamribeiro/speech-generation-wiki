---
slug: subjective-evaluation
concept: subjective-evaluation
render_type: overview
title: Subjective Evaluation of Generated Speech
aliases: [listening tests, perceptual evaluation, MOS evaluation, human evaluation]
status: established
last_reviewed: 2026-08-02
source_digest_date: 2026-07-27
paper_count: 180
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
> Subjective evaluation measures how listeners perceive generated speech: naturalness,
> intelligibility, identity, prosody, emotion, interaction, and social suitability. Human judgment
> remains necessary because automatic metrics and learned judges miss important failures, but the
> result depends strongly on the question, listeners, references, anchors, and deployment context.

## Current State

**There is no single perceptual quality axis.** Naturalness, intelligibility, speaker identity,
expressiveness, and interaction success are distinct; improvement on one does not guarantee another.
Holistic MOS is useful for broad naturalness comparisons but insufficient for controllable,
multilingual, cloning, and spoken-dialogue systems. Dimension-specific tests are now core evaluation
infrastructure.

**Automatic metrics frequently disagree with listeners.** Learned quality predictors generally
correlate better than classical signal metrics, yet remain sensitive to domain, quality range, and
rating scale. Speaker embeddings can be confounded by recording, language, and architecture.
Automatic scores are useful screening instruments, not interchangeable substitutes for perceptual
evidence ([[2204.02152|UTMOS]], [[2507.02176|Analyzing Speaker Similarity Assessment]]).

**Protocol design changes the conclusion.** Listener language, cultural context, hearing profile,
reference conditions, instructions, anchors, and recruitment affect MOS-style results. Pairwise,
ranking, and contrastive designs often discriminate small differences more reliably than isolated
absolute ratings, but remain relative to the selected alternatives.

**Preference optimization turns evaluation into a training objective.** Human or proxy preferences
can improve a targeted perceptual dimension beyond supervised training, while optimizing one reward
can degrade naturalness, identity, or prosodic diversity. Evaluation must therefore include held-out
dimensions that were not optimized ([[2406.00654|Enhancing Zero-Shot TTS with Human Feedback]],
[[2025.acl-long.598]]).

**Context and population validity are gaining importance.** Low-resource, Indigenous-language, and
participatory studies show that imported English-centric protocols can miss educational, cultural,
and community requirements. Spoken-agent benchmarks similarly need rubrics for turn-taking,
semantic success, and interaction behavior rather than speech naturalness alone.

**Benchmarks are becoming multidimensional.** Newer evaluations separate semantic success,
naturalness, identity, prosody, and interaction instead of compressing them into one leaderboard.
Composite scores remain risky unless component results and judge reliability stay visible; a system
can otherwise compensate for a severe failure on one axis with small gains elsewhere.

## Method Landscape

- **Absolute category ratings** include MOS-style naturalness or quality judgments. They are easy to
  aggregate but sensitive to scale use, context, and listener population.
- **Pairwise preference and ranking** increase discrimination and support preference training. They
  establish relative order within a comparison set, not absolute acceptability.
- **Reference-based attribute tests** measure identity, accent, emotion, prosody, or similarity
  against anchors. Reference selection can introduce its own confounds.
- **Task and interaction rubrics** evaluate semantic completion, turn-taking, helpfulness, or role
  behavior in spoken systems. Multidimensional reporting is essential.
- **Participatory and qualitative methods** add cultural, pedagogical, accessibility, and usability
  context that scalar laboratory scores omit.

## Key Trade-offs

- **Scale versus diagnosticity:** one MOS number is cheap to compare but hides which dimension
  changed.
- **Absolute interpretation versus discrimination:** pairwise tests detect small differences while
  depending on the chosen baseline.
- **Laboratory control versus ecological validity:** clean prompts and trained listeners improve
  consistency but may not predict in-the-wild use.
- **Automation versus perceptual coverage:** learned judges reduce cost while failing under domain
  shift or fine-grained paralinguistic tasks.
- **Aggregate power versus population fairness:** pooling listeners increases sample size but can
  conceal language, demographic, and cultural disparities.

## Open Questions

- Which protocol best balances ecological validity, diagnostic resolution, cost, and replication?
- How should listener language, demographics, hearing profile, and cultural context enter analysis?
- When can learned or LLM-based judges reduce human testing safely?
- How can preference training improve one attribute without suppressing unmeasured dimensions?
- Which references and anchors make identity, prosody, and style comparable across systems?

## Go Deeper

Read [[concepts/subjective-evaluation-in-depth|Subjective Evaluation of Generated Speech: In
Depth]] for protocol choices, metric disagreement, preference training, bias, participatory methods,
spoken-agent rubrics, and practical study design.

## Scope

This synthesis covers 180 papers from 2016 through September 2025, including 160 from 2025. It
spans TTS, evaluation research, spoken agents, VC, codecs, and singing. Fifteen clusters are strongly
supported, one is emerging, and one is contested. The graph is broad but heterogeneous: many papers
use listening tests as support for a system rather than as controlled research on evaluation itself.
Protocol conclusions are strongest where evaluation methods are studied directly across conditions.

---

_This page is generated from `wiki/_claims/subjective-evaluation.yaml` (digest date: 2026-07-27).
For complete structured claims and paper-level provenance, use the source YAML._
