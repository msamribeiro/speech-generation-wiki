---
concept: emotion-synthesis
render_type: in-depth
title: "Emotional and Expressive Speech Synthesis: In Depth"
source_digest_date: 2026-10-31
paper_count: 98
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

Companion to the [[concepts/emotion-synthesis|Overview]]. This page explains what the reviewed research collectively shows. Citations are representative; complete provenance is in `wiki/_claims/emotion-synthesis.yaml`.

## Findings at a Glance

- **Explicit emotion conditioning improves recognizability and perceived expression.** The size of the gain depends on taxonomy, data balance, and the neutral baseline.
- **Natural-language instructions support richer control than fixed labels.** Instruction understanding and acoustic realization can fail independently.
- **Fine-grained temporal conditioning improves within-utterance dynamics.** Alignment, transition smoothing, and local error accumulation remain limiting factors.
- **Speaker and emotion representations can be separated usefully but not completely.** Habitual expression contributes to perceived identity.
- **Emotion strength trades against naturalness, intelligibility, and identity.** Stronger control can reverse gains on untargeted dimensions.
- **Dialogue, visual, semantic, and retrieval context improve affect modeling.** Irrelevant or misattributed context can instead degrade output.
- **Dedicated data and objectives are needed for laughter, sighs, fillers, and emphasis.** Lexical metrics often penalize these intended events.
- **Preference optimization can improve emotional alignment beyond supervised tuning.** Reward hacking and evaluator brittleness are first-order concerns.
- **Automatic emotion metrics do not replace human evaluation.** Agreement varies by language, category, system family, and whether the target is a label or a trajectory.
- **Sustained emotional coherence remains emerging.** Most evidence concerns isolated or short synthetic interactions rather than long realistic dialogue.

## Scope

The source graph contains 98 papers published through the Q4 2025 evidence window and synthesizes 17 claim clusters: 16 strongly supported and one emerging. It covers emotional TTS, expressive voice conversion, spoken dialogue, multimodal affect, reference and instruction control, paralinguistic events, and preference optimization. The evidence includes listening tests, emotion-recognition measures, acoustic analysis, benchmarks, and automatic judges. Many datasets are acted, categorical, demographically narrow, or automatically labeled; many dialogue studies are single-turn or synthetically constructed. Strong support therefore applies to the encoded control mechanisms and observed trade-offs, not to universal cultural interpretation or long-horizon coherence.

The graph also combines studies with different targets: some synthesize an emotion category, some transfer a reference style, and others judge whether a conversational response is affectively appropriate. These outcomes are related but not interchangeable. Reported gains are interpreted within each task rather than pooled into a common emotional-quality scale.

## Research Landscape

Emotion synthesis combines three layers: interpreting an intended affect, representing its temporal structure, and rendering it acoustically. Autoregressive speech language models often merge these layers in token generation and increasingly add preference post-training. Flow-matching systems support parallel synthesis and local editing, while transformer encoder–decoders place emotion at sentence, word, phoneme, or frame level. Hybrid systems separate semantic affect planning from acoustic rendering and may add speaker, retrieval, visual, or dialogue pathways.

GAN-based systems remain useful for expressive transfer and adversarial separation; diffusion systems support flexible iterative conditioning; VAE-style latent spaces support interpolation and factorization. These are not exclusive categories. A model may use an instruction encoder, factorized speaker and emotion latents, flow-based acoustic generation, adversarial losses, and a neural vocoder together. The central questions are where affect enters, how locally it can change, and which other attributes move with it.

## What the Research Shows

### Explicit conditioning works, within its taxonomy

**Current assessment:** explicit emotion conditioning improves recognizable and perceived expression, but the result is bounded by label design, class balance, and data realism.

Category labels provide the clearest supervised signal and repeatedly outperform neutral or uncontrolled baselines ([[2025.findings-naacl.38|Prompt-Guided Emotive TTS]], [[2507.04349|TTS-CtrlNet]]). Continuous valence–arousal–dominance coordinates add graded intensity, and combining categories with continuous axes broadens control. These approaches establish that affect can be manipulated, not that the emotion taxonomy is complete or culturally invariant.

Acted corpora often exaggerate cues and underrepresent ambiguous or mixed states. Rare classes, speaker demographics, and recording conditions are confounded with emotion. A classifier can recognize the same artifacts the synthesizer learned, inflating apparent success. The appropriate conclusion is that explicit controls improve output relative to matched baselines under the represented taxonomy.

**Most informative evidence:** [[2025.findings-naacl.38|Prompt-Guided Emotive TTS]], [[2507.04349|TTS-CtrlNet]].

**What remains unresolved:** balanced naturalistic datasets and cross-cultural listener studies that separate emotion from speaker, text, and recording condition.

