---
concept: zero-shot-tts
render_type: in-depth
title: "Zero-Shot Text-to-Speech: In Depth"
source_digest_date: 2026-07-26
paper_count: 203
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

For the short state-of-the-art view, see [[concepts/zero-shot-tts|Zero-Shot Text-to-Speech]]. This
page synthesizes decision-relevant conclusions; complete paper-to-claim provenance remains in
`wiki/_claims/zero-shot-tts.yaml`.

## Findings at a Glance

- Reference-conditioned systems reliably clone unseen voices without speaker-specific fine-tuning,
  but performance is conditional on prompt quality, duration, and domain match.
- Discrete codec tokens, continuous acoustic targets, and hybrid semantic–acoustic systems offer
  different balances of intelligibility, identity, controllability, and generation cost.
- Broader speaker and acoustic diversity improves generalization, while proprietary data makes the
  architectural contribution to frontier results difficult to isolate.
- Cross-lingual cloning remains vulnerable to language, accent, and prosody leakage, but the failure
  is scope-dependent rather than universal.
- Intelligibility and speaker similarity frequently trade off; neither automatic axis fully
  predicts listener-perceived cloning quality.
- Prompt robustness, long-form alignment, pronunciation modeling, and acceleration are now core
  system requirements rather than peripheral refinements.
- Speaker-encoder choice can change measured similarity and system rankings, limiting comparisons
  across papers.
- Explicit style and prosody control extends cloning beyond timbre transfer, but is rarely tested
  jointly with identity preservation under matched conditions.
- The strongest recent systems combine several stages; their quality does not prove that any one
  component or representation is independently superior.
- Safety evaluation lags capability evaluation despite the direct misuse potential of voice
  cloning.

## Scope

The claim graph contains 203 papers published from 2017 through September 2025. It is strongly
weighted toward recent work: 164 papers are from 2025, 24 from 2024, and ten from 2023. Most entries
study TTS, while voice conversion, spoken conversational agents, codecs, evaluation, and singing
provide adjacent evidence. The graph therefore captures both direct zero-shot synthesis and systems
whose representation, conditioning, or evaluation results materially affect cloning.

The evidence includes nine method families and eighteen concept-level clusters. Sixteen clusters
are strongly supported; two are contested. Strong status means repeated support within this graph,
not that every comparison is matched. Papers often differ in prompt length, training data,
language, speaker encoder, acoustic target, decoder, and listener protocol. Large proprietary
corpora recur among frontier systems, and successor papers from one organization or backbone are
not treated as fully independent confirmations.

“Zero-shot” means inference for an unseen speaker without target-speaker parameter updates. Papers
that use a few target utterances for adapter or low-resource fine-tuning remain relevant to the
boundary of the problem, but their results do not demonstrate zero-shot inference. Likewise,
voice-conversion and singing systems provide evidence about identity and style transfer without
necessarily establishing text-to-speech intelligibility.

The graph contains historical systems such as [[1712.05884|Tacotron 2]] and
[[2010.05646|HiFi-GAN]], but their role is lineage or infrastructure. The modern zero-shot paradigm
is more directly represented by [[2301.02111|VALL-E]], [[2209.03143|AudioLM]], and later continuous
and hybrid systems. Surveys help frame the landscape but do not count as experimental replication.

## Research Landscape

The field is organized less by a single architecture contest than by where it places linguistic
planning and acoustic reconstruction. **Autoregressive speech-token language models**, the largest
family, serialize codec or semantic tokens and condition on a reference prompt. They inherit the
strengths of language modeling and in-context learning, but long token sequences increase latency
and expose repetition, omission, and alignment failures.

**Hybrid autoregressive–non-autoregressive systems** plan semantic or coarse acoustic structure
sequentially and reconstruct detail with flow matching or diffusion. This division can protect
intelligibility while avoiding fully autoregressive waveform-detail prediction. It also creates an
interface: an error in semantic planning cannot be repaired reliably by the acoustic stage, while a
strong decoder can make an inadequate representation sound plausible.

**Non-autoregressive flow-matching systems** form the second largest family. Models such as
[[2025.acl-long.313|F5-TTS]] show that continuous acoustic generation can produce convincing
zero-shot speech without codec-token prediction. Solver schedules, duration modeling, and reference
conditioning govern the quality–latency trade-off. Diffusion systems occupy a related but smaller
family and use iterative denoising rather than a flow trajectory.

