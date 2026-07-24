---
concept: speech-to-speech
render_type: in-depth
title: "Speech-to-Speech Systems: In Depth"
source_digest_date: 2026-07-19
paper_count: 60
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

This review asks what the 60 papers currently encoded for speech-to-speech systems collectively
show, where independent studies converge, and where comparisons remain too confounded to support a
firm conclusion. For a concise introduction and practical recommendations, see
[[concepts/speech-to-speech|Speech-to-Speech Systems]]. For record-level provenance, use
`wiki/_claims/speech-to-speech.yaml`.

## Findings at a Glance

- **Cascaded ASR–language-model–TTS systems remain the strongest general-purpose baseline.** They
  lead many current spoken-instruction and reasoning comparisons, although architecture, training
  scale, and component maturity are rarely controlled at the same time.
- **Modularity is also the most reliable way to preserve language ability.** Systems that freeze or
  lightly adapt a text-language-model backbone and attach a speech decoder generally retain more of
  the backbone's reasoning and instruction-following capability than systems trained jointly on
  speech throughout.
- **Low latency is not the same as good interaction.** Full-duplex systems can listen and speak at
  once, but fast interruption response often comes with false yielding, premature takeovers, weak
  backchannels, or reduced semantic coherence.
- **Semantic performance is ahead of acoustic and paralinguistic understanding.** Emotion, ambient
  sound, speaker traits, and other non-verbal signals remain weaker than content understanding across
  several recent benchmarks.
- **Single-turn evaluation is too forgiving.** Context recall, emotional coherence, informational
  density, and bias can worsen as conversations continue.
- **Automatic judges are useful for broad separations, not subtle speech quality.** They are least
  trustworthy for paralinguistic behavior, non-verbal output, and near-tied systems.
- **Two apparent contradictions remain genuinely unsettled.** The evidence disagrees about whether
  staged ASR/TTS pretraining helps instruction-following, and retrieval helps factual knowledge more
  clearly than it helps conversational memory.

## Scope

The reviewed corpus covers three related but non-identical tasks: open-domain spoken dialogue,
full-duplex interaction, and speech-to-speech translation. They share a speech-in/speech-out
interface, but optimize different outcomes. Dialogue systems emphasize reasoning and response
quality; full-duplex systems emphasize overlap, interruption, and latency; translation systems
emphasize cross-lingual content, voice preservation, and simultaneous decoding. Evidence from one
task is therefore not automatically evidence for another.

The corpus also mixes system papers, diagnostic benchmarks, surveys, deployment studies, and a
small number of historical systems. Primary experiments carry the most weight for performance
claims. Surveys help establish taxonomy but are not independent replications. Known arXiv and
conference versions of the same work are treated as one underlying evidence point even when both
records remain in the graph.

## Research Landscape

| Family | Practical idea | Best-supported advantage | Main unresolved cost |
|--------|----------------|--------------------------|----------------------|
| Cascaded ASR + LM + TTS | Optimize recognition, reasoning, and synthesis separately | Strong task accuracy, debuggability, replaceable components | Added orchestration and possible loss of non-verbal information at transcription |
| Modular LM + speech decoder | Preserve a text LM and attach a streaming speech generator | Retains more text reasoning while enabling direct speech output | Often limited to one voice, language, or output style |
| Native joint speech-text LM | Generate interleaved or parallel speech and text tokens in one model | Direct generation and a natural path to full duplex | Knowledge forgetting and weaker reasoning or paralinguistic behavior |
| Dual-stream full duplex | Give user and agent audio separate streaming paths | Interruption handling without flattening both speakers into one stream | Evidence is concentrated in small models and restricted test settings |
| Simultaneous speech translation | Couple translation with voice-preserving speech generation | Cross-lingual voice transfer and streaming policies | Language-pair dependence and limited reproducibility of frontier systems |

This table is a navigation aid, not a ranking. Several systems combine families, and differences in
data, backbone scale, output voice, and evaluation design prevent a clean architecture-only league
table.

## What the Research Shows

### Cascade baseline

**Assessment: strongly supported within the reviewed corpus.**

Across general spoken question answering, instruction following, role playing, and production
deployment studies, systems built from mature ASR, text-language-model, and TTS components remain
difficult for end-to-end speech models to beat. [[2410.17196|VoiceBench]] reports a large composite
advantage for a Whisper-plus-text-LM pipeline over the strongest open end-to-end system it tests.
[[2506.23049|AURA]] likewise reports a wide gap on knowledge-intensive VoiceBench tasks, and
[[2508.02013|SpeechRole]] finds that a cascade using a strong text model surpasses the evaluated
end-to-end alternatives. In a production interview setting, [[2507.16835]] finds that changing the
speech recognizer affects measured quality more than swapping the language model or TTS provider.