### Instructions broaden control but split the task

**Current assessment:** natural-language descriptions enable richer and more compositional control than fixed labels, while introducing separate semantic-understanding and acoustic-realization failures.

Instruction-conditioned systems can request nuanced mixtures, pragmatic styles, intensity, pacing, or paralinguistic behavior that do not fit a small label inventory. CosyVoice and later instruction-following benchmarks show the breadth of this interface ([[2407.05407|CosyVoice]], [[2506.16381|InstructTTSEval]]). Text prompts can generalize across speakers without requiring a matching emotional reference clip.

However, a model may understand a request and fail to render it, or produce an acoustic cue that an evaluator mislabels. Audio references usually carry richer prosody and often retain a naturalness advantage, while also leaking speaker, content, and corpus properties. Open-ended descriptions are difficult to score because two valid renditions may realize the instruction differently.

**Most informative evidence:** [[2407.05407|CosyVoice]], [[2506.16381|InstructTTSEval]], [[2512.09504|DMP-TTS]].

**What remains unresolved:** listener protocols and grounded rubrics that separate prompt interpretation from acoustic execution for open-ended styles.

### Emotion unfolds within the utterance

**Current assessment:** word-, phoneme-, frame-, and sub-sentence conditioning produces more credible dynamics than one utterance-level label, provided alignment and transitions remain stable.

Time-varying systems can emphasize a word, change emotional intensity, or transition between states rather than applying one global style ([[2507.04349|TTS-CtrlNet]], [[2510.05758|EMORL-TTS]]). Phoneme-aware embeddings and local rewards improve temporal precision; teacher-generated transitions can scale training examples.

Finer granularity increases the cost of alignment errors. Local predictions can accumulate, abrupt transitions can sound unnatural, and requested event count or duration may not be verified. The late non-verbal study shows improved recognition from fine-grained tags, but its coarse comparison uses different source data and its control accuracy is not objectively measured ([[2605.25504|Fine-Grained Non-Verbal Expression Control]]).

**Most informative evidence:** [[2507.04349|TTS-CtrlNet]], [[2510.05758|EMORL-TTS]], [[2605.25504|Fine-Grained Non-Verbal Expression Control]].

**What remains unresolved:** matched studies of control granularity that report alignment, transition quality, event accuracy, and listener trajectory judgments together.

### Identity and emotion remain partially coupled

**Current assessment:** factorized pathways and disentanglement objectives improve independent control of voice and expression, but complete independence is neither demonstrated nor necessarily desirable.

Explicit predictors, contrastive objectives, perturbation, and hierarchical fusion reduce speaker leakage into emotion representations and vice versa ([[2508.02038|Marco-Voice]], [[interspeech-2025-1101|ZSDEVC]]). This enables transferring emotion to a target speaker while preserving more of that speaker’s timbre.

Habitual rhythm, pitch range, voice quality, and expressive manner are part of perceived identity. Removing them completely can make a generated voice less recognizable or natural. Cross-lingual transfer adds accent and language as further coupled variables. The relevant target is low cross-attribute interference under controlled changes rather than statistical independence.

**Most informative evidence:** [[2508.02038|Marco-Voice]], [[interspeech-2025-1101|ZSDEVC]], [[2512.09504|DMP-TTS]].

**What remains unresolved:** factorial human studies that cross unseen speakers, emotions, languages, accents, and habitual prosody.

### Stronger control exposes non-monotonic trade-offs

**Current assessment:** increasing emotion or style strength often improves recognition before reducing naturalness, intelligibility, identity, or diversity.

This pattern appears across guidance weights, reference intensity, compositional controls, and preference optimization. The late non-verbal study provides a clear small-scale example: adding fine-grained vocal events improves expressiveness and emotion recognition while slightly reducing naturalness for every tested emotion ([[2605.25504|Fine-Grained Non-Verbal Expression Control]]). Lexical metrics may additionally treat intended laughter or fillers as errors.

One selected operating point cannot reveal this curve. Systems should report multiple strengths and evaluate each on separate axes. A high emotion-classification score may reflect exaggerated cues rather than a listener-preferred rendition, while a conservative setting may sound natural but weakly controlled.

**Most informative evidence:** [[2507.00808|Multi-Interaction TTS]], [[2507.04349|TTS-CtrlNet]], [[2605.25504|Fine-Grained Non-Verbal Expression Control]].

**What remains unresolved:** matched intensity sweeps with listener-rated emotion, naturalness, identity, intelligibility, and preference.

### Context improves appropriateness

**Current assessment:** textual, dialogue, visual, semantic, and retrieved context improve affect selection, while long-horizon emotional coherence remains insufficiently demonstrated.

