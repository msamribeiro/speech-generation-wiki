---
concept: autoregressive-codec-tts
render_type: in-depth
title: "Autoregressive Codec Text-to-Speech: In Depth"
source_digest_date: 2026-07-25
paper_count: 165
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

For the concise view, see [[concepts/autoregressive-codec-tts|Autoregressive Codec TTS]]. Complete
paper-to-claim provenance remains in `wiki/_claims/autoregressive-codec-tts.yaml`.

## Findings at a Glance

- Conditional prediction of codec tokens makes in-context zero-shot voice cloning practical.
- Semantic–acoustic token hierarchies balance linguistic coherence and waveform detail, but do not
  cleanly separate content, identity, and prosody.
- Tokenizer quality, rate, and structure constrain downstream generation as strongly as the language
  model does.
- Autoregressive decoding incurs sequential latency and remains vulnerable to omission, repetition,
  insertion, and runaway alignment failures.
- Explicit monotonic, duration, interleaving, or reordering constraints improve robustness while
  introducing supervision and portability costs.
- Scaling data and models improves complete systems, but architecture, tokenizer, corpus, and recipe
  changes remain confounded.
- Multilingual codec LMs transfer voice across languages, with recurring accent–identity trade-offs.
- Unified and multistream autoregression extends codec TTS into streaming and full-duplex dialogue.
- Preference optimization improves targeted quality beyond supervised training while exposing
  reward hacking and prosodic-diversity risks.
- Masked, parallel, and continuous alternatives show that full discrete causality is not universally
  required; matched evidence remains limited.

## Scope

The graph contains 165 papers published from 2016 through September 2025. TTS provides the majority
of direct evidence, with spoken conversational agents, codecs, voice conversion, evaluation, and
singing contributing downstream or infrastructure evidence. The 117 papers from 2025 make this a
rapidly evolving snapshot rather than a settled ranking of systems.

Fifteen clusters are strongly supported. One cluster—whether masked or parallel generators can
match autoregressive quality while reducing latency—is contested. Continuous-representation
autoregression is emerging from only two principal papers. Thirteen method families overlap because
modern systems commonly combine semantic autoregression, parallel acoustic generation, tokenizer
infrastructure, alignment constraints, and preference post-training.

Historical waveform autoregression such as [[1609.03499|WaveNet]] supplies lineage for sequential
generation but not direct evidence about modern codec-token behavior. [[2209.03143|AudioLM]] and
[[2301.02111|VALL-E]] establish central semantic/acoustic and in-context paradigms. Successor systems
sharing Microsoft, Google, or other organizational lineages are related evidence, not fully
independent confirmations. Surveys provide taxonomy rather than experimental replication.

## Research Landscape

The foundational template predicts one coarse or semantic codec stream autoregressively and then
generates residual acoustic codebooks in parallel. This **hierarchical AR+NAR** arrangement reduces
the longest sequential path while retaining an LM-like planning stage. Fully autoregressive
multicodebook systems instead keep every token causal, simplifying the objective while multiplying
sequence length.

The largest recent family uses **autoregressive tokens with a continuous renderer**. A language
model plans semantic or coarse acoustic structure; flow matching, diffusion, or another continuous
decoder reconstructs detail. These systems preserve the strengths of causal planning without asking
the LM to predict every acoustic codebook.

**Unified speech–text models** share a language-model backbone across modalities. **Full-duplex
multistream systems** delay or parallelize token streams so listening and speaking can overlap.
These families move beyond TTS, but their speech-generation quality remains dependent on the same
token and decoder decisions.

Alignment-constrained, preference-optimized, and scaled general-purpose families describe
interventions around the generator rather than mutually exclusive architectures. Masked and
parallel codec alternatives provide the main competing hypothesis to full autoregression, while
continuous-token autoregression tests whether sequence modeling can survive without quantization.

## What the Research Shows

### Codec prompting establishes zero-shot cloning

**Current assessment:** discrete speech-token language modeling enables unseen-speaker cloning from
short acoustic prompts without target-specific fine-tuning.

[[2301.02111|VALL-E]] is the best-known demonstration, with [[2306.12925|AudioPaLM]] and later
systems providing evidence across larger multimodal and multilingual settings. The architecture
turns reference speech into in-context conditioning rather than a fixed speaker label. That change
made speaker generalization an inference-time capability.

The degree of cloning depends on prompt duration, noise, channel, content, and training-speaker
coverage. Speaker embedding scores also differ by evaluator. The evidence establishes useful
identity transfer, not a universal fidelity level or prompt requirement.

