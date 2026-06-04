---
slug: gan-vocoder
title: GAN Vocoder
aliases: [HiFi-GAN, MelGAN, adversarial vocoder, neural vocoder, GAN-based waveform synthesis, causal vocoder, streaming vocoder]
related_concepts: [diffusion-tts, flow-matching, neural-codec, transformer-enc-dec-tts, streaming-tts, voice-conversion]
last_updated: 2026-06-05
status: mature-infrastructure
---

## Executive Summary

> [!abstract]
> GAN vocoders convert low-level acoustic representations (typically mel-spectrograms) into full-resolution waveforms using adversarial training. HiFi-GAN established the paradigm in 2020 and remains the standard neural vocoder in 2025 TTS and VC systems. The active frontier has shifted from quality improvements to causal/streaming adaptation for real-time applications.

## Current Status

mature-infrastructure — GAN vocoders are universally adopted as the waveform synthesis stage in TTS and VC pipelines; HiFi-GAN and its derivatives (BigVGAN, Vocos) are the de facto standard. Quality improvements have largely plateaued; current research focuses on causal streaming variants for low-latency applications.

## Why This Matters

Vocoder quality is often the bottleneck in overall TTS/VC perceptual quality. Modern GAN vocoders achieve near-transparent quality compared to ground-truth audio for clean speech. However, adapting them for streaming (online, causal) operation is non-trivial: transposed convolutions produce checkerboard artifacts when zero-padded to enforce causality, and frame-by-frame autoregressive vocoders are too slow for real-time applications.

## Core Idea

A GAN vocoder is a generative adversarial network that converts a low-level acoustic representation (typically a mel-spectrogram) into a full-resolution waveform. The generator synthesizes the waveform from the conditioning spectrogram; the discriminator distinguishes real from generated audio and provides adversarial training signal. GAN vocoders are used as the final stage in most TTS and VC pipelines, converting the system's mel-spectrogram output into audio.

The dominant GAN vocoder architecture is HiFi-GAN (Kong et al., 2020), which uses multi-period discriminators (MPD) and multi-scale discriminators (MSD) together with a generator built from transposed convolutions and residual blocks. BigVGAN and Vocos are recent improvements. For real-time applications, the vocoder must be made causal (no access to future frames).

## Methods and Variants

**HiFi-GAN.** The standard architecture: generator with transposed-convolution upsampling and multi-receptive-field fusion (MRF) residual blocks; MPD + MSD discriminators. High quality for offline synthesis.

**Causal HiFi-GAN (zero-padding).** Replace all convolutions with left-padded causal convolutions; upsampling with zero-padded transposed convolutions. Simple but introduces spectral artifacts at frame boundaries due to asymmetric padding in the upsampling layers.

**Causal Shuffle Vocoder (Conan, [[2507.14534]]).** Replace transposed convolutions with causal convolutions + pixel shuffle (channel-to-time). Pixel shuffle rearranges r×C channels into C channels at r× the frame rate, achieving temporal upsampling purely by reordering without kernel interpolation over future frames. This avoids checkerboard artifacts while maintaining strict causality. Supports streaming with a fixed per-frame latency.

**Vocos.** Replaces mel-based upsampling with frequency-domain ISTFT vocoder, achieving high quality with a simpler architecture. Used as NAR decoder in FlexiCodec-TTS [[2510.00981]].

## Major Claims

Claims are generalised propositions aggregated from paper evidence. The full claim registry with supporting paper lists is in `wiki/concepts/_evidence/gan-vocoder.yaml`.

### Strongly Supported

- GAN vocoders offer the best quality-to-speed trade-off for waveform synthesis and are the dominant choice in 2025 TTS and VC systems.
  Supporting: [[2025.acl-long.682]], [[2025.acl-industry.42]], [[2025.acl-long.654]], [[2508.12001]]

- Adversarial training with multiple discriminators (MPD, MSD, STFT-based) is essential for high-fidelity waveform and codec reconstruction.
  Supporting: [[2025.acl-long.654]], [[interspeech-2025-0554]], [[2025.naacl-long.110]]

- Zero-padding transposed convolutions in causal HiFi-GAN introduce checkerboard spectral artifacts; pixel-shuffle upsampling eliminates this while preserving strict causality.
  Supporting: [[2507.14534]]