Chain-Talker separates understanding from rendering for empathetic conversational speech, and audio-visual systems use scenes and faces as complementary affect evidence ([[2025.findings-acl.101|Chain-Talker]], [[2508.16188|Emotion-Aware Audio-Visual Modeling]]). Retrieval can supply semantic or prosodic exemplars when a label is too coarse. Q4 dialogue tuning and emotional-intelligence benchmarks further emphasize appropriateness across turns ([[2511.00850|MULTI-Bench]]).

Context pipelines can compound errors. A wrong face attribution, irrelevant retrieved example, or mistaken dialogue interpretation can steer the voice incorrectly. Most studies use short or synthetic interactions, so they do not establish sustained engagement, gradual affect change, or recovery from misinterpretation over long conversations.

**Most informative evidence:** [[2025.findings-acl.101|Chain-Talker]], [[2508.16188|Emotion-Aware Audio-Visual Modeling]], [[2511.00850|MULTI-Bench]].

**What remains unresolved:** long, unscripted, culturally varied dialogue evaluations with trajectory-level human judgments.

### Data scale does not remove annotation bias

**Current assessment:** automatic annotation, captioning, synthetic dialogue, and augmentation expand emotional data, but reproduce classifier, prompt, and source-distribution biases.

MIKU-PAL illustrates scalable multimodal paralinguistic labeling, and Q4 systems increasingly filter expressiveness or generate style-rich dialogue automatically ([[2505.15772|MIKU-PAL]]). This makes temporal and open-ended controls feasible where curated labels are scarce.

Automatic labels are often scored by related models, creating self-referential evaluation. Synthetic source diversity can be narrower than its sample count suggests, and scaling one source may eventually reduce generalization. Acted-versus-naturalistic differences and overlapping acoustic correlates still limit clean taxonomy learning.

**Most informative evidence:** [[2505.15772|MIKU-PAL]], [[2512.14291|GLM-TTS]].

**What remains unresolved:** independent listener validation and source-diversity audits for automatically labeled or generated emotional corpora.

### Paralinguistic events need dedicated objectives

**Current assessment:** laughter, sighs, fillers, emphasis, and similar events require data and rewards beyond ordinary lexical TTS training.

These events have their own timing, duration, and acoustic structure. Treating them as generic style often yields weak or misplaced realizations. Dedicated tokens, local conditioning, and event-specific rewards improve recognizability ([[2506.16381|InstructTTSEval]], [[2510.05758|EMORL-TTS]], [[2605.25504|Fine-Grained Non-Verbal Expression Control]]).

Standard ASR-based evaluation may count an intentional event as insertion noise, while speaker metrics may respond to laughter or breath rather than identity. Better event control can therefore worsen lexical and identity proxies. This reinforces the need for event accuracy, perceptual appropriateness, naturalness, and content preservation as separate outcomes.

**Most informative evidence:** [[2506.16381|InstructTTSEval]], [[2510.05758|EMORL-TTS]], [[2605.25504|Fine-Grained Non-Verbal Expression Control]].

**What remains unresolved:** validated event-level datasets and listener protocols covering timing, duration, appropriateness, and interaction with speech content.

### Preference optimization helps and can be gamed

**Current assessment:** preference optimization improves emotional alignment beyond supervised tuning, but robust rewards and independent multidimensional evaluation are required.

DPO, GRPO, CTC preference objectives, and token-level rewards provide convergent evidence that listener- or proxy-derived preferences can refine emotional output ([[2510.05758|EMORL-TTS]], [[2512.04552|RRPO]]). Robust policy optimization explicitly addresses noisy or exploitable reward signals, while larger systems use regeneration and multiple reward axes.

Reward models may favor exaggerated acoustic archetypes, classifier-recognizable shortcuts, or degenerate trajectories. Training longer can saturate or reverse gains. If the same evaluator defines preferences and reports success, the result is circular. Held-out listeners, prompts, reward models, and unoptimized dimensions are necessary.

**Most informative evidence:** [[2510.05758|EMORL-TTS]], [[2512.04552|RRPO]], [[2512.14291|GLM-TTS]].

**What remains unresolved:** reward models that resist acoustic hacking while preserving subtle, culturally appropriate, listener-preferred expression.

### Human evaluation remains necessary

**Current assessment:** automatic emotion and style metrics are useful for screening but do not replace perceptual evaluation across systems, languages, and fine-grained trajectories.

Classifiers measure category recognizability, acoustic distances measure selected correlates, and automated judges can apply rubrics. None alone establishes naturalness, appropriateness, identity preservation, or temporal coherence. Agreement varies by emotion and quality range, and general-purpose language judges may miss audio-only or unscripted cues.

Calibrated audio-aware judges and human-trained evaluators improve scalability, but must be validated for the exact domain and attribute. A trajectory-sensitive measure may be needed for within-utterance change, while cross-language judgments require competent listeners and balanced data.

