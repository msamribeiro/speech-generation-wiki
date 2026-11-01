---
concept: subjective-evaluation
render_type: in-depth
title: "Subjective Evaluation of Generated Speech: In Depth"
source_digest_date: 2026-10-31
paper_count: 255
generation:
  schema_version: 2
  date: "2026-10-31"
  stage: render
  mode: full
  runtime: codex
  provider: openai
  agent: speech-generation-render-agent
  model: "gpt-5"
  commit: "b672c6b"
---

Companion to the [[concepts/subjective-evaluation|Overview]]. This page explains what the reviewed research collectively shows. Citations are representative; complete provenance is in `wiki/_claims/subjective-evaluation.yaml`.

## Findings at a Glance

- **Naturalness, intelligibility, identity, expression, and interaction are distinct perceptual axes.** A single aggregate score cannot establish overall quality.
- **Automatic metrics frequently rank systems differently from listeners.** The direction and size of disagreement depend on the task, domain, quality range, and population.
- **Learned quality predictors are not universally superior to classical metrics.** Aggregate correlation can hide utterance-level errors, saturation, and codec- or distortion-specific reversals.
- **MOS is useful but protocol-sensitive.** Listener composition, anchors, instructions, references, and scale interpretation can change the outcome.
- **Pairwise and ranking designs usually improve discrimination.** Their result is relative to the selected alternatives and rated dimension.
- **Fine-grained control requires dimension-specific tests.** Naturalness alone cannot validate emotion, accent, rate, melody, voice quality, or synchronization control.
- **Preference optimization can improve targeted perception while degrading other dimensions.** Reward saturation and hacking make independent held-out evaluation necessary.
- **Automated judges require scoped human validation.** Reliability is strongest for clear, rubric-grounded differences and weaker for fine-grained or non-verbal attributes.
- **Population and domain shift affect conclusions.** High-resource panels and clean laboratory speech do not automatically transfer to low-resource, culturally specific, or in-the-wild settings.
- **Codec, vocoder, and efficiency choices set perceptual ceilings.** Evaluation must distinguish system-level gains from artifacts introduced by the representation or decoder.

## Scope

The graph contains 255 papers published from 2016 through the Q4 2025 evidence window and synthesizes 17 claim clusters: 14 strongly supported and three contested. It covers TTS, voice conversion, spoken conversational agents, codecs, vocoders, singing, preference alignment, low-resource languages, and multimodal speech. Evidence includes method-focused evaluation research, benchmarks, system listening tests, metric studies, participatory work, and preference-training experiments. Many papers use a perceptual study as validation rather than investigate the protocol itself; those studies support task-specific conclusions but carry less weight for general claims about evaluation design. Scores are not pooled across unmatched listener groups, datasets, scales, or instructions.

## Research Landscape

Subjective evaluation is not one method family. Absolute category ratings, pairwise preferences, rankings, reference-based attribute tests, multidimensional task rubrics, and participatory studies answer different questions. MOS-style ratings estimate an absolute-seeming quality level, while pairwise tests measure relative preference within a comparison set. Reference tests make identity, accent, emotion, or prosody explicit, but the anchor influences the judgment. Interactive and multimodal benchmarks additionally need semantic, temporal, and behavioral rubrics.

The systems under evaluation also shape the protocol. Autoregressive and transformer systems expose alignment, sampling, and long-form errors; GAN and codec systems expose waveform and compression artifacts; flow and diffusion systems trade inference budget against quality; latent systems raise diversity and identity-drift questions. A valid study chooses dimensions and stimuli around the expected failures rather than applying one naturalness score to every architecture.

Automation cuts across these methods. Learned MOS predictors, speaker encoders, codec metrics, text-based LLM judges, and audio-language-model judges can scale evaluation. Their role is strongest when calibrated against humans for the same task, domain, attribute, and scoring granularity. They are not a separate ground truth.

## What the Research Shows

### Quality is multidimensional

**Current assessment:** naturalness, intelligibility, speaker identity, expressiveness, prosody, and interaction success should be reported separately because they can improve or degrade independently.

