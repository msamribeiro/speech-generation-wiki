---
slug: emotion-synthesis
title: Emotion Synthesis
aliases: [expressive TTS, affective speech synthesis, emotional TTS, style transfer]
related_concepts: [prosody-control, instruction-conditioned-tts, disentanglement, subjective-evaluation, spoken-language-model]
last_updated: 2026-05-30
---
## What it is

Emotion synthesis in speech refers to the generation of speech that conveys a target emotional state — such as happiness, sadness, anger, surprise, or empathy — through appropriate paralinguistic cues: pitch contour, speaking rate, energy, voice quality, and temporal patterning. It is a sub-problem of expressive TTS and a core capability for empathetic spoken dialogue systems.

Emotion synthesis can be conditioned on discrete emotion labels (a fixed taxonomy), continuous valence-arousal dimensions, natural language style descriptions (see [[instruction-conditioned-tts]]), or reference audio clips. In speech-to-speech (SCA) systems, emotion synthesis additionally requires detecting and responding to the user's emotional state — a joint perception and generation task.

## Why it matters

Natural human speech is inherently emotional. Flat, neutral TTS is appropriate for information delivery but fails in conversational, customer service, entertainment, and accessibility contexts where emotional resonance matters. For spoken dialogue agents, empathetic responses require the system to detect user emotion from audio and generate a contextually appropriate emotional response — without losing the emotional signal that an ASR/TTS cascade would discard.

## Current state of the art

As of 2025, multiple approaches to emotion synthesis have been integrated into the corpus:

**Empathetic SCA (OpenS2S [[2025.emnlp-demos.70]])**: end-to-end empathetic speech LM with automated data construction (50K EN + 50K ZH pairs). URO-Bench UnderEmotion 46.9 EN / 67.68 ZH.

**AR duration-controlled emotion TTS (IndexTTS2 [[2506.21619]])**: GRL-based emotion–speaker disentanglement with a Conformer-based emotion perceiver, three-stage training curriculum, and T2E module (DeepSeek-R1 distilled to Qwen3 LoRA for natural language emotion routing). EMOS 4.22 vs. 3.37 for MaskGCT on a 12-speaker × 7-emotion evaluation. Strongest emotional expressiveness for a zero-shot AR system in the corpus.

**Training-free emotion steering (EmoSteer-TTS [[2508.03543]])**: activation steering in pre-trained DiT flow-matching models (F5-TTS, E2-TTS, CosyVoice2) at inference time without any retraining. Enables emotion conversion (E-SIM 0.280 F5+EmoSteer vs. 0.240 unsteered CosyVoice2), interpolation, erasure, and multi-emotion blending. The key finding is that pre-trained flow-matching models implicitly encode emotional information in their DiT activations, accessible without specialized training.

**Difference-in-means emotion embeddings with disentanglement (Marco-Voice [[2508.02038]])**: rotational emotion embeddings extracted as difference-in-means between emotional and neutral speech from same speakers, combined with cross-orthogonal speaker-emotion constraint and in-batch contrastive learning. Achieves emotional expression MOS 4.225 vs. CosyVoice2 3.240; SPK-SIM 0.8275 vs. 0.605 for CosyVoice2 in voice cloning. Both the REE technique and EmoSteer's difference-in-means approach demonstrate that direction arithmetic in representation space is an effective tool for emotion control.

**Multi-reward RL for emotion (GLM-TTS [[2512.14291]])**: GRPO RL with an emotion reward (and a separate laughter reward) alongside CER and SIM rewards. Demonstrates that including emotion in the RL reward trades off against pronunciation accuracy — revealing a fundamental tension in multi-attribute RL alignment for speech.

## Key variants and sub-approaches

**Emotion label conditioning.** A fixed discrete taxonomy (happy, sad, angry, neutral, etc.) is used as a one-hot or learned embedding conditioning signal. Widely used but limited by taxonomy size and cross-domain generalization.

**Reference audio style transfer.** The speaking style (including emotion) of a reference clip is transferred to synthesized speech via a style encoder. Conan [[2507.14534]] captures local speaking style at chunk level using clustering VQ. Marco-Voice [[2508.02038]] extracts a speaker-independent emotion direction from paired emotional/neutral reference clips of the same speaker.

**Training-free activation steering.** EmoSteer-TTS [[2508.03543]] steers emotion by adding difference-in-means vectors to intermediate DiT layer activations at inference time, requiring no labeled emotional training data for the target model.

