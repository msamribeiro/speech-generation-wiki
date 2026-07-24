---
concept: evaluation-metrics
render_type: in-depth
title: "Evaluation Metrics: In Depth"
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

This page explains what the reviewed evaluation literature collectively supports. For the short
state-of-the-art view, see [[concepts/evaluation-metrics|Evaluation Metrics]]. Citations are
representative; the complete paper-to-claim graph remains in
`wiki/_claims/evaluation-metrics.yaml`.

## Findings at a Glance

- No single metric captures overall speech-generation quality; useful evaluation is necessarily
  multidimensional.
- WER and CER are convenient content proxies but do not reliably measure perceived intelligibility.
- Speaker-verification embeddings do not fully track perceived identity and can produce
  counterintuitive rankings.
- Automatic MOS predictors and reference-based distortion metrics become unreliable under domain
  shift, at finer granularity, and near the quality ceiling.
- LLM/audio-LM judges are useful for structured semantic or rubric-based evaluation but remain
  unreliable for fine prosody and non-verbal audio.
- Clean, in-domain benchmarks understate failures that emerge in realistic acoustic and
  conversational conditions.
- Codec reconstruction quality does not predict semantic retention or downstream generation
  quality.
- Modern subjective protocols are becoming less discriminative, but proposed replacements are not
  yet standardized.
- Fairness, clinical suitability, full-duplex timing, and multi-turn behavior require dedicated
  evaluation rather than another aggregate quality score.
- Optimizing a single measured property can degrade diversity or another unmeasured perceptual
  dimension.

## Scope

The claim graph contains 285 papers through Q3 2025, making this the broadest integrated concept in
the current wiki. It spans TTS, voice conversion, neural codecs, vocoders, spoken conversational
agents, full-duplex systems, clinical and accessibility applications, watermarking, and
low-resource language evaluation.

Most papers are not metric papers in the narrow sense. Many are system papers whose experiments
expose an evaluation failure: a metric reverses a human ranking, an apparently strong reconstruction
score fails downstream, or a benchmark misses a capability that matters in deployment. The method
families therefore group papers by evaluation dimension rather than architecture.

This breadth supports cross-task conclusions but prevents a single quantitative meta-analysis.
Metrics, datasets, listening protocols, and target populations differ substantially. Survey entries
support taxonomy, while primary studies provide the direct evidence. Large umbrella families such
as objective-metric divergence and WER-based evaluation are evidence that the same caution recurs
across unrelated systems, not one unified experimental program.

## Research Landscape

Speech evaluation can be understood as five interacting layers.

**Human perceptual and interaction studies** remain the closest measure of how speech is heard and
used. MOS, CMOS, MUSHRA, preference tests, deception tests, and interactive protocols answer
different questions; their results depend on listener population, anchors, prompts, and test
design.

**Task-derived proxies** use another model to infer a target property: ASR errors for content,
speaker-verification embeddings for identity, emotion classifiers for affect, or VAD for timing.
They scale well but inherit the proxy model's biases and blind spots.

**Signal and perceptual metrics** such as PESQ, STOI, MCD, and spectral distances compare generated
and reference audio or estimate distortion. They are useful diagnostics under controlled
conditions, but they do not define holistic quality.

**Learned evaluators** include SSL-based quality predictors and LLM/audio-LM judges. They can learn
broader patterns than hand-designed metrics, yet their validity depends on training domain,
granularity, prompting, and whether the target information is available to the model.

**Capability-specific benchmark suites** evaluate what generic quality scores omit: multi-turn
memory, full-duplex overlap, paralinguistic reasoning, fairness, codec behavior under noise,
pronunciation, clinical suitability, and provenance attacks. This is the fastest-growing part of
the landscape.

## What the Research Shows

### Content intelligibility

**Current assessment:** ASR-derived error rates are valuable content diagnostics but are
insufficient as standalone measures of how intelligible speech is to people.

The finding recurs across autoregressive codec TTS, codec benchmarking, accent and pronunciation
evaluation, prosody modeling, and second-language listener studies. [[2403.16973|VoiceCraft]]
provides a particularly clear warning: synthesized speech can obtain lower ASR error than ground
truth while receiving worse human intelligibility judgments. [[interspeech-2025-0984|SITool]]
likewise motivates perceptually grounded codec intelligibility tests rather than relying only on
recognizer output.

