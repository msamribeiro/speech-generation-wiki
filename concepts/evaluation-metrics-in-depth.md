---
concept: evaluation-metrics
render_type: in-depth
title: "Evaluation Metrics: In Depth"
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

This page explains what the reviewed evaluation research supports. For the short version, see
[[concepts/evaluation-metrics|Evaluation Metrics]]. The citations are representative. The complete
paper-to-claim graph is in `wiki/_claims/evaluation-metrics.yaml`.

## Findings at a Glance

- No single metric measures overall speech-generation quality. Useful evaluation needs several
  dimensions.
- WER and CER are useful content checks, but they do not reliably measure human intelligibility.
- Speaker-verification embeddings do not fully represent perceived identity.
- Automatic MOS predictors and reference-based distortion metrics lose reliability under domain
  shift, at fine granularity, and near the quality ceiling.
- LLM and audio-LM judges help with structured semantic or rubric-based tests. They remain weak on
  fine prosody and non-verbal audio.
- Clean, in-domain benchmarks hide failures that appear in realistic conditions.
- Codec reconstruction quality does not predict semantic retention or downstream generation.
- Standard listening tests are becoming less able to separate high-quality systems. Proposed
  replacements are not yet standard.
- Fairness, clinical suitability, full-duplex timing, and multi-turn behavior need dedicated tests.
- Improving one measured property can reduce diversity or another unmeasured quality.
- Watermarking, accent, demographic fairness, and assistive speech need tests designed for their
  target populations and conditions.
- Spoken systems can sound natural while still trailing similar text models in reasoning and
  dialogue state.

## Scope

The graph contains 346 papers through Q4 2025. It is the largest integrated concept in the wiki.
It covers TTS, voice conversion, neural codecs, vocoders, spoken agents, full-duplex systems,
clinical and accessibility applications, watermarking, and low-resource languages.

Most entries are not metric papers. Many are system papers whose experiments expose a test
failure. A metric can reverse a human ranking. A strong reconstruction score can fail to predict
downstream use. A benchmark can miss a capability that matters in deployment. The method families
therefore group papers by evaluation purpose, not by model architecture.

The breadth supports conclusions across tasks, but it prevents one numerical meta-analysis.
Metrics, datasets, listening tests, and target populations differ too much. Surveys support the
taxonomy. Primary studies provide direct evidence. Large families such as objective-metric
divergence show that the same warning occurs across unrelated systems; they are not one unified
experiment.

## Research Landscape

Speech evaluation has five connected layers.

**Human listening and interaction tests** measure how people hear and use speech. MOS, CMOS,
MUSHRA, preference tests, deception tests, and interactive tests answer different questions. Their
results depend on listener populations, anchors, prompts, and test design.

**Task proxies** use another model to estimate a target property. Examples include ASR errors for
content, speaker embeddings for identity, emotion classifiers for affect, and VAD for timing. They
scale well, but inherit the proxy model's bias and blind spots.

**Signal and perceptual metrics** include PESQ, STOI, MCD, and spectral distances. They compare
generated and reference audio or estimate distortion. They are useful in controlled conditions,
but they do not define overall quality.

**Learned evaluators** include SSL-based quality predictors and LLM or audio-LM judges. They can
learn broader patterns than hand-designed metrics. Their validity still depends on training data,
test granularity, prompting, and access to the information being judged.

**Capability-specific benchmarks** measure what generic scores miss. Examples include multi-turn
memory, overlap and interruption behavior, paralinguistic reasoning, fairness, codec performance
under noise, pronunciation, clinical suitability, and watermark attacks. This is the fastest-
growing part of the landscape.

## What the Research Shows

### Content intelligibility

**Current assessment:** ASR error rates are useful content checks, but they are not sufficient
measures of human intelligibility.

This result appears in autoregressive codec TTS, codec benchmarks, accent and pronunciation tests,
prosody research, and second-language listener studies. [[2403.16973|VoiceCraft]] gives a clear
example: synthetic speech can have lower ASR error than ground truth but worse human
intelligibility. [[interspeech-2025-0984|SITool]] also motivates perceptual codec tests instead of
relying only on recognizer output.

WER is not meaningless. It measures whether one recognizer recovers the expected text under its
own training conditions. That can matter for machine use. It does not measure ease of listening,
robustness across accents, or preservation of clinically important speech features.

