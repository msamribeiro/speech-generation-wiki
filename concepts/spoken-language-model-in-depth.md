---
concept: spoken-language-model
render_type: in-depth
title: "Spoken Language Models: In Depth"
source_digest_date: 2026-07-27
paper_count: 127
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

For the concise view, see [[concepts/spoken-language-model|Spoken Language Models]]. Complete
paper-to-claim provenance remains in `wiki/_claims/spoken-language-model.yaml`.

## Findings at a Glance

- Discrete speech tokens make large-scale modeling tractable, while representation choices trade
  semantic content against voice, prosody, and timing.
- Low and variable frame rates reduce sequence cost when allocation follows information density;
  excessive compression damages speech detail.
- Shared backbones can support understanding and generation, without equal competence across
  reasoning, acoustics, and interaction.
- Speech–text alignment and interleaving preserve much pretrained text intelligence while adding
  speech capabilities.
- Incremental tokenization and decoding reach interactive latency, but reported numbers often omit
  endpointing and other pipeline costs.
- Full duplex requires explicit overlap, interruption, backchannel, and turn-transition behavior.
- Paralinguistic reasoning remains weaker than transcript-level semantics.
- Multi-turn conversation needs memory and state management; longer context alone is insufficient.
- Cascaded and end-to-end systems retain complementary strengths rather than one universal winner.
- Multidimensional live evaluation and safety evidence lag rapid capability growth.

## Scope

The graph contains 127 papers from 2022 through September 2025, with 101 from 2025. Ninety-six
papers address spoken conversational agents, while TTS, codecs, evaluation, voice conversion, and
singing provide adjacent evidence. Forty-one papers play infrastructure roles, and four provide
historical framing. Adoption inside a system supports viability, not independent proof that a token
or alignment mechanism is optimal.

Sixteen clusters are strongly supported. Safety, bias, privacy, misuse, and provenance form one
emerging cluster supported by limited populations and threat models. Six method families overlap:
autoregressive speech LMs dominate, while hybrid multistage systems and flow decoders separate
reasoning from acoustic rendering. Transformer encoder–decoder, adversarial codec, and variational
front ends provide alternatives or infrastructure.

Comparisons are heavily confounded by base text LM, tokenizer, training data, instruction tuning,
decoder, and hardware. A fluent demonstration does not establish reasoning parity, full-duplex
robustness, or safe deployment. Offline replay and synthetic conversations are weaker evidence than
live interaction.

## Research Landscape

Autoregressive speech LMs convert speech into semantic or acoustic tokens and extend a causal
language model across modalities. Some generate speech tokens directly; others reason internally in
text and render speech through a separate decoder. Low-rate tokenization reduces the gap between
speech sequence length and text sequence length.

Hybrid systems retain cascaded modules for ASR, text reasoning, or TTS while sharing training or
routing. They offer inspectability and strong component performance. End-to-end systems preserve
timing and non-verbal information that transcripts discard. Dynamic hybrids try to select between
these paths.

Flow and latent decoders map language-model plans into continuous acoustics, reducing token-by-token
waveform detail. Full-duplex systems add multiple delayed or parallel streams so input and output
overlap. The architecture question is therefore where to place semantic reasoning, acoustic
planning, and conversational control—not simply whether a system is “end to end.”

## What the Research Shows

### Tokens make speech modeling tractable

**Current assessment:** discrete semantic and codec tokens convert waveforms into compact sequences
that can be modeled at language-model scale.

[[2209.03143|AudioLM]] establishes hierarchical audio modeling, and [[2301.02111|VALL-E]] shows
in-context generation through codec tokens. Later spoken models reuse this interface for dialogue
and multimodal reasoning.

Compactness does not guarantee preservation. Token rate, codebooks, and objectives decide whether
linguistic content, identity, prosody, and non-verbal events survive. A strong decoder can make a
lossy token sound plausible without recovering the original information.

### Semantic and acoustic information trade off

**Current assessment:** tokenizers face a persistent balance between intelligible semantic content
and fine acoustic, speaker, and prosodic fidelity.

Semantically supervised units simplify reasoning and long context. Acoustic codecs retain voice and
timing but lengthen sequences and complicate prediction. Hybrid systems assign content to one stream
and detail to another.

The split is porous. Prosody carries linguistic and interaction meaning; identity influences how an
utterance is interpreted. Representation evaluation should include semantic tasks, voice and
prosody, reconstruction, and downstream conversation rather than one codec score.

