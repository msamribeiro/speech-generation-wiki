---
id: "interspeech-2025-0575"
title: "VoiceMark: Zero-Shot Voice Cloning-Resistant Watermarking Approach Leveraging Speaker-Specific Latents"
authors: [Haiyun Li, Zhiyong Wu, Xiaofeng Xie, Jingran Xie, Yaoxun Xu, Hanyang Peng]
organization: Tsinghua University
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-05-30
task: [VC, TTS]
architecture: [VAE]
conditioning: [speaker-conditioned, zero-shot]
training: [supervised]
model_size: "not reported"
codec_used: "SpeechTokenizer (RVQ-based)"
datasets_train: [VCTK]
datasets_eval: [VCTK, LibriSpeech]
metrics:
  - name: ACC
    value: 0.964
    system: VoiceMark
    testset: VCTK (CosyVoice zero-shot VC)
  - name: ACC
    value: 0.979
    system: VoiceMark
    testset: VCTK (F5-TTS zero-shot VC)
  - name: ACC
    value: 0.957
    system: VoiceMark
    testset: VCTK (MaskGCT zero-shot VC)
  - name: PESQ
    value: 2.2
    system: VoiceMark
    testset: VCTK
  - name: SMOS
    value: 4.25
    system: VoiceMark
    testset: VCTK (15 subjects, 40 samples)
  - name: ACC
    value: 0.508
    system: AudioSeal (baseline)
    testset: VCTK (CosyVoice zero-shot VC)
code_available: true
demo_available: true
url: https://www.isca-archive.org/interspeech_2025/li25g_interspeech.html
related_concepts: [voice-conversion, zero-shot-tts, neural-codec, disentanglement]
related_papers: [2407.05407, 2410.06885]
---
> [!abstract] Interspeech · 2025 · Conference
> **Haiyun Li et al.** (Tsinghua University) · [→ Paper](https://www.isca-archive.org/interspeech_2025/li25g_interspeech.html) · Demo: ✓ · Code: ✓
>
> VoiceMark is the first watermarking method that survives zero-shot voice cloning by embedding watermarks into speaker-specific latents of a neural codec rather than into raw audio, achieving over 95% detection accuracy where prior methods fall to ~50%.

## Problem

Existing VC-resistant watermarking methods embed marks into model weights via training-time exposure, which entirely fails in zero-shot VC. Zero-shot VC models (CosyVoice, F5-TTS, MaskGCT) synthesize cloned audio directly from a short audio prompt without any model training, so there is no opportunity to bake the watermark into the model. Additionally, the zero-shot synthesis process changes content, duration, and speed, causing traditional audio watermarks to be filtered or distorted beyond recovery — prior SOTA methods reach only ~50% bit accuracy (near random) after zero-shot VC, rendering them useless for attribution.

## Method

VoiceMark consists of three components built on a pretrained SpeechTokenizer RVQ model:

**Disentanglement of speaker-specific latents.** Using a pretrained RVQ codec inspired by SpeechTokenizer, the encoder compresses input audio into latents. An 8-layer RVQ quantizes these into codebooks: VQ layer 1 is distilled to capture content (using HuBERT supervision); VQ layers 2–8 capture speaker-specific information (timbre, pitch, prosody). Content latents are left unmodified; only speaker-specific latents are used for watermark embedding.

**Cross-attention watermark embedder.** A 4-layer transformer decoder embeds an n-bit watermark (default n=16) into the speaker-specific latents via cross-attention, where speaker latents are queries and the projected watermark sequence is keys/values. A VAD-based binary cross-entropy loss constrains the embedder to act only on speech-containing frames (labelled 1 by dual-threshold VAD), ignoring silence and voiceless frames. The modified latents are recombined with content latents and decoded to produce watermarked audio.

**Transformer watermark decoder.** A separate 8-layer transformer encoder extracts the watermark from speaker-specific latents of any audio — whether original or synthesized by a zero-shot VC model. Learnable CLS tokens corresponding to the watermark hexadecimal positions are concatenated with speaker latents and decoded via self-attention.

**VC-simulated augmentation.** The decoder is trained with augmentations that simulate zero-shot VC distortions: random frame masking (20%), segment shuffling in 50 ms windows (50%), segment replacement with original audio (50%), EnCodec encode/decode, and speed/amplitude/filter perturbations. Crucially, the decoder never sees actual VC model outputs during training — robustness is achieved entirely through these simulated distortions.

The watermark decoder is trained jointly with the embedder using a weighted loss combining VAD cross-entropy, speaker cosine similarity loss, multi-scale mel spectrogram loss, adversarial loss, and watermark decoding cross-entropy.

## Key Results

On three zero-shot VC models (CosyVoice, F5-TTS, MaskGCT), VoiceMark achieves bit-wise accuracy of 0.964 / 0.979 / 0.957 respectively, with False Attribution Rate (FAR, 1-vs-99 candidates) of 0.112 / 0.070 / 0.141. In contrast, AudioSeal, WavMark, and Timbre all fall to ACC ≈ 0.5 and FAR ≈ 1.0 — effectively random performance. On traditional audio editing (resampling, amplitude, filtering, MP3), VoiceMark performs comparably to baselines, with 0.965–0.995 ACC and 0.014–0.109 FAR.

Audio quality: PESQ 2.2, SI-SNR 2.01, STOI 0.89, SMOS 4.25 ± 0.13. This is lower than AudioSeal (PESQ 4.32, SMOS 4.67) but comparable to SpeechTokenizer (SMOS 4.63) and better than EnCodec directly (SMOS 2.21). The quality degradation stems from the codec-based watermarking pipeline.

Ablation confirms that speaker-specific latent embedding (vs. generic waveform-level embedding as in AudioSeal) is the key innovation: replacing the embedder with AudioSeal's waveform architecture drops CosyVoice ACC from 0.964 to 0.663.

## Novelty Assessment

The central insight — that zero-shot VC models must transfer speaker-specific latents to produce high-similarity clones, and therefore a watermark embedded in those latents will also be transferred — is genuinely novel and well-reasoned. Prior work had not observed this latent-space invariance property. The VAD-based loss and VC-simulated augmentations are practical engineering contributions that address the silence/voiceless-frame and distortion problems. The architecture builds on SpeechTokenizer and standard transformer components; the novelty is in the application and the training regime, not the architecture itself.

## Limitations and Open Questions

The model is trained only on VCTK, a small clean dataset, which limits robustness to out-of-distribution audio editing (the paper notes lower-than-1.0 ACC on traditional editing, likely due to this). The audio quality impact (PESQ 2.2 vs. 4.32 for AudioSeal) is significant and may be prohibitive for some use cases. The approach does not address adversarial attacks specifically designed to remove latent-space watermarks. Coverage is limited to English; multilingual generalization is untested. Finally, the approach assumes the zero-shot VC model must preserve speaker-specific latents for high similarity — models that do not operate this way could evade detection.

## Wiki Connections

VoiceMark builds directly on the zero-shot VC capabilities of [[2407.05407]] (CosyVoice) and [[2410.06885]] (F5-TTS) as the attack models it is designed to survive. It leverages the SpeechTokenizer disentanglement scheme from the [[neural-codec]] concept. The speaker-specific latent disentanglement is central to the [[disentanglement]] concept. As a tool for protecting [[voice-conversion]] and [[zero-shot-tts]] outputs, it represents a security/attribution layer on top of those systems.
