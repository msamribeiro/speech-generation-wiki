---
id: interspeech-2025-0468
title: "DualCodec: A Low-Frame-Rate, Semantically-Enhanced Neural Audio Codec for Speech Generation"
authors: ["Jiaqi Li", "Xiaolong Lin", "Zhekai Li", "Shixi Huang", "Yuancheng Wang", "Chaoren Wang", "Zhenpeng Zhan", "Zhizheng Wu"]
organization: "The Chinese University of Hong Kong, Shenzhen / Baidu"
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-06-03
task: ["codec"]
architecture: ["VAE", "GAN"]
conditioning: []
training: ["supervised", "self-supervised"]
model_size: "95–100M params"
codec_used: "DualCodec 12.5Hz / 25Hz (proposed)"
datasets_train: ["Emilia (100K hours, multilingual)"]
datasets_eval: ["LibriSpeech test-clean", "Seed-TTS-Eval"]
metrics:
  - name: MUSHRA
    value: 88.2
    system: DualCodec 12.5Hz (16384+1024)
    testset: LibriSpeech test-clean
  - name: MUSHRA
    value: 86.2
    system: DualCodec 25Hz (16384+1024)
    testset: LibriSpeech test-clean
  - name: UTMOS
    value: 4.11
    system: DualCodec 12.5Hz (16384+4096)
    testset: LibriSpeech test-clean
  - name: PESQ
    value: 3.11
    system: DualCodec 12.5Hz (16384+4096)
    testset: LibriSpeech test-clean
  - name: WER
    value: 3.4
    system: VALL-E + DualCodec 25Hz
    testset: Seed-TTS-Eval EN
  - name: WER
    value: 2.49
    system: VALL-E + DualCodec 25Hz
    testset: Seed-TTS-Eval ZH
code_available: true
demo_available: true
url: "https://www.isca-archive.org/interspeech_2025/li25e_interspeech.html"
related_concepts: ["neural-codec", "self-supervised-speech", "autoregressive-codec-tts", "spoken-language-model", "zero-shot-tts", "disentanglement"]
related_papers: ["2301.02111", "2406.02430"]
field_significance:
  level: moderate
  type: ["architectural-novelty"]
generation:
  date: 2026-06-03
  model: claude-sonnet-4-6
  commit: "55d0339"
---