### Low-rate tokens reduce sequence cost

**Current assessment:** low and variable frame rates improve efficiency when tokens are allocated by
information density, with an eventual cost to timing and detail.

[[2408.16725|Mini-Omni]] and later low-rate work reduce speech positions so the LM can handle longer
contexts and respond faster. Variable-rate tokenizers spend capacity around changes rather than
uniformly over silence or steady regions.

Aggressive compression damages intelligibility, timing precision, identity, or prosody. Rate claims
must include tokenizer latency, decoder lookahead, and downstream quality. Average rate can also
hide bursts that matter for streaming.

### Shared backbones unify understanding and generation

**Current assessment:** one language-model backbone can accept and emit speech across conversational
tasks, while unified capability does not imply equal quality on every dimension.

[[2305.11000|SpeechGPT]], [[2306.12925|AudioPaLM]], and [[2409.06666|LLaMA-Omni]] show different
ways to connect speech with pretrained LMs. Shared representations support transfer and reduce the
need for separate task models.

Text reasoning, acoustic expression, recognition, and turn policy can still have different
bottlenecks. Unified systems may trail a cascade using a stronger text LM or specialized ASR/TTS.
Evaluation must retain task-specific baselines.

### Alignment preserves text intelligence

**Current assessment:** connectors, paired objectives, and speech–text alignment transfer much of a
pretrained text LM's reasoning while adding speech input and output.

Transcript-linked training teaches correspondence without rebuilding linguistic knowledge.
Interleaved speech and text additionally lets the model reason in one modality and communicate in
another. [[2402.05755|Spirit LM]] and synthetic interleaving work show the breadth of this approach.

Transcript-accessible knowledge transfers more readily than emotion, speaker traits, hesitation, or
non-verbal cues. Synthetic interleaving can inherit teacher errors and unnatural timing. Alignment
should therefore measure preserved text ability and newly acquired acoustic competence separately.

### Streaming reaches interactive latency

**Current assessment:** incremental tokenization, generation, and decoding allow spoken models to
respond at interactive latency, while reporting remains implementation-specific.

[[2408.16725|Mini-Omni]], [[2409.06666|LLaMA-Omni]], and [[2410.00037|Moshi]] demonstrate streaming
paths. The model can begin speaking before a complete response is generated.

Published latency may exclude voice activity detection, endpointing, network transfer, or initial
speech encoding. Real-time factor also differs from time to first meaningful audio. Systems should
report the whole interactive pipeline and performance under concurrency.

### Full duplex requires conversation control

**Current assessment:** natural duplex interaction requires listening while speaking plus explicit
policies for overlap, interruption, backchannels, and turn transitions.

[[2408.02622|Language Model Can Listen While Speaking]] and [[2410.00037|Moshi]] use parallel or
delayed streams to remove strict turn segmentation. This enables interruption and overlapping audio
in principle.

Many evaluations use scripted or synthetic dialogues. Successful overlap is not merely low latency:
the model must recognize whether speech is a backchannel, interruption, or noise and decide whether
to continue. Live, multilingual evaluation remains limited.

### Paralinguistic reasoning remains weaker

**Current assessment:** spoken LMs handle transcript semantics more reliably than prosody, emotion,
speaker traits, and non-verbal vocal behavior.

Benchmarks repeatedly reveal gaps in acoustic-only cues even when text reasoning is strong. The
problem combines tokenizer loss, alignment objectives, data labeling, and evaluator reliability.
Raw audio access does not ensure that a model uses the information.

Tests should use counterfactual pairs with identical transcripts and different prosody or speaker
state, then require behavior that depends on the acoustic difference. Text-derived labels alone can
teach shortcuts.

### Speech introduces reasoning and robustness gaps

**Current assessment:** replacing text with speech produces measurable reasoning, instruction, and
robustness degradation even when the underlying LM is capable.

Errors can enter through recognition, tokenization, modality alignment, or output generation. A
single end-to-end score cannot isolate them. Cascaded transcript controls reveal how much capability
is lost at the speech interface.

Evaluation should compare text input, clean speech, accented or noisy speech, and end-to-end spoken
output under the same tasks. This turns “speech reasoning” into diagnosable stages.

### Memory requires retrieval, not just context

**Current assessment:** multi-turn spoken interaction needs explicit conversational state and
long-horizon retrieval; a larger context window alone does not ensure relevant recall.