This conclusion recurs across synthesis, conversion, codec, dialogue, and multimodal systems. A model can produce intelligible speech that sounds unlike the intended speaker, preserve identity while flattening prosody, or improve emotional recognition while reducing naturalness. Spoken agents can deliver fluent audio while failing semantic or turn-taking objectives. Q4 benchmarks increasingly make these dimensions explicit ([[2025.emnlp-main.1160|C3]], [[2025.emnlp-main.1447|MULTIVOX]], [[2511.00850|MULTI-Bench]]).

Composite scores can be useful for summary, but they conceal compensation: a small gain on several easy dimensions can hide a severe failure on one material axis. The most defensible evaluation reports each component and explains any aggregation rule. Task success, acoustic delivery, identity, and interaction should not be interchangeable weights by default.

**Most informative evidence:** [[2025.emnlp-main.1160|C3]], [[2025.emnlp-main.1447|MULTIVOX]], [[2511.05143|Selected Perceptual Voice Qualities]].

**What remains unresolved:** common rubrics that remain diagnostic across TTS, conversion, dialogue, singing, and multimodal output without forcing unrelated tasks onto one scale.

### MOS depends on who listens and how

**Current assessment:** MOS remains useful for broad naturalness comparisons, but its result depends on listener population, cultural context, reference conditions, instructions, anchors, and scale design.

An apparent absolute score is produced by a particular sample of listeners under a particular protocol. Native-language competence affects pronunciation and prosody judgments; cultural expectations affect style and social appropriateness; hearing profile and playback equipment affect artifact sensitivity. Reusing training or development listeners, changing panel composition, or comparing scores from different scales can create differences unrelated to the systems.

The late PACC page makes the reporting problem concrete. Its listener and test-set details are absent and its score scales are inconsistent, so it cannot support an absolute quality ranking ([[2605.18001|Bridging the Gap]]). This does not invalidate MOS itself. It reinforces the need to report recruitment, eligibility, stimuli, repetitions, anchors, instructions, scale endpoints, exclusions, uncertainty, and statistical analysis.

**Most informative evidence:** [[2605.18001|Bridging the Gap]], [[2025.iwclul-1.3|South Sámi TTS]].

**What remains unresolved:** portable reporting and analysis standards that preserve local cultural validity while enabling cross-laboratory replication.

### Pairwise tests improve discrimination

**Current assessment:** pairwise preference, ranking, and contrastive designs often resolve small perceptual differences more reliably than isolated absolute scores, but they do not establish absolute acceptability.

Relative judgments reduce the burden of mapping perception onto a numerical scale. They are especially useful for near-tied systems, attribute controls, and preference-data collection. The comparison can ask which sample is more natural, more similar, more expressive, or better aligned, making the judged dimension explicit.

The result remains conditional on the candidates. A system can win against a weak baseline and still be poor, and rankings can change between naturalness and identity. Pairwise results also become expensive as the comparison set grows. Arena and adaptive designs reduce coverage cost but introduce sampling and transitivity assumptions.

**Most informative evidence:** [[interspeech-2025-2283|Pairwise Accent Evaluation]], [[2406.00654|Human-Feedback Zero-Shot TTS]].

**What remains unresolved:** designs that combine pairwise discrimination with interpretable absolute thresholds and efficient coverage of many systems.

### Automatic metrics are task-dependent tools

**Current assessment:** automatic quality, intelligibility, and speaker metrics provide useful diagnostics but frequently disagree with human perception and should not be treated as universal substitutes.

Classical signal metrics require references and may penalize harmless transformations or miss perceptually salient artifacts. Learned MOS predictors can correlate better with aggregate listener scores, as UTMOS illustrates, but can saturate near transparent quality and fail under language, domain, or distortion shift ([[2204.02152|UTMOS]]). Q4 neural-codec evidence directly contests a general learned-metric advantage: several classical measures match or outperform learned predictors for particular codec conditions ([[2511.19734|Objective Metrics for Neural Audio Codecs]]).

