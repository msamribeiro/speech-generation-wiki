---
concept: subjective-evaluation
render_type: in-depth
title: "Subjective Evaluation of Generated Speech: In Depth"
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

For the concise view, see [[concepts/subjective-evaluation|Subjective Evaluation of Generated
Speech]]. This page synthesizes conclusions rather than listing studies; complete provenance remains
in `wiki/_claims/subjective-evaluation.yaml`.

## Findings at a Glance

- Naturalness, intelligibility, identity, prosody, emotion, and interaction success are distinct
  perceptual dimensions and should not be collapsed into one score.
- Automatic metrics frequently rank systems differently from listeners; learned predictors improve
  aggregate correlation but remain domain- and scale-sensitive.
- MOS results depend on listener population, language, cultural context, instructions, references,
  anchors, and recruitment.
- Pairwise and ranking designs often discriminate small differences better than isolated absolute
  ratings, while providing only relative conclusions.
- Preference optimization improves targeted perception but can damage untargeted dimensions when
  rewards or stopping criteria are incomplete.
- Embedding-based speaker similarity is especially confounded and remains emerging as a direct
  substitute for perceived identity.
- Attribute control requires attribute-specific listening tests; naturalness alone cannot validate
  emotion, accent, prosody, or style.
- Domain and language shift can invalidate conclusions drawn from clean, high-resource conditions.
- Participatory evaluation reveals cultural, educational, and usability outcomes that scalar scores
  omit.
- LLM and audio-language-model judges are promising for scale but contested for fine-grained and
  non-verbal attributes.

## Scope

The graph contains 180 papers published from 2016 through September 2025; 160 are from 2025. TTS is
the largest task, followed by evaluation-focused work, spoken conversational agents, voice
conversion, codecs, and singing. Every paper supplies core evidence because subjective judgments
are used to support system claims, but only a subset studies evaluation methodology directly.

The evidence includes controlled metric studies, listening tests, preference-training experiments,
benchmark proposals, low-resource evaluations, and participatory workshops. Fifteen clusters are
strongly supported, one is emerging, and one is contested. Results from repeated use of the same
metric or listener platform do not constitute independent validation of that instrument. Historical
papers establish common protocols; surveys and benchmarks organize evidence but do not replace
listener experiments.

The synthesis distinguishes a protocol's ability to discriminate the submitted systems from its
validity for a deployment population. A test can be statistically consistent among recruited
listeners and still ask the wrong question, omit a relevant group, or fail under language and domain
shift. It also distinguishes automatic prediction of ratings from human evaluation itself.

## Research Landscape

Subjective evaluation cuts across generation architectures rather than forming architecture-based
method families. The claim graph records autoregressive, transformer encoder–decoder, GAN,
diffusion, flow, variational, and hybrid systems because protocol behavior must be interpreted across
the systems being compared. Architecture membership provides context for artifacts, not a taxonomy
of listening methods.

The core protocol families are absolute ratings, pairwise or ranking judgments, reference-based
attribute comparisons, task-specific rubrics, and participatory qualitative methods. Absolute MOS
compresses an impression into a familiar scale. Pairwise tests ask which sample is preferred or
better on one attribute. Reference-based designs compare identity, accent, emotion, or prosody
against an anchor. Spoken-agent evaluations add semantic, interactive, and temporal rubrics.
Participatory work asks whether the system is useful and appropriate in its community context.

Automatic evaluation forms a parallel layer. Classical signal measures compare waveforms or
spectra; learned quality predictors estimate listener ratings; ASR and speaker encoders provide
intelligibility and identity proxies; LLM and audio-language models increasingly judge content,
emotion, or interaction. These instruments can reduce evaluation cost, but each inherits a training
distribution and operational definition.

## What the Research Shows

### Perception is multidimensional

**Current assessment:** naturalness, intelligibility, identity, expressiveness, and interaction
behavior are separate axes; gains on one do not guarantee gains on another.

This conclusion recurs from foundational TTS and vocoder work through modern zero-shot and spoken-
agent systems. A model can be intelligible but sound unnatural, resemble a speaker while carrying
the wrong accent, or generate fluent audio while failing the conversational task. Studies such as
[[2406.18009|E2 TTS]] and preference-aligned TTS report multiple metrics precisely because one axis
cannot stand in for the others.

The implication is structural: evaluations should start from the system claim, then select a test
for each relevant dimension. Composite headline scores may aid dashboards, but component results
must remain visible. Otherwise an improvement in a high-variance or heavily weighted dimension can
hide regression elsewhere.