### Semantic and acoustic streams divide the problem

**Current assessment:** separating semantic planning from acoustic refinement improves coherence
while retaining detail, with unavoidable leakage between levels.

[[2209.03143|AudioLM]], [[2308.16692|SpeechTokenizer]], and codec-language models show why a coarse
linguistic stream is easier to model over long contexts. Residual or acoustic stages then restore
speaker, pitch, and waveform detail. RVQ makes this staged design convenient because codebooks can
be generated in different orders or with different models.

The hierarchy is tokenizer-dependent. Semantic supervision may suppress expressive cues, while
early codebooks can still contain speaker information and later codebooks can retain content. The
levels should be probed and tested through interventions rather than named “semantic” or “acoustic”
by position alone.

### The tokenizer is a downstream bottleneck

**Current assessment:** semantic alignment, frame rate, bitrate, codebook utilization, and
reconstruction quality constrain both generation quality and modeling cost.

[[2305.02765|HiFi-Codec]] and [[2308.16692|SpeechTokenizer]] illustrate different priorities in
reconstruction and semantic organization. A codec can sound strong under ground-truth
reconstruction yet produce tokens that are difficult for an LM to predict. Conversely, easy
semantic tokens may omit identity or prosody that no decoder can faithfully recover.

Codec rankings therefore do not transfer automatically between generators, languages, or tasks.
Useful evaluations include reconstructed-reference ceilings, token prediction difficulty, sequence
rate, listener quality, intelligibility, and identity under one downstream system.

### Sequential decoding imposes latency

**Current assessment:** causal token generation has a deployment-relevant throughput cost even when
streaming reduces time to first audio.

Every temporal token depends on prior output. Multiple codebooks can multiply positions or require
careful interleaving. Streaming emits partial speech early, but it does not remove total sequential
computation. Hardware, batching, token rate, and decoder lookahead make published latency numbers
difficult to compare.

Hybrid AR-plus-flow or diffusion systems shorten the causal path by predicting only semantic or
coarse tokens sequentially. They shift compute to a continuous renderer and may improve throughput,
but the added stage can dominate latency or introduce a new interface error. Reports should
separate first-audio latency, total synthesis time, and sustained concurrency.

### Alignment failures define robustness

**Current assessment:** omission, repetition, insertion, and runaway loops remain characteristic
autoregressive failures, and WER alone does not capture their perceptual severity.

[[2406.05370|VALL-E 2]] and analysis across newer systems show that a plausible local next token
does not guarantee global text coverage. Long utterances, unusual text, noisy prompts, and sampling
increase exposure to drift. A short loop can sound locally natural while being unusable.

Robustness evaluation should report failure rates by length and input difficulty, not only average
WER. It should distinguish pronunciation errors from skipped text, duplicated phrases, truncation,
and looping. Candidate reranking can suppress failures at additional inference cost and can conceal
the base sampler's reliability.

### Explicit constraints improve alignment

**Current assessment:** monotonic guidance, duration information, phoneme interleaving, and sequence
reordering consistently reduce alignment errors.

[[2401.07333|ELLA-V]] reorders sequences to guide alignment; [[2404.03204|RALL-E]] adds explicit
reasoning-like alignment support. Other systems use durations, monotonic attention, or interleaved
text tokens. These methods reduce the search space and make text progress observable.

Their cost is dependency. Forced aligners and duration labels may be unreliable for spontaneous,
expressive, code-switched, or low-resource speech. A constraint tuned to one tokenizer or language
may not transfer. The open problem is robust intrinsic alignment without expensive labels or
reranking.

### Scaling helps but remains confounded

**Current assessment:** larger and more diverse training plus greater model capacity generally
improve intelligibility, identity, and expressive generalization, while causal attribution to scale
alone is weak.

[[2402.08093|BASE TTS]], [[2301.02111|VALL-E]], and large proprietary systems demonstrate the
direction of improvement. More speakers and acoustic conditions expose the model to variation it
must handle at inference. Capacity supports longer context and more complex token distributions.

Most scaling studies also replace the tokenizer, filtering, alignment, decoder, or post-training
recipe. Public reproduction is constrained by proprietary data. The evidence supports scaling the
complete recipe, not a universal parameter-count law.

### Multilingual transfer remains conditional

**Current assessment:** multilingual codec LMs transfer identity across languages from short
prompts, with accent naturalness and speaker similarity often pulling in different directions.

