---
title: "Field Overview"
source_digest_date: 2026-07-22
source_concepts: [flow-matching, speech-to-speech, evaluation-metrics, rlhf-speech, disentanglement]
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
> Across the five concepts currently integrated in depth, speech generation is becoming faster,
> more interactive, and more controllable—but improvements increasingly depend on whether systems
> preserve capabilities that their objectives and benchmarks do not directly measure. This is a
> partial field view derived from the reviewed corpus, not an exhaustive account of speech
> generation research.

> [!warning]
> **Work in progress.** This synthesis currently stops at Q3 2025 and includes only the five
> completed concept integrations listed below. Later papers in the catalog have not necessarily
> reached the claim graphs or this field-level account.

## What the Integrated Corpus Shows

### Generation speed

Within the reviewed flow-matching literature, continuous-output generation offers a practical
speed advantage over conventional diffusion across TTS, voice conversion, vocoding, and related
representations. Few-step methods now use several distinct mechanisms, including distillation,
learned priors, scheduling, caching, and specialized solvers. Their headline step counts are not
interchangeable: lower compute can reduce naturalness, prosodic diversity, memory efficiency, or
training simplicity in different ways ([[2025.acl-long.313|F5-TTS]],
[[2025.acl-long.1043|OZSpeech]], [[2507.14988|DMOSpeech 2]]).

The same distinction appears in spoken interaction. Full-duplex systems can listen while speaking
and respond rapidly to interruption, but low latency alone does not produce good turn-taking.
False yielding, premature takeover, weak backchannels, and loss of semantic coherence remain
visible failure modes ([[2507.23159|Full-Duplex-Bench v1.5]], [[2509.22243|FLEXI]]). Across these
concepts, speed is best understood as one dimension of capability rather than a proxy for overall
system quality.

### Modularity and capability retention

The reviewed speech-to-speech evidence favors modularity when general language ability matters.
Cascaded ASR–language-model–TTS systems remain strong general-purpose baselines, while systems
that preserve a pretrained text backbone and attach a speech decoder often retain reasoning and
instruction-following more reliably than jointly trained speech models
([[2410.17196|VoiceBench]], [[2411.00774|Freeze-Omni]],
[[2025.findings-acl.1051|LLMVoX]]).

Flow-matching systems reveal a related choice. Pure non-autoregressive generation offers parallel
decoding, whereas autoregressive-plus-flow hybrids separate semantic or stylistic planning from
acoustic rendering at the cost of sequential latency. The relevant question is therefore not
simply whether a model is “end to end,” but which capabilities are coupled, which components can
be trained independently, and where information is discarded.

### Control and separation

Disentanglement makes independent control of content, speaker identity, prosody, emotion, and style
practical. Explicit factorization supports attribute transfer and substitution across TTS, voice
conversion, codecs, and expressive generation ([[2403.03100|NaturalSpeech 3]],
[[2025.acl-long.346|ControlSpeech]], [[2508.02038|Marco-Voice]]). Yet tighter separation repeatedly
costs reconstruction quality, intelligibility, speaker similarity, or useful phonetic detail.

This evidence argues against treating maximum independence as the goal. A more practical target is
enough separation for the intended control while preserving the information required by the
downstream generator. Pitch illustrates the difficulty: generic latent separation often leaves F0
entangled, while dedicated signal-level mechanisms perform better in the systems represented here
([[interspeech-2025-0347|PeriodCodec]], [[interspeech-2025-1397|VibE-SVC]]). Whether that advantage
generalizes beyond the current mechanisms remains unresolved.

### Post-training and reward scope

Across the young RLHF-for-speech corpus, preference and reward optimization improve the dimensions
they explicitly target across discrete, flow, diffusion, and hybrid generators. The recurring
problem is reward scope. Better WER, likelihood, speaker similarity, or reward-model scores can
coexist with flatter prosody, reduced diversity, weaker emotional appropriateness, or evasive
dialogue behavior ([[2025.acl-long.598]], [[2509.18531]],
[[2508.16332|Vevo2]]).

Multi-objective rewards reduce some one-dimensional failures, but the literature does not yet
establish principled weighting or a general defense against reward hacking. Post-training also
cannot reliably create capabilities absent from the base model or repair qualities controlled by
a frozen downstream acoustic model or vocoder. These limitations connect alignment directly to
evaluation: a system can only be optimized safely when the evaluation suite represents the
qualities that should be preserved.

### Evaluation

The broadest conclusion across the integrated concepts is that no single score captures speech
quality. Naturalness, intelligibility, speaker identity, prosody, diversity, emotion, robustness,
latency, and interaction quality can move independently. WER and CER may disagree with perceived
intelligibility; speaker embeddings can mis-rank synthetic voices; automatic MOS predictors can
reverse human rankings; and reconstruction quality does not necessarily predict whether codec
tokens are useful for generation ([[2403.16973|VoiceCraft]], [[2406.18009|E2 TTS]],
[[2025.naacl-srw.6]], [[2506.10274]]).

The measurement gap becomes larger for spoken agents. Transcript-based evaluation discards timing,
emotion, speaker traits, ambient sound, and non-verbal behavior, while single-turn tests conceal
failures that accumulate over a conversation ([[2508.18240|MTalk-Bench]],
[[2507.23159|Full-Duplex-Bench v1.5]]). Learned judges make broad evaluation more scalable, but the
reviewed evidence supports capability-specific use rather than treating them as general substitutes
for listening or interaction tests.

## Shared Research Priorities

Across the five completed integrations, several questions recur:

- How can faster generation preserve naturalness, speaker identity, and prosodic diversity?
- Which forms of modularity preserve language capability without discarding useful acoustic
  information?
- How much attribute separation is sufficient for control, and how should residual leakage be
  measured directly?
- Can preference optimization protect qualities that are absent from its reward?
- Which evaluation suites remain valid across architectures, domains, languages, and multi-turn
  interaction?

## Read by Concept

| Concept | Overview | In Depth |
|---------|----------|----------|
| Flow Matching | [[concepts/flow-matching\|Overview]] | [[concepts/flow-matching-in-depth\|In Depth]] |
| Speech-to-Speech Systems | [[concepts/speech-to-speech\|Overview]] | [[concepts/speech-to-speech-in-depth\|In Depth]] |
| Evaluation Metrics | [[concepts/evaluation-metrics\|Overview]] | [[concepts/evaluation-metrics-in-depth\|In Depth]] |
| RLHF for Speech | [[concepts/rlhf-speech\|Overview]] | [[concepts/rlhf-speech-in-depth\|In Depth]] |
| Disentanglement | [[concepts/disentanglement\|Overview]] | [[concepts/disentanglement-in-depth\|In Depth]] |

## Scope

This overview draws only from the five claim graphs completed through Q3 2025: flow matching
(97 papers), speech-to-speech systems (60), evaluation metrics (285), RLHF for speech (29), and
disentanglement (100). Counts overlap because papers can inform multiple concepts. Eighteen tracked
concepts have not yet completed integration, so this page should not be read as a comprehensive
ranking of paradigms or a full history of the field. Representative citations are selected for
readability; complete paper-to-claim provenance remains in the corresponding `_claims/*.yaml`
files.

The coverage boundary will advance through Q4 2025 and later periods until it catches up with
current research. Future field views will be complemented by period-over-period reports,
venue-specific synthesis, and citation analysis.