The conclusion should not be simplified to “cascades are intrinsically better.” Most comparisons
change several variables at once: end-to-end systems may have smaller backbones, less training data,
fixed voices, immature decoding, or different inputs. The comparative study in
[[2025.naacl-demo.21|ESPnet-SDS]] is explicitly not strictly matched, and the production
configurations in [[2507.16835]] ran in different time windows. Surveys in [[2411.13577|WavChat]]
and [[2504.08528]] reinforce the broad pattern but summarize overlapping literature rather than add
new experimental trials.

The most defensible reading is therefore narrower: **with the systems and evaluations currently
represented here, mature components provide a stronger general semantic and task-performance
baseline than joint speech models.** The evidence does not establish that the advantage will survive
matched backbones, matched data, and equally mature speech generation.

**Most informative evidence:** [[2410.17196|VoiceBench]], [[2506.23049|AURA]],
[[2508.02013|SpeechRole]], [[2025.naacl-demo.21|ESPnet-SDS]].

**What would change the assessment:** a matched comparison holding the backbone, data, evaluation
inputs, and speech-output quality constant while changing only cascade versus end-to-end coupling.

### Preserving language reasoning

**Assessment: strongly supported, with architectural scope limits.**

Several systems converge on the same engineering lesson: adapting the entire language model to
speech can erode capabilities learned during text pretraining. [[2411.00774|Freeze-Omni]] reports a
smaller gap from its text-only backbone than joint speech systems show from theirs.
[[2025.findings-acl.1051|LLMVoX]], [[2505.15670|SALM-Duplex]], and
[[interspeech-2025-0874|NVDuplex]] keep the backbone frozen or substantially decouple it from speech
generation, while still enabling streaming spoken responses. Together they support modularity as a
practical defense against knowledge and instruction-following degradation.

This evidence explains the recent prominence of systems that attach an autoregressive,
flow-matching, or aligned TTS decoder to an existing text model. It does not show that native joint
speech-text models are incapable of retaining reasoning at larger scale. [[2410.00037|Moshi]] and
related systems pursue a different objective—joint acoustic and semantic modeling for direct,
full-duplex interaction—and may accept a semantic benchmark cost in exchange for capabilities that
a transcript-centered design cannot express.

**Most informative evidence:** [[2411.00774|Freeze-Omni]],
[[2025.findings-acl.1051|LLMVoX]], [[2505.15670|SALM-Duplex]],
[[interspeech-2025-0874|NVDuplex]].

**Boundary:** current support comes largely from modular streaming systems. It should not be read as
a universal claim that freezing is always preferable to joint training.

### Full-duplex floor control

**Assessment: strongly supported across overlapping benchmark lineages.**

Full-duplex systems are designed to listen while speaking, stop when interrupted, and decide when
to retain or yield the conversational floor. The evidence shows that these behaviors cannot be
collapsed into one latency number. [[2507.23159|Full-Duplex-Bench v1.5]] finds that the fastest or
most interruption-responsive system can also yield incorrectly to background or non-addressed
speech. [[2509.22243|FLEXI]] similarly shows opposing failure profiles: one system jumps in too
readily while others almost never jump in but take over during pauses. [[interspeech-2025-0739|FD-Bench]]
finds that integrated duplex mechanisms handle interruptions more successfully than systems that
depend on external voice-activity detection alone.

Backchannels remain a particularly visible gap. A model may stop promptly when interrupted yet
still fail to produce well-timed acknowledgments or contextually appropriate continuation.
[[2503.04721|Full-Duplex-Bench]] and [[2509.22243|FLEXI]] show that pause handling, backchannels,
and interruption response produce different system rankings. This makes “full duplex” a capability
class, not evidence that human-like interaction has been achieved.

The benchmark evidence is not fully independent: the two Full-Duplex-Bench versions share authors
and overlapping systems. Even so, the pattern recurs in FD-Bench and FLEXI, which reduces the risk
that it is merely an artifact of one benchmark.

**Practical implication:** report at least interruption response, false yielding or jump-in,
pause takeover, backchannel timing, and semantic continuation. A single average latency hides the
central trade-off.

### Paralinguistic understanding

**Assessment: strongly supported across several diagnostic benchmarks.**