**Most informative evidence:** [[2403.16973|VoiceCraft]],
[[interspeech-2025-0984|SITool]], [[2508.12001|FNH-TTS]].

**What remains unresolved:** how to combine human intelligibility, recognizer performance, and
listener-population effects in an affordable benchmark.

### Speaker identity

**Current assessment:** embedding similarity and EER-based verification do not reliably measure
perceived identity across synthesis, conversion, anonymization, and accessibility settings.

Some automatic metrics rank synthetic speech above the reference. Other systems receive similar
embedding scores but sound different to listeners. [[2406.18009|E2 TTS]] and
[[2025.naacl-long.242|StyleTTS-ZS]] are part of a pattern also seen in voice conversion and
anonymization. The embeddings focus on traits useful to the verification model, not every identity
cue that listeners use.

The gap matters when style, accent, age, pathology, or background sound interacts with identity.
An assistive voice can match timbre but still feel unsuitable to its user. An anonymized voice can
move in embedding space while keeping some identity cues. Pair automatic similarity with human
identity or suitability tests for the target use.

**Most informative evidence:** [[2406.18009|E2 TTS]],
[[2025.naacl-long.242|StyleTTS-ZS]], [[interspeech-2025-1726]].

### Automatic quality prediction

**Current assessment:** SSL representations support useful and data-efficient quality predictors,
but validity does not transfer automatically across languages, domains, systems, or levels of
detail.

[[2204.02152|UTMOS]] established the most mature learned-quality family in the graph. Later work
adds listener models, pairwise or contrastive ranking, mixture-of-experts designs, and reward-
shaped objectives. This makes learned predictors useful infrastructure.

The same research shows their limits. A predictor can correlate with average system MOS but fail
on individual utterances ([[2507.06116]]). A model trained on one language or conventional TTS can
fail on other languages, new generation methods, or out-of-domain content. Near the quality
ceiling, small predicted differences can reverse human rankings ([[2508.00317]]).

Use a predictor after validation on the target domain and granularity. Do not treat a MOS-like
output as human MOS without that validation.

**Most informative evidence:** [[2204.02152|UTMOS]], [[2508.00317]],
[[2507.06116]], [[2509.20485]].

### Reference-based metrics

**Current assessment:** PESQ, STOI, MCD, and related metrics provide useful diagnostics. They can
still disagree with listeners and with one another across vocoders, codecs, and sampling
conditions.

The clearest reversals occur when architectures produce different types of error. A system can
have more sample-level distortion but smoother or more natural perceptual structure. Sampling
rate, reference alignment, and representation can change a metric ranking without changing
listener preference in the same way.

These metrics remain useful when their measured distortion matches the research question. The
problem begins when one controlled diagnostic becomes an overall quality label.

**Most informative evidence:** [[2507.01611]], [[2305.02765]],
[[iclr-2025-uxDFlPGRLX|FlowDec]], [[interspeech-2025-0984|SITool]].

### Learned judges

> [!warning]
> **Capability-scoped:** learned judges can match people on some structured semantic tasks. They
> remain unreliable for fine prosody and non-verbal or raw-waveform cues.

The strongest uses are instruction following, style adherence, broad dialogue quality, and
checklist rubrics. [[2506.16381|InstructTTSEval]] uses structured instruction judgments.
[[2508.02013|SpeechRole]] and [[2506.21875|WildSpeech-Bench]] show that careful rubrics can support
useful model-based evaluation. Repeated scoring and checklist decomposition can improve stability.

The negative evidence is direct. [[2509.19928]] shows weak judgment of fine prosodic diversity.
[[2508.18240|MTalk-Bench]] shows weaknesses on paralinguistic and non-verbal dimensions. These
results do not reject learned judges in general. They identify properties for which the judge
lacks access, resolution, or calibration.

Use model judges for structured and reproducible questions. Validate each rubric against people.
Keep human tests for dimensions carried in the waveform rather than the transcript.

**What remains unresolved:** whether better audio-native training can close the gap, or whether
fine human judgment remains necessary.

### Robustness under domain shift

**Current assessment:** clean, neutral, in-domain tests understate failures under noise,
reverberation, spontaneous content, unfamiliar speakers, and wild recordings.