**Most informative evidence:** [[2506.16381|InstructTTSEval]], [[2511.00850|MULTI-Bench]], [[2605.25504|Fine-Grained Non-Verbal Expression Control]].

**What remains unresolved:** multilingual, cross-system calibration linking automatic trajectory scores to fine-grained listener judgments.

## Where Findings Disagree

### Is more emotion always better?

No. Recognition usually rises with stronger cues until naturalness, intelligibility, identity, or diversity begins to fall. The location of that optimum is system-, emotion-, and listener-dependent. Comparisons should show the curve rather than one selected setting.

### Are text instructions better than audio references?

They solve different problems. Text offers open-ended, editable control and may generalize without matching audio. References carry detailed timing and voice-quality information and often sound more natural, but leak speaker, content, and recording properties. Hybrid systems may combine them, with additional attribution risk.

### Does disentanglement remove identity–emotion coupling?

It reduces interference, but does not erase it. Identity includes habitual expression, and the same pitch or timing cues can encode linguistic, emotional, and speaker information. Intervention-level control is a more defensible claim than independence.

### Do current systems sustain emotional coherence?

Evidence supports utterance-level and short-context appropriateness. Direct evidence for stable, culturally appropriate affect across long unscripted dialogue remains too sparse for a stronger conclusion, so the graph retains this as emerging.

## How the Field Is Changing

Emotion synthesis is moving from utterance-level categorical labels toward natural-language descriptions, continuous affect axes, localized controls, and compositional attributes. Flow matching and speech language models increasingly separate semantic affect planning from acoustic realization, while retrieval and multimodal context provide alternatives to fixed labels.

Q4 strengthens preference-based control across autoregressive, flow, CTC, and hybrid systems, but also establishes reward hacking and reward robustness as central design problems. Data work is moving from acted categorical corpora toward spontaneous speech, synthetic style-rich dialogue, automatic filtering, and teacher-generated transitions. The evaluation target is correspondingly broadening from emotion-classification accuracy to listener judgments of appropriateness, trajectory, naturalness, intelligibility, identity, and cross-turn coherence.

## Implications

- **Specify the control representation.** State whether emotion is categorical, continuous, instructional, reference-derived, local, or contextual, and what each choice cannot express.
- **Evaluate compositional interference.** Test emotion jointly with speaker, dialect, rate, prosody, and paralinguistic events rather than only one control at a time.
- **Report strength curves.** Show how recognition, naturalness, identity, and intelligibility change as guidance or reward strength increases.
- **Treat non-verbal events as intended content.** Use event-specific timing and appropriateness measures instead of relying only on ASR error.
- **Audit automatic and synthetic labels.** Validate against independent listeners and characterize source diversity, demographic balance, and cultural scope.
- **Separate reward training from final evaluation.** Use held-out judges, prompts, listeners, and unoptimized perceptual axes.
- **Test realistic context.** Distinguish short synthetic dialogue success from sustained emotional coherence in long, unscripted interaction.

## Representative Reading Path

1. **Start with explicit emotional conditioning.** Read [[2025.findings-naacl.38|Prompt-Guided Emotive TTS]] and [[2507.04349|TTS-CtrlNet]] to compare context-aware global control with time-varying local control.
2. **Move from labels to instructions.** Use [[2407.05407|CosyVoice]] and [[2506.16381|InstructTTSEval]] to understand the breadth and evaluation difficulty of natural-language style requests.
3. **Study identity–emotion separation.** Read [[interspeech-2025-1101|ZSDEVC]] and [[2512.09504|DMP-TTS]] for diffusion and multimodal factorization approaches.
4. **Add dialogue and multimodal context.** Compare [[2025.findings-acl.101|Chain-Talker]], [[2508.16188|Emotion-Aware Audio-Visual Modeling]], and [[2511.00850|MULTI-Bench]].
5. **Examine reward-based control.** Read [[2510.05758|EMORL-TTS]] with [[2512.04552|RRPO]] to see both fine-grained gains and the need for robust reward optimization.
6. **Inspect the late non-verbal evidence.** Read [[2605.25504|Fine-Grained Non-Verbal Expression Control]] for a direct example of recognition and expressiveness improving while naturalness declines slightly.
7. **Follow the compositional frontier.** Use [[2512.18699|Task Vector in TTS]] and [[2512.14291|GLM-TTS]] to examine dialectal emotion, multi-attribute control, and large-scale preference post-training.

## Structured Source

This page selects citations for human readability. The complete claim graph, paper roles, source sections, caveats, and reassessment state are available in `wiki/_claims/emotion-synthesis.yaml`.

---

_This page is generated from `wiki/_claims/emotion-synthesis.yaml` (digest date: 2026-10-31)._