[[2303.03926|VALL-E X]] establishes cross-lingual codec modeling, while later multilingual systems
expand language coverage. The shared token space makes paired bilingual recordings unnecessary in
some settings.

Evidence remains concentrated in a limited set of language pairs. Text front ends, phonological
distance, and pretraining coverage change results. Evaluation should use native listeners and
separate intelligibility, accent, identity, and prosody rather than relying on one embedding.

### Unified and duplex systems broaden autoregression

**Current assessment:** shared speech–text vocabularies enable understanding and generation in one
backbone, while delayed or parallel streams permit streaming and full-duplex interaction.

[[2305.11000|SpeechGPT]] and [[2306.12925|AudioPaLM]] establish unified speech–text modeling.
[[2408.02622|Language Model Can Listen While Speaking]] and [[2410.00037|Moshi]] extend causal token
models to simultaneous input and output. The same sequential machinery becomes a conversational
policy and an acoustic generator.

Unified capability does not mean equal competence. Text reasoning, acoustic expressiveness,
latency, and interruption handling can favor different system designs. Many duplex evaluations use
synthetic conversations or offline simulation, so live interaction evidence remains narrower.

### Preference optimization improves targeted quality

**Current assessment:** preference and reinforcement-learning post-training improve intelligibility,
similarity, or perceived quality beyond supervised training, with structural reward risks.

[[2406.00654|Enhancing Zero-Shot TTS with Human Feedback]] and [[2406.02430|Seed-TTS]] show how
preferences redirect an existing codec LM. Later systems use learned or automatic rewards at larger
scale. Post-training is attractive because it changes behavior without rebuilding the tokenizer or
decoder.

An optimized proxy can be gamed. Intelligibility rewards may flatten prosody; similarity rewards may
harm pronunciation; predicted quality can diverge from listeners. Multi-dimensional held-out
evaluation, regularization, and early stopping are necessary.

### Alternatives challenge full discrete causality

**Current assessment:** masked and parallel codec models can reduce latency and sometimes match
autoregressive quality, but the comparison is contested and often unmatched.

[[2305.09636|SoundStorm]] demonstrates efficient parallel acoustic generation, and
[[2025.acl-long.65|Autoregressive Speech Synthesis without Vector Quantization]] questions discrete
codec prediction from another direction. Supporting and contradicting studies use different data,
tokenizers, prompts, and compute, preventing a universal winner.

Continuous-token autoregression, represented by [[2025.findings-naacl.184|Continuous Speech
Tokenizer in TTS]] and [[2412.16846|KALL-E]], is emerging. It avoids quantization loss while
reintroducing regression and rendering challenges. The useful research question is which parts of
the process need causality, not whether one label wins globally.

### Sampling creates diversity and instability

**Current assessment:** stochastic sampling supplies prosodic variation but increases output
variance and failure risk.

Temperature, top-k or top-p filtering, and candidate selection alter both diversity and reliability.
Deterministic decoding can sound repetitive; unconstrained sampling can drift. Reranking improves
the selected sample at extra cost and complicates fair system comparison.

Studies should publish sampling settings, number of candidates, rejection criteria, and variance
across repeated generations. One favorable sample is not evidence of a stable operating point.

### Controllability depends on imperfect separation

**Current assessment:** separating linguistic content, speaker identity, and style in codec tokens
improves controllability while introducing fidelity and identity trade-offs.

AudioLM-style hierarchies, [[2308.16692|SpeechTokenizer]], and controllable codec systems assign
different factors to prompts, semantic codes, or acoustic stages. This lets a system change text or
style while retaining a reference voice. The practical benefit is strong enough to recur across
TTS, VC, and spoken-generation pipelines.

The factors are not cleanly independent. Speaker embeddings can contain language and channel;
semantic tokens can retain pitch; acoustic prompts can transfer unwanted prosody. Stronger
invariance may improve attribute substitution while weakening the details listeners use to identify
a voice. Classifier probes demonstrate decodability but not necessarily causal control.

Evaluation should use counterfactual interventions: hold text and speaker fixed while changing
style, then measure the requested change and every preservation axis. Identity, intelligibility,
prosody, and naturalness should be tested together. A system should not be called disentangled only
because one probe performs poorly.

### Automatic metrics miss perceived failures

**Current assessment:** automatic intelligibility and signal-quality measures do not reliably
predict perceived naturalness or the severity of characteristic autoregressive errors.

[[2403.16973|VoiceCraft]] and human-feedback work show why WER or predicted quality is incomplete.
A repeated phrase may remain intelligible but be unacceptable; a speaker embedding may reward
identity while missing unnatural timing; a quality predictor may saturate near high quality. Metric
agreement also changes with codec, corpus, listener protocol, and error type.

