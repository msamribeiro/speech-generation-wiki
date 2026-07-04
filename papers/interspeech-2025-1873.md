---
id: "interspeech-2025-1873"
title: "Can AI Understand Mandarin Speech Prosody? A Framework and Benchmark Showcase"
authors: ["Zilong Wang", "Xiaoxue Zhang", "Xinyang Jiang", "Kaitao Song", "Jue Yu"]
organization: "Microsoft Research Asia / Tongji University"
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-07-04
task: ["SCA", "evaluation"]
architecture: []
conditioning: []
training: []
model_size: "not reported"
codec_used: "none"
datasets_train: ["not applicable (evaluation only)"]
datasets_eval: ["MSPB (Mandarin Speech Prosody Benchmark, 178 test items)"]
metrics:
  - name: accuracy
    value: 59.70
    system: GPT-4o (best model)
    testset: MSPB
code_available: null
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/wang25v_interspeech.html"
related_concepts: ["evaluation-metrics", "spoken-language-model"]
related_papers: []
field_significance:
  level: "moderate"
  type: ["evaluation-contribution", "dataset-contribution"]
generation:
  date: 2026-07-04
  agent: speech-generation-ingest-agent
  model: claude-sonnet-4-6
  commit: "318f1b4"
---

> [!abstract] Interspeech · 2025 · Conference
> **Wang et al.** (Microsoft Research Asia / Tongji University) · [→ Paper](https://www.isca-archive.org/interspeech_2025/wang25v_interspeech.html) · Demo: ? · Code: ?
>
> Introduces MSPB, a linguistically grounded Mandarin benchmark of 178 prosodic comprehension items spanning eight tasks, and demonstrates that current Speech LLMs substantially underperform human participants on prosody-dependent tasks while approaching human-level accuracy on context-rich tasks.

## Problem

Existing audio benchmarks (AIR-Bench, Audiobench, SD-Eval, StyleTalk, SALMON) cover speech recognition, emotion identification, speaker characteristics, and environmental sounds, but do not systematically evaluate prosodic comprehension across the syntax-semantics-pragmatics interface. Speech LLMs have moved toward end-to-end audio processing precisely to preserve prosodic information that ASR-to-text pipelines discard, yet no benchmark tests whether this capability is actually realized. Mandarin Chinese requires particular attention because lexical tone and sentential intonation interact in ways absent from Western-centric benchmark designs.

## Method

MSPB contains 178 phonetically recorded test items produced by a female Mandarin phonetician in a sound-treated environment (44.1 kHz, 16-bit, Sennheiser head-worn microphone). Items span eight tasks: intonation identification, prosodic disambiguation, prosodic focus marking, focus operator mapping, scalar implicature, irony detection, and emotional prosody identification with and without context. Each item was produced three times and selected by phonetic verification in Praat; all items were reviewed by linguistic experts and piloted with 20 native Mandarin speakers. Intonation items use a 2-alternative forced-choice design; the remaining seven tasks use 4-alternative forced choice.

Six Speech LLMs were evaluated under zero-shot MCQA prompting with text instructions and audio stimuli: GPT-4o-Audio (OpenAI), Gemini-1.5-Pro and Gemini-2-Flash (Google), Qwen2-Audio-7B-Instruct (Alibaba Cloud), GLM-4-Voice (Zhipu AI), and MiniCPM-o 2.6 (OpenBMB). Response text was mapped to choice labels using GPT-4o-based semantic mapping to neutralize format variation. Accuracy was computed across five independent runs per condition.

A parallel human baseline enrolled 27 native Mandarin participants (26F/1M) performing the same tasks with button responses, no time limit, and randomized stimulus order.

## Key Results

All six Speech LLMs underperformed human participants across most tasks. GPT-4o achieved the highest average model accuracy at 59.70%, followed by Gemini-2-Flash (59.28%), Gemini-1.5-Pro (56.05%), MiniCPM-o 2.6 (50.44%), GLM-4-Voice (47.66%), and Qwen2-Audio-7B-Instruct (46.41%). Human participants achieved 81.01%-95.67% accuracy across tasks, with near-ceiling performance on emotional prosody with context (95.67%) and intonation (95.29%).

Models performed best on context-rich tasks: on irony and focus operator tasks, Gemini-1.5-Pro, Gemini-2-Flash, GPT-4o, and MiniCPM-o reached statistically human-comparable accuracy via pairwise tests with Bonferroni correction. All models failed substantially on prosodic focus marking and scalar implicature, which require interpreting subtle acoustic prosodic cues rather than leveraging contextual reasoning. A mixed-design ANOVA confirmed significant main effects of testee type (F(6,49) = 189.77, p < 0.001), task category (F(7,343) = 116.86, p < 0.001), and their interaction (F(42,343) = 23.80, p < 0.001). Notably, Gemini-1.5-Pro declined to respond to emotional prosody without context items, introducing an evaluation gap attributable to safety restrictions.

## Novelty Assessment

MSPB is the first benchmark designed specifically for Mandarin prosodic comprehension across the syntax-semantics-pragmatics interface, filling a gap left by all existing audio benchmarks. The benchmark design is linguistically principled, the audio is professionally recorded, and the expert validation process is credible. The evaluation methodology is standard MCQA prompting with LLM-based response normalization; there is no novel technical contribution in model architecture or training. The paper's value is the benchmark itself and the diagnostic finding that current Speech LLMs exploit contextual cues more effectively than prosodic cues, a result that is actionable for future model development.

## Field Significance

Moderate — MSPB addresses a genuine gap in evaluation infrastructure for Mandarin prosody in speech LLM research. The eight-task battery reveals a specific failure mode (contextual cue reliance over acoustic prosody) that aggregate audio benchmarks cannot expose. The benchmark provides a reusable diagnostic tool for future prosodic integration work in spoken conversational systems.

## Claims

- **supports:** End-to-end Speech LLMs have not yet achieved robust prosodic comprehension, particularly for tasks that require interpretation of subtle acoustic prosodic cues without supporting contextual information.
  > *Evidence:* Across eight Mandarin prosodic tasks, all six evaluated Speech LLMs fell substantially below human-participant accuracy (human range 81.01%-95.67%; best model GPT-4o at 59.70%), with the gap widest on prosodic focus marking and scalar implicature tasks where acoustic prosody is the primary cue. *(§3.2.2, §3.3)*

- **complicates:** Aggregate benchmark accuracy scores obscure systematic task-dependent performance disparities in Speech LLMs.
  > *Evidence:* Several models achieved statistically human-comparable accuracy on irony and focus operator tasks (context-rich) while failing significantly on prosodic focus marking and emotional prosody without context (prosody-dependent), showing that average performance masks strong variation in which prosodic capabilities are and are not acquired. *(§3.3)*

- **supports:** Existing general-purpose audio benchmarks are insufficient for diagnosing prosodic comprehension weaknesses in Speech LLMs, especially for tonal languages.
  > *Evidence:* AIR-Bench, Audiobench, SD-Eval, StyleTalk, and SALMON collectively lack tasks for prosodic disambiguation, focus marking, scalar implicature, and the tone-intonation interaction characteristic of Mandarin, motivating MSPB's eight-task battery of 178 linguistically-validated items. *(§1, §2.1)*

- **complicates:** Safety and compliance restrictions in proprietary Speech LLMs can invalidate prosodic benchmark evaluations for specific task categories.
  > *Evidence:* Gemini-1.5-Pro declined to respond to emotional prosody without context items, producing no evaluable output for that task category and leaving a gap in the cross-model comparison. *(§3.2.2)*

## Limitations and Open Questions

The benchmark is limited to single-round, zero-shot evaluation in Mandarin with a single speaker's voice, restricting generalization to multi-speaker and multi-accent settings. Multi-turn conversational evaluation and multilingual extension are identified as future directions. The 178-item dataset is relatively small, and the ANOVA reports performance at the group level without per-item difficulty analysis that could pinpoint specific prosodic cue confounders.

## Wiki Connections

- [[evaluation-metrics|Evaluation Metrics]] — MSPB contributes a new task battery for prosodic comprehension absent from all existing audio benchmarks, expanding evaluation coverage for speech LLMs.
- [[spoken-language-model|Spoken Language Model]] — the paper evaluates six leading Speech LLMs (GPT-4o, Gemini series, Qwen2-Audio, GLM-4-Voice, MiniCPM-o) on their ability to interpret prosodic cues in Mandarin speech.