**Natural language instruction conditioning.** Free-form text descriptions of emotional style are used as conditioning (see [[instruction-conditioned-tts]]). ControlSpeech [[2025.acl-long.346]] and IndexTTS2 [[2506.21619]] (T2E module) both support natural language emotion routing.

**End-to-end empathetic SLM.** [[2025.emnlp-demos.70]] (OpenS2S) trains a full speech-in speech-out model to detect and respond to user emotion through multi-stage training and automated empathetic data synthesis.

## Comparison to alternatives

# TODO: expand

## Year-on-year trajectory

2024: FillerSpeech [[2025.emnlp-main.1730]] (in corpus) addresses filler word insertion as a paralinguistic naturalness feature adjacent to emotion synthesis. Interspeech 2025 adds three contributions: MIKU-PAL ([[interspeech-2025-0648]]) provides the largest open-source emotionally labeled speech dataset (131.2h, 26 emotion categories using Gemini 2.0 Flash for automated labeling at Fleiss κ=0.93 vs. human κ=0.43); EME-TTS ([[interspeech-2025-0754]]) jointly models emphasis and emotion via an Emphasis Perception Enhancement (EPE) block preventing emotional prosody from distorting word-level emphasis; DiffRO ([[interspeech-2025-0704]]) demonstrates that RL-based emotion control via a differentiable multi-task reward model can teach a codec LM to generate laughter, sobs, and breaths without any emotion-labeled RL training data. 2025: OpenS2S [[2025.emnlp-demos.70]] is the first fully open-source empathetic SLM in corpus. FireRedTTS-2 ([[2509.02020]]) demonstrates implicit emotion inference from chat context without explicit emotion labels. IndexTTS2 [[2506.21619]] introduces a three-stage training curriculum for robust emotion conditioning with GRL-based disentanglement, showing the strongest emotional expressiveness for a zero-shot system; its T2E module (LLM distillation for soft emotion routing) provides a scalable path to natural language emotion control. Marco-Voice [[2508.02038]] demonstrates that difference-in-means emotion embeddings combined with cross-orthogonal disentanglement substantially improve both speaker similarity and emotional naturalness simultaneously. EmoSteer-TTS [[2508.03543]] reveals that training-free activation steering can provide emotion control for any DiT-based flow-matching model, suggesting that emotional information is implicitly encoded in pre-trained models. GLM-TTS [[2512.14291]] shows that multi-reward RL can include emotion as a trainable objective, revealing a pronunciation/emotion trade-off. The 2025 trajectory reveals two parallel directions: (1) better supervision for explicit emotion conditioning during training (IndexTTS2, Marco-Voice, GLM-TTS); (2) post-hoc control for pre-trained models without labeled emotional data (EmoSteer-TTS).

## Open questions

- How well does automated LLM+TTS empathetic data (as in OpenS2S [[2025.emnlp-demos.70]]) substitute for human-annotated empathetic dialogues?
- Can streaming voice conversion systems (Conan [[2507.14534]]) reliably capture and transfer emotion at chunk level, or does emotion modeling require full-utterance context?
- EmoSteer-TTS [[2508.03543]] assumes linearity of emotional representations in DiT activations — is this assumption valid, and does it hold for models not trained on any emotional data?
- GLM-TTS [[2512.14291]] shows emotion reward trades off against pronunciation accuracy in RL; is this fundamental, or can a better reward weighting or curriculum resolve it?
- IndexTTS2 [[2506.21619]] uses 7 basic emotion categories; how can continuous or compositional emotion control be achieved in AR systems without an exponential growth in reference data?
- Vevo2 [[2508.16332]] extends emotion expressiveness to singing via chromagram prosody; does this prosody space generalize to spoken emotion as well?

## Papers