**Encoder–decoder, GAN, and continuous-latent approaches** demonstrate that codec LMs are not the
only viable route. Some are attractive for efficient or real-time deployment; others preserve
continuous detail or provide explicit control. **Modular hybrids** combine several of these ideas,
which makes family membership overlap. A taxonomy label identifies the principal generation
pattern, not a clean experimental treatment.

Across families, three cross-cutting choices dominate: how the prompt is represented, how content
and speaker information are separated, and which acoustic details are delegated to a decoder.
Those choices explain more of the recurring evidence than model brand names do.

## What the Research Shows

### Reference conditioning establishes the capability

**Current assessment:** multiple independent architectures preserve unseen-speaker identity from
reference audio without target-specific fine-tuning, establishing zero-shot cloning as a practical
capability within the reviewed corpus.

The evidence spans early speaker-encoder conditioning, codec-language models, continuous flow
systems, encoder–decoder pipelines, and real-time voice conversion. [[2301.02111|VALL-E]] made
in-context codec prompting a central paradigm; [[2025.acl-long.1043|OZSpeech]] demonstrates a
one-step flow formulation; and [[2025.acl-demo.37|RT-VC]] extends reference conditioning into a
real-time conversion setting. The convergence matters because the capability does not depend on one
generator family.

The common result is narrower than “perfect identity cloning.” Studies use different speaker
encoders, prompt distributions, and similarity protocols. Some preserve broad identity while
missing accent, habitual prosody, or recording-specific cues. Others improve embedding similarity
at the cost of linguistic accuracy. Reference conditioning establishes transfer, while the degree
and perceptual meaning of similarity remain evaluation questions.

### Prompt conditions bound fidelity

**Current assessment:** prompt duration, quality, channel, style, and selection materially constrain
cloning; reference processing cannot be treated as neutral input preparation.

Evidence across VALL-E-style, flow, and robust-cloning systems shows degradation with noisy,
out-of-domain, very short, or stylistically mismatched prompts. [[2411.19770|Noro]] specifically
targets noise-robust one-shot conversion, while [[2312.15821|Audiobox]] and
[[2025.acl-long.1043|OZSpeech]] expose different conditioning strategies. The repeated pattern
supports robustness as a system property rather than a benchmark afterthought.

The corpus does not establish a universal minimum number of seconds. Prompt-duration studies vary
in speech rate, phonetic coverage, channel, speaker familiarity, and model architecture. Selection
can matter as much as raw length: a longer but noisy or unrepresentative clip may be worse than a
short clean one. Deployment should therefore validate prompt acceptance, quality estimation, and
fallback behavior under the intended capture conditions.

### Representation choices redistribute errors

**Current assessment:** discrete semantic structure improves linguistic modelability, continuous
targets retain acoustic detail, and hybrid designs balance the two; none dominates on every cloning
dimension.

[[2209.03143|AudioLM]] and [[2301.02111|VALL-E]] illustrate semantic/acoustic or codec-token
hierarchies. Explicit semantic structure helps the generator maintain content and supports
language-model scaling. [[2025.acl-long.65|Autoregressive Speech Synthesis without Vector
Quantization]] shows that continuous targets can support high-fidelity synthesis without discrete
codec prediction. [[2502.07243|Vevo]] separates content, style, and timbre through self-supervised
representations.

These approaches fail differently. A semantic bottleneck may remove fine identity or prosodic
detail. A continuous generator may retain detail while requiring a more expensive decoder or
alignment mechanism. A hybrid system can obtain both, but interfaces create attribution problems:
improved output may come from the semantic planner, acoustic generator, vocoder, or data recipe.
Matched representation studies remain rarer than end-to-end system comparisons.

### Disentanglement enables control but remains incomplete

**Current assessment:** separating content, speaker identity, and prosody improves controllability
and transfer, but downstream success does not prove statistically independent representations.

[[2025.acl-long.346|ControlSpeech]] supports simultaneous speaker and language-style control, while
[[2502.07243|Vevo]] and related factorized systems show how dedicated streams permit attribute
substitution. This enables more than timbre copying: systems can target emotion, rhythm, accent, or
speaking style while retaining a reference identity.