The pattern appears in voice cloning, intelligibility, binaural synthesis, expressive speech, and
spoken instruction following. [[2403.16973|VoiceCraft]] shows a gap between audiobook-style and
wild content. [[2506.21875|WildSpeech-Bench]] shows that speech-to-text benchmark performance does
not predict speech-to-speech behavior in realistic interactions.

Robust tests need separate conditions: clean and noisy, in-domain and shifted, read and
spontaneous, short and long, single-turn and multi-turn. An average must not hide the worst
condition that matters in deployment.

### Codec evaluation

**Current assessment:** acoustic reconstruction, semantic retention, and downstream generation are
partly independent. No codec metric predicts all three.

[[2508.20660|CodecBench]] shows that semantic preservation can come at the cost of paralinguistic
detail. [[2025.naacl-srw.6]] and [[2506.10274]] show that better reconstruction does not always
produce better language-model tokens. More codebooks can improve reconstruction while adding
redundancy and reducing generative performance.

Noise and reverberation add another dimension. [[interspeech-2025-0355]] shows non-linear codec
behavior that helps explain why clean rankings do not transfer to degraded input. Test a codec
through the downstream generator when generation is the intended use.

### Subjective-test saturation

**Current assessment:** MOS, CMOS, and MUSHRA remain essential, but standard protocols can struggle
to separate modern high-quality systems.

The problem is statistical and conceptual. Small scale differences are hard to interpret near the
ceiling. Reference-matching tests also do not always ask whether speech sounds human.
[[interspeech-2025-0854]] shows a ranking-by-elimination test finding a preference that a MUSHRA-
style setup misses. [[interspeech-2025-2765]] proposes deception rate as a direct test of human
likeness.

Deception testing is a single-paper emerging direction, not a replacement standard. It also
answers a narrower question than naturalness or usability. Subjective tests must state what they
measure and adapt as systems approach their anchors.

### Social and interaction-specific evaluation

**Current assessment:** aggregate scores hide failures tied to demographic groups, clinical
speech, conversational context, and timing.

Fairness rankings can change across tasks. Single-turn tests can understate multi-turn bias
([[2509.12171]], [[2510.02352]]). Metrics trained on typical speech can misjudge dysarthric or
neuroprosthetic speech. A system can remove diagnostic or identity-bearing features while trying
to “normalize” the output.

Full-duplex tests expose a different problem. Latency alone cannot distinguish fast response from
incorrect yielding or premature takeover. [[2507.23159|Full-Duplex-Bench v1.5]] and
[[2509.22243|FLEXI]] use separate measures for interruption response, false yielding, jump-in,
floor holding, and backchannel timing.

These protocols define success for the application. They are not optional additions to a generic
scorecard.

### Metric optimization

**Current assessment:** optimizing intelligibility, duration accuracy, or sampling speed can reduce
prosodic diversity or another unmeasured property.

This pattern appears in preference optimization, flow-matching distillation, and explicit duration
modeling. [[2507.14988|DMOSpeech 2]] and [[2509.19928]] show that gains on a target metric can
narrow output variation. Teacher-guided or hybrid sampling can recover some of it.

Evaluation is therefore part of the training objective. If the test suite excludes diversity, the
optimizer has no reason to preserve it. Use held-out human measures and untargeted diagnostics when
training directly against automatic rewards.

### Q4 broadens consequential evaluation

**Current assessment:** fairness, accessibility, provenance, and spoken reasoning need dedicated
tests because aggregate quality scores hide their most important failures.

Demographic tests must separate speakers, tasks, languages, and turns. [[2509.12171|Preservation
of Language Understanding Capabilities in Speech-aware Large Language Models]] shows that adding
speech can change language understanding unevenly. [[2510.02352]] treats bias as a multi-turn
property. Clinical and assistive settings add another boundary: typical-speech metrics may not
measure suitability for a target user ([[interspeech-2025-1726|Voice Reconstruction through
Large-Scale TTS Models]]).

Provenance tests need realistic transformations. [[interspeech-2025-2328|A Watermark for Auto-
Regressive Speech Generation Models]] shows weaker detection after re-encoding, time stretching,
pitch shift, or reverberation. Passing a basic additive-noise test does not prove robust
provenance.

Spoken-agent tests must also keep semantic ability visible. A system can sound natural while
trailing a similar text model in reasoning, dialogue state, or instruction following.
[[2412.15649|SLAM-Omni]] is part of this comparison. Evaluation needs content and state measures
alongside voice, latency, and interaction quality.

