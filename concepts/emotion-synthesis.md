---
slug: emotion-synthesis
title: Emotion Synthesis
aliases: [expressive TTS, affective speech synthesis, emotional TTS, style transfer]
related_concepts: [prosody-control, instruction-conditioned-tts, disentanglement, subjective-evaluation, spoken-language-model]
last_updated: 2026-05-27
---

# Emotion Synthesis

## What it is

Emotion synthesis in speech refers to the generation of speech that conveys a target emotional state — such as happiness, sadness, anger, surprise, or empathy — through appropriate paralinguistic cues: pitch contour, speaking rate, energy, voice quality, and temporal patterning. It is a sub-problem of expressive TTS and a core capability for empathetic spoken dialogue systems.

Emotion synthesis can be conditioned on discrete emotion labels (a fixed taxonomy), continuous valence-arousal dimensions, natural language style descriptions (see [[instruction-conditioned-tts]]), or reference audio clips. In speech-to-speech (SCA) systems, emotion synthesis additionally requires detecting and responding to the user's emotional state — a joint perception and generation task.

## Why it matters

Natural human speech is inherently emotional. Flat, neutral TTS is appropriate for information delivery but fails in conversational, customer service, entertainment, and accessibility contexts where emotional resonance matters. For spoken dialogue agents, empathetic responses require the system to detect user emotion from audio and generate a contextually appropriate emotional response — without losing the emotional signal that an ASR/TTS cascade would discard.

## Current state of the art

As of 2025, the primary corpus contribution to emotion synthesis in the SCA context is [[2025.emnlp-demos.70]] (OpenS2S), which demonstrates an empathetic end-to-end speech language model. Key aspects:
- Emotion detection: trained on SER datasets (IEMOCAP, MELD, CMU-MOSEI, MEAD, ESD) via a two-stage pipeline (semantic alignment → emotional alignment).
- Empathetic response generation: automated data construction using LLM-generated emotion labels and CosyVoice2 for speech rendering. The pipeline produces 50K EN + 50K ZH empathetic dialogue pairs.
- Evaluation: URO-Bench UnderEmotion (46.9 EN, 67.68 ZH score), competitive with Kimi-Audio while using significantly less training data.

A limitation is that all empathetic responses in the SFT dataset use a fixed young-female voice, constraining response speaker diversity. The "empathy" is learned through associations between input emotion and response style rather than cognitive understanding.

# TODO: expand — more corpus papers on discrete emotion label TTS, reference audio style transfer, and continuous valence-arousal control are needed before this section can be fully developed.

## Key variants and sub-approaches

**Emotion label conditioning.** A fixed discrete taxonomy (happy, sad, angry, neutral, etc.) is used as a one-hot or learned embedding conditioning signal. Widely used but limited by taxonomy size and cross-domain generalization.

**Reference audio style transfer.** The speaking style (including emotion) of a reference clip is transferred to synthesized speech via a style encoder. Conan [[2507.14534]] captures local speaking style at chunk level using clustering VQ.

**Natural language instruction conditioning.** Free-form text descriptions of emotional style are used as conditioning (see [[instruction-conditioned-tts]]). ControlSpeech [[2025.acl-long.346]] demonstrates this for open-vocabulary style including emotion.

**End-to-end empathetic SLM.** [[2025.emnlp-demos.70]] (OpenS2S) trains a full speech-in speech-out model to detect and respond to user emotion through multi-stage training and automated empathetic data synthesis.

## Comparison to alternatives

# TODO: expand

## Year-on-year trajectory

2024: FillerSpeech [[2025.emnlp-main.1730]] (in corpus) addresses filler word insertion as a paralinguistic naturalness feature adjacent to emotion synthesis. 2025: OpenS2S [[2025.emnlp-demos.70]] is the first fully open-source empathetic SLM in corpus; its automated data construction pipeline represents a new approach to scaling emotional training data.

## Open questions

- How well does automated LLM+TTS empathetic data (as in OpenS2S [[2025.emnlp-demos.70]]) substitute for human-annotated empathetic dialogues?
- Can streaming voice conversion systems (Conan [[2507.14534]]) reliably capture and transfer emotion at chunk level, or does emotion modeling require full-utterance context?
- What is the right evaluation protocol for empathetic speech? URO-Bench is a starting point but measures comprehension of emotion rather than naturalness of emotional delivery.

## Papers

| ID | Title | Venue | Year | Key use of this concept |
|----|-------|-------|------|------------------------|
| [[2025.emnlp-demos.70]] | OpenS2S: Advancing Fully Open-Source End-to-End Empathetic Large Speech Language Model | EMNLP | 2025 | First fully open-source empathetic speech-to-speech LLM; automated pipeline synthesizes 100K empathetic bilingual dialogues; multi-stage training for emotion detection and empathetic response generation |