### Emerging

- GAN-style adversarial post-training can accelerate flow-matching TTS inference (4-step) without a dedicated vocoder stage.
  Supporting: [[interspeech-2025-0455]]

- Flow-matching vocoders using GAN-style auxiliary losses can match or exceed BigVGAN/HiFi-GAN quality while offering faster inference.
  Supporting: [[2025.naacl-long.110]]

- Adversarial training is not necessary for high-quality mel-spectrogram vocoders that explicitly predict the phase spectrum; frequency-weighted phase supervision can substitute for discriminator-provided phase signal.
  Supporting: [[2508.07711]]

- The necessity of GAN in neural vocoders is contingent on the presence of explicit phase prediction: removing adversarial training without explicit phase modelling causes a quality gap that targeted improvements do not fully close.
  Supporting: [[2508.07711]]

- Inserting an explicit bandwidth extension step (via a diffusion model) prior to vocoding can improve both spectrogram visual realism and perceived audio quality in singing voice synthesis, but only when the vocoder is redesigned to handle the higher-resolution intermediate.
  Supporting: [[2508.01796]]

- Multi-scale STFT discriminators in the time-frequency domain are effective GAN training components for neural codecs targeting general audio, achieving large ViSQOL advantages over waveform-domain baselines at low bitrates.
  Supporting: [[2508.05207]]

- 1.58-bit quantisation-aware training applied selectively to GAN vocoder convolutional layers is more effective than naive channel reduction at equivalent model sizes, achieving 83% model compression with reasonable quality retention.
  Supporting: [[interspeech-2025-1122]]

## Relationship to Other Concepts

### Competes With
- [[diffusion-tts]] — diffusion vocoders (DiffWave, WaveGrad) achieve high quality but require many inference steps, making them impractical for real-time synthesis where GAN vocoders dominate.
- [[flow-matching]] — flow-matching vocoders (WaveFM, [[2025.naacl-long.110]]) are an emerging alternative that can incorporate GAN-style losses; positioned as a potential successor in vocoder applications.

### Commonly Paired With
- [[transformer-enc-dec-tts]] — GAN vocoders are the standard waveform synthesis backend for FastSpeech2-style and VITS-based enc-dec TTS systems.
- [[neural-codec]] — Vocos-style ISTFT decoders are used in neural codec architectures (Language-Codec, [[2025.acl-long.654]]) with GAN discriminator training.
- [[streaming-tts]] — causal GAN vocoder variants (Causal Shuffle Vocoder, [[2507.14534]]) are the enabling component for real-time streaming VC and TTS.
- [[voice-conversion]] — GAN vocoders are universally used as the waveform synthesis stage in VC pipelines, including streaming VC systems.

## Representative Papers

### Foundational
- [[2507.14534]] — introduces the Causal Shuffle Vocoder, the first artifact-free strictly causal adaptation of HiFi-GAN for streaming VC applications.

### Influential
- [[2025.acl-long.654]] — demonstrates that GAN training with multiple discriminators (MPD, MRD, MSD, complex STFT) is essential for high-fidelity codec reconstruction in Language-Codec.
- [[2508.12001]] — replaces HiFi-GAN with Vocos in a VITS-based system and adds advanced multi-scale discriminators, achieving MOS 4.48 on LJSpeech (highest among evaluated systems).

### Recent Highlights
- [[interspeech-2025-0554]] — shows a Conv2d mel-spectrogram discriminator is the highest-impact single component (+0.41 NISQA) in consistency flow matching TTS acceleration.
- [[2025.naacl-long.110]] — flow-matching vocoder (WaveFM) using GAN-style auxiliary losses benchmarks against BigVGAN and HiFi-GAN, signalling a potential shift in the vocoder paradigm.

## Open Questions