[[2025.coling-industry.29|CarMem]] studies structured long-term memory for assistants. Later systems
track dialogue history, user information, or summaries. Speech adds recognition uncertainty and
turn-boundary ambiguity to text-memory problems.

Memory tests should measure correct use, contradiction, privacy, and forgetting over realistic time
gaps. Storing every acoustic turn is expensive and can retain sensitive identity information.

### Long-form output needs hierarchical planning

**Current assessment:** long-form speech benefits from separating discourse continuity from local
acoustic rendering.

Hierarchical and multiscale systems plan content over longer horizons while generating speech in
manageable segments. [[2508.19205|VibeVoice]] is representative of the push toward extended
generation.

Local naturalness can remain high while discourse repeats, drifts, or contradicts itself. Long-form
evaluation must track semantic coherence and accumulated acoustic errors, not sample isolated clips.

### Alignment improves spoken behavior

**Current assessment:** reward and preference post-training improve helpfulness, naturalness, or
behavioral control beyond supervised training, with cross-dimension risks.

[[2502.11946|Step-Audio]] and later alignment studies optimize spoken behavior directly. This can
address interaction failures not represented in transcript likelihood.

Rewards omit qualities. A helpfulness judge may ignore voice stability; a naturalness reward may
miss factual accuracy. Model selection needs held-out semantic, acoustic, and interaction measures,
plus human checks for reward gaming.

### Instruction data expands capability

**Current assessment:** large and diverse speech-instruction and conversational datasets broaden
task coverage and generalization, while scale can conceal imbalance and synthetic-label noise.

[[2305.11000|SpeechGPT]], [[2409.06666|LLaMA-Omni]], and later instruction-tuned systems use paired
or interleaved data to teach speech following, dialogue, and generation. Coverage across intents,
voices, environments, and conversational structures supplies behaviors that pretraining likelihood
alone does not define.

Quantity is not interaction diversity. Synthetic dialogues can inherit a text teacher's style and
omit hesitation, overlap, repair, emotion, or culturally specific turn behavior. Demographic and
language imbalance can produce uneven recognition and recommendations. Dataset reports should
describe speakers, languages, acoustic conditions, dialogue sources, synthetic proportions, and
interaction structures, then evaluate the same strata.

### Benchmarks must be multidimensional and live

**Current assessment:** text-only accuracy and aggregate speech quality miss reasoning, prosody,
turn-taking, memory, and robustness failures in spoken systems.

Newer benchmarks use full-duplex, multi-turn, role-playing, prosody, and situated tasks. They expose
cases where a transcript is correct but timing is disruptive, where speech is natural but reasoning
fails, or where the model ignores acoustic emotion. This broader coverage is strongly supported
across the graph.

Benchmark conclusions depend on judge reliability, prompt coverage, language, and whether dialogue
is replayed or live. Automated judges may evaluate semantic rubrics more reliably than voice or
non-verbal behavior. Strong benchmarks publish component scores and human calibration instead of
hiding dimensions inside one leaderboard.

### Long-form evaluation must track accumulation

**Current assessment:** hierarchical planning helps extended speech, while local clip quality
understates discourse drift and accumulated acoustic errors.

Long-form models divide discourse planning, semantic tokens, and acoustic rendering across scales.
This lets the system maintain a voice and topic beyond one sentence. The relevant failures include
repetition, contradiction, forgotten entities, speaker drift, and discontinuities between chunks.

Evaluation should score discourse and acoustics over duration, record failure time, and include
memory-dependent questions. Selecting short excerpts from a long output can conceal the exact
failure mode the architecture is intended to solve.

### Acoustic-only information needs direct tests

**Current assessment:** speech–text alignment efficiently transfers transcript-level knowledge,
but competence in emotion, hesitation, emphasis, laughter, and speaker state requires training and
evaluation that cannot be reconstructed from text alone.

Synthetic transcripts and derived labels are scalable, yet can turn acoustic understanding into a
shortcut classification problem. Counterfactual recordings with identical words and different
delivery reveal whether behavior actually depends on speech. The response should change only when
the acoustic cue is relevant, which also tests overreaction to incidental style.

This design matters for safety and accessibility. Misreading distress, sarcasm, uncertainty, or a
speech impairment can change advice even when the transcript is correct. Evaluations should include
native and atypical speech, report subgroup results, and use human-validated acoustic contrasts.