Recent evaluation work repeatedly finds that semantic response quality is ahead of paralinguistic
and non-verbal understanding. [[2508.18240|MTalk-Bench]] reports that evaluated systems perform
better on semantic rubrics than on paralinguistic and ambient-sound dimensions.
[[2509.03940|VoxRole]] finds acoustic appropriateness to be the weakest judged dimension across all
seven tested models. [[2508.17623|EMO-Reasoning]] shows a large gap from human performance on
emotional reasoning, while [[2506.21875|WildSpeech-Bench]] finds that conclusions drawn from
speech-to-text benchmarks do not reliably predict performance on more realistic audio interactions.

This gap matters because a speech-to-speech interface is valuable precisely where the transcript is
insufficient: emotion, identity, timing, emphasis, ambient sound, and non-verbal vocal behavior. A
system can therefore score well on spoken QA while still failing at the dimensions that motivate
direct audio modeling.

The studies do not isolate one cause. The bottleneck may lie in tokenization, training data,
backbone adaptation, generation, or the benchmarks themselves. The convergence is in the observed
failure pattern, not yet in its mechanism.

### Multi-turn evaluation

**Assessment: strongly supported, with heterogeneous outcome measures.**

[[2025.findings-acl.470]] finds that open voice-interaction models recall and use earlier dialogue
context substantially less effectively than their text backbones. [[2508.17623|EMO-Reasoning]]
reports weaker emotional reasoning over multiple turns, and [[2508.18240|MTalk-Bench]] finds that
systems struggle to maintain conversational quality across longer interactions.
[[2510.02352|FairDialogue]] adds a different but related warning: demographic effects can accumulate
or become clearer over multi-turn exchanges even when the text content is controlled.

These papers measure recall, emotional coherence, information density, and bias rather than one
common construct. They should not be pooled into a single numerical “multi-turn degradation” claim.
They do, however, agree on the practical conclusion that single-turn evaluation systematically
misses important failure modes.

### Automatic evaluation

**Assessment: strongly supported for the limitations; method choice remains task-dependent.**

Automatic judges can identify large semantic differences and make broad benchmark sweeps feasible.
Their reliability weakens when systems are close, when the important information is acoustic rather
than textual, or when outputs contain non-verbal behavior. Evidence from
[[2508.18240|MTalk-Bench]], [[2508.09600]], [[2508.17623|EMO-Reasoning]], and the production study
[[2507.16835]] points in the same direction despite different evaluation protocols.

The appropriate response is not to discard automatic evaluation. It is to use it as one layer:
automatic semantic and task metrics for scale, targeted acoustic measures for diagnosable
properties, and human listening or interaction tests for paralinguistic behavior and close
comparisons. The current evidence does not justify treating an LLM judge as a general substitute for
human evaluation of spoken interaction.

## Where Findings Disagree

### Staged pretraining

> [!warning]
> **Contested:** staged ASR or TTS pretraining improves instruction-following in some systems and
> degrades it in others; current comparisons do not isolate the cause.

[[2502.17239|Baichuan-Audio]] finds that staged pretraining reduces the gap between speech and text
performance. [[2412.15649|SLAM-Omni]] reports the opposite pattern: adding ASR or TTS pretraining
before dialogue fine-tuning lowers instruction-following scores with little recognition benefit.
The conference record [[2025.findings-acl.115]] represents the same underlying SLAM-Omni work and
must not be counted as an independent replication. [[2410.11190]] and
[[2411.00774|Freeze-Omni]] provide related evidence that staged or frozen-backbone training can
preserve general capability, but they do not test the same instruction-following question directly.

The disagreement is confounded by backbone scale, data volume, freezing policy, and the meaning of
“staged.” It is therefore premature to recommend staged pretraining in general. A controlled,
scale-matched ablation is required.

### Retrieval and memory

**Status: scope-dependent and currently contested in the graph.**

[[2506.23049|AURA]] and [[2507.16632|Step-Audio 2]] show benefits from retrieval or tool use on
knowledge-intensive questions and external actions. [[2025.findings-acl.470]] finds that retrieval
does not reliably repair recall of earlier conversational turns. These results may not be a true
mechanistic contradiction: factual retrieval and episodic dialogue memory require different
indexes, queries, and evidence selection.

For now, the evidence supports retrieval for external knowledge more clearly than for conversational
memory. Systems should evaluate those functions separately rather than report a single RAG result.

## How the Field Is Changing

The historical trajectory is not simply a shift from cascades to end-to-end models. Early systems
such as [[2305.11000|SpeechGPT]] and [[2305.15255|Spectron]] established that language-model-style
decoding could operate over speech representations, but offered limited evaluation or impractical
generation costs. [[2410.00037|Moshi]] made joint acoustic-semantic generation and full-duplex
interaction a central design target. More recent work increasingly preserves a pretrained text
backbone and moves speech generation into a modular streaming decoder, reflecting the evidence that
joint speech adaptation can damage existing language ability.

