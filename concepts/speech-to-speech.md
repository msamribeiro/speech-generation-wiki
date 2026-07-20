---
slug: speech-to-speech
title: Speech-to-Speech Systems
aliases: [speech-to-speech, S2S, spoken dialogue systems, speech language models, end-to-end spoken agents]
status: established
last_reviewed: 2026-07-20
source_digest_date: 2026-07-19
generation:
  schema_version: 2
  date: "2026-07-20"
  stage: render
  mode: full
  runtime: codex
  provider: openai
  agent: speech-generation-render-agent
  model: "gpt-5"
  commit: "0078612"
---

> [!abstract]
> Speech-to-speech systems accept speech and produce speech, spanning open-domain spoken dialogue, full-duplex interaction, and speech-to-speech translation. Across the 60 papers reviewed here, modular and end-to-end designs have made real-time interaction practical, but cascaded ASR–language-model–TTS systems still provide a strong quality baseline while current end-to-end systems struggle most with paralinguistic understanding, long conversations, and robust evaluation.

## Current Assessment

The reviewed evidence shows a field moving from demonstrations of direct speech-token generation toward modular streaming systems and diagnostic evaluation. Native joint speech-text language models remain important, especially for low-latency and full-duplex research, but many 2025 systems preserve a strong text-language-model backbone and attach a separately trained streaming speech decoder. That modular pattern improves latency and protects text reasoning or instruction-following, while avoiding the need to relearn all language capability in a joint speech-token stream.

Cascaded systems remain difficult to beat on broad benchmarks because their ASR, reasoning, and TTS components can each be optimized independently. End-to-end systems can reduce first-response latency and preserve acoustic context that a transcript discards, but the evidence does not show a general quality victory. Evaluation has become a central research contribution in its own right: multiple benchmarks agree that semantic task scores conceal weaker emotion, speaker, ambient-sound, interruption, and multi-turn behavior.

## Evidence Boundaries

This page covers 60 integrated papers through Q3 2025. The corpus mixes primary systems, benchmarks, surveys, translation systems, and deployment studies. Survey papers support taxonomy and synthesis but are not independent experimental replications; duplicate arXiv/conference pairs are treated as single evidence points where identified. Several systems use proprietary data or undisclosed commercial baselines, limiting controlled comparison.

“Speech-to-speech” is used here for systems with both speech input and speech output. Pure turn-taking detectors and speech-in/text-out systems were intentionally excluded during integration. The three included sub-paradigms—spoken dialogue, full-duplex interaction, and speech-to-speech translation—share interfaces but not always objectives or metrics, so claims are stated at the narrowest common scope.

## Major Claims

### Strongly Supported

- **Cascaded ASR–language-model–TTS systems remain stronger than current end-to-end systems on many broad task and quality benchmarks.**
  Evidence: [[2506.23049|AURA]], [[2025.naacl-demo.21|ESPnet-SDS]], [[2507.16835]], [[2508.18240]], [[2509.03940]].
  Caveat: Comparisons often confound architecture with training scale, component maturity, and proprietary deployment tuning.

- **Separating speech generation from a frozen or lightly adapted text-language-model backbone better preserves text reasoning and instruction-following than unrestricted speech fine-tuning.**
  Evidence: [[2411.00774]], [[2025.findings-acl.1051]], [[2505.15670]], [[interspeech-2025-0874|NVDuplex]].
  Caveat: The result supports modularity within the reviewed systems, not a proof that native joint models cannot close the gap at larger scale.

- **Semantic competence is ahead of paralinguistic and non-verbal audio competence.**
  Evidence: [[2504.08528]], [[2508.18240]], [[2509.03940]], [[2507.18119]].
  Caveat: Benchmarks differ in emotion, ambient sound, and speaker-trait coverage, but converge on the same broad gap.

- **Spoken dialogue quality degrades over longer multi-turn interactions.**
  Evidence: [[2508.17623]], [[2508.18240]], [[2025.findings-acl.470]].
  Caveat: Different studies measure coherence, information density, recall, or bias; they establish a shared degradation pattern rather than one failure mechanism.