The problem is not that WER is meaningless. It measures whether a particular recognizer recovers
the expected transcript under its own training biases. That is often relevant to downstream
machine consumption. It is not equivalent to ease of understanding for listeners, robustness
across accents, or preservation of clinically relevant speech characteristics.

**Most informative evidence:** [[2403.16973|VoiceCraft]],
[[interspeech-2025-0984|SITool]], [[2508.12001|FNH-TTS]].

**What remains unresolved:** how to combine human intelligibility, recognizer performance, and
listener-population effects in a protocol that remains affordable at benchmark scale.

### Speaker identity

**Current assessment:** embedding cosine similarity and EER-based verification do not reliably
represent perceived speaker identity across synthesis, conversion, anonymization, and
accessibility settings.

The reviewed evidence includes cases where automatic similarity ranks synthetic speech above the
ground-truth reference and where systems with similar embedding scores differ perceptually.
[[2406.18009|E2 TTS]] and [[2025.naacl-long.242|StyleTTS-ZS]] contribute to a broader pattern also
seen in voice conversion and anonymization: embeddings emphasize the traits useful to the
verification model, not necessarily the full identity listeners perceive.

This matters most when style, accent, age, pathology, or background sound interacts with identity.
An assistive voice can be close in timbre yet feel unsuitable to its user; an anonymized voice can
move in embedding space without eliminating every identity cue. The appropriate response is to
pair automatic similarity with human identity or suitability judgments designed for the use case.

**Most informative evidence:** [[2406.18009|E2 TTS]],
[[2025.naacl-long.242|StyleTTS-ZS]], [[interspeech-2025-1726]].

### Automatic quality prediction

**Current assessment:** SSL representations provide an effective, data-efficient foundation for
automatic quality prediction, but predictor validity does not transfer automatically across
languages, domains, systems, or evaluation granularity.

[[2204.02152|UTMOS]] established the most mature learned-quality lineage in the graph. Later work
improves it with listener modeling, pairwise or contrastive ranking, mixture-of-experts designs, and
reward-shaped objectives. This supports learned predictors as useful infrastructure.

The same literature also establishes their limits. A predictor can correlate well with average
system MOS while performing poorly on individual utterances ([[2507.06116]]). Models trained on
one language or conventional TTS can fail on other languages, non-standard generation paradigms,
or out-of-domain content. Near the quality ceiling, small predicted differences may reverse human
rankings ([[2508.00317]]).

The practical distinction is between *calibrated use* and *universal substitution*. A predictor
validated on the target domain and granularity can support iteration and screening. It should not
be treated as human MOS merely because its output has a MOS-like scale.

**Most informative evidence:** [[2204.02152|UTMOS]], [[2508.00317]],
[[2507.06116]], [[2509.20485]].

### Reference-based metrics

**Current assessment:** PESQ, STOI, MCD, and related intrusive metrics provide diagnostic
information but can disagree with subjective quality and with one another across vocoders, codecs,
and sampling conditions.

The clearest reversals occur when architectures produce different error structures. A system may
have more sample-level distortion but smoother or more natural perceptual structure. Codec and
vocoder comparisons show that changing sampling rate, reference alignment, or representation can
alter a metric ranking without producing an equivalent listener preference.

These metrics remain useful when the distortion they measure corresponds to the research question.
The failure comes from turning one controlled diagnostic into an overall quality label.

**Most informative evidence:** [[2507.01611]], [[2305.02765]],
[[iclr-2025-uxDFlPGRLX|FlowDec]], [[interspeech-2025-0984|SITool]].

### Learned judges

> [!warning]
> **Capability-scoped:** learned judges can approximate human evaluation for structured semantic
> tasks, but they remain unreliable for fine-grained prosody and non-verbal or raw-waveform cues.

Their strongest uses include instruction following, style adherence, broad dialogue quality, and
checklist-based rubrics.

The positive evidence is operationally important. [[2506.16381|InstructTTSEval]] uses structured
instruction-following judgments, while [[2508.02013|SpeechRole]] and
[[2506.21875|WildSpeech-Bench]] show that rubric design can make broad model-based evaluation
useful. Repeated scoring and checklist decomposition can improve stability.