> [!abstract] Interspeech · 2025 · Conference
> **Jiaqi Li et al.** (The Chinese University of Hong Kong, Shenzhen / Baidu) · [→ Paper](https://www.isca-archive.org/interspeech_2025/li25e_interspeech.html) · Demo: ✓ · Code: ✓
>
> DualCodec introduces a dual-stream encoding architecture that directly encodes SSL features into the first-layer RVQ tokens rather than distilling them, achieving simultaneously low frame rate (12.5–25 Hz), high audio quality, and strong semantic preservation in a single open-source codec system.

## Problem

Codec-based speech language models (SLMs) face a three-way tension: semantic fidelity in RVQ-1 tokens (which drives intelligibility and prosody for autoregressive generation), audio reconstruction quality, and inference efficiency via low frame rate. Prior work resolves at most two of these at once. SpeechTokenizer uses SSL distillation to improve RVQ-1 semantics but operates at 50 Hz and struggles with tonal languages due to insufficient pitch encoding. Mimi achieves 12.5 Hz with distillation but still exhibits degraded audio quality at low bitrates. DAC delivers strong audio reconstruction but carries no semantic structure in its tokens. No existing open-source codec satisfies all three requirements, and the distillation approach leaves a measurable semantic gap — even with distillation, RVQ-1 tokens decoded alone yield high WER, particularly for Chinese.

## Method

DualCodec uses two parallel encoding streams that are jointly optimised end-to-end. The SSL encoding stream takes 16th-layer features from a frozen w2v-BERT-2.0 model (600M parameters), downsamples them via 1D average pooling to the target frame rate (factor 2 for 25 Hz, factor 4 for 12.5 Hz), and passes them through a ResNet encoder (ConvNeXt blocks, 13M parameters) before VQ quantisation. The resulting discrete code becomes RVQ-1. The waveform encoding stream follows the DAC architecture — a CNN encoder with strided convolutions reaches the same frame rate, then N-1 RVQ layers quantise the residual between the waveform feature and the RVQ-1 feature; the decoder sums both streams back before upsampling to audio. This residual subtraction is the disentanglement mechanism: RVQ-1 captures semantic content from SSL directly, while RVQ-rest is responsible only for the acoustic residual.

![The dual encoding method for neural audio codecs. The upper stream is SSL encoding, and the lower stream is waveform encoding. Given a speech input, the SSL feature is obtained from a pretrained w2v-bert-2 model and then encoded as the codec's first-layer token (RVQ-1). The remaining RVQ layers encodes the residual between the waveform feature and the RVQ-1 feature, and outputs audio. The framework is trained end-to-end requiring an additional L2 SSL feature loss in addition to codec training losses.](assets/interspeech-2025-0468/figure-1.png)

Training combines an MSE SSL reconstruction loss on the downsampled SSL features with the standard DAC objective: multi-scale mel spectrogram loss, quantisation commitment loss with straight-through estimator, and adversarial loss using Multi-Period Discriminator and Multi-Scale STFT Discriminator. RVQ dropout is applied during training so the model degrades gracefully when fewer codebook layers are used at inference. The system is trained on the 100K-hour multilingual Emilia dataset on 8 A100 GPUs for 400K steps.

During TTS inference only the codec decoder is needed; both encoding streams are required only at TTS training time to generate supervision tokens.

The paper releases two primary configurations: DualCodec 25Hz (100M params) and DualCodec 12.5Hz (95M params), both open-sourced along with DualCodec-based VALL-E and AR+SoundStorm TTS systems.

## Key Results

On codec audio reconstruction quality (LibriSpeech-test-clean, normalised to 75 tokens/s, ~0.75 kbps), DualCodec 25Hz with 16384 codebook (G2) achieves MUSHRA 86.2 and UTMOS 4.08, exceeding WavTokenizer-large (MUSHRA 81.0, UTMOS 3.87) and Mimi (MUSHRA 72.8). The best variant, DualCodec 12.5Hz with 16384+4096 codebooks (G5), reaches MUSHRA 88.2 and UTMOS 4.11. These are the strongest results in the low-bitrate regime across all systems tested.

On semantic preservation, dual encoding with the 16384 RVQ-1 codebook (C3, 25Hz) reduces English WER from 55.4% (plain DAC, no enhancement) to 2.98% and Chinese WER to 2.91%, closely matching ground truth (2.13% / 1.25%). The 12.5Hz variant (D1) achieves 6.94% / 6.36% — a meaningful gap from ground truth but far below the distillation baseline.

In downstream TTS on Seed-TTS-Eval, VALL-E with DualCodec 25Hz achieves EN WER 3.4% and ZH WER 2.49%, versus 8.16% / 10.5% for VALL-E with Mimi and 15.4% / 21.5% with SpeechTokenizer. Speaker similarity (SIM-O) also improves substantially: 0.57 EN / 0.67 ZH versus 0.48 EN / 0.55 ZH for Mimi. The 12.5Hz variant cuts RTF to 0.16 (matching Mimi's speed) while maintaining much stronger WER and SIM.

> [!note]
> Audio quality evaluation uses a fixed 75 tokens/s normalisation across all systems. Bitrates therefore vary (0.75–0.93 kbps) because codebook sizes differ. The codec and TTS evaluations use different test sets (LibriSpeech vs. Seed-TTS-Eval), so reconstruction quality rankings do not directly transfer to TTS rankings.

## Novelty Assessment

The core novelty is the dual-stream encoding design: rather than distilling SSL features as a secondary training loss applied to an otherwise waveform-centric codec (SpeechTokenizer, Mimi), DualCodec treats SSL encoding as a first-class encoding path that *directly produces* the RVQ-1 tokens. The waveform stream then encodes only the residual. This architectural choice, as opposed to distillation, is what drives the large WER gap versus prior work: dual encoding reduces EN WER from 28.4% (distillation) to 5.59% at 25Hz with the same codebook size.

The remaining components are established: DAC architecture for the waveform stream, ConvNeXt ResNet blocks, w2v-BERT-2.0 as the frozen SSL model, and RVQ dropout. The training objective combines known losses. The primary contribution is therefore in the encoding architecture, not in the training recipe or data scale. The multilingual evaluation (English + Chinese) is a useful contribution beyond most prior codec papers, which focus on English-only.

## Field Significance

Moderate — DualCodec provides a concrete architectural alternative to SSL distillation for semantic codec design, showing that direct SSL encoding of RVQ-1 tokens substantially outperforms distillation on both English and Chinese intelligibility while maintaining competitive audio quality at low frame rates. As an open-source 12.5 Hz codec with strong multilingual semantics, it addresses a gap in the available toolchain for building efficient, semantically-aware SLM-based TTS systems.

## Claims

- Directly encoding SSL features as a first-class codec stream produces stronger semantic preservation in RVQ-1 tokens than distillation from an SSL model, particularly for tonal languages where pitch fidelity is critical.
- Operating a neural codec at lower frame rates with more RVQ layers at fixed token rate improves audio quality over higher-frame-rate codecs with fewer layers at the same bitrate.
- Semantic quality of RVQ-1 tokens is a primary determinant of downstream TTS intelligibility in autoregressive codec-based systems, independent of codec audio reconstruction quality.
- An SSL-based semantic stream in a codec encoder can improve perceptual audio quality beyond what waveform-only codecs achieve, even when using the same decoder architecture.

## Limitations and Open Questions

The 12.5 Hz DualCodec-based TTS lags behind the 25 Hz variant in both WER and speaker similarity, indicating that the more aggressive downsampling introduces a ceiling on semantic accuracy that affects TTS quality. The paper acknowledges this gap as the primary remaining challenge.

The SSL model (w2v-BERT-2.0, 600M parameters, frozen) is required at TTS training time but not inference. This makes the training pipeline heavier than pure waveform codec approaches. It is also unclear whether the approach generalises to SSL models other than w2v-BERT-2.0, or whether the chosen 16th layer feature is optimal across languages beyond English and Mandarin.

Codec evaluation uses a single controlled bitrate band (~0.75 kbps); performance at higher bitrates typical of studio-quality TTS is not evaluated. The subjective test has only 8 participants, limiting statistical confidence for MUSHRA comparisons between closely-scoring systems.

## Wiki Connections

Core concepts: [[neural-codec]] · [[self-supervised-speech]] · [[autoregressive-codec-tts]] · [[spoken-language-model]] · [[disentanglement]] · [[zero-shot-tts]]

In-corpus papers: [[2301.02111]] (VALL-E, used as the TTS backbone in downstream experiments) · [[2406.02430]] (Seed-TTS-Eval benchmark used for TTS evaluation)