- **Automatic judges are useful only within a limited operating range and are especially weak for paralinguistic or near-tied comparisons.**
  Evidence: [[2508.18240]], [[2508.09600]], [[2508.17623]], [[2507.16835]].
  Caveat: Large score gaps and text-dominated tasks can still yield useful automatic rankings.

- **Joint or interleaved speech-text training improves cross-modal alignment and transfers text-model capability into speech.**
  Evidence: [[2402.05755]], [[2410.03751]], [[2411.17607]].
  Caveat: One supporting entry is survey evidence, and improved alignment does not by itself close the end-to-end reasoning gap.

- **Integrated interruption detection is more robust than relying only on an external voice-activity detector for barge-in behavior.**
  Evidence: [[interspeech-2025-0739]], [[2410.11190]], [[2411.00774]].
  Caveat: Speaker-conditioned VAD remains a complementary improvement rather than direct evidence for removing VAD entirely.

### Emerging and Contested

- **In-context acoustic prompts may preserve cross-lingual voice better than speaker embeddings in speech-to-speech translation.**
  Evidence: [[2303.03926|VALL-E X]].
  Caveat: This direct comparison is backed by one paper.

> [!warning]
> **Retrieval and tool use help knowledge-intensive spoken QA, but retrieval does not reliably improve recall of earlier dialogue turns.** [[2506.23049|AURA]] and [[2507.16632|Step-Audio 2]] support knowledge and tool benefits; [[2025.findings-acl.470]] qualifies the claim for conversational-context recall. The conflict may reflect different retrieval targets rather than a single method failing or succeeding universally.

## Method Families

**Native joint speech-text token language models.** A single decoder produces interleaved or parallel speech and text streams. This family makes direct interaction and full-duplex generation possible without a downstream TTS module, but still faces semantic, reasoning, and paralinguistic gaps ([[2305.11000|SpeechGPT]], [[2410.00037|Moshi]], [[2410.11190]]).

**Dual-stream full-duplex systems.** Separate user and agent audio paths support listening while speaking and avoid flattening both sides into one stream. The design is promising for interruption and latency, but evidence is concentrated in smaller models and limited task settings ([[2408.02622]], [[2505.15670]], [[interspeech-2025-0874|NVDuplex]]).

**Modular language model plus streaming speech decoder.** A frozen or lightly adapted text backbone feeds a separately trained autoregressive or flow-based speech decoder through gating, CTC, or cross-attention. This is the clearest 2025 pattern in the reviewed graph because it preserves text capability while enabling speech output and streaming ([[2409.06666|LLaMA-Omni]], [[2411.00774]], [[2501.06282]]).

**Speech-to-speech and simultaneous translation.** These models optimize cross-lingual content, voice preservation, and latency policies rather than open-domain conversation. Targeted streaming training, gradual modality transition, and reinforcement learning can match or exceed fully streaming training, but proprietary evidence and language-pair dependence remain concerns ([[2303.03926|VALL-E X]], [[2025.acl-long.817]], [[interspeech-2025-1595]], [[2507.17527]]).

**Cascaded pipelines.** Separate ASR, text reasoning, and TTS components remain the comparison baseline and often the strongest production choice. They trade higher pipeline complexity and potential latency for mature components, inspectable intermediate text, and easier component replacement ([[2506.23049|AURA]], [[2025.naacl-demo.21|ESPnet-SDS]], [[2507.16835]]).

## How to Interpret Older and Newer Evidence

The 2023 systems establish native spoken-language-model and cross-lingual prompting lineages; they demonstrate feasibility but do not represent current benchmark maturity. Work from 2024 develops joint streams, duplex interaction, and modular decoders. The 2025 evidence shifts toward benchmarks, matched architectural comparisons, and production-oriented latency. Frontier claims from proprietary streaming translation or tool-using agents should be read as demonstrations pending replication, while survey entries support taxonomy rather than experimental independence.

## Current Tensions