Speaker similarity adds its own confounds. Embedding scores depend on the verification model, reference duration, language, channel, pitch, and architecture; higher cosine similarity does not guarantee stronger perceived identity ([[2507.02176|Speaker Similarity Assessment]]). The appropriate use is dimension-specific screening and diagnosis, calibrated to the transformation and operating range.

**Most informative evidence:** [[2204.02152|UTMOS]], [[2507.02176|Speaker Similarity Assessment]], [[2511.19734|Objective Metrics for Neural Audio Codecs]].

**What remains unresolved:** cross-domain calibration studies that identify robust metric combinations and uncertainty estimates for each task and artifact class.

### Automated judges need scoped validation

**Current assessment:** LLM and audio-language-model judges can reproduce some human judgments under grounded, matched conditions, but reliability is contested for fine-grained, non-verbal, near-tied, and unscripted attributes.

Text-based judges can evaluate transcript-level correctness or instruction following, while audio-capable judges can additionally inspect delivery. Task-specific tuning, explicit rubrics, repeated sampling, and reference grounding improve agreement. Spoken-dialogue benchmarks use these tools to scale complex evaluations that would be expensive for listeners ([[2025.emnlp-main.1160|C3]], [[2511.00850|MULTI-Bench]]).

The boundary is important. A high aggregate correlation does not establish reliable utterance-level emotion, prosody, turn-taking, or non-verbal judgment. Real unscripted audio and audio-only cues remain harder than verbal content. One Q4 prosody-diversity study provides evidence that a carefully targeted automatic measure can work, but that success narrows rather than removes the validation requirement ([[2509.19928|Prosody Diversity Benchmark]]).

**Most informative evidence:** [[2025.emnlp-main.1160|C3]], [[2511.00850|MULTI-Bench]], [[2509.19928|Prosody Diversity Benchmark]].

**What remains unresolved:** prospective transfer tests that determine when calibration on one domain, model family, or quality range remains valid on another.

### Fine-grained control needs matched questions

**Current assessment:** emotion, accent, prosody, speaking rate, melody, voice quality, and audiovisual timing require attribute-specific perceptual tests rather than holistic naturalness alone.

Control claims concern whether a requested attribute changed in the intended way. Naturalness can stay high while control fails, or decline slightly while recognizability and expressiveness improve. The late non-verbal emotion study demonstrates the latter pattern: fine-grained events improve emotion recognition and expressiveness but slightly lower naturalness in a small 15-listener test ([[2605.25504|Fine-Grained Non-Verbal Expression Control]]). That result is refining evidence, not a universal operating point.

Attribute tests need validated labels and anchors. Emotion categories, accent definitions, voice-quality terms, and style instructions can be culturally or linguistically specific. Word-level and transition-level controls may require time-localized judgments rather than one utterance score.

**Most informative evidence:** [[2605.25504|Fine-Grained Non-Verbal Expression Control]], [[2511.05143|Selected Perceptual Voice Qualities]].

**What remains unresolved:** reliable time-localized protocols for controls whose success and naturalness vary within an utterance.

### Control strength has an operating point

**Current assessment:** stronger guidance, compression, transformation, or preference optimization does not produce monotonic perceptual gains; evaluation should report curves rather than one selected setting.

Increasing a control weight can improve adherence while degrading naturalness, identity, intelligibility, or diversity. Compression can reduce bitrate until a perceptual ceiling breaks; retrieval scale can improve coverage before introducing mismatch; reward training can improve the optimized axis before saturating or exploiting the proxy. These reversals appear across architectures and tasks.

The relevant question is not whether control helps, but where the operating point lies for each dimension. A single tuned setting hides the shape of the trade-off and may be selected on the same listeners or metrics used for final reporting.

**Most informative evidence:** [[2406.00654|Human-Feedback Zero-Shot TTS]], [[2025.acl-long.598|Preference-Aligned Zero-Shot TTS]], [[2605.25504|Fine-Grained Non-Verbal Expression Control]].

**What remains unresolved:** standardized multi-axis sweeps with held-out listeners and explicit stopping criteria for guidance and reward optimization.