Leakage remains common. A speaker code may carry language or recording cues; an acoustic prompt may
transfer unwanted prosody; a content representation may retain pitch. Classifier probes and
embedding scores reveal some leakage but do not establish causal separation. The practical standard
should be controlled counterfactual behavior—change one attribute while measuring preservation of
the others—under multiple prompts and languages.

### Data scale drives generalization

**Current assessment:** broader speaker, language, and acoustic diversity improves zero-shot
generalization, but data advantages are often inseparable from architecture and curation.

Large-scale systems such as [[2402.08093|BASE TTS]] show capabilities that emerge with broad
training coverage. Low-resource studies, including [[2025.findings-naacl.279|BnTTS]], demonstrate
how adaptation can recover some gaps, while multilingual work shows that coverage matters for
cross-lingual identity. The direction of the evidence is consistent: a model generalizes better to
conditions represented by a diverse pre-training population.

The causal estimate is weak because data scale rarely changes alone. Frontier systems introduce
new generators, larger models, filtering, pronunciation front ends, and preference stages together.
Proprietary datasets prevent matched reproduction. Claims that an architecture is responsible for
quality should therefore be narrower than claims that the complete system achieves it.

### Cross-lingual leakage is scope-dependent

**Current assessment:** cross-lingual cloning frequently transfers unwanted accent, language, or
prosody from the prompt, but at least one contradictory result shows leakage is not inevitable.

Support comes from multilingual control and intelligibility studies including
[[2025.acl-long.346|ControlSpeech]] and [[2025.acl-long.598]]. Language coverage, phonological
distance, text front ends, and prompt language all affect the result. The contradicting evidence in
[[2509.19928|Prosody Diversity Benchmark]] prevents a universal formulation and suggests that model
or evaluation conditions can suppress the failure.

The useful conclusion is diagnostic: cross-lingual evaluation must measure pronunciation, accent,
identity, and prosody separately. A single similarity score can reward retention of language cues
that listeners perceive as incorrect accent transfer. Balanced language-pair matrices are more
informative than one source–target direction.

### Intelligibility and identity do not move together

**Current assessment:** zero-shot systems frequently trade linguistic accuracy against speaker
similarity, and apparent improvements depend on guidance settings and metric choice.

[[2025.acl-long.313|F5-TTS]], [[2025.acl-long.598]], and codec-LM analyses show that reducing WER can
change similarity or prosodic diversity. Stronger text or semantic conditioning improves
pronunciation, while stronger prompt conditioning can preserve identity at the cost of content
errors. This is not an immutable law, but a recurring operating curve.

Evaluation should report both axes at multiple settings rather than select one checkpoint using a
composite score. Speaker-encoder choice itself changes rankings, as demonstrated by benchmark work
such as [[2504.20581|ClonEval]]. Human identity judgments are needed to calibrate automatic
embeddings, especially across languages and recording conditions.

### Speed and long-form robustness are coupled

**Current assessment:** acceleration can preserve usable short-form quality, while long-form
generation requires explicit alignment, chunking, or memory controls to prevent accumulated
errors.

Flow solvers, non-autoregressive acoustic stages, lower token rates, and parallel decoding reduce
latency. Comparisons are difficult because papers report real-time factor, time to first audio, or
end-to-end latency on different hardware. An acceleration result supports its evaluated mechanism,
not a universal speed ranking.

Long-form synthesis adds failures invisible in sentence benchmarks: repetition, skipped text,
speaker drift, discontinuities at chunk boundaries, and growing misalignment. Hierarchical systems
such as [[iclr-2025-868masI331|HALL-E]] and systems with explicit memory or monotonic alignment show
the value of structure. Robustness should be measured as failure rates over duration, not only
average short-utterance quality.

### Style control expands the target

**Current assessment:** zero-shot cloning increasingly targets style, emotion, rhythm, and prosody
in addition to timbre, but joint control-and-preservation evaluation remains inconsistent.

[[2025.acl-long.346|ControlSpeech]] and [[2025.findings-acl.687|TCSinger 2]] illustrate the move
toward independent style and identity control across speech and singing. The advance changes what
“successful cloning” means: matching a speaker embedding while flattening their prosody is no
longer sufficient for expressive applications.

Studies often measure the controlled attribute and identity in separate experiments. The stronger
test holds text, speaker, language, and prompt constant while varying the requested style, then asks
listeners whether the intended attribute changed without identity loss. Standardized protocols for
that test remain an open need.

### Synthetic data and safeguards

