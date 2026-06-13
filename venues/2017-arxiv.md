---
title: "arXiv 2017"
venue: arXiv
year: 2017
papers_ingested: 3
last_updated: 2026-06-13
---

## Overview

The three arXiv 2017 papers in this corpus represent two distinct contributions: two TTS foundational works and one ML infrastructure paper. [[1703.10135]] (Tacotron) introduced the first end-to-end seq2seq TTS system from characters, achieving naturalness competitive with statistical parametric synthesis (MOS 3.82 vs. 3.69) without any hand-engineered linguistic features. [[1712.05884]] (Tacotron 2) refined this by using mel spectrograms as an intermediate representation and conditioning WaveNet on them, reaching near-human naturalness (MOS 4.526 vs. 4.582 for ground truth) and establishing the two-stage acoustic model and neural vocoder pipeline that dominated TTS until the codec LM era. [[1711.05101]] (AdamW, decoupled weight decay) is included as a foundational ML method used across virtually all modern speech systems. Tacotron and Tacotron 2 together define the transition from feature-engineering TTS to learned TTS, setting the quality benchmark that later non-autoregressive and codec-based systems were measured against.

## Papers

| ID | Title |
|----|-------|
| 1711.05101 | Decoupled Weight Decay Regularization |
| 1712.05884 | Natural TTS Synthesis by Conditioning WaveNet on Mel Spectrogram Predi |
| 1703.10135 | Tacotron: Towards End-to-End Speech Synthesis |
