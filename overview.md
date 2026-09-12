---
title: "Field Overview"
source_digest_date: "2026-09-12"
source_registry_date: "2026-09-12"
source_registry_digest: "sha256:530764ef596592dc955589b091e47471a7542596e911a4ebe40ca23a16b2fde5"
source_snapshot: 2025-Q3
source_snapshot_digest: "sha256:33ef4cf849c6e202e327b28cf39d95ea6128b78e474ae0514861b56490df463b"
evidence_cutoff: "2025-09-30"
assessment_as_of: "2026-09-12"
paper_count: 497
concept_membership_count: 2226
claim_cluster_count: 406
broader_claim_count: 11
source_concepts: [autoregressive-codec-tts, diffusion-tts, disentanglement, emotion-synthesis, evaluation-metrics, fine-tuning, flow-matching, gan-vocoder, instruction-conditioned-tts, multilingual-tts, neural-codec, prosody-control, rlhf-speech, self-supervised-speech, singing, speaker-adaptation, speech-to-speech, spoken-language-model, streaming-tts, subjective-evaluation, transformer-enc-dec-tts, voice-conversion, zero-shot-tts]
generation:
  schema_version: 2
  date: "2026-09-12"
  stage: render
  mode: full
  runtime: codex
  provider: openai
  agent: speech-generation-render-agent
  model: "gpt-5"
  commit: "96be404"
---

> [!abstract]
> Speech generation is no longer organized around a single acoustic-model choice. The central
> design problem is how to divide linguistic planning, acoustic reconstruction, voice and style
> control, interaction, and evaluation so that improving one capability does not silently damage
> another. This overview synthesizes all 23 integrated concept graphs and their human-reviewed
> cross-concept relationships.

> [!warning]
> **Retrospective Q3 boundary.** This is a current synthesis assessed on 2026-09-12 from the
> reviewed corpus published through 2025-09-30. It describes what that bounded evidence supports
> under the later review, not what reviewers contemporaneously knew at the end of Q3 2025.

## What Changes Decisions

### Evaluation defines what counts as progress

Speech quality is multidimensional. Semantic correctness and intelligibility, naturalness,
speaker identity, prosody and emotion, signal fidelity, latency, and interaction behavior can
change independently. A transcript score or one aggregate quality score can therefore conceal a
regression on an unmeasured dimension. This matters most when a system claims an architectural
speedup, stronger voice transfer, or more expressive control: the evaluation must measure the
quality placed at risk, not merely the optimized target ([[2406.18009|E2 TTS]],
[[2507.23159|Full-Duplex-Bench v1.5]], [[2509.24457|Neural-codec metric assessment]]).

Learned quality predictors are useful within their validation domain and generally track human
ratings better than classical signal-distortion measures. Their rankings can change across
languages, recording conditions, synthesis architectures, quality ranges, and utterance-versus-
system aggregation. Likewise, LLM and audio-language-model judges can separate clearly different
systems but are less reliable for near ties, fine-grained prosody or emotion, non-verbal behavior,
and domain shifts. Neither class of evaluator should replace a matched listening test without
validation for the same task, attribute, domain, and scoring granularity
([[2508.18240|MTalk-Bench]], [[2509.18531|No Verifiable Reward for Prosody]]).

This conclusion consolidates the overlapping evidence in **evaluation metrics**, **subjective
evaluation**, **instruction-conditioned TTS**, **emotion synthesis**, **speech-to-speech
systems**, **spoken language models**, and **neural codecs**. The narrower qualifications remain
important: system-level correlation does not establish utterance-level reliability, and a metric
validated for naturalness says little by itself about identity, expression, or interaction.

### Representation interfaces set the capability frontier

Speech tokens impose a recurring semantic–acoustic trade-off. Representations optimized for text
alignment and linguistic discrimination are easier for a language model to use for intelligible,
coherent generation, but retain less speaker, prosodic, and waveform detail. Reconstruction-
oriented codec tokens preserve more acoustic information but give the language model a harder
prediction problem. Frame rate, codebook structure, training objective, and decoder capacity move
the operating point; there is no universally best tokenizer ([[2209.03143|AudioLM]],
[[2301.02111|VALL-E]]).

Hierarchical systems mitigate this conflict by using semantic representations for longer-range
planning and acoustic tokens or continuous generators for reconstruction. This division supports
coherence without discarding all fine detail, and it connects **autoregressive codec TTS**,
**neural codecs**, **self-supervised speech representations**, **zero-shot TTS**,
**speech-to-speech systems**, and **spoken language models**. It does not eliminate the interface:
semantic-stage errors constrain later reconstruction, while an overly strong semantic objective
can suppress prosody or identity ([[2403.03100|NaturalSpeech 3]]).