### Preference optimization changes evaluation burden

**Current assessment:** preference-based optimization can improve targeted perception beyond supervised or metric-only training, but it requires independent evaluation of reward robustness and untargeted dimensions.

Preference learning turns comparisons into training data. Human-feedback TTS and domain-focused alignment show that targeted intelligibility or quality can improve ([[2406.00654|Human-Feedback Zero-Shot TTS]], [[2025.acl-long.598|Preference-Aligned Zero-Shot TTS]]). Proxy-generated preferences can scale the process, but also make evaluation circular when the training reward and reported judge share assumptions or models.

Cross-dimension trade-offs are recurrent. Better intelligibility can weaken identity, stronger emotional alignment can affect naturalness, and reward optimization can suppress prosodic diversity. Reward saturation and hacking mean that more optimization is not automatically better. Final studies need held-out judges, listeners, prompts, and dimensions not used for training.

**Most informative evidence:** [[2406.00654|Human-Feedback Zero-Shot TTS]], [[2025.acl-long.598|Preference-Aligned Zero-Shot TTS]].

**What remains unresolved:** preference pipelines whose data construction, reward model, stopping rule, and external validation can be audited independently.

### Population validity is part of validity

**Current assessment:** listener language, demographics, cultural context, and community goals affect perceived quality and should be reported rather than averaged away.

Demographic representation and social perceptions can produce disparities in quality and identity judgments. Majority accents and familiar voices may receive systematically different ratings because training data, evaluators, and social stereotypes interact. In low-resource or endangered-language settings, a large conventional panel may be impossible or invalid if listeners do not understand the language.

The South Sámi study shows why participatory evaluation matters: community-scaled review adds linguistic, cultural, and revitalization context that a generic MOS panel cannot supply ([[2025.iwclul-1.3|South Sámi TTS]]). Small samples reduce population-level statistical power, but can increase construct validity for the actual community.

**Most informative evidence:** [[2025.iwclul-1.3|South Sámi TTS]].

**What remains unresolved:** analysis methods that respect small-community evidence and uncertainty without treating non-native scale as a substitute for relevance.

### Representation and decoder set a ceiling

**Current assessment:** codec and vocoder artifacts propagate into downstream evaluation, so system-level listening tests must distinguish representation limits from generator behavior.

A speech model cannot recover identity, pitch, transient, or phase detail that its codec has discarded reliably. Likewise, an acoustic model may be blamed for artifacts introduced by the vocoder, or a faster decoder may appear worse because its conditioning differs. Q4 codec and vocoder studies broaden evaluation beyond clean speech to dynamic bitrate, streaming latency, aliasing, harmonic content, singing, and downstream token usefulness.

Perceptual speed comparisons need the same discipline. One-step diffusion, speculative decoding, streaming codec decoders, and adaptive solvers may reduce latency through different mechanisms. Human tests are necessary to determine whether the exact acceleration preserves quality, while matched hardware and model size are needed for causal attribution.

**Most informative evidence:** [[2511.19734|Objective Metrics for Neural Audio Codecs]], [[2512.12129|Child Voice Conversion Study]].

**What remains unresolved:** evaluations that jointly report representation reconstruction, downstream generation, human perception, latency, and resource use under matched conditions.

## Where Findings Disagree

### Are learned metrics better than classical metrics?

Often, but not uniformly. Learned predictors tend to correlate better on the distributions and scales they were trained to model. Classical measures can remain competitive for codec-specific distortions or signal properties. The current assessment is task-dependent: use matched human calibration rather than selecting a metric family by reputation.

### Can automated judges replace listeners?

They can reduce human workload for validated coarse screening, transcript-level criteria, and clearly separated systems. Evidence remains mixed for near ties, prosody, emotion, non-verbal cues, unscripted audio, and domain transfer. Replacement is therefore a scoped empirical claim, not a property of the judge model alone.

### Is MOS too flawed to use?

