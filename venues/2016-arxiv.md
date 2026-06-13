---
title: "arXiv 2016"
venue: arXiv
year: 2016
papers_ingested: 1
last_updated: 2026-06-13
---

## Overview

The single arXiv 2016 paper in this corpus is [[1609.03499]] (WaveNet), a foundational work from DeepMind that established raw-waveform autoregressive generation as the dominant neural TTS paradigm. WaveNet demonstrated that dilated causal convolutions spanning receptive fields of hundreds of milliseconds produce substantially higher naturalness than all prior vocoder-based systems (MOS 4.21 vs. 3.86 for concatenative synthesis). The paper also established multi-speaker conditioning as a viable design pattern, with speaker embeddings enabling a single model to represent 109 voices. The core tension WaveNet identified shaped TTS research for the following decade: sample-level autoregressive generation is too slow for real-time deployment, motivating GAN vocoders, flow-based vocoders, and ultimately the codec-and-LM paradigm that decouples acoustic quality from inference speed.

## Papers

| ID | Title |
|----|-------|
| 1609.03499 | WaveNet: A Generative Model for Raw Audio |