### Architecture choice is now a bottleneck choice

The reviewed evidence does not support one generator as uniformly superior. Autoregressive codec
models retain causal semantic planning and flexible prompting, but token-by-token decoding creates
latency and alignment costs. Parallel transformer encoder–decoder systems remove that serial path,
yet require explicit duration, pitch, and variance modeling to avoid skipped, repeated, or flat
speech ([[1703.10135|Tacotron]], [[2006.04558|FastSpeech 2]]).

Diffusion models offer flexible conditional generation and iterative refinement but pay for it in
sampling cost. Flow matching often reaches a useful quality–latency point with fewer integration
steps, although comparisons remain confounded when target representations, solvers, backbones,
data, or hardware differ ([[2025.acl-long.313|F5-TTS]]). GAN vocoders remain a strong waveform-
decoding choice when low latency matters: discriminator design, periodic and spectral structure,
and training on predicted rather than only clean conditioning features are central to quality
([[2010.05646|HiFi-GAN]], [[2206.04658|BigVGAN]]). Iterative flow or diffusion vocoders can improve
some reconstruction dimensions, while GAN decoding generally retains the speed advantage.

The practical choice is consequently where to place sequential reasoning, parallel acoustic
generation, and perceptual reconstruction. This conclusion combines local evidence from
**autoregressive codec TTS**, **transformer encoder–decoder TTS**, **diffusion TTS**,
**flow matching**, **GAN vocoders**, **neural codecs**, and **streaming TTS** without treating
their headline latency figures as directly interchangeable.

### Control works through partial, not perfect, separation

Separately conditioning linguistic content, speaker identity or timbre, prosody, emotion, and
style generally enables more targeted transfer than forcing every attribute through one shared
bottleneck. This pattern recurs across codec, self-supervised, diffusion, autoregressive, voice-
conversion, speaker-adaptation, emotional, prosodic, and zero-shot systems
([[2025.acl-long.346|ControlSpeech]], [[2508.16332|Vevo2]]).

The factors are not statistically independent, and stronger separation is not always better.
Accent, timing, habitual expression, and pitch can jointly carry content and perceived identity;
removing their cross-factor information can reduce intelligibility, naturalness, or speaker
fidelity. In this overview, “disentanglement” therefore means reduced interference under an
evaluated intervention. It does not mean that a probe, auxiliary classifier, or downstream
similarity score has proved causal isolation of the latent factors.

Natural-language instructions extend control beyond fixed labels, but their evaluation inherits
the same problem: a holistic naturalness score cannot establish that the requested style or
emotion was followed. Evidence from **instruction-conditioned TTS**, **prosody control**, and
**emotion synthesis** supports attribute-specific listening or recognition tests alongside
naturalness, intelligibility, and identity measures.

### Unseen-voice transfer is feasible but condition-dependent

Reference-audio conditioning enables TTS and voice-conversion systems to generate an unseen
speaker without speaker-specific parameter updates at inference. That establishes zero-shot
feasibility, not uniform identity fidelity. Reference duration and content, noise and channel,
language and accent match, expressive style, training-speaker coverage, and the evaluation
protocol all affect the result ([[2303.03926|VALL-E X]], [[2403.03100|NaturalSpeech 3]]).

