---
slug: gan-vocoder
title: GAN Vocoder
aliases: [HiFi-GAN, MelGAN, adversarial vocoder, neural vocoder, GAN-based waveform synthesis, causal vocoder, streaming vocoder]
related_concepts: [diffusion-tts, flow-matching, neural-codec, transformer-enc-dec-tts, streaming-tts, voice-conversion]
last_updated: 2026-05-30
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
| [[2025.emnlp-main.989]] | VocalNet: Speech LLMs with Multi-Token Prediction for Faster and High-Quality Generation | EMNLP | 2025 | Uses HiFi-GAN vocoder (via CosyVoice2's pipeline) as the final stage; identifies the GAN vocoder (combined with flow-matching) as the dominant latency component in streaming speech LLMs |
| [[2025.acl-long.682]] | Recent Advances in Speech Language Models: A Survey | ACL | 2025 | Surveys GAN-based vocoders (MelGAN, HiFi-GAN, Fre-GAN, BigVGAN) and their role as token-to-speech synthesizers; distinguishes direct synthesis (HiFi-GAN on tokens) from input-enhanced synthesis (flow-matching → mel → HiFi-GAN as in CosyVoice) |
| [[2025.acl-industry.42]] | Scaling Under-Resourced TTS: A Data-Optimized Framework with Advanced Acoustic Modeling for Thai | ACL | 2025 | Uses a GAN-based waveform decoder (L1 + mel spectrogram + GAN perceptual losses) as the acoustic synthesis stage; demonstrates GAN vocoder in a low-resource production TTS system for a tonal language |
| [[2025.acl-demo.37]] | RT-VC: Real-Time Zero-Shot Voice Conversion with Speech Articulatory Coding | ACL | 2025 | Uses a DDSP (Differentiable Digital Signal Processing) harmonic-plus-noise vocoder instead of HiFi-GAN for waveform synthesis; the DDSP vocoder conditioned on articulatory features and speaker embedding via FiLM layers — an alternative to GAN vocoders for interpretable streaming VC |
| [[2025.acl-long.654]] | Language-Codec: Bridging Discrete Codec Representations and Speech Language Models | ACL | 2025 | Uses a Vocos-style Fourier-based decoder (Conv1d ResBlocks + ConvNeXt + ISTFT) and a multi-discriminator GAN training setup (MPD, MRD, MSD, complex STFT discriminator); demonstrates that GAN training with multiple discriminators is essential for high-fidelity codec reconstruction |
| [[2508.12001]] | FNH-TTS: A Fast, Natural, and Human-Like Speech Synthesis System | arXiv | 2025 | Replaces VITS's HiFi-GAN vocoder with VOCOS (Fourier-based) and adds two advanced multi-scale discriminators; MOS 4.48 on LJSpeech (highest among evaluated systems); demonstrates GAN vocoder replacement as a practical quality improvement for VITS-based systems |
| [[interspeech-2025-0455]] | APTTS: Adversarial Post-training in Latent Flow Matching for Fast and High-fidelity TTS | Interspeech | 2025 | Prompt-conditioned JCU discriminator applies GAN-style adversarial and feature-matching losses to a latent flow-matching TTS generator during post-training; enables 4-step inference with MOS 3.58 |
| [[interspeech-2025-0554]] | RapFlow-TTS: Rapid and High-Fidelity TTS with Improved Consistency Flow Matching | Interspeech | 2025 | Conv2d mel-spectrogram discriminator provides adversarial + feature-matching losses as the highest-impact component (NISQA 3.78→4.19) in consistency flow matching TTS acceleration |
| [[interspeech-2025-0406]] | Zero-Shot Mono-to-Binaural Speech Synthesis | Interspeech | 2025 | WaveFit GAN-based vocoder (pretrained on LibriLight 60k hours) is applied iteratively to a geometric warp-initialized binaural estimate; implicit speech distribution of the GAN enables zero-shot binaural rendering without binaural training data |
| [[interspeech-2025-0854]] | Bridging the Training–Inference Gap in TTS | Interspeech | 2025 | GAN U-Net postprocessor trained with pix2pix-style adversarial loss for low-resource spectrogram enhancement; MUSHRA 74.8 vs. 50.9 for the ForwardTacotron baseline |
| [[2025.naacl-long.110]] | WaveFM: A High-Fidelity and Efficient Vocoder Based on Flow Matching | NAACL | 2025 | Flow-matching vocoder benchmarks against BigVGAN and HiFi-GAN; uses GAN-style auxiliary losses (adversarial STFT loss, feature matching) enabled by reparameterized clean-waveform prediction objective |
