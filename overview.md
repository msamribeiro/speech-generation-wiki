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
  date: "2026-09-13"
  stage: render
  mode: full
  runtime: codex
  provider: openai
  agent: speech-generation-render-agent
  model: "gpt-5"
  commit: "96be404"
---

> [!abstract]
> Modern speech generation is about much more than making a voice sound natural. A useful system
> must also get the words right, preserve the intended speaker, express the right style, respond
> at the right time, and behave well outside a clean test set. This overview brings together all
> 23 integrated concept areas to explain where those goals reinforce one another—and where they
> pull in different directions.

> [!warning]
> **Retrospective Q3 boundary.** This overview was assessed on 2026-09-12 using reviewed evidence
> published through 2025-09-30. It describes what that bounded evidence supports under the later
> review, not what researchers necessarily knew at the end of Q3 2025.

## What Matters Across the Field

### A good score is not the same as good speech

Speech quality has several parts. A system can pronounce the words correctly and still sound
unnatural. It can sound natural while losing the target speaker's identity. It can preserve the
voice but flatten the emotion or respond awkwardly in conversation. That is why a transcript
score or a single overall quality score is rarely enough. Evaluations need to measure the
dimensions that matter for the task, especially any quality that a new method might put at risk
([[2406.18009|E2 TTS]], [[2025.naacl-srw.6|Codec-LM Co-design]],
[[2507.23159|Full-Duplex-Bench v1.5]], [[2509.24457|Neural-codec metric assessment]]).

Automatic evaluators still have an important role. Learned quality predictors usually follow
human ratings better than older signal-distortion measures when the test data resembles their
training data. But their rankings can change with the language, recording conditions, synthesis
architecture, or level of analysis. A metric that works across whole systems may still be poor at
judging individual utterances ([[2508.00317|Speech Quality Assessment review]],
[[2509.18823|Neural Audio Codec Embedding Distances]]).

The same caution applies to LLM and audio-language-model judges. They can often separate systems
that are clearly different, but become less dependable for close comparisons, fine-grained
prosody or emotion, non-verbal sounds, and unfamiliar domains. Before replacing listeners, an
automatic judge should be checked against people on the same task, domain, attribute, and scoring
scale ([[2508.18240|MTalk-Bench]], [[2509.13989|Instruction-Perception Gap]],
[[2509.19928|Prosody Diversity Benchmark]]).

This shared lesson appears in the work on **evaluation metrics**, **subjective evaluation**,
**instruction-conditioned TTS**, **emotion synthesis**, **speech-to-speech systems**, **spoken
language models**, and **neural codecs**. The practical message is simple: measure each important
quality separately, and be clear about what an automatic score has actually been validated to
measure.

### Speech tokens have to serve two different purposes

Language models want speech tokens that make words and sentence structure easy to predict. Audio
decoders want tokens that keep the details of the voice: rhythm, pitch, timbre, and fine waveform
structure. Improving tokens for one purpose often makes them less useful for the other. Tokens
trained mainly for linguistic discrimination tend to support clearer, more coherent generation;
tokens trained mainly for reconstruction tend to preserve richer sound detail
([[2209.03143|AudioLM]], [[2308.16692|SpeechTokenizer]],
[[2025.naacl-srw.6|Codec-LM Co-design]]).

Many systems handle this by splitting the job. A semantic representation plans the words and
longer-range structure, then acoustic tokens or a continuous generator rebuild the voice and
waveform. This is the basic idea behind several hierarchical systems, from codec language models
to zero-shot TTS and spoken agents ([[2301.02111|VALL-E]], [[2305.09636|SoundStorm]],
[[2403.03100|NaturalSpeech 3]]).

The split helps, but it does not make the problem disappear. If the semantic stage makes an error,
the acoustic stage usually cannot repair the missing meaning. If semantic supervision is too
strong, the representation may throw away useful speaker or prosodic information. Long-form
generation adds another challenge: both the linguistic plan and the acoustic realization must
remain consistent over time ([[2412.18603|Long-Form Speech Generation with Spoken Language Models]],
[[2509.26276|Optimizing Speech Language Models for Acoustic Consistency]]).