- **Cascade quality versus end-to-end latency and acoustic context.** Cascades lead many general benchmarks, while end-to-end or modular speech systems can respond earlier and retain information discarded by transcription ([[2506.23049|AURA]], [[interspeech-2025-0874|NVDuplex]]).
- **Native joint modeling versus modular preservation of language ability.** Joint streams simplify direct generation; frozen-backbone systems more reliably protect text reasoning in current evidence ([[2410.00037|Moshi]], [[2025.findings-acl.1051]]).
- **Semantic tokens versus acoustic fidelity.** Semantic tokens align well with text and language models, while acoustic codecs preserve voice and paralinguistic detail; hybrid representations attempt to retain both ([[2412.15649|SLAM-Omni]], [[2410.00037|Moshi]]).
- **Staged pretraining versus direct dialogue fine-tuning.** [[2412.15649|SLAM-Omni]] and [[2502.17239|Baichuan-Audio]] reach opposing conclusions under different scales and training regimes, so staging itself cannot yet be isolated as the cause.
- **Knowledge retrieval versus conversational memory.** External retrieval helps factual or tool-oriented tasks but may not retrieve the right evidence for multi-turn context ([[2507.16632|Step-Audio 2]], [[2025.findings-acl.470]]).

## Decision Implications

- Use a cascade when broad task accuracy, debuggability, or component replacement is the priority; treat an end-to-end system’s latency advantage as a separate axis from overall quality ([[2506.23049|AURA]], [[2025.naacl-demo.21|ESPnet-SDS]]).
- Consider a modular frozen-backbone-plus-speech-decoder design when preserving text reasoning and instruction-following matters ([[2025.findings-acl.1051]], [[interspeech-2025-0874|NVDuplex]]).
- Test full-duplex systems with real overlap, interruption, and backchannel behavior rather than voice-activity detection alone ([[interspeech-2025-0739]], [[2508.07375]]).
- Include human evaluation for paralinguistic quality and close comparisons; automated judges are insufficient there ([[2508.18240]], [[2508.09600]]).
- Evaluate over multi-turn sessions because single-turn scores can hide coherence, recall, and bias degradation ([[2508.17623]], [[2025.findings-acl.470]]).

## Open Questions

- Will better language-model backbones close the cascade-versus-end-to-end gap under matched data and components ([[interspeech-2025-0874|NVDuplex]])?
- Does staged modality pretraining help or harm instruction-following when scale and data are controlled ([[2412.15649|SLAM-Omni]], [[2502.17239|Baichuan-Audio]])?
- Can training data and architecture solve backchannels and barge-in, or is a different duplex objective required ([[interspeech-2025-0739]], [[2410.11190]])?
- Can retrieval be designed for conversational episodes rather than only external factual knowledge ([[2507.16632|Step-Audio 2]], [[2025.findings-acl.470]])?
- Which representation best balances semantic alignment, voice fidelity, and paralinguistic information ([[2410.03751]], [[2410.00037|Moshi]])?

## Recommended Reader Path

1. [[2305.11000|SpeechGPT]] — early native speech-text language-model formulation.
2. [[2410.00037|Moshi]] — influential joint acoustic-semantic full-duplex design.
3. [[2409.06666|LLaMA-Omni]] — representative modular streaming speech decoder.
4. [[interspeech-2025-0874|NVDuplex]] — evidence for asymmetric duplex design and backbone-quality effects.
5. [[2506.23049|AURA]] — cascade comparison plus retrieval and tool-use evidence.
6. [[2508.18240]] — broad diagnostic evidence for multi-turn and paralinguistic gaps.
7. [[2507.16835]] — production-oriented comparison of end-to-end and cascaded systems.

## Related Concepts and Pages

- [[spoken-language-model|Spoken Language Model]] — the language-model foundation for many end-to-end systems.
- [[streaming-tts|Streaming TTS]] — speech-output latency and chunked decoding techniques.
- [[autoregressive-codec-tts|Autoregressive Codec TTS]] — token generation used by native and modular systems.
- [[neural-codec|Neural Audio Codec]] — acoustic and semantic token representations.
- [[evaluation-metrics|Evaluation Metrics]] — benchmark and judge reliability issues.
- [[evidence/speech-to-speech|Evidence Dossier]] — complete claim, family, and paper-level provenance.

---

_This page is generated from `wiki/_claims/speech-to-speech.yaml` (digest date: 2026-07-19). For the full paper inventory, claim support matrix, and reassessment queue, see [[evidence/speech-to-speech|Evidence Dossier]]._
