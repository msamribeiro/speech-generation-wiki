---
id: "interspeech-2025-2815"
title: "From Static to Dynamic: Enhancing AAC with Generative Imagery and Zero-Shot TTS"
authors: ["Juliana Francis", "Joakim Gustafsson", "Éva Székely"]
organization: "KTH Royal Institute of Technology"
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-07-05
task: [TTS]
architecture: []
conditioning: [zero-shot]
training: []
model_size: "not reported"
codec_used: "not reported"
datasets_train: ["not applicable"]
datasets_eval: ["not applicable"]
metrics: []
code_available: true
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/francis25_interspeech.html"
related_concepts: [zero-shot-tts, speaker-adaptation]
related_papers: []
field_significance:
  level: "low"
  type: ["engineering-integration"]
generation:
  date: 2026-07-05
  agent: speech-generation-ingest-agent
  model: claude-sonnet-4-6
  commit: "2829243"
---

> [!abstract] Interspeech · 2025 · Conference
> **Francis et al.** (KTH Royal Institute of Technology) · [→ Paper](https://www.isca-archive.org/interspeech_2025/francis25_interspeech.html) · Demo: ? · Code: ✓
>
> An AAC application for minimally verbal children with ASD that integrates DALL-E 3 for on-demand visual symbol generation and XTTS for personalized zero-shot voice synthesis, decoupling communicative capability from static symbol sets and predefined TTS voices.

## Problem

Traditional Speech-Generating Devices (SGDs) and picture-based AAC systems confine users to a predefined inventory of visual symbols and pre-assigned TTS voices. For minimally verbal children with ASD, this limits communicative scope and personal relevance: vocabulary reflecting individual interests, cultural contexts, or regional linguistic diversity is difficult or impossible to add. High device costs and requirements for clinician-mediated configuration further restrict access, while existing interventions tend to focus narrowly on requesting rather than broader communicative functions.

## Method

The system is a cross-platform AAC application designed to mirror the layout of the established SGD Proloquo2Go, reducing the learning curve for existing users. The core display presents configurable button grids where each button stores a word or phrase. Pressing a button appends its text to the pending speech output; long-pressing enters an editing mode for repositioning, renaming, or removal. Buttons can also function as folders, grouping context-related vocabulary across multiple pages.

For visual symbols, users either upload images from local storage or generate new ones on demand via DALL-E 3. The button's text label is inserted into a fixed prompt template sent to DALL-E 3, which returns four candidate images for selection. This eliminates dependence on a curated symbol library.

For voice output, XTTS is integrated as the zero-shot TTS backend. Users record a short sentence or upload an existing audio clip; XTTS conditions synthesis on this reference alongside the target text to produce speech in the selected voice. XTTS is chosen explicitly for its multilingual capabilities, enabling the system to serve users across diverse linguistic backgrounds without language-specific models.

An optional text expansion module (currently using GPT-4o) rewrites terse AAC inputs into more conversationally natural utterances. The authors flag that production deployment should substitute a locally run, fine-tuned model to prevent conversational data from being shared with third-party servers. The full codebase is publicly available on GitHub.

## Key Results

No formal evaluation is reported. The paper is a system demonstration rather than an experimental study. No MOS, speaker similarity, WER, or usability metrics are provided. Future work is identified as the appropriate venue for controlled comparisons against existing AAC solutions, longitudinal assessments of communicative skill development, and evaluation across cultural and linguistic settings.

## Novelty Assessment

The contribution is engineering integration. DALL-E 3, XTTS, and GPT-4o are off-the-shelf generative AI systems applied here to the AAC domain for minimally verbal users with ASD. No new architecture, training procedure, or synthesis technique is introduced. The design insight, that a modular architecture decouples AAC capability from fixed symbol and voice inventories, is user-centered and well-motivated, but the technical novelty relative to the speech synthesis literature is minimal. The paper's value is in demonstrating feasibility for an underserved application domain rather than advancing core TTS methodology.

## Field Significance

Low. This paper illustrates how off-the-shelf zero-shot TTS and generative image synthesis can extend communicative capability in assistive technology, but reports no empirical evaluation and proposes no new synthesis technique. Its contribution to the speech generation field is narrow, primarily signalling that zero-shot TTS has reached a maturity level where assistive communication researchers can integrate it without deep TTS expertise.

## Claims

- **supports:** Zero-shot TTS enables personalized voice output in assistive speech applications by conditioning synthesis on short user-supplied reference audio, without requiring model retraining or large speaker-specific corpora.
  > *Evidence:* XTTS is integrated into the AAC platform so that users record or upload a short voice clip; synthesis is then conditioned on this reference to produce speech in the target voice for any input text, with no fine-tuning step. *(§2.1)*

- **complicates:** Integrating cloud-based generative AI into assistive applications with privacy-sensitive users creates tension between expressive capability and data security when user speech or conversational content must be transmitted to third-party services.
  > *Evidence:* The paper flags that GPT-4o-based text expansion sends conversational content to third-party servers, and recommends replacing it with a locally run LLM before broad deployment; the same privacy concern applies to cloud-based image generation components. *(§2.3, §2.4)*

## Limitations and Open Questions

No user study or controlled evaluation is included, leaving the system's effectiveness relative to current AAC solutions undemonstrated. Long-term effects on communicative skill development, social engagement, and generalization across linguistic and cultural settings are identified as open questions for future longitudinal research. The text expansion feature relies on GPT-4o, creating a privacy exposure that the authors explicitly defer to future work for remediation. Use of DALL-E 3, which may be trained on data sourced without permission, raises ethical concerns that the modular design aims to accommodate but does not resolve.

## Wiki Connections

- [[zero-shot-tts|Zero-Shot TTS]] — XTTS serves as the zero-shot TTS backend, conditioning voice synthesis on short reference recordings provided by the user to produce a personalized voice without retraining.
- [[speaker-adaptation|Speaker Adaptation]] — voice personalization is a central design goal; users adapt the synthesized voice by providing a reference audio clip at inference time, with no model fine-tuning required.