The negative evidence is equally direct. [[2509.19928]] finds that judge-based evaluation does not
adequately capture fine prosodic diversity, and [[2508.18240|MTalk-Bench]] exposes weaknesses on
paralinguistic and non-verbal dimensions. These are not blanket refutations: they identify
capabilities for which the judge lacks access, resolution, or calibration.

The appropriate design is hierarchical. Use model judges for structured, reproducible questions;
validate each rubric against people; and retain human evaluation for dimensions whose evidence is
carried in the waveform rather than its transcript.

**What remains unresolved:** whether audio-native judges with better temporal and perceptual
training can close this gap, or whether fine human judgment remains an irreducible requirement.

### Robustness under domain shift

**Current assessment:** evaluation restricted to studio-quality, neutral, in-domain speech
systematically understates failures under noise, reverberation, spontaneous content, unfamiliar
speakers, and wild recording conditions.

This pattern appears across voice cloning, intelligibility, binaural synthesis, expressive speech,
and spoken instruction following. [[2403.16973|VoiceCraft]] shows a gap between audiobook-style and
wild content, while [[2506.21875|WildSpeech-Bench]] demonstrates that speech-to-text benchmark
performance does not predict speech-to-speech behavior in realistic interactions.

Robust evaluation therefore needs condition axes, not one “test set”: clean versus noisy,
in-domain versus shifted, read versus spontaneous, short versus long, and single-turn versus
multi-turn. Aggregate averages should not hide the worst condition that matters in deployment.

### Codec evaluation

**Current assessment:** acoustic reconstruction, semantic retention, and downstream generative
utility are partly orthogonal; no codec metric reliably predicts all three.

[[2508.20660|CodecBench]] shows that codecs preserving semantic content can sacrifice
paralinguistic detail. [[2025.naacl-srw.6]] and [[2506.10274]] show that stronger reconstruction
does not necessarily yield better tokens for a downstream language model; adding codebooks can
improve reconstruction while increasing redundancy and degrading generative performance.

Noise and reverberation add a fourth axis. [[interspeech-2025-0355]] finds non-linear codec behavior
that helps explain why rankings from clean reconstruction do not transfer to mixed or degraded
inputs. A codec intended for generation should therefore be evaluated through the downstream model,
not only through resynthesis.

**Most informative evidence:** [[2508.20660|CodecBench]], [[2025.naacl-srw.6]],
[[2506.10274]], [[interspeech-2025-0355]].

### Subjective-test saturation

**Current assessment:** MOS, CMOS, and MUSHRA remain essential, but conventional protocols can
struggle to separate high-quality modern systems or detect when synthetic output approaches the
reference.

The issue is partly statistical and partly conceptual. Small scale differences become difficult to
interpret near the ceiling, and reference-matching tests may not ask whether speech sounds
genuinely human. [[interspeech-2025-0854]] shows a ranking-by-elimination protocol detecting a
preference that a MUSHRA-style setup misses. [[interspeech-2025-2765]] proposes human-fooling or
deception rate as a more direct test of human likeness.

Deception testing is currently a single-paper emerging direction, not a replacement standard. It
also answers a narrower question than naturalness or usability. The more general conclusion is
that subjective protocols must specify the construct they intend to measure and evolve as systems
approach their anchors.

### Social and interaction-specific evaluation

**Current assessment:** aggregate quality and accuracy mask failures tied to demographic groups,
clinical speech, conversational context, and timing behavior.

Fairness studies find that rankings can change across tasks and that single-turn tests understate
multi-turn bias ([[2509.12171]], [[2510.02352]]). Clinical and assistive applications show that
metrics calibrated on typical speech can misjudge dysarthric or neuroprosthetic speech; a system may
“normalize” diagnostic or identity-bearing features rather than preserve them.

Full-duplex evaluation reveals a different omission. Latency alone cannot distinguish a system that
responds quickly from one that yields incorrectly to background speech or takes over during a
pause. [[2507.23159|Full-Duplex-Bench v1.5]] and [[2509.22243|FLEXI]] motivate separate measures for
interruption response, false yielding, jump-in, floor holding, and backchannel timing.

These specialized protocols should not be bolted on after a generic scorecard. They define what
success means for the application.

### Metric optimization

**Current assessment:** optimizing intelligibility, duration accuracy, or accelerated sampling can
reduce prosodic diversity or another unmeasured property.