## Where Findings Disagree

### Learned and human judgment

The disagreement is mainly about scope. Positive studies test semantic instructions, style labels,
checklists, or large quality differences. Negative studies test fine prosody, non-verbal cues, or
near-tied systems. Learned judges can be validated parts of a test suite. They are not universal
replacements for people.

### Codec reconstruction metrics

Reconstruction is not irrelevant. The disagreement concerns what it predicts. Signal metrics are
useful for resynthesis. Downstream studies show that a representation optimized for reconstruction
can be worse for semantic or generative use. A codec test must state which meaning of “quality” it
targets.

### Standard or task-specific protocols

Shared benchmarks improve comparison, but universal metrics fail on specialized properties. A
likely design is a common core for content, quality, identity, and robustness, plus task modules
for timing, emotion, fairness, clinical use, or downstream codec performance.

## How the Field Is Changing

Earlier evaluation centered on acoustic TTS quality: MOS, intelligibility proxies, speaker
similarity, and signal distortion. These measures remain important. The 2025 corpus adds spoken-
agent diagnostics, full-duplex timing, multi-turn behavior, fairness, and paralinguistic reasoning.

Learned evaluators also change the cost of testing. SSL-based MOS predictors support routine
screening. LLM and audio-LM judges form a middle layer between narrow metrics and full human
studies. Capability-specific validity research appeared almost immediately, which limits the risk
of assuming that language judgment transfers to acoustic judgment.

The field is moving from “Which system has the highest score?” to “Which properties does this test
actually observe?” This shift is visible in codec co-design, wild benchmarks, multi-turn tests,
and protocols built around concrete interaction failures.

It also changes how results should be reported. A compact scorecard can keep each number attached
to the property it measures, show uncertainty or group-level variation, and expose hidden
trade-offs. The aim is not a longer results table. It is a clearer argument from evidence to
claim.

Q4 strengthens this modular direction. Full-duplex tests separate stop latency from false
yielding. Fairness and clinical tests name the population for which a score is valid. Provenance
tests list transformations. Optimization studies monitor diversity lost while a target reward
improves.

## Implications

- **Define the target before choosing the metric.** Naturalness, intelligibility, identity,
  prosody, diversity, robustness, and usefulness are different properties.
- **Use a small, justified set of metrics.** More columns do not guarantee a better test.
- **Validate proxies on the target domain and granularity.** System-level correlation on English
  read speech does not prove utterance-level validity elsewhere.
- **Include the downstream system when testing representations.** Reconstruction alone is not
  enough for generative-token selection.
- **Use people for perceptual, consequential, or close decisions.** Use automatic methods for
  scale and diagnosis, not as unqualified ground truth.
- **Report results for each important condition.** A clean average must not hide noise, domain,
  demographic, clinical, or multi-turn failures.
- **Test untargeted properties during optimization.** This helps detect reward hacking and loss of
  diversity.

## Representative Reading Path

1. **Start with learned quality prediction.** Read [[2204.02152|UTMOS]].
2. **See a proxy fail.** Read [[2403.16973|VoiceCraft]] for WER versus human intelligibility.
3. **Survey the evaluation problem.** Read [[2508.00317]].
4. **Separate codec quality dimensions.** Read [[2508.20660|CodecBench]].
5. **Move to realistic conditions.** Read [[2506.21875|WildSpeech-Bench]].
6. **Test multi-turn and non-verbal behavior.** Read [[2508.18240|MTalk-Bench]].
7. **Measure interaction, not only latency.** Read [[2507.23159|Full-Duplex-Bench v1.5]].
8. **Study task-specific fairness.** Read [[2510.02352]].
9. **Test provenance under realistic changes.** Read
   [[interspeech-2025-2328|A Watermark for Auto-Regressive Speech Generation Models]].

## Structured Source

This page selects and groups findings for readability. The complete graph contains all 346 paper
entries, evidence roles, source sections, caveats, method families, and reassessment state. It is
available in `wiki/_claims/evaluation-metrics.yaml`.

---

_This page is generated from `wiki/_claims/evaluation-metrics.yaml` (digest date: 2026-09-20). For complete structured claims and paper-level provenance, use the source YAML._