This trade-off ties together **autoregressive codec TTS**, **neural codecs**, **self-supervised
speech representations**, **zero-shot TTS**, **speech-to-speech systems**, and **spoken language
models**. There is no universally best tokenizer. Frame rate, codebook structure, training
objective, and decoder capacity all change where a representation sits between easy language
modeling and faithful audio reconstruction.

### Choosing an architecture means choosing a bottleneck

There is no single generator that wins on every requirement. Autoregressive codec models are good
at step-by-step semantic planning and flexible prompting, but token-by-token decoding costs time
and can accumulate alignment errors. Parallel transformer encoder–decoder systems remove much of
that serial work, but then need explicit duration, pitch, and variation models to avoid skipped,
repeated, or flat speech ([[1703.10135|Tacotron]], [[2006.04558|FastSpeech 2]]).

Diffusion models are flexible and can refine an output over several steps, but those steps add
latency. Flow matching often reaches useful quality with fewer steps. Even so, a headline step
count is not a fair comparison unless the systems use similar data, representations, solvers,
hardware, and latency definitions ([[iclr-2025-hQvX9MBowC|DiTTo-TTS]],
[[2025.acl-long.313|F5-TTS]]).

The same choice appears at the waveform decoder. GAN vocoders remain attractive when very fast
generation matters. Their quality depends heavily on discriminator design, periodic structure,
and whether they were trained on the imperfect features they will see at deployment
([[2010.05646|HiFi-GAN]], [[2206.04658|BigVGAN]]). Flow and diffusion vocoders can improve some
parts of reconstruction through iterative refinement, while distillation can narrow the speed gap
([[iclr-2025-tQ1PmLfPBL|PeriodWave]], [[2025.naacl-long.110|WaveFM]]).

For streaming systems, it also helps to separate time to first audio from total generation time.
A model can begin speaking quickly and still have expensive sequential decoding afterward.
Causal codecs, vocoders, and voice-conversion systems can approach offline quality, but only when
their representations and training are adapted for limited future context
([[2509.16195|FocalCodec-Stream]], [[2507.14534|Conan]],
[[interspeech-2025-1819|Low-Latency Vocoder Analysis]]).

The most useful question is therefore not “Which architecture is best?” It is “Where should this
system spend its sequential reasoning, parallel acoustic generation, and perceptual reconstruction
budget?” The answer depends on whether the main constraint is latency, long-form stability,
control, compute, or final waveform quality.

### Control works because factors are partly separable

It is easier to control a system when content, speaker identity, prosody, emotion, and style do not
all share one undifferentiated representation. Separate conditioning paths let a user change one
attribute while trying to preserve the others. This pattern appears across TTS, voice conversion,
neural codecs, and expressive generation ([[2025.acl-long.346|ControlSpeech]],
[[2403.03100|NaturalSpeech 3]], [[2508.16332|Vevo2]]).

But “disentangled” does not mean perfectly independent. Accent, timing, pitch, identity, and
habitual expression naturally overlap. Removing every trace of one factor can also remove
information needed for another, reducing intelligibility, naturalness, or speaker similarity.
Direct pitch mechanisms and targeted preservation losses often help precisely because a generic
latent split does not capture every dependency ([[interspeech-2025-0347|PeriodCodec]],
[[interspeech-2025-1394|DiEmo-TTS]], [[interspeech-2025-1101|ZSDEVC]]).

Natural-language instructions make control more flexible than fixed labels, but they also make
success harder to measure. A sample may sound good while ignoring the requested style, or follow
the instruction while harming the voice. Fine-grained emotion work shows the same issue: a global
score can miss changes within an utterance ([[2509.20378|Fine-Grained Emotional Speech Synthesis]],
[[2509.13989|Instruction-Perception Gap]]). In this overview, disentanglement means
reduced interference under a tested intervention—not proof that the latent factors are
statistically independent.

### Zero-shot voice transfer works, but not equally well everywhere

