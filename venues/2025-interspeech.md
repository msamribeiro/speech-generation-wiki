---
title: "Interspeech 2025"
venue: Interspeech
year: 2025
papers_ingested: 29
last_updated: 2026-06-02
---

# Interspeech 2025

## Overview

Interspeech 2025 is the largest single venue in the corpus with 27 ingested papers, spanning TTS, VC, SCA, codec design, and evaluation. The batch reveals five dominant themes.

**Flow-matching acceleration** is the most concentrated technical cluster: APTTS ([[interspeech-2025-0455]]) introduces adversarial post-training in a latent FM framework (4-step inference, RTF 0.052), while RapFlow-TTS ([[interspeech-2025-0554]]) applies consistency flow matching to Matcha-TTS (2-NFE, MOS 4.01). Together they confirm that few-step FM synthesis is now practical without quality collapse, using adversarial and consistency mechanisms respectively. The Swedish accentedness paper ([[interspeech-2025-0762]]) also uses Matcha-TTS as a backbone, extending FM to code-switching acoustic control.

**RLHF for TTS** expands beyond the established DPO/GRPO paradigm: DiffRO ([[interspeech-2025-0704]]) introduces differentiable token-level reward optimization for codec LMs (Gumbel-Softmax bypassing FM+vocoder, achieving WER 0.78% on seed-tts-eval zh), while DLPO ([[interspeech-2025-0063]]) demonstrates RLHF for diffusion TTS via task-specific loss regularization. These two papers together establish that RLHF is viable across both the codec LM and the diffusion paradigm.

**Robustness and reliability** is a recurring practical theme: NR-LauraTTS ([[interspeech-2025-0319]]) introduces codec-domain speech denoising (1.10G FLOPs vs. 38.93G for waveform SE); the monotonic attention paper ([[interspeech-2025-0551]]) addresses repetition/omission failures in LLM-based TTS without external aligners; and the dysarthric speaker TTS paper ([[interspeech-2025-0596]]) extends zero-shot TTS to pathological speech.

**Evaluation and dataset contributions** are unusually numerous: HiFiTTS-2 ([[interspeech-2025-0989]], 36.7k hours 22kHz), MIKU-PAL ([[interspeech-2025-0648]], 131.2h emotion-labeled), math TTS evaluation ([[interspeech-2025-0779]]), Spanish MOS corpus ([[interspeech-2025-0973]]), and low-resource Punjabi/Urdu corpora ([[interspeech-2025-0469]]).

**Hybrid codec and tokenization** includes two notable contributions: PAST ([[interspeech-2025-0669]]) challenges the need for SSL pseudo-label distillation in hybrid tokenizers by using supervised CTC + phoneme classification directly on RVQ-1, while the speech tokenization grid search ([[interspeech-2025-0310]]) demonstrates that N=80ms + K=16384 HuBERT K-means tokenization beats original-resolution GSLM tokenization with 50% data reduction.

## Papers

| ID | Title |
|----|-------|
| interspeech-2025-0047 | Revival with Voice: Multi-modal Controllable Text-to-Speech Synthesis |
| interspeech-2025-0063 | Fine-Tuning Text-to-Speech Diffusion Models Using Reinforcement Learning with Human Feedback |
| interspeech-2025-0143 | Multimodal Prosody Modeling: A Use Case for Multilingual Sentence Mode Prediction |
| interspeech-2025-0253 | Long-Context Speech Synthesis with Context-Aware Memory |
| interspeech-2025-0310 | Exploring the Effect of Segmentation and Vocabulary Size on Speech Tokenization for Speech Language Models |
| interspeech-2025-0319 | Improving Noise Robustness of LLM-based Zero-shot TTS via Discrete Acoustic Token Denoising |
| interspeech-2025-0902 | VoiceQualityVC: A Voice Conversion System for Studying the Perceptual Effects of Voice Quality in Speech |
| interspeech-2025-0406 | Zero-Shot Mono-to-Binaural Speech Synthesis |
| interspeech-2025-0408 | Improving User Impression of Spoken Dialogue Systems by Controlling Para-linguistic Expression Based on Intimacy |
| interspeech-2025-0455 | APTTS: Adversarial Post-training in Latent Flow Matching for Fast and High-fidelity Text-to-Speech |
| interspeech-2025-0554 | RapFlow-TTS: Rapid and High-Fidelity Text-to-Speech with Improved Consistency Flow Matching |
| interspeech-2025-0551 | Monotonic Attention for Robust Text-to-Speech Synthesis in Large Language Model Frameworks |
| interspeech-2025-0575 | VoiceMark: Zero-Shot Voice Cloning-Resistant Watermarking Approach Leveraging Speaker-Specific Latents |
| interspeech-2025-0596 | Facilitating Personalized TTS for Dysarthric Speakers Using Knowledge Anchoring and Curriculum Learning |
| interspeech-2025-0648 | MIKU-PAL: An Automated and Standardized Multimodal Method for Speech Paralinguistic and Affect Labeling |
| interspeech-2025-0669 | PAST: Phonetic-Acoustic Speech Tokenizer |
| interspeech-2025-0704 | Differentiable Reward Optimization for LLM based TTS system |
| interspeech-2025-0723 | Counterfactual Activation Editing for Post-hoc Prosody and Mispronunciation Correction in TTS Models |
| interspeech-2025-0754 | EME-TTS: Unlocking the Emphasis and Emotion Link in Speech Synthesis |
| interspeech-2025-0762 | Intrasentential English in Swedish TTS: perceived English-accentedness |
| interspeech-2025-0779 | Intelligibility of Text-to-Speech Systems for Mathematical Expressions |
| interspeech-2025-0787 | Gradual modeling of the Lombard effect by modifying speaker embeddings from a Text-To-Speech model |
| interspeech-2025-0469 | Developing High-Quality TTS for Punjabi and Urdu: Benchmarking against MMS Models |
| interspeech-2025-0854 | Bridging the Training–Inference Gap in TTS: Training Strategies for Robust Generative Postprocessing for Low-Resource Speakers |
| interspeech-2025-0973 | A Dataset for Automatic Assessment of TTS Quality in Spanish |
| interspeech-2025-0989 | HiFiTTS-2: A Large-Scale High Bandwidth Speech Dataset |
| interspeech-2025-1034 | Non-Standard Accent TTS Support via Large Multi-Accent Frontend Pronunciation Knowledge Transfer |
| interspeech-2025-0196 | SPCODEC: Split and Prediction for Neural Speech Codec |
| interspeech-2025-0203 | ClapFM-EVC: High-Fidelity and Flexible Emotional Voice Conversion with Dual Control from Natural Language and Speech |