Across **speaker adaptation**, **voice conversion**, and **zero-shot TTS**, broader training
coverage of speakers and acoustic conditions is consistently associated with better unseen-voice
similarity and robustness. Total hours alone are not a causal explanation because language and
accent balance, filtering, transcription quality, pretraining, architecture, and model capacity
usually change at the same time. **Multilingual TTS** makes the boundary especially visible:
shared training and phonetic representations help low-resource languages, while imbalance can
bias quality toward high-resource languages and aggregate scores can hide per-language failures
([[2025.naacl-short.69|Multilingual synthesis for Ojibwe, Mi'kmaq, and Maliseet]],
[[2508.07426|Scalable Controllable Accented TTS]]).

Continued pretraining and parameter-efficient modules can extend language or speaker coverage, but
the evidence is uneven. The dedicated **fine-tuning** graph contains one paper: adapter tuning
preserved existing TTS capability better than full fine-tuning while adapting a lightweight
multilingual GAN system, an emerging result that is not yet architecture-independent
([[2508.18006|Unseen Speaker and Language Adaptation with Adapters]]).

### Full-duplex interaction requires behavioral control

A model is not an effective full-duplex conversational partner merely because it can accept audio
while emitting speech. It must also decide whether incoming speech is a genuine interruption, a
brief overlap, or a listener backchannel, and then continue, yield, or respond without losing the
meaning of its ongoing turn. This connects multi-stream autoregression, streaming generation,
speech-to-speech dialogue, and spoken language modeling ([[2411.00774|Freeze-Omni]],
[[2507.23159|Full-Duplex-Bench v1.5]]).

Current evaluations expose a specific trade-off. Systems that react more quickly to user speech
or barge-ins are more likely to stop or yield prematurely when they hear a short overlap or
backchannel. Systems that preserve their response and semantic coherence tend to respond more
slowly to genuine interruptions. The operating point depends on the scenario, thresholding, and
the latency and error profile of the underlying speech model; no evaluated system in the reviewed
corpus is strong across both behaviors ([[2509.22243|FLEXI]]). Time to first audio, total decoding
latency, interruption response, and turn-taking accuracy should therefore be reported separately.

### Post-training improves what its signal represents

Preference- and reward-based post-training has improved explicitly represented attributes across
autoregressive, diffusion, flow-matching, emotional-speech, and spoken-language-model systems.
Reported gains include intelligibility, speaker similarity, naturalness, emotional alignment, and
instruction following relative to each study's supervised or metric-only baseline
([[2025.acl-long.598|Preference alignment for zero-shot TTS]], [[2508.16332|Vevo2]]).

Those improvements do not establish a general perceptual gain. Outcomes depend on base-model
capability, preference-pair construction, reward coverage, objective weighting, and optimization
stability. Aggressive optimization can flatten prosody or reduce untargeted quality, and repeated
alignment cannot reliably create a capability absent from the base model
([[2509.18531|No Verifiable Reward for Prosody]]). Multi-objective rewards reduce one-dimensional
failure but do not remove the need for held-out listening tests that cover qualities omitted from
the reward. This is the shared conclusion across **RLHF for speech**, **subjective evaluation**,
**emotion synthesis**, **flow matching**, **diffusion TTS**, **autoregressive codec TTS**, and
**spoken language models**.

### Singing exposes the limits of speech-derived assumptions

The **singing** graph is small—ten papers, with all conclusions still emerging—but it provides an
important stress test. Speech references can support singer-identity transfer only when systems
address the cross-modal mismatch; high-pitch codec reconstruction benefits from singing data; and
identity, style accuracy, naturalness, and dynamic techniques such as vibrato remain difficult to
optimize together ([[interspeech-2025-0816|Bridging Speech and Singing]],
[[interspeech-2025-1397|VibE-SVC]], [[2509.15629|Singing Voice Conversion Challenge 2025]]).

Explicit F0 or energy controls and synchronized visual timing cues open useful control paths, but
their evidence is task- and dataset-specific ([[2509.07038|Phoneme-Level Energy Control]],
[[2509.22718|PerformSinger]]). Singing therefore cautions against assuming that an architecture,
tokenizer, identity metric, or adaptation recipe validated on speech transfers unchanged to longer
pitch trajectories and tightly constrained musical timing.

## Cross-Cutting Trade-offs

- **Semantic abstraction versus acoustic detail:** easier language modeling usually removes some
  speaker, prosodic, or waveform information; hierarchical reconstruction mitigates rather than
  erases the loss.
- **Parallelism versus explicit structure:** removing causal decoding reduces serial latency but
  shifts responsibility to duration, alignment, variance, and control models.
- **Independent control versus retained identity:** stronger factor separation improves targeted
  manipulation until it removes cues jointly carrying identity, accent, timing, or expression.
- **Fast interruption response versus preserving the speaking turn:** greater sensitivity to
  incoming audio reduces barge-in delay but increases premature stopping on overlaps and
  backchannels.
- **Optimization strength versus coverage:** reward or metric optimization improves represented
  attributes while leaving unrepresented qualities vulnerable.
- **Shared multilingual capacity versus uneven coverage:** transfer helps low-resource languages,
  but imbalance and front-end gaps can move errors into pronunciation, accent, duration, or
  speaker fidelity.

## Research Priorities

1. Evaluate systems with task-specific multidimensional protocols and report where automatic
   judges have been validated against matched listeners.
2. Compare tokenizers and generators under matched data, compute, target representation, decoder,
   hardware, and latency definitions.
3. Measure attribute leakage through interventions and direct probes, not only downstream quality
   or classifier accuracy.
4. Separate speaker count, language and accent coverage, recording diversity, and total hours in
   scaling and adaptation studies.
5. Evaluate spoken agents over overlap, backchannels, interruption, and multi-turn memory rather
   than inferring interaction quality from single-turn response scores.
6. Test whether post-training preserves untargeted qualities and generalizes beyond the reward
   model's data and scoring granularity.
7. Expand controlled evidence for low-resource languages, atypical voices, singing, and other
   conditions that expose failures hidden by aggregate English-speech benchmarks.

## Read by Concept

| Concept | Overview | In Depth |
|---------|----------|----------|
| Autoregressive Codec TTS | [[concepts/autoregressive-codec-tts\|Overview]] | [[concepts/autoregressive-codec-tts-in-depth\|In Depth]] |
| Diffusion Text-to-Speech | [[concepts/diffusion-tts\|Overview]] | [[concepts/diffusion-tts-in-depth\|In Depth]] |
| Disentanglement | [[concepts/disentanglement\|Overview]] | [[concepts/disentanglement-in-depth\|In Depth]] |
| Emotional and Expressive Speech Synthesis | [[concepts/emotion-synthesis\|Overview]] | [[concepts/emotion-synthesis-in-depth\|In Depth]] |
| Evaluation Metrics | [[concepts/evaluation-metrics\|Overview]] | [[concepts/evaluation-metrics-in-depth\|In Depth]] |
| Fine-Tuning Foundation Models | [[concepts/fine-tuning\|Concept page]] | Render pending |
| Flow Matching | [[concepts/flow-matching\|Overview]] | [[concepts/flow-matching-in-depth\|In Depth]] |
| GAN Vocoders | [[concepts/gan-vocoder\|Overview]] | [[concepts/gan-vocoder-in-depth\|In Depth]] |
| Instruction-Conditioned Text-to-Speech | [[concepts/instruction-conditioned-tts\|Overview]] | [[concepts/instruction-conditioned-tts-in-depth\|In Depth]] |
| Multilingual Text-to-Speech | [[concepts/multilingual-tts\|Overview]] | [[concepts/multilingual-tts-in-depth\|In Depth]] |
| Neural Audio Codecs | [[concepts/neural-codec\|Overview]] | [[concepts/neural-codec-in-depth\|In Depth]] |
| Prosody Control | [[concepts/prosody-control\|Overview]] | [[concepts/prosody-control-in-depth\|In Depth]] |
| RLHF for Speech | [[concepts/rlhf-speech\|Overview]] | [[concepts/rlhf-speech-in-depth\|In Depth]] |
| Self-Supervised Speech Representations | [[concepts/self-supervised-speech\|Overview]] | [[concepts/self-supervised-speech-in-depth\|In Depth]] |
| Singing Voice Synthesis and Conversion | [[concepts/singing\|Concept page]] | Render pending |
| Speaker Adaptation | [[concepts/speaker-adaptation\|Overview]] | [[concepts/speaker-adaptation-in-depth\|In Depth]] |
| Speech-to-Speech Systems | [[concepts/speech-to-speech\|Overview]] | [[concepts/speech-to-speech-in-depth\|In Depth]] |
| Spoken Language Models | [[concepts/spoken-language-model\|Overview]] | [[concepts/spoken-language-model-in-depth\|In Depth]] |
| Streaming Text-to-Speech | [[concepts/streaming-tts\|Overview]] | [[concepts/streaming-tts-in-depth\|In Depth]] |
| Subjective Evaluation | [[concepts/subjective-evaluation\|Overview]] | [[concepts/subjective-evaluation-in-depth\|In Depth]] |
| Transformer Encoder–Decoder TTS | [[concepts/transformer-enc-dec-tts\|Overview]] | [[concepts/transformer-enc-dec-tts-in-depth\|In Depth]] |
| Voice Conversion | [[concepts/voice-conversion\|Overview]] | [[concepts/voice-conversion-in-depth\|In Depth]] |
| Zero-Shot Text-to-Speech | [[concepts/zero-shot-tts\|Overview]] | [[concepts/zero-shot-tts-in-depth\|In Depth]] |

## Scope and Traceability

The 23 concept graphs contain 2,226 paper memberships but only 497 unique papers; 1,729 memberships
are overlaps because one paper can inform several concepts. They contain 406 local claim clusters
(280 strongly supported, 112 emerging, and 14 contested) and 173 method families. The overview
presents the 11 human-approved broader claims once and uses the 49 approved direct relationships
to interpret narrower connections. The 15 preserved agent proposals are excluded.

Evidence is bounded by canonical publication dates through 2025-09-30. The retrospective
assessment was completed on 2026-09-12 and is reproducibly recorded in the published
`2025-Q3` snapshot. The field overview itself remains a replaceable current-state render from the
living concept graphs and reviewed registry; the snapshot is the authority for the separate
historical quarterly report.

Every field conclusion traces from a human-approved broader claim or a concept-local claim cluster
to its role-specific paper evidence, then to the paper-page claim and cited source section.
Representative citations are selected for readability rather than exhaustively listed here. The
complete machine-readable provenance remains in `wiki/_claims/*.yaml` and
`wiki/_claims/_reconciliation/registry.yaml`.
