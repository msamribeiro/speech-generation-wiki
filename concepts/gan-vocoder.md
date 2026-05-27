---
slug: gan-vocoder
title: GAN Vocoder
aliases: [HiFi-GAN, MelGAN, adversarial vocoder, neural vocoder, GAN-based waveform synthesis, causal vocoder, streaming vocoder]
related_concepts: [diffusion-tts, flow-matching, neural-codec, transformer-enc-dec-tts, streaming-tts, voice-conversion]
last_updated: 2026-05-26
---

# GAN Vocoder

## What it is

A GAN vocoder is a generative adversarial network that converts a low-level acoustic representation (typically a mel-spectrogram) into a full-resolution waveform. The generator synthesizes the waveform from the conditioning spectrogram; the discriminator distinguishes real from generated audio and provides adversarial training signal. GAN vocoders are used as the final stage in most TTS and VC pipelines, converting the system's mel-spectrogram output into audio.

The dominant GAN vocoder architecture is HiFi-GAN (Kong et al., 2020), which uses multi-period discriminators (MPD) and multi-scale discriminators (MSD) together with a generator built from transposed convolutions and residual blocks. BigVGAN and Vocos are recent improvements. For real-time applications, the vocoder must be made causal (no access to future frames).

## Why it matters

Vocoder quality is often the bottleneck in overall TTS/VC perceptual quality. Modern GAN vocoders achieve near-transparent quality compared to ground-truth audio for clean speech. However, adapting them for streaming (online, causal) operation is non-trivial: transposed convolutions produce checkerboard artifacts when zero-padded to enforce causality, and frame-by-frame autoregressive vocoders are too slow for real-time applications.

## Current state of the art

[[2507.14534]] (Conan) introduces the Causal Shuffle Vocoder, a strictly causal adaptation of HiFi-GAN for streaming VC. It replaces transposed convolutions with causal convolutions followed by pixel shuffle (channel-to-time rearrangement), eliminating checkerboard artifacts while maintaining causality. Ablation confirms the Causal Shuffle Vocoder outperforms zero-padding causal HiFi-GAN (CMOS-Q improvement of +0.21 in favor of CSV).

## Key variants and sub-approaches

**HiFi-GAN.** The standard architecture: generator with transposed-convolution upsampling and multi-receptive-field fusion (MRF) residual blocks; MPD + MSD discriminators. High quality for offline synthesis.

**Causal HiFi-GAN (zero-padding).** Replace all convolutions with left-padded causal convolutions; upsampling with zero-padded transposed convolutions. Simple but introduces spectral artifacts at frame boundaries due to asymmetric padding in the upsampling layers.

**Causal Shuffle Vocoder (Conan, [[2507.14534]]).** Replace transposed convolutions with causal convolutions + pixel shuffle (channel-to-time). Pixel shuffle rearranges r×C channels into C channels at r× the frame rate, achieving temporal upsampling purely by reordering without kernel interpolation over future frames. This avoids checkerboard artifacts while maintaining strict causality. Supports streaming with a fixed per-frame latency.

**Vocos.** Replaces mel-based upsampling with frequency-domain ISTFT vocoder, achieving high quality with a simpler architecture. Used as NAR decoder in FlexiCodec-TTS [[2510.00981]].

## Comparison to alternatives

Diffusion vocoders (DiffWave, WaveGrad) achieve high quality but require many inference steps — impractical for real-time. WaveNet autoregressive vocoders are high quality but extremely slow (sample-by-sample generation). GAN vocoders offer the best quality/speed trade-off and are universally used in 2025 TTS and VC systems.

## Year-on-year trajectory

2020: HiFi-GAN established GAN vocoders as the standard. 2022–2024: BigVGAN and Vocos improved quality further. 2025: Streaming/causal adaptation is the active frontier for VC applications. [[2507.14534]] demonstrates a pixel-shuffle-based solution that eliminates zero-padding artifacts in causal streaming vocoders.

## Open questions

- Is pixel shuffle the optimal approach for causal upsampling, or are other artifact-free alternatives possible?
- Can causal GAN vocoders achieve quality parity with non-causal systems on perceptual evaluation?
- How sensitive is causal vocoder quality to the right-context size in partially-causal settings?

## Papers

| ID | Title | Venue | Year | Key use of this concept |
|----|-------|-------|------|------------------------|
| [[2507.14534]] | Conan: A Chunkwise Online Network for Zero-Shot Adaptive Voice Conversion | arXiv (ASRU 2025) | 2025 | Introduces Causal Shuffle Vocoder: pixel-shuffle-based causal HiFi-GAN that eliminates checkerboard artifacts while maintaining strict causality for streaming VC |