The pattern appears in preference optimization, flow-matching distillation, and explicit duration
modeling. [[2507.14988|DMOSpeech 2]] and [[2509.19928]] both show that improvements to a targeted
metric can narrow output variation; teacher-guided or hybrid sampling partially recovers it.

This makes evaluation part of the training objective, not merely a reporting choice. If the metric
suite excludes diversity, the optimizer has no reason to preserve it. Held-out human measures and
untargeted diagnostic dimensions are therefore essential when a system is trained directly against
automatic rewards.

## Where Findings Disagree

### Learned and human judgment

The apparent disagreement is resolved by scope. Studies supporting learned judges generally target
semantic instructions, style labels, checklists, or large quality differences. Negative studies
target fine prosody, non-verbal cues, and near-tied systems. The current evidence supports learned
judges as validated components of an evaluation suite, not universal human substitutes.

### Codec reconstruction metrics

No single side claims reconstruction is irrelevant. The disagreement concerns its role. Signal
metrics are useful for resynthesis quality, while downstream studies show that a representation
optimized for reconstruction can be worse for semantic or generative use. Codec evaluation must
state which meaning of “quality” is intended.

### Standardized or task-specific protocols

Shared benchmarks improve comparability, but universal metrics repeatedly fail at specialized
properties. The likely direction is a common core—content, quality, identity, robustness—combined
with task modules for dialogue timing, emotion, fairness, clinical use, or codec downstream
performance.

## How the Field Is Changing

The older evaluation center of gravity was acoustic TTS quality: MOS, intelligibility proxies,
speaker similarity, and signal distortion. Those measures remain important, but the 2025 corpus
shows rapid expansion into spoken-agent diagnostics, full-duplex timing, multi-turn behavior,
fairness, and paralinguistic reasoning.

Learned evaluators are also changing the cost structure. SSL-based MOS predictors have become
mature enough for routine screening, while LLM/audio-LM judges create a middle layer between narrow
automatic metrics and full human studies. Their rise has been accompanied almost immediately by
capability-specific validity studies, which is a healthy correction against assuming that language
judgment transfers automatically to acoustic judgment.

The field is moving from asking “Which system has the highest score?” toward “Which dimensions
does this protocol actually observe?” That shift is most visible in codec co-design, wild
benchmarks, multi-turn evaluation, and tests built around concrete interaction failures.

## Implications

- **Define the construct before choosing the metric.** Naturalness, intelligibility, identity,
  prosody, diversity, robustness, and usefulness are not interchangeable.
- **Use a compact, justified suite.** More columns do not guarantee better evaluation; each metric
  should have a clear interpretive role.
- **Validate proxies on the target domain and granularity.** System-level correlation on English
  read speech does not establish utterance-level validity elsewhere.
- **Include the downstream system when evaluating representations.** Codec reconstruction alone is
  insufficient for generative-token selection.
- **Reserve human evaluation for decisions that are perceptual, consequential, or close.** Use
  automatic methods for scale and diagnosis, not as unqualified ground truth.
- **Report condition-wise results.** Clean averages should not conceal noise, domain, demographic,
  clinical, or multi-turn failures.
- **Evaluate untargeted dimensions during optimization.** This is the most direct defense against
  reward hacking and diversity collapse.

## Representative Reading Path

1. [[2204.02152|UTMOS]] — the foundational SSL-based automatic MOS predictor.
2. [[2403.16973|VoiceCraft]] — a clear example of WER diverging from human intelligibility.
3. [[2508.00317]] — a broad view of current speech-quality-assessment challenges.
4. [[2508.20660|CodecBench]] — why codec evaluation must separate acoustic and semantic quality.
5. [[2506.21875|WildSpeech-Bench]] — why conventional speech benchmarks miss wild behavior.
6. [[2508.18240|MTalk-Bench]] — multi-turn, paralinguistic, and judge-validity evidence.
7. [[2507.23159|Full-Duplex-Bench v1.5]] — interaction evaluation beyond latency.
8. [[2510.02352]] — why fairness evaluation must be task- and turn-specific.

## Structured Source

This page selects and groups findings for human readability. The complete claim graph—including all
285 paper entries, roles, source sections, caveats, method-family memberships, and reassessment
state—is available in `wiki/_claims/evaluation-metrics.yaml`.

---

_This page is generated from `wiki/_claims/evaluation-metrics.yaml` (digest date: 2026-07-21).
Citations are selected for human readability; complete provenance remains in the source YAML._
