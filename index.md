---
title: "Speech Generation Wiki"
---

![Speech Generation Wiki](assets/speech-generation-wiki.png)

> [!abstract]
> A living systematic review of the state of the art in synthetic speech, covering **text-to-speech (TTS)**, **voice conversion (VC)**, and **spoken conversational agents (SCA)**. 363 papers ingested · 24 concepts · Last updated 2026-06-30.

This wiki ingests papers from Interspeech, ICASSP, ACL, EMNLP, arXiv, and industry technical reports on a rolling basis. Each paper contributes structured **evidence** and **claims** (generalised propositions the paper supports, refutes, or complicates). Claims are aggregated into **concept pages** that synthesise what the field collectively knows. Concept pages and evidence digests feed periodic **field reports** that track how the frontier is moving.

## Where the Field Stands

Two paradigms dominate open-weight zero-shot TTS as of mid-2025: **pure flow matching** (F5-TTS) and **LLM + flow-matching hybrids** (CosyVoice 2). Preference alignment via DPO is becoming standard post-training across architectures, no longer limited to autoregressive models. Neural codecs are increasingly recognised as a first-class quality bottleneck, not just audio compression infrastructure. Meanwhile, spoken conversational agents are converging on aligned multimodal LLMs over pure end-to-end speech language models.

→ [[overview|Full field overview: paradigms, trends, tensions, and open questions]]

## Concepts

Each concept page synthesises what the field knows, with claims, representative papers, and open questions.

**Core methods:** the architectural families that build speech synthesis systems

[[flow-matching|Flow Matching]] · [[diffusion-tts|Diffusion TTS]] · [[autoregressive-codec-tts|Autoregressive Codec TTS]] · [[transformer-enc-dec-tts|Transformer Enc-Dec TTS]] · [[gan-vocoder|GAN Vocoder]]

**Capabilities:** what these systems can do

[[zero-shot-tts|Zero-Shot TTS]] · [[voice-conversion|Voice Conversion]] · [[multilingual-tts|Multilingual TTS]] · [[emotion-synthesis|Emotion Synthesis]] · [[prosody-control|Prosody Control]] · [[streaming-tts|Streaming TTS]] · [[spoken-language-model|Spoken Language Model]] · [[speech-to-speech|Speech-to-Speech]] · [[instruction-conditioned-tts|Instruction-Conditioned TTS]] · [[singing|Singing Voice Synthesis]]

**Foundations:** representations, training paradigms, and cross-cutting techniques

[[neural-codec|Neural Audio Codec]] · [[self-supervised-speech|Self-Supervised Speech]] · [[disentanglement|Disentanglement]] · [[speaker-adaptation|Speaker Adaptation]] · [[rlhf-speech|RLHF for Speech]] · [[fine-tuning|Fine-Tuning]]

**Evaluation:** how the field measures synthesis quality

[[evaluation-metrics|Evaluation Metrics]] · [[subjective-evaluation|Subjective Evaluation]]

→ [[concepts/index|All concepts with paper counts]]

## Papers and Venues

363 papers ingested from arXiv (2019–2026), Interspeech 2025, ACL 2025, EMNLP 2025, NAACL 2025, ICLR 2026, NeurIPS 2025, and industry technical reports.

→ [[papers/index|Browse all 363 papers]]

→ [[venues/index|Browse by venue]]

## Reports

Field reports synthesise changes across multiple ingestion passes, tracking shifts in dominant approaches, emerging trends, and contested claims. The first report is due — 363 papers ingested.

→ [[reports/index|All reports]]