A short reference recording can let TTS or voice-conversion systems speak in the voice of someone
who was not in the training set. That is a real and well-supported capability. It does not mean
that every reference works equally well. Duration, spoken content, noise, channel, language,
accent, emotion, and the evaluation method can all change how much identity is preserved
([[2303.03926|VALL-E X]], [[2312.01479|OpenVoice]],
[[2403.03100|NaturalSpeech 3]]).

Training diversity matters here. Across **speaker adaptation**, **voice conversion**, and
**zero-shot TTS**, systems trained on a broader range of speakers and recording conditions tend to
generalize better to unseen voices. Still, total hours alone do not explain the gain. Speaker,
language, and accent coverage; filtering; transcription quality; pretraining; architecture; and
model capacity usually change together.

**Multilingual TTS** makes these limits easy to see. Shared training and common phonetic
representations can help low-resource languages, but an imbalanced corpus can pull quality toward
high-resource languages. Average scores then hide pronunciation or speaker-identity problems in
individual languages ([[2025.naacl-short.69|Multilingual synthesis for Ojibwe, Mi'kmaq, and Maliseet]],
[[2025.acl-short.81|Vietnamese Zero-Shot TTS]], [[2508.14049|MahaTTS]]).

Language and dialect also need explicit care. A common phonetic alphabet can reduce spelling-to-
sound ambiguity, while language identifiers or routing can prevent a shared model from averaging
distinct accents and speaking styles ([[2312.01479|OpenVoice]],
[[2509.22727|DiaMoE-TTS]]). Continued pretraining and lightweight adapters can extend coverage,
but the dedicated **fine-tuning** evidence is still small. Its single paper found that adapters
preserved an existing multilingual GAN-TTS system better than full fine-tuning during new-speaker
and new-language adaptation; that result is promising, not yet architecture-independent
([[2508.18006|Unseen Speaker and Language Adaptation with Adapters]]).

### A full-duplex model must manage the conversation

Letting a model listen while it speaks is only the first step. A useful full-duplex agent must
also decide what incoming speech means. Is the user genuinely interrupting? Is it a brief overlap?
Is it a listener saying “mm-hm” to encourage the speaker to continue? The agent then has to keep
speaking, yield, or respond without losing the meaning of its current turn
([[2411.00774|Freeze-Omni]], [[2410.17799|OmniFlatten]]).

Responding sooner is not always better. Systems that react quickly to any user sound are more
likely to stop unnecessarily when they hear a short overlap or backchannel. Systems that protect
their current response tend to react more slowly to genuine interruptions. In the reviewed
evaluations, no system handles both sides strongly across interaction scenarios
([[2503.04721|Full-Duplex-Bench]], [[interspeech-2025-0739|FD-Bench]],
[[2507.23159|Full-Duplex-Bench v1.5]], [[2509.22243|FLEXI]]).

This is the concrete meaning of the responsiveness-versus-floor-holding trade-off: the system is
balancing quick interruption response against its ability to continue speaking when it still has
the conversational turn. The right balance depends on the setting, interruption threshold, and
errors or delays in the underlying speech model. Time to first audio, total decoding time,
interruption response, backchannel behavior, and turn-taking accuracy should be measured
separately.

### Post-training improves what the feedback can see

Preference and reward training can improve the qualities that appear in the feedback signal.
Across several speech architectures, studies report better intelligibility, speaker similarity,
naturalness, emotion, or instruction following than their supervised or metric-only baselines
([[2025.acl-long.598|Preference Alignment for Zero-Shot TTS]],
[[2025.acl-long.997|Align-SLM]], [[2509.25416|Emotion-Aligned Diffusion TTS]]).

The catch is that the optimizer cannot protect a quality it cannot see. A reward focused on word
accuracy may flatten prosody. A preference model trained in one domain may favor the wrong cues in
another. Poorly constructed preference pairs or overly long optimization can also reduce held-out
quality ([[interspeech-2025-0063|RLHF for Diffusion TTS]],
[[2509.18531|No Verifiable Reward for Prosody]], [[2509.18928|Speech DPO]]).

Multi-objective rewards help, but the weights still express a choice about which failures matter
most. Post-training also cannot reliably create a capability missing from the base model. The
evidence across **RLHF for speech**, **subjective evaluation**, **emotion synthesis**, **flow
matching**, **diffusion TTS**, **autoregressive codec TTS**, and **spoken language models** therefore
supports a bounded conclusion: feedback optimization is useful, but its gains are only as broad
as the signal and validation used to guide it.

### Singing is a useful stress test

The **singing** evidence is much smaller—ten papers, with all conclusions still emerging—but it
tests assumptions that are easy to miss in ordinary speech. Speech references can guide singer
identity only if the system handles the mismatch between speaking and singing. Codecs need
training examples that cover high pitches. Identity, style, naturalness, vibrato, and musical
timing are difficult to improve together ([[interspeech-2025-0816|Bridging Speech and Singing]],
[[interspeech-2025-1397|VibE-SVC]], [[2509.15629|Singing Voice Conversion Challenge 2025]]).

Explicit pitch or energy controls make some aspects easier to steer, and synchronized video can
provide timing information when phoneme durations are unavailable. These are useful early results,
not general solutions ([[2509.07038|Phoneme-Level Energy Control]],
[[2509.22718|PerformSinger]]). Singing reminds us that a tokenizer, identity metric, or adaptation
recipe that works on speech may not transfer unchanged to long pitch movements and tightly fixed
musical timing.

## The Recurring Trade-offs

- **Easy language modeling versus rich audio detail.** More abstract tokens simplify word and
  sentence prediction but usually discard some information about voice and prosody.
- **Parallel speed versus explicit structure.** Removing token-by-token decoding saves time, but
  duration, alignment, pitch, and variation then need to be modeled elsewhere.
- **Independent control versus preserved identity.** Stronger factor separation makes targeted
  changes easier until it also removes cues that carry accent, timing, expression, or identity.
- **Quick interruption response versus continuing the current turn.** A more sensitive agent
  reacts faster to genuine barge-ins but is also more likely to stop for harmless overlap.
- **Strong optimization versus broad quality.** Reward or metric optimization improves what it
  measures while leaving other qualities exposed.
- **Shared multilingual capacity versus even coverage.** Joint training helps transfer, but weak
  languages and dialects can disappear inside an acceptable average score.

## Questions Worth Following

1. Can automatic evaluators remain reliable across languages, architectures, domains, and close
   system comparisons—and where are human listeners still essential?
2. Which tokenizer properties predict both easy language modeling and good reconstructed speech
   when data, decoder, and compute are controlled?
3. How should attribute leakage be measured directly, rather than inferred from one downstream
   score or classifier?
4. Which parts of unseen-voice generalization come from more hours, and which come from speaker,
   language, accent, and recording diversity?
5. Can spoken agents handle overlap, backchannels, interruption, and long conversations without
   trading away semantic coherence?
6. Can preference training protect qualities that are absent from its reward and validation data?
7. Which lessons from ordinary speech survive in low-resource languages, atypical voices, and
   singing?

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

The 23 concept graphs contain 2,226 paper memberships but 497 unique papers. The difference—1,729
memberships—comes from papers that inform more than one concept. The graphs contain 406 local claim
clusters: 280 strongly supported, 112 emerging, and 14 contested. This overview presents the 11
human-approved broader claims once and uses the 49 approved direct relationships to interpret
narrower connections. The 15 preserved agent proposals are not used as evidence.

Evidence is limited by canonical publication dates through 2025-09-30. The retrospective
assessment was completed on 2026-09-12 and is recorded in the published `2025-Q3` snapshot. This
overview remains a replaceable current-state rendering of the living concept graphs and reviewed
registry; the snapshot is the authority for the separate historical quarterly report.

Every conclusion traces from a human-approved broader claim or a local claim cluster to paper-level
evidence, then to the paper page and cited source section. Citations here are representative rather
than exhaustive. Complete provenance remains in `wiki/_claims/*.yaml` and
`wiki/_claims/_reconciliation/registry.yaml`.