### Automatic metrics diverge from listeners

**Current assessment:** automatic quality and task metrics frequently rank systems differently from
human perceptual judgments, with disagreement varying by task, domain, and quality range.

Evidence spans classical signal metrics, ASR-based intelligibility, speaker embeddings, and learned
quality models. Disagreement is especially consequential near high quality, where predictors can
saturate while listeners still hear artifacts. A metric may also reward properties correlated with
its training data rather than the intended perceptual construct.

This does not make automatic metrics useless. They support rapid iteration, large-scale screening,
and reproducible diagnostics. The appropriate use is calibration: validate the metric against human
judgments under the intended domain, then retain periodic listening tests and inspect cases where
metrics disagree.

### Learned quality predictors improve but do not solve validity

**Current assessment:** learned perceptual predictors generally correlate better with human ratings
than classical signal measures, but aggregate correlation conceals utterance-level errors, domain
shift, and scale sensitivity.

[[2204.02152|UTMOS]] exemplifies the learned-MOS approach. Later mixture-of-experts and embedding
models target robustness across datasets. Their advantage comes from learning perceptual patterns
instead of assuming waveform distance equals quality.

Correlation is not interchangeability. A predictor trained on one rating scale or language can
shift systematically elsewhere. System-level correlation can remain high while individual samples
are unreliable. Predicted MOS must be labeled as predicted, and a new architecture or domain should
trigger revalidation rather than automatic trust.

### MOS is context-sensitive

**Current assessment:** MOS remains useful for broad naturalness comparisons when rigorously
reported, but its numeric value depends on population and protocol.

Listeners use rating scales relative to the samples, anchors, and expectations they encounter.
Native-language competence affects pronunciation judgments; cultural experience shapes style and
social perception; instructions determine whether “quality” means naturalness, pleasantness, or
technical cleanliness. Reference conditions and recruitment platforms also shift scores.

Low-resource work such as [[2025.americasnlp-1.1|Shipibo-Konibo TTS]] shows why importing a standard
protocol can be inappropriate when native listeners are scarce and community goals differ. Strong
reporting includes listener language, demographics relevant to the task, screening, compensation,
instructions, randomization, anchors, sample assignment, uncertainty, and exclusion rules.

### Pairwise tests improve discrimination

**Current assessment:** pairwise preference, ranking, and contrastive designs often detect
perceptual differences more reliably than isolated absolute scores.

Pairwise judgments reduce variation in how listeners use an absolute scale and focus attention on a
specified attribute. They are central to preference training and system comparison, from
[[2406.00654|Enhancing Zero-Shot TTS with Human Feedback]] to modern benchmark arenas.

Their limitation is relativity. Winning against one baseline does not establish acceptable
absolute quality, and the comparison set can change rankings. Studies should select meaningful
baselines, balance presentation order, report ties or indifference when allowed, and avoid turning a
pairwise win rate into an absolute quality claim.

### Preference training exposes reward trade-offs

**Current assessment:** preference-based post-training can improve targeted perceptual attributes,
while optimization against incomplete human or proxy rewards can degrade unmeasured qualities.

[[2025.acl-long.598]] improves zero-shot intelligibility through preference alignment. Other work
targets emotion, prosody, identity, or conversational coherence. The repeated positive direction
establishes preference optimization as more than a single-system result.

The same literature records naturalness, identity, melody, or diversity regression when one reward
is pushed too far. Proxy-generated preferences can amplify metric bias; even human labels cover
only the question asked. Safe evaluation therefore reserves held-out dimensions, uses multi-
objective checkpoint selection, and continues listening after the optimized reward improves.

### Speaker similarity needs perceptual calibration

**Current assessment:** embedding similarity is useful within controlled protocols but is an
emerging and confounded substitute for perceived identity.

[[2507.20140|Do Not Mimic My Voice]] and [[2507.02176|Analyzing and Improving Speaker Similarity
Assessment]] show that recording conditions, language, architecture, and reference design influence
scores. Two embeddings can rank systems differently, while neither captures every identity cue a
listener uses.

Speaker verification and synthesis evaluation ask related but different questions. Verification
seeks discriminative identity under its training distribution; cloning asks whether generated
speech sounds like a reference while remaining natural and linguistically correct. Calibration
should use listener identity judgments across languages, channels, and demographic groups.

### Attribute control needs targeted tests

**Current assessment:** emotion, prosody, accent, rhythm, and style require dimension-specific
perceptual tests rather than holistic naturalness scores.