| ID | Title | Venue | Year | Key use of this concept |
|----|-------|-------|------|------------------------|
| [[2025.emnlp-demos.70]] | OpenS2S: Advancing Fully Open-Source End-to-End Empathetic Large Speech Language Model | EMNLP | 2025 | First fully open-source empathetic speech-to-speech LLM; automated pipeline synthesizes 100K empathetic bilingual dialogues; multi-stage training for emotion detection and empathetic response generation |
| [[2509.02020]] | FireRedTTS-2: Towards Long Conversational Speech Generation for Podcast and Chatbot | arXiv | 2025 | Demonstrates implicit emotion inference from chat context history (no explicit emotion labels); achieves 76.7–93.3% accuracy across 6 emotions (surprise, sadness, happiness, concern, apology, anger) via SFT on a 15-hour curated emotion corpus |
| [[2506.21619]] | IndexTTS2: Emotionally Expressive and Duration-Controlled AR Zero-Shot TTS | arXiv | 2025 | GRL-based emotion–speaker disentanglement with Conformer emotion perceiver; T2E module distills DeepSeek-R1 into Qwen3 LoRA for natural language emotion routing; EMOS 4.22 on 12-speaker × 7-emotion evaluation |
| [[2508.02038]] | Marco-Voice Technical Report | arXiv | 2025 | Rotational emotion embeddings via difference-in-means from paired emotional/neutral speech; cross-orthogonal constraint prevents speaker-emotion entanglement; emotional expression MOS 4.225 vs. 3.240 for CosyVoice2 |
| [[2508.03543]] | EmoSteer-TTS: Training-Free Emotion-Controllable TTS via Activation Steering | arXiv | 2025 | Training-free emotion control via difference-in-means steering vectors in DiT intermediate layers; supports conversion, interpolation, erasure, blending across F5-TTS, E2-TTS, CosyVoice2 |
| [[2412.10117]] | CosyVoice 2: Scalable Streaming Speech Synthesis with Large Language Models | arXiv | 2024 | Instruction fine-tuning with emotion, speaking rate, dialect, and paralinguistic tokens ([laughter], [breath]) integrated into base training |
| [[2512.14291]] | GLM-TTS Technical Report | arXiv | 2025 | Multi-reward GRPO RL with emotion reward and laughter reward alongside pronunciation and similarity; demonstrates pronunciation/emotion trade-off in multi-reward alignment |
| [[2406.02430]] | Seed-TTS: A Family of High-Quality Versatile Speech Generation Models | arXiv | 2024 | RL-SER variant uses emotion recognition accuracy as reward; improves emotion category accuracy from 0.13–0.53 to 0.78–0.91 for surprise/happy/sad/angry |
| [[2508.16332]] | Vevo2: A Unified and Controllable Framework for Speech and Singing Voice Generation | arXiv | 2025 | Chromagram-based prosody tokenizer captures melodic/emotional prosody across speech and singing; GRPO post-training with joint intelligibility and prosody similarity rewards |
| [[2025.emnlp-main.40]] | Towards Controllable Speech Synthesis in the Era of LLMs: A Systematic Survey | EMNLP | 2025 | Surveys emotion control strategies: style tagging, reference audio, natural language description, instruction-guided; identifies simultaneous multi-attribute control as an open problem |
| [[2508.11273]] | EmoSSLSphere: Multilingual Emotional Speech Synthesis with Spherical VAE | arXiv | 2025 | Combines spherical AVD emotion vectors with HuBERT k-means prosody tokens in a FastSpeech2-style model; emotion and language conditioned in a unified framework; nMOS 4.13 on English ESD |
| [[2510.05758]] | EMORL-TTS: Reinforcement Learning for Fine-Grained Emotion Control in TTS | ICASSP | 2026 | GRPO RL applied to frozen LLM-based TTS for fine-grained continuously controllable emotion intensity; mean emotion accuracy 0.88 vs. 0.85 for EmoSphere++ — RL improves over SFT-only (0.81) for emotion control |
| [[2601.03888]] | IndexTTS 2.5 Technical Report | arXiv | 2026 | Extends IndexTTS 2's emotion-controlled synthesis to 4 languages via semantic codec compression; RLAIF emotional alignment fine-tuning as one of three optimization components |
| [[interspeech-2025-0648]] | MIKU-PAL: Automated Multimodal Method for Speech Paralinguistic and Affect Labeling | Interspeech | 2025 | Automated pipeline using Gemini 2.0 Flash for zero-shot emotion labeling at Fleiss κ=0.93 (vs. human 0.43); releases 131.2h MIKU-EmoBench covering 26 emotion categories; fine-tuned Fish-Speech achieves MOS 4.12 and emotion similarity 0.92 |
| [[interspeech-2025-0754]] | EME-TTS: Unlocking the Emphasis and Emotion Link in Speech Synthesis | Interspeech | 2025 | First joint modeling of emphasis and emotion in TTS; EPE block prevents emotional prosody from distorting word-level emphasis; MOS 4.22 vs. EmoSpeech 4.14; emphasis recognition accuracy 0.78 vs. 0.73 without EPE |
| [[interspeech-2025-0704]] | Differentiable Reward Optimization for LLM based TTS system | Interspeech | 2025 | MTR-based DiffRO teaches a codec LM to generate laughter, sobs, and breaths for emotion via RL reward without emotion-labeled RL data; emotion accuracy Happy zh/en 1.00/0.92, outperforming F5-TTS |