**Current assessment:** synthetic and pseudo-labeled speech can broaden training coverage, but the
evidence remains smaller than for real-data scaling and does not resolve error reinforcement or
misuse risk.

[[2406.02430|Seed-TTS]] and [[2209.03143|AudioLM]] contribute evidence that generated or weakly
labeled material can extend conditions seen during training. This is attractive for languages,
speakers, and styles scarce in curated corpora. Its value depends on filtering: synthetic
pronunciation, speaker leakage, or flattened prosody can be learned again by the target model. The
graph supports synthetic expansion as an input source, not an automatic substitute for diverse
recordings.

The same capability that makes personalization accessible creates authorization and identity
risks. [[2507.20140|Do Not Mimic My Voice]] examines speaker-identity unlearning, while protection
work tests prompt or content defenses. These papers show that safety can be an explicit objective,
but the graph does not establish one safeguard that survives every architecture or adversarial
prompt. Capability studies should test protected-speaker attacks and measure whether safeguards
harm legitimate accessibility uses; lower average similarity alone is not a safety evaluation.

## Where Findings Disagree

The graph records two contested conclusions. Cross-lingual leakage is supported broadly but not
universally. Automatic metrics often diverge from human perception, yet one study reports stronger
agreement under its protocol. These contradictions narrow the claims: leakage and metric mismatch
depend on the language pair, model, quality range, metric, and listener task.

The corpus does not identify a universal winner between discrete and continuous targets.
End-to-end systems change too many factors simultaneously. Nor does it establish one optimal prompt
length, speaker encoder, solver, or semantic/acoustic split. The absence of a single answer is a
property of the evidence, not an invitation to average incomparable numbers.

## How the Field Is Changing

From 2023 through Q3 2025, the center shifted from encoder–decoder and GAN-centered cloning toward
autoregressive speech-token models, flow matching, and hybrid autoregressive-plus-continuous
pipelines. The shift is architectural, but also reflects much larger and more diverse data.

The target has expanded from short English sentences and timbre similarity toward cross-lingual,
expressive, streaming, and long-form synthesis. Robust prompt handling and alignment are now first-
class requirements. Evaluation has not advanced as uniformly: WER or CER and one speaker embedding
still dominate many reports, while identity perception and prompt robustness remain fragmented.

## Implications

- Validate reference capture as part of the product, including noise, channel, duration, and style.
- Report intelligibility, perceived identity, prosody, and naturalness separately.
- Treat speaker encoders as measurement instruments requiring calibration, not ground truth.
- Attribute gains cautiously when architecture, data scale, filtering, and preference alignment all
  change together.
- Evaluate long-form failure rates and streaming latency using explicit definitions and matched
  hardware.
- Separate zero-shot inference from few-shot adaptation in both claims and comparisons.
- Add misuse resistance and authorization tests alongside quality benchmarks.

## Representative Reading Path

1. **Understand codec prompting.** Read [[2301.02111|VALL-E]] with [[2209.03143|AudioLM]] to see
   why semantic and acoustic tokens made in-context voice conditioning practical.
2. **Compare a continuous alternative.** Use [[2025.acl-long.313|F5-TTS]] and
   [[2025.acl-long.65|Autoregressive Speech Synthesis without Vector Quantization]] to examine what
   changes when codec-token prediction is removed.
3. **Study independent control.** Read [[2025.acl-long.346|ControlSpeech]] and
   [[2502.07243|Vevo]] for content, identity, language-style, and prosody factorization.
4. **Interrogate generalization.** Pair [[2402.08093|BASE TTS]] with
   [[2025.findings-naacl.279|BnTTS]] to contrast scale-driven zero-shot coverage and low-resource
   adaptation.
5. **Audit evaluation.** Use [[2504.20581|ClonEval]] and [[2509.19928|Prosody Diversity Benchmark]]
   to test what speaker embeddings and aggregate quality scores omit.
6. **Extend to long-form and real time.** Read [[iclr-2025-868masI331|HALL-E]] and
   [[2025.acl-demo.37|RT-VC]] for duration and latency constraints beyond sentence-level cloning.

## Structured Source

Complete structured claims and provenance are available in
[`wiki/_claims/zero-shot-tts.yaml`](../_claims/zero-shot-tts.yaml).

---

_This page is generated from `wiki/_claims/zero-shot-tts.yaml` (digest date: 2026-07-26)._