### Routing can combine complementary systems

**Current assessment:** dynamic routing between end-to-end and cascaded paths is a plausible way to
combine latency and acoustic sensitivity with high-quality text reasoning, but matched deployment
evidence remains limited.

Simple conversational turns may benefit from a direct low-latency path. Knowledge-intensive or
high-stakes queries may benefit from explicit transcription, tool use, and an inspectable response
before synthesis. A router adds its own errors and latency, so it should be evaluated on correct path
selection, fallback behavior, and the cost of switching modalities mid-conversation.

### Cascades and end-to-end systems complement each other

**Current assessment:** cascades retain strong reasoning and controllability, while end-to-end paths
preserve timing and paralinguistic information; comparisons do not establish one universal winner.

Cascades expose transcripts and allow component replacement. End-to-end models reduce interfaces
and can react directly to acoustic cues. Hybrid routing can choose a path by task or latency need.

Studies frequently compare different base LMs, data, and decoders. Matched comparisons should hold
reasoning capacity and speech components constant and include failure attribution.

### Evaluation and safety lag capability

**Current assessment:** multidimensional benchmarks expose gaps missed by text accuracy or aggregate
quality, while deployment safety evidence remains emerging.

Benchmarks now cover reasoning, prosody, role-playing, turn-taking, memory, and duplex behavior.
Judge reliability, prompt coverage, language, and live versus replayed interaction affect results.

[[interspeech-2025-2328|A Watermark for Autoregressive Speech Generation]] and
[[2510.02352|Evaluating Bias in Spoken Dialogue LLMs]] provide early provenance and bias evidence.
Limited populations and threat models prevent broad safety claims. Privacy, impersonation, and
generated-speech provenance require explicit deployment testing.

## Where Findings Disagree

The graph contains no formally contested cluster, but architecture claims remain scope-dependent.
Cascades and end-to-end systems optimize different strengths. Low-rate tokens help efficiency until
they remove needed cues. Unified models provide breadth without guaranteeing specialization.

The safety cluster is emerging because passing one watermark, bias, or privacy benchmark does not
cover alternate attacks or populations. Fluent interaction should not be interpreted as deployment
readiness.

## How the Field Is Changing

The field progressed from offline codec language modeling in 2022–2023 to streaming, duplex, and
omnimodal interaction in 2024–2025. Token research targets lower or variable frame rates and clearer
semantic–acoustic separation.

Speech–text interleaving and connectors preserve pretrained text intelligence while adding speech.
Evaluation shifted toward multi-turn, live, role-playing, prosody, and reasoning benchmarks.
Preference alignment is increasingly common, alongside new safety trade-offs.

## Implications

- Report whole-pipeline latency, including endpointing and encoding.
- Test semantic and paralinguistic competence separately.
- Compare text, speech-input, and spoken-output controls for reasoning tasks.
- Evaluate duplex behavior in live overlap and interruption scenarios.
- Treat memory as retrieval, privacy, and forgetting—not only context length.
- Retain cascaded baselines when claiming end-to-end superiority.
- Monitor semantic, acoustic, and interaction dimensions during alignment.
- Test bias, impersonation, privacy, and provenance across languages and populations.

## Representative Reading Path

1. **Begin with speech tokens.** Read [[2209.03143|AudioLM]] and [[2301.02111|VALL-E]].
2. **Connect speech and text.** Read [[2305.11000|SpeechGPT]], [[2306.12925|AudioPaLM]], and
   [[2402.05755|Spirit LM]].
3. **Study streaming.** Pair [[2408.16725|Mini-Omni]] with [[2409.06666|LLaMA-Omni]].
4. **Examine full duplex.** Read [[2408.02622|Language Model Can Listen While Speaking]] and
   [[2410.00037|Moshi]].
5. **Add memory and alignment.** Read [[2025.coling-industry.29|CarMem]] and
   [[2502.11946|Step-Audio]].
6. **Audit deployment risks.** Use [[interspeech-2025-2328|Speech Watermarking]] and
   [[2510.02352|Bias in Spoken Dialogue LLMs]].

## Structured Source

Complete structured claims and provenance are in
[`wiki/_claims/spoken-language-model.yaml`](../_claims/spoken-language-model.yaml).

---

_This page is generated from `wiki/_claims/spoken-language-model.yaml` (digest date: 2026-07-27)._
