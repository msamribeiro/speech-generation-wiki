---
id: "interspeech-2025-bokkahallisatish25_interspeech"
title: "Hear Me Out: Interactive evaluation and bias discovery platform for speech-to-speech conversational AI"
authors: ["Shree Harsha Bokkahalli Satish", "Gustav Eje Henter", "Éva Székely"]
organization: "KTH Royal Institute of Technology"
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-07-05
task: ["SCA", "evaluation"]
architecture: []
conditioning: []
training: []
model_size: "not reported"
codec_used: "not reported"
datasets_train: ["not applicable"]
datasets_eval: ["not applicable"]
metrics: []
code_available: null
demo_available: true
url: "https://www.isca-archive.org/interspeech_2025/bokkahallisatish25_interspeech.html"
related_concepts: ["spoken-language-model", "voice-conversion", "evaluation-metrics", "subjective-evaluation", "speech-to-speech"]
related_papers: []
field_significance:
  level: "low"
  type: ["evaluation-contribution"]
generation:
  date: 2026-07-05
  agent: speech-generation-ingest-agent
  model: claude-sonnet-4-6
  commit: "2829243"
---

> [!abstract] Interspeech · 2025 · Conference
> **Bokkahalli Satish et al.** (KTH Royal Institute of Technology) · [→ Paper](https://www.isca-archive.org/interspeech_2025/bokkahallisatish25_interspeech.html) · Demo: ✓ · Code: ?
>
> An interactive evaluation platform that uses real-time voice conversion to expose how speaker characteristics influence speech foundation model responses, enabling side-by-side bias discovery through paired comparison of original versus transformed voice inputs.

## Problem

Speech foundation models promise to understand and utilize prosodic and paralinguistic information — including speaker characteristics such as gender, age, and accent — yet systematic tools for exploring how these characteristics shape model behavior are limited. Existing benchmarks such as VoxDialogue, Spoken StereoSet, and VoxEval each evaluate speaker attributes in isolation, treating them as independent dimensions rather than exploring their intersections. None offer an interactive, exploratory experience through which users can directly observe and reflect on speaker-dependent biases in real time.

## Method

'Hear Me Out' is a demonstration platform assembled from four components. A speech foundation model block allows users to select from multiple deployed speech LMs, which accept raw audio (and optionally text prompts), encode it via a speech tokenizer, process it through an autoregressive language model, and generate spoken output via a vocoder. A real-time voice conversion module lets users modify speaker characteristics (gender, age, accent) and re-ask a question with the transformed voice, creating a matched-pair design in which only the perceived speaker identity changes. An automated metrics panel accompanies each response with quantitative measures: speech rate and articulation rate (syllables per second), mean and standard deviation of fundamental frequency (F0), sentiment analysis via AudioFlamingo2, multi-dimensional audio quality assessment via AudioboxAesthetics (content usefulness, enjoyment, production quality, production complexity), and semantic textual similarity between paired responses. Finally, the interactive UI presents original and transformed-voice responses side by side, with the automated metrics visualised in a radar plot, enabling immediate perceptual and quantitative comparison.

## Key Results

'Hear Me Out' is a system demonstration rather than an empirical study; no aggregate benchmark numbers are reported. The platform's contribution lies in its design: by pairing real-time voice conversion with side-by-side response display and automated metrics, it provides a methodology for observing speaker-dependent response variation in speech LMs that standard benchmarks do not support. The system is publicly accessible at https://shreeharsha-bs.github.io/Hear-Me-Out/.

## Novelty Assessment

The contribution is engineering integration: each component (voice conversion, speech LMs, sentiment analysis, audio quality assessment) is an existing tool assembled into an interactive evaluation interface. The novelty lies in the experimental methodology, specifically using voice conversion as a controlled perturbation to expose model sensitivity to speaker identity, rather than in any individual algorithmic advance. The paper is explicit about its scope: 'Hear Me Out' is positioned as a complementary tool to quantitative benchmarks, not a replacement, and no findings about bias prevalence or model ranking are reported.

## Field Significance

Low — 'Hear Me Out' introduces a useful interactive methodology for exploring speaker-dependent biases in speech LMs, but reports no empirical findings and deploys no novel technique. Its value is as a demonstration tool that reframes bias evaluation as an interactive, experiential activity rather than a purely quantitative exercise, which may encourage researchers to ask questions that structured benchmarks do not surface.

## Claims

- **supports:** Interactive evaluation tools that pair voice conversion with side-by-side response comparison can surface speaker-dependent biases in speech foundation models that automated benchmarks do not capture.
  > *Evidence:* The platform lets users re-ask questions with voice-converted inputs and directly observe changes in response tone, phrasing, and behavior, positioning human interactive comparison as the primary bias discovery mechanism rather than the automated metrics. *(§2.2, §2.4)*

- **complicates:** Automated speech evaluation metrics are insufficient on their own for assessing fairness and speaker-dependent behavior in speech foundation models.
  > *Evidence:* Even with speech rate, pitch statistics, sentiment analysis, and semantic similarity available, the authors present human interactive comparison as the primary mode of bias discovery, arguing that quantitative measures miss subtle differences in model behavior across speaker identities. *(§2.3, §2.4)*

- **supports:** Existing benchmarks for speech foundation models treat speaker attributes independently, limiting the ability to detect intersectional or compounded identity-based biases.
  > *Evidence:* Survey of VoxDialogue, Spoken StereoSet, and VoxEval reveals that each evaluates dimensions such as gender or age in isolation; VoxEval explicitly excludes bias and harmful content analysis as outside its scope. *(§1)*

## Limitations and Open Questions

The platform reports no empirical benchmark results, so claims about its effectiveness at bias discovery remain qualitative and user-dependent. The voice conversion module's perceptual fidelity (whether transformed voices are actually perceived as intended) is not systematically evaluated, which could confound interpretation of response differences. The selection of speech LMs available in the platform is not enumerated. Large-scale studies to obtain statistically reliable metrics are identified as future work.

## Wiki Connections

- [[spoken-language-model|Spoken Language Model]] — the platform targets speech foundation models (speech LMs) as the systems under evaluation, probing how they respond to varied speaker characteristics at the paralinguistic and extralinguistic level.
- [[voice-conversion|Voice Conversion]] — real-time voice conversion is the core experimental mechanism, enabling controlled manipulation of speaker characteristics to produce matched-pair inputs for comparison.
- [[evaluation-metrics|Evaluation Metrics]] — the automated metrics panel (speech rate, pitch, sentiment, audio quality, semantic similarity) is introduced as a complement to human interactive evaluation for quantifying response differences.
- [[subjective-evaluation|Subjective Evaluation]] — the interactive UI is explicitly designed for human perceptual comparison, positioning user experience as the primary mode of bias discovery rather than automated scoring alone.
- [[speech-to-speech|Speech-to-Speech]] — the platform evaluates end-to-end speech-to-speech conversational AI systems that process audio input and generate spoken output, contrasting with the conventional ASR+LLM+TTS pipeline.
