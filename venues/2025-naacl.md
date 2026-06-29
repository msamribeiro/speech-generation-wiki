---
title: "NAACL 2025"
venue: NAACL
year: 2025
papers_ingested: 8
last_updated: 2026-05-30
---

# NAACL 2025

## Overview

NAACL 2025 is represented by three papers in the corpus. All challenge prior assumptions in TTS from different angles:

[[2025.findings-naacl.184]] (Continuous Speech Tokenizer in TTS) proposes retaining encoder outputs directly rather than discretizing them via RVQ, demonstrating measurable high-frequency information retention advantages (8 kHz retention 0.55 vs. 0.34 for discrete) and improved downstream TTS quality (WER 6.59% vs. 12.73% for VALL-E on LibriSpeech).

[[2025.naacl-long.242]] (StyleTTS-ZS) takes the opposite approach to improving efficiency: instead of eliminating the codec, it compresses prosodic variation into a fixed-length K=50 RVQ latent and runs diffusion only over that compact space. A one-step distillation requires only 10k samples, achieving 10-20x faster inference than prior diffusion SOTA while reaching CMOS -0.032 vs. ground truth on LibriSpeech test-clean — effectively human-parity naturalness, with highest expressiveness (pitch+energy CV) among compared systems.

[[2025.naacl-long.110]] (WaveFM) extends the flow-matching paradigm to the vocoder stage: a mel-conditioned prior (instead of Gaussian noise) reduces transport distance, while a reparameterized clean-waveform prediction objective enables GAN-style auxiliary losses. Adapted consistency distillation achieves single-step inference at 303× real-time, matching BigVGAN-base quality.

Together, the NAACL 2025 TTS papers cover a broad methodological range: continuous tokenization challenges whether discrete VQ is needed at all; compact latent diffusion demonstrates that focusing the generative model on a small well-structured latent enables dramatic efficiency gains; and WaveFM demonstrates that flow matching is viable as a vocoder architecture, extending FM's reach beyond the acoustic modeling stage.

## Papers

| ID | Title |
|----|-------|
| 2025.findings-naacl.184 | Continuous Speech Tokenizer in Text To Speech |
| 2025.naacl-long.242 | StyleTTS-ZS: Efficient High-Quality Zero-Shot TTS with Distilled Time-Varying Style Diffusion |
| 2025.naacl-long.110 | WaveFM: A High-Fidelity and Efficient Vocoder Based on Flow Matching |
| 2025.naacl-srw.6 | Towards Codec-LM Co-design for Neural Codec Language Models |
| 2025.findings-naacl.298 | Gender Bias in Instruction-Guided Speech Synthesis Models |
| 2025.findings-naacl.471 | The Role of Prosody in Spoken Question Answering |
| 2025.naacl-long.464 | ManaTTS Persian: a recipe for creating TTS datasets for lower resource languages |
| 2025.naacl-long.619 | ProSE: Diffusion Priors for Speech Enhancement |