No. MOS is efficient and interpretable for broad naturalness judgments under a well-reported common protocol. It becomes misleading when scales, listeners, contexts, or rated attributes differ, or when one aggregate score is used to validate fine-grained control.

### Do pairwise tests solve protocol sensitivity?

They reduce scale-mapping difficulty and improve discrimination, but remain sensitive to the candidate set, presentation order, question wording, and rated axis. They complement rather than eliminate absolute and multidimensional evaluation.

## How the Field Is Changing

Subjective evaluation has expanded from aggregate naturalness MOS toward multidimensional tests of identity, prosody, emotion, interaction, synchronization, voice quality, accent, and social perception. Pairwise preference and ranking became central both for measurement and for preference-based training during 2024–2025.

Automation is growing in parallel. Learned MOS predictors, embeddings, LLM judges, and audio-language-model judges enable larger studies, while the evidence base increasingly documents domain shift, saturation, and fine-grained reliability failures. Q4 benchmarks directly evaluate generated audio from spoken-dialogue and multimodal systems rather than relying on transcripts alone. Low-resource and participatory studies are also adapting protocols to native-listener availability and cultural goals instead of importing standard MOS unchanged.

## Implications

- **Define the decision before selecting the test.** Naturalness, identity, emotion, accent, task success, and interaction require different questions and references.
- **Report the full protocol.** Include recruitment, language competence, demographics where relevant, playback controls, stimuli, anchors, instructions, exclusions, uncertainty, and analysis.
- **Keep dimensions visible.** Do not let a composite score conceal failure on a material axis.
- **Calibrate automation locally.** Validate metrics and judges on the same domain, attribute, quality range, and scoring granularity before reducing human evaluation.
- **Evaluate preference systems externally.** Separate training rewards and proxy judges from final listeners, prompts, and held-out perceptual dimensions.
- **Report operating-point curves.** Guidance, compression, reward duration, retrieval scale, and transformation strength should be evaluated across settings.
- **Treat community relevance as evidence quality.** A smaller competent or affected population may provide more valid judgments than a larger convenient panel.
- **Attribute system bottlenecks carefully.** Separate codec, vocoder, generator, and acceleration effects wherever the design permits.

## Representative Reading Path

1. **Start with scalable perceptual prediction.** Read [[2204.02152|UTMOS]] to understand why learned quality models became attractive and what their correlation claims do and do not establish.
2. **Study identity as a distinct perceptual axis.** Use [[2507.02176|Speaker Similarity Assessment]] to see how speaker metrics depend on model, language, reference, and protocol.
3. **Connect evaluation to training.** Compare [[2406.00654|Human-Feedback Zero-Shot TTS]] and [[2025.acl-long.598|Preference-Aligned Zero-Shot TTS]] to understand targeted gains and cross-dimension evaluation burdens.
4. **Examine automated spoken-system evaluation.** Read [[2025.emnlp-main.1160|C3]], [[2025.emnlp-main.1447|MULTIVOX]], and [[2511.00850|MULTI-Bench]] for multidimensional rubrics and judge-calibration problems.
5. **Test metric-family assumptions.** Read [[2511.19734|Objective Metrics for Neural Audio Codecs]] for evidence that learned predictors are not uniformly superior.
6. **See why fine-grained control changes the protocol.** Read [[2511.05143|Selected Perceptual Voice Qualities]] and [[2605.25504|Fine-Grained Non-Verbal Expression Control]] for attribute-specific and non-monotonic perceptual outcomes.
7. **Add community and cultural validity.** Read [[2025.iwclul-1.3|South Sámi TTS]] for a setting where participatory, native-listener evidence matters more than conventional panel scale.
8. **Close with a reporting failure case.** Use [[2605.18001|Bridging the Gap]] to see why underspecified listeners and score scales cannot support an absolute system ranking.

## Structured Source

This page selects citations for human readability. The complete claim graph, paper roles, source sections, caveats, and reassessment state are available in `wiki/_claims/subjective-evaluation.yaml`.

---

_This page is generated from `wiki/_claims/subjective-evaluation.yaml` (digest date: 2026-10-31)._