[[2025.coling-main.352|DiffStyleTTS]] and related control systems demonstrate hierarchical or
fine-grained variation. A naturalness test can confirm that controlled output remains plausible but
cannot show whether the intended attribute changed. Attribute labels, anchors, and listener cultural
competence become part of construct validity.

The strongest design measures the target attribute and preservation dimensions together. Increasing
control strength may improve recognizability while reducing identity or naturalness. Reporting an
operating curve reveals this non-monotonic behavior better than one selected checkpoint.

### Domain and language shift limit transfer

**Current assessment:** subjective conclusions from clean, high-resource, single-language settings
often fail to transfer to low-resource or in-the-wild use.

Domain shift changes recording conditions, text, speaker population, and listener expectations at
once. [[2025.chipsal-1.18|Nepali TTS Vocoder Study]] and low-resource dataset work show that choices
validated on common English corpora may rank differently elsewhere. Cross-domain comparisons are
hard precisely because several factors change together.

Evaluation should sample deployment conditions and stratify results rather than rely on one pooled
mean. Native-language listeners and context-appropriate materials are essential when pronunciation
or cultural style is part of the claim.

### Bias and participatory validity matter

**Current assessment:** demographic representation and social perception produce systematic
differences in quality and identity judgments, while participatory methods reveal consequences that
scalar laboratory tests omit.

Training-data imbalance, listener composition, stereotypes, and acoustic confounds can all affect
ratings. [[1904.02882|LibriTTS]] provides widely used data whose population shapes downstream
systems; later studies examine perceived likability, gendered voice qualities, and disparities.

[[2025.computel-main.6|Participatory Ojibwe TTS Evaluation]] shows a complementary objective:
community members assess educational usefulness and cultural fit, not merely acoustic naturalness.
Small participatory studies trade population-level generalization for contextual validity. They
should complement, not be forced into, standard MOS aggregation.

Their reporting logic is correspondingly different. A laboratory listening test estimates a
population response under standardized sampling assumptions; a workshop can reveal requirements,
failure meanings, and adoption constraints that were absent from the laboratory question. Strong
participatory reports document who set the criteria, how community members used the system, which
themes recurred, and where views differed. They should avoid claiming population prevalence from a
small group, while conventional studies should avoid treating a larger anonymous panel as evidence
of cultural appropriateness. Using both methods produces complementary validity: quantitative
comparability for perceptual differences and situated evidence about whether those differences
matter in practice.

### Automated judges remain contested

**Current assessment:** LLM and audio-language-model judges approximate some human judgments but are
not uniformly reliable for fine-grained, non-verbal, or paralinguistic attributes.

Benchmarks such as [[2508.02013|SpeechRole]] and [[2508.18240|MTalk-Bench]] use rubric-based or arena
evaluation for spoken systems. Automated judges scale semantic and interaction assessment beyond
small panels. Repeated sampling and specific rubrics can improve stability.

The contested evidence, including [[2509.19928|Prosody Diversity Benchmark]], shows that reliability
varies with modality and attribute. A text-capable judge may assess semantic compliance while
missing subtle timing, voice identity, or affect. Human audits and per-dimension reliability should
be reported before replacing listeners.

### Efficiency claims require listening tests

**Current assessment:** human listening is necessary to establish whether codec, vocoder, pruning,
or acceleration gains preserve perceived quality.

Foundational systems such as [[1609.03499|WaveNet]], [[2010.05646|HiFi-GAN]],
[[2105.06337|Grad-TTS]], and [[2210.13438|EnCodec]] pair efficiency or architectural claims with
perception because objective measures do not capture all artifacts. Later work varies model size,
steps, hardware, and data together, which limits causal attribution.

Matched listening tests should compare operating points at explicit latency, bitrate, or compute.
Without that control, “same quality” may mean only that one automatic metric changed little.

### Benchmarks need multidimensional coverage

**Current assessment:** speech benchmarks need rubrics that separate semantic success,
naturalness, identity, prosody, and interaction, while preserving component scores and judge
reliability.

Spoken systems make the limitation of one-dimensional quality clear. A response can be semantically
correct but interrupt the user, use an inappropriate voice, or fail to signal turn completion.
[[2506.23049|AURA]], [[2025.acl-long.911|DNASpeech]], and full-duplex benchmark work encode
different combinations of task completion, situated behavior, audio quality, and timing.