Automatic measures remain valuable for screening large sample sets. They should be paired with
listener tests targeted at identity, naturalness, prosody, and failure severity. Repeated sampling
is essential because a stochastic model's distribution cannot be represented by one output per
sentence. Reconstructed-reference controls help isolate tokenizer artifacts from generation errors.

### Infrastructure and downstream claims differ

**Current assessment:** a codec or tokenizer can enable autoregressive TTS without independently
proving the quality of the complete generator.

Infrastructure papers establish bitrate, reconstruction, token structure, or decoder behavior.
Downstream systems establish that those tokens can support cloning, multilingual synthesis, or
dialogue. Historical papers establish lineage. Keeping these roles separate prevents adoption of a
tokenizer from being counted as repeated experimental confirmation that it is superior.

The same discipline applies to surveys and scaled successor systems. A taxonomy can explain why a
semantic–acoustic split is useful but does not replicate it. A successor sharing data, tokenizer,
and decoder extends a lineage rather than creating fully independent evidence. Confidence is
strongest where qualitative findings recur across organizations and representations.

This distinction also changes procurement and reproduction decisions. A practitioner selecting a
tokenizer should demand evidence under the intended generator and language, while a researcher
claiming a generation advance should include a fixed-tokenizer control. Otherwise a system-level
gain can be attributed to whichever component is emphasized in the paper even though several parts
changed. Recording the reconstructed ceiling, token rate, base LM, decoder, sampling policy, and
training data makes later comparisons scientifically usable.

## Where Findings Disagree

The formal disagreement concerns whether masked and parallel models match autoregressive quality.
Evidence supports lower latency, while quality conclusions reverse across tokenizers and datasets.
The graph therefore treats fully causal superiority as unresolved.

Other tensions are conditional rather than contradictory: semantic abstraction helps coherence but
can reduce prosody; scaling helps complete systems but does not isolate cause; alignment constraints
improve robustness but reduce portability; stochasticity improves diversity while weakening
determinism.

## How the Field Is Changing

After 2022, the field moved from sample- and frame-level autoregression toward compact semantic and
codec tokens. VALL-E-style hierarchical AR+NAR generation became a major template. By 2024–2025,
hybrids increasingly retained autoregressive planning while delegating acoustics to flow, diffusion,
or parallel decoders.

Tokenizer quality, alignment, and streaming replaced basic waveform fidelity as the main
differentiators. Unified and duplex models broadened the task from TTS to conversation. Preference
post-training emerged as a separate lever for improving mature base architectures.

## Implications

- Evaluate tokenizer ceilings and downstream prediction together.
- Report failure categories and length-stratified robustness, not WER alone.
- Separate first-audio latency, total synthesis time, and throughput.
- Treat prompt conditions and sampling settings as part of the method.
- Use native-listener, multidimensional tests for multilingual voice transfer.
- Match data, tokenizer, and compute before comparing causal and parallel generators.
- Monitor untargeted perception during preference optimization.
- Test duplex claims in live interaction, not only replayed or synthetic dialogue.

## Representative Reading Path

1. **Establish the token hierarchy.** Read [[2209.03143|AudioLM]] and
   [[2308.16692|SpeechTokenizer]].
2. **See in-context codec TTS.** Read [[2301.02111|VALL-E]] with
   [[2303.03926|VALL-E X]] for zero-shot and multilingual transfer.
3. **Study robustness.** Pair [[2401.07333|ELLA-V]] with [[2404.03204|RALL-E]].
4. **Compare parallel acoustics.** Read [[2305.09636|SoundStorm]] and
   [[2025.acl-long.65|Autoregressive Speech Synthesis without Vector Quantization]].
5. **Follow unified interaction.** Read [[2305.11000|SpeechGPT]],
   [[2408.02622|Language Model Can Listen While Speaking]], and [[2410.00037|Moshi]].
6. **Examine post-training.** Use [[2406.00654|Enhancing Zero-Shot TTS with Human Feedback]] and
   [[2406.02430|Seed-TTS]] to connect rewards with perceptual trade-offs.

## Structured Source

Complete structured claims and provenance are in
[`wiki/_claims/autoregressive-codec-tts.yaml`](../_claims/autoregressive-codec-tts.yaml).

---

_This page is generated from `wiki/_claims/autoregressive-codec-tts.yaml` (digest date: 2026-07-25)._