Evaluation has changed just as substantially. Broad spoken-QA scores are being supplemented by
benchmarks for interruption, backchannels, paralinguistic understanding, multi-turn coherence,
emotional reasoning, and bias. This shift matters because it changes what counts as progress: a
system can improve latency or semantic accuracy while becoming worse at floor control, acoustic
appropriateness, or long-context interaction.

The frontier is therefore moving along several axes at once rather than converging on one
architecture. Native joint models remain important for direct audio interaction; modular systems
offer a pragmatic route to stronger reasoning; and diagnostic benchmarks are exposing weaknesses
that neither family can currently claim to have solved.

## Implications

- **Treat architecture labels as incomplete explanations.** Compare backbone scale, training data,
  output representation, voice restrictions, and evaluation inputs before attributing a result to
  “cascade” or “end to end.”
- **Evaluate latency and interaction separately.** First-packet latency should be accompanied by
  interruption response, false yielding, pause takeover, backchannels, and semantic continuation.
- **Protect the capability being inherited from the text model.** If instruction-following and
  reasoning are deployment priorities, frozen or lightly adapted backbones currently have the
  clearest support.
- **Test what transcripts remove.** Emotion, timing, speaker characteristics, ambient sound, and
  non-verbal vocal behavior should be first-class evaluation dimensions for any system justified by
  direct audio modeling.
- **Use multi-turn and human evaluation for consequential comparisons.** Single-turn automatic
  scores are insufficient when differences are subtle or primarily paralinguistic.

## Representative Reading Path

This is a curated route through the evidence, not a strength ranking or full inventory.

1. **Begin with native speech-text language modeling.** Read [[2305.11000|SpeechGPT]] for an early
   unified next-token formulation and its historical importance despite limited quantitative
   evaluation.
2. **Then examine joint acoustic-semantic generation.** Read [[2410.00037|Moshi]] for a
   foundational full-duplex joint-token design and the semantic trade-offs that accompany it.
3. **Compare modular streaming alternatives.** Read [[2409.06666|LLaMA-Omni]] alongside
   [[2411.00774|Freeze-Omni]] to contrast decoder attachment with frozen-backbone strategies.
4. **Test the case for cascades.** Use [[2410.17196|VoiceBench]] and [[2506.23049|AURA]] for broad
   comparative and knowledge-intensive evidence on the current cascade advantage.
5. **Study full-duplex failure modes.** Read [[2503.04721|Full-Duplex-Bench]],
   [[2507.23159|Full-Duplex-Bench v1.5]], and [[2509.22243|FLEXI]] to separate interruption,
   false yielding, pause takeover, and backchannel behavior.
6. **Move from single-turn scores to conversational evaluation.** Read
   [[2508.18240|MTalk-Bench]], [[2508.17623|EMO-Reasoning]], and
   [[2025.findings-acl.470]] for evidence on what semantic and single-turn metrics miss.
7. **Finish with simultaneous speech translation.** Compare [[2303.03926|VALL-E X]],
   [[2025.acl-long.817]], and [[interspeech-2025-1595]] for voice prompting, boundary-aware
   decoding, and interleaved training.

## Limits of This Review

- The 60-paper graph is a reviewed corpus, not proof of complete field coverage.
- Spoken dialogue, full-duplex interaction, and speech translation use different evaluation regimes;
  results are not freely transferable between them.
- Several comparisons confound architecture with model scale, data, component maturity, or
  proprietary deployment tuning.
- Surveys and duplicate publication records inflate raw citation counts unless independence is
  checked explicitly.
- Some frontier results rely on proprietary training data, undisclosed commercial baselines, or
  author-created benchmarks without independent validation.
- Two claim clusters in the source graph have only one supporting paper and should remain emerging.

## Structured Source

This page selects and groups evidence for human readability. The complete claim graph, including
all paper roles, source sections, caveats, confidence and relevance fields, method-family
memberships, and reassessment state, is available in
`wiki/_claims/speech-to-speech.yaml`.

Readers can inspect that YAML directly or clone the repository and analyze it with their own tools
or agents. The Markdown page is intentionally comprehensive at the level of important conclusions,
not exhaustive at the level of records.

---

_This page is generated from `wiki/_claims/speech-to-speech.yaml` (digest date: 2026-07-19).
Citations are selected for human readability; complete provenance remains in the source YAML._