Hierarchical rubrics improve diagnosis by asking narrow questions at the right level. Arena-style
comparisons can aggregate preferences without pretending every failure shares one scale. The danger
is hidden compensation: a composite leaderboard may mask a critical identity, safety, or latency
failure. Benchmarks should publish per-dimension scores, uncertainty, missing judgments, and judge
reliability for each rubric.

### Control strength is perceptually non-monotonic

**Current assessment:** stronger guidance, compression, transformation, or attribute control does
not produce monotonic perceptual improvement; useful evaluation reports an operating curve rather
than one selected setting.

Across style control, voice conversion, codecs, and preference training, increasing intervention
strength can clarify the target attribute while degrading naturalness, identity, or intelligibility.
[[2507.04817|Fast-VGAN]] illustrates explicit pitch and duration control, while related studies show
that perceptual optima differ by attribute and system.

Selecting one point after inspecting the test set can conceal the trade-off. Predefined settings,
multiple strengths, and dimension-specific listening reveal whether gains are robust or a narrow
optimum. When deployment exposes a user control, evaluation should cover its usable range and
failure boundaries.

Operating curves also make disagreement interpretable. Two listener groups may prefer different
strengths without disagreeing about the underlying samples, and a deployment may select a different
point for assistive reading than for expressive entertainment. Publishing the curve, listener
strata, and selection rule preserves that information instead of turning a context-dependent choice
into a universal system ranking.

## Where Findings Disagree

The primary contested cluster concerns automated LLM and audio-language-model judges. Some studies
find useful agreement on structured semantic or role-playing rubrics; others show unreliable
fine-grained prosody judgments. The narrow conclusion is task-specific substitutability, not a
general replacement for listeners.

Speaker embedding similarity is emerging rather than contested because direct support is still
thin. MOS and pairwise tests are not universal competitors: MOS supports broad absolute
interpretation, while pairwise tests support discrimination. The appropriate design depends on the
claim and quality range.

## How the Field Is Changing

Subjective evaluation has expanded from aggregate naturalness MOS toward multidimensional identity,
prosody, emotion, interaction, and social-perception tests. Pairwise preferences became prominent
in 2024–2025 as both evaluation data and training signals, exposing cross-dimension reward trade-offs.

Learned predictors and multimodal judges increasingly scale evaluation, while evidence of domain
shift and saturation has grown alongside them. Low-resource and participatory studies adapt
protocols to native-listener availability and cultural goals. Spoken-agent benchmarks add
hierarchical rubrics, arenas, and query-specific checklists beyond audio quality.

## Implications

- Define perceptual constructs before selecting metrics or listener questions.
- Report naturalness, intelligibility, identity, expressiveness, and interaction separately.
- Calibrate automatic metrics and speaker encoders against the intended population and domain.
- Use pairwise tests for discrimination without overstating absolute quality.
- Reserve held-out perceptual dimensions when preferences become training rewards.
- Report listener recruitment, language, instructions, anchors, assignment, and uncertainty.
- Stratify demographic and language results where pooling could conceal disparities.
- Combine laboratory listening with participatory or in-context evaluation when social use matters.
- Audit automated judges by attribute and retain human review for paralinguistic claims.

## Representative Reading Path

1. **Understand the naturalness baseline.** Read [[1609.03499|WaveNet]] and
   [[2010.05646|HiFi-GAN]] for the role of listening tests in foundational synthesis and vocoding.
2. **Study learned quality prediction.** Read [[2204.02152|UTMOS]] while separating system-level
   correlation from utterance-level validity.
3. **Compare absolute and preference protocols.** Pair [[2105.06337|Grad-TTS]] with
   [[2406.00654|Enhancing Zero-Shot TTS with Human Feedback]].
4. **Examine multidimensional alignment.** Read [[2025.acl-long.598]] and
   [[2507.02176|Analyzing Speaker Similarity Assessment]].
5. **Add community context.** Read [[2025.americasnlp-1.1|Shipibo-Konibo TTS]] and
   [[2025.computel-main.6|Participatory Ojibwe TTS Evaluation]].
6. **Evaluate spoken agents and automated judges.** Use [[2508.02013|SpeechRole]],
   [[2508.18240|MTalk-Bench]], and [[2509.19928|Prosody Diversity Benchmark]].

## Structured Source

Complete structured claims and provenance are in
[`wiki/_claims/subjective-evaluation.yaml`](../_claims/subjective-evaluation.yaml).

---

_This page is generated from `wiki/_claims/subjective-evaluation.yaml` (digest date: 2026-07-27)._