- Is pixel shuffle the optimal approach for causal upsampling, or are other artifact-free alternatives possible?
- Can causal GAN vocoders achieve quality parity with non-causal systems on perceptual evaluation?
- How sensitive is causal vocoder quality to the right-context size in partially-causal settings?
- FreeGAN [[2508.07711]] shows GAN is unnecessary for explicit-phase vocoders on VCTK 16 kHz; does this finding generalise to 24 kHz or 44.1 kHz targets, or to zero-shot conditions with out-of-domain speakers?
- The LSE+Vocos2D pipeline [[2508.01796]] inserts a 32-step diffusion model prior to vocoding; can this be accelerated (e.g. via flow matching or consistency distillation) without losing the spectrogram realism gains?
- [[interspeech-2025-1122]] shows the GAN vocoder stage is more sensitive to aggressive quantisation than the acoustic model in a lightweight JETS system; does this pattern hold for larger-scale or transformer-heavy vocoders?

## Trend Summary

2020: HiFi-GAN established GAN vocoders as the standard. 2022–2024: BigVGAN and Vocos improved quality further. 2025: Streaming/causal adaptation is the active frontier for VC applications. [[2507.14534]] demonstrates a pixel-shuffle-based solution that eliminates zero-padding artifacts in causal streaming vocoders. Integration pass 5 adds: FreeGAN [[2508.07711]] provides the clearest empirical evidence that GAN discriminators are dispensable for vocoders with explicit phase prediction — isolating the phase modelling capability as the active ingredient rather than adversarial training itself; this also documents a concrete UTMOS/subjective MOS divergence for vocoder evaluation. The LSE+Vocos2D pipeline [[2508.01796]] proposes bandwidth extension as an intermediate spectrogram super-resolution step for singing synthesis, with a 2D-convolution vocoder redesign handling the expanded frequency resolution. SpectroStream [[2508.05207]] establishes multi-scale STFT discriminators as effective for full-band general audio codec training, outperforming waveform-domain DAC baselines at low bitrates. Meitei Mayek TTS [[2508.06870]] and MultiGen [[2508.08715]] confirm that HiFi-GAN remains the default waveform synthesis backend in applied low-resource TTS pipelines. Integration pass 6 adds BitTTS [[interspeech-2025-1122]], which demonstrates 1.58-bit QAT for on-device TTS, finding that the GAN vocoder (HiFi-GAN) is consistently more sensitive to aggressive quantisation than the acoustic model — a practical data point for model compression in the TTS vocoder space.

## All Papers

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
| [[2508.01796]] | Enhancing Spectrogram Realism in Singing Voice Synthesis | arXiv | 2025 | Vocos2D: 2D-convolution vocoder with discriminator augmentation and shortcut conditioning; combined with diffusion-based LSE step achieves MOS 4.176 vs. Vocos baseline 3.576 for singing synthesis |
| [[2508.05207]] | SpectroStream | arXiv | 2025 | Multi-scale STFT discriminator at 6 window lengths for GAN training of a general audio codec; operates on time-frequency domain; 76.3% preference over DAC at 2.7 kbps |
| [[2508.06870]] | TTS for Meitei Mayek Script | arXiv | 2025 | HiFi-GAN as standard vocoder in first TTS for Manipuri script; confirms GAN vocoder remains the default in new low-resource applied TTS pipelines |
| [[2508.07711]] | Is GAN Necessary for Mel-Spectrogram-based Neural Vocoder? | arXiv | 2025 | FreeGAN: GAN-free explicit-phase vocoder matches BigVGAN MOS on VCTK; GAN dispensable only for explicit-phase architectures; documents UTMOS vs. subjective MOS divergence |
| [[interspeech-2025-1122]] | BitTTS: Highly Compact Text-to-Speech Using 1.58-bit Quantization and Weight Indexing | Interspeech | 2025 | HiFi-GAN vocoder in a JETS-based system; demonstrates GAN vocoder's disproportionate sensitivity to aggressive quantisation; 83% model compression via 1.58-bit QAT with weight indexing |
| [[2508.18006]] | Unseen Speaker and Language Adaptation for Lightweight TTS with Adapters | arXiv | 2025 | MultiBand MelGAN as vocoder in 2M-param GAN-based TTS; convolutional vocoder adapters (50K params) contribute to cross-lingual quality; vocoderside adapters drive accent nativeness |
| [[2509.02244]] | Spectrogram Patch Codec | arXiv | 2025 | HiFi-GAN vocoder trained on codec-reconstructed spectrograms; codec-artefact conditioning improves vocoder robustness; alternative to RVQ with single-stage 2D patch VQ |
