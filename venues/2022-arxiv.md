---
title: "arXiv 2022"
venue: arXiv
year: 2022
papers_ingested: 8
last_updated: 2026-06-14
---

## Overview

The 2022 arXiv preprints in this corpus represent a pivotal year for neural speech and audio generation. Three foundational infrastructure papers appeared: [[2209.03143]] (AudioLM) established the semantic-token plus acoustic-token hierarchical generation paradigm that anchors the codec LM lineage; [[2210.13438]] (EnCodec) provided the RVQ streaming codec whose design became the standard for codec-based TTS; and [[2210.02747]] (Flow Matching) formalized the conditional flow matching framework that most subsequent non-autoregressive TTS systems adopt. On the evaluation and conditioning side, [[2207.12598]] (Classifier-Free Guidance) introduced the guidance mechanism now used universally in conditional diffusion and flow-matching systems, and [[2204.02152]] (UTMOS) established the automatic MOS predictor that became the community's primary quality proxy. [[2206.04658]] (BigVGAN) and [[2212.04356]] (Whisper) completed the year's infrastructure: BigVGAN provided a universal neural vocoder via periodic activations, and Whisper provided a robust multilingual ASR model that TTS evaluation pipelines use for WER computation. Taken together, 2022 is the year when the modern foundational stack (codec + flow matching + SSL-assisted evaluation) was assembled.

## Papers

| ID | Title |
|----|-------|
| 2212.04356 | Robust Speech Recognition via Large-Scale Weak Supervision |
| 2210.13438 | High Fidelity Neural Audio Compression |
| 2204.02152 | UTMOS: UTokyo-SaruLab System for VoiceMOS Challenge 2022 |
| 2210.02747 | Flow Matching for Generative Modeling |
| 2209.03143 | AudioLM: a Language Modeling Approach to Audio Generation |
| 2206.04658 | BigVGAN: A Universal Neural Vocoder with Large-Scale Training |
| 2207.12598 | Classifier-Free Diffusion Guidance |
| 2206.08317 | Paraformer: Fast and Accurate Parallel Transformer for Non-autoregress |
