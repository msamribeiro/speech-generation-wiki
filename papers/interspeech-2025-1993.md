---
id: "interspeech-2025-1993"
title: "Defending Unauthorized Voice Cloning with Watermark-Aware Codecs"
authors: ["Jiankun Zhao", "Lingwei Meng", "Chengxi Deng", "Helen Meng", "Xixin Wu"]
organization: The Chinese University of Hong Kong
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-06-03
task: [TTS, VC]
architecture: [autoregressive-LM]
conditioning: [zero-shot, speaker-conditioned]
training: [supervised, fine-tuning]
model_size: "not reported"
codec_used: "EnCodec (modified watermark-aware variant)"
datasets_train: ["LibriSpeech clean-100", "LibriTTS-R"]
datasets_eval: ["LibriSpeech test-clean", "LibriTTS-R test-clean"]
metrics:
  - name: MOS
    value: 3.64
    system: O2 (proposed, unwatermarked prompt)
    testset: LibriTTS-R test-clean
  - name: MOS
    value: 1.27
    system: O2 (proposed, watermarked prompt)
    testset: LibriTTS-R test-clean
  - name: WER
    value: 15.0
    system: O2 (unwatermarked prompt)
    testset: LibriTTS-R test-clean
  - name: WER
    value: 51.0
    system: O2 (watermarked prompt)
    testset: LibriTTS-R test-clean
  - name: SPK-SIM
    value: 0.85
    system: O2 (unwatermarked prompt)
    testset: LibriTTS-R test-clean
  - name: SPK-SIM
    value: 0.74
    system: O2 (watermarked prompt)
    testset: LibriTTS-R test-clean
  - name: PESQ
    value: 3.21
    system: O2 codec reconstruction
    testset: LibriSpeech test-clean
  - name: STOI
    value: 0.94
    system: O2 codec reconstruction
    testset: LibriSpeech test-clean
code_available: null
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/zhao25k_interspeech.html"
related_concepts: ["zero-shot-tts", "neural-codec", "voice-conversion", "autoregressive-codec-tts", "evaluation-metrics"]
related_papers: ["2301.02111"]
field_significance:
  level: moderate
  type: [engineering-integration, conceptual-contribution]
generation:
  date: 2026-06-03
  model: claude-sonnet-4-6
  commit: "55d0339"
---

> [!abstract] Interspeech · 2025 · Conference
> **Zhao et al.** (The Chinese University of Hong Kong) · [→ Paper](https://www.isca-archive.org/interspeech_2025/zhao25k_interspeech.html) · Demo: ? · Code: ?
>
> A method to prevent open-source zero-shot TTS models from cloning copyrighted voices by training the codec encoder to output silence when it detects a watermarked speech prompt, degrading generation quality without any model-level hard-coded logic that attackers could bypass.

## Problem

Zero-shot TTS models are capable of imitating a voice from a short audio prompt, creating a direct pathway for unauthorized voice cloning: an attacker collects watermarked, copyright-protected audio from public sources and feeds it as a speaker prompt. Existing countermeasures — passive deepfake detectors and post-hoc watermark detectors — can identify synthesized or watermarked audio after the fact, but do not prevent cloning from occurring. A hard-coded "if-else" gate between a watermark detector and the TTS model is trivially removable in an open-source setting. A parametric solution embedded in the model itself is therefore needed to make bypass attacks costly.

## Method

The key observation is that virtually all state-of-the-art zero-shot TTS systems route the speaker prompt through a pre-trained codec encoder before passing speaker information to the generative model. By modifying the codec encoder's training objective, the paper embeds watermark detection implicitly: when the encoder encounters watermarked audio, it is trained to reconstruct a silent (all-zero) waveform rather than the original signal; for clean audio, reconstruction proceeds normally. The TTS model trained on top of this modified codec therefore receives silent codec codes whenever a watermarked prompt is supplied, causing it to generate degraded or silent output.

To make the codec robust against watermark removal attacks, 13 types of audio distortions (speed adjustment, resampling, echo, noise, low/high/band-pass filters, smoothing, dynamic range changes, and MP3/AAC/EnCodec compression) are applied to training utterances as augmentation. Watermarked and clean utterances are sampled with equal probability, and distorted versions are sampled at one-tenth the rate of clean ones to preserve reconstruction quality. The paper uses EnCodec as the backbone codec (4 residual blocks, downsample rates 8-5-4-2, 8 codebook layers of size 1024) and VALL-E as the downstream zero-shot TTS system. The TTS model is retrained on clean data using the modified codec encoder, meaning that adapting it back to a standard codec would require substantial retraining effort — forming the primary barrier against adaptation attacks.

Robustness is also achieved structurally: because the codec's code distribution under watermarked input differs significantly from the baseline codec's distribution, any attacker attempting to swap in an unmodified codec faces a distribution mismatch that degrades synthesis quality.

## Key Results

On LibriSpeech test-clean, the watermark detection accuracy (ACC) of the proposed O2 system reaches 89.3% averaged across 14 attack types, versus a 50% random baseline and 94.5% for the AudioSeal detector upper bound. Most attack types are handled at near-100% ACC; the main weaknesses are speed adjustment (FS, 50%) and low-pass filter (LP, 59.5%), though these attacks also severely degrade unwatermarked prompts, making them self-defeating.

On the TTS evaluation, O2 achieves MOS 3.64, WER 15.0%, and speaker similarity 0.85 on clean prompts — comparable to the baseline B1 (MOS 3.74, WER 14.1%, SIM 0.88). When given watermarked prompts, O2 collapses to MOS 1.27, WER 51.0%, and SIM 0.74, demonstrating effective degradation. The baseline systems B1 and B2 show negligible quality change between watermarked and unwatermarked prompts, confirming they have no rejection ability. Codec reconstruction quality under O2 (PESQ 3.21, STOI 0.94) remains close to the clean baseline (PESQ 3.63, STOI 0.96), indicating that watermark-awareness introduces only modest reconstruction overhead.

## Novelty Assessment

The contribution is primarily engineering-integration with a conceptual novelty: the idea of embedding watermark detection directly inside a codec encoder's training objective, rather than as a separate module, is to the authors' knowledge the first work of this kind for open-source TTS defense. The underlying components — EnCodec, VALL-E, AudioSeal watermarking — are all existing systems. The training modification (changing the reconstruction target to silence for watermarked inputs) is simple and effective. The robustness augmentation with 13 audio distortions follows established practice from the watermarking literature. The "silence ratio" evaluation metric is new but straightforward. The primary contribution is demonstrating that this configuration works in practice and characterising its robustness profile across a wide range of attacks.

## Field Significance

Moderate — This paper addresses a real and growing security concern for open-source zero-shot TTS, introducing a parametric protection mechanism that resists the obvious bypass (removing a hard-coded gate). Its contribution is conceptual in framing the problem and pragmatic in the solution; the underlying techniques are not new. It is most relevant as a proof-of-concept that codec-level intervention can function as an access-control layer in the TTS pipeline, rather than as a performance advance. Generalization beyond AudioSeal-watermarked prompts and beyond the VALL-E architecture remains untested.

## Claims

- Embedding watermark detection directly into codec encoder training is a viable alternative to post-hoc or hard-coded watermark gates for protecting open-source zero-shot TTS models.
- Neural codec architectures are a natural intervention point for access-control in speaker-conditioned TTS because they mediate all speaker information transfer from prompt to synthesis.
- Training-time augmentation with common audio distortions substantially improves a codec's robustness to watermark removal attacks without degrading reconstruction quality on clean audio.
- Codec-level defenses for voice cloning create a structural barrier to adaptation attacks because TTS models trained on modified codec distributions cannot be trivially swapped to unprotected codecs without retraining.

## Limitations and Open Questions

> [!warning]
> The defense is effective only against speech watermarked with the specific watermarking model (AudioSeal) used during codec training. A copyrighted voice that is unwatermarked — or that is protected with a different, unseen watermarking system — receives no protection. The attacker simply needs to avoid using an AudioSeal-watermarked prompt.

The evaluation is conducted only on the VALL-E architecture and EnCodec backbone; whether the approach generalizes to other zero-shot TTS architectures (e.g., flow-matching or diffusion-based systems) is not tested. All data is clean studio speech (LibriSpeech/LibriTTS-R); robustness in noisy or spontaneous speech conditions is unknown. The speed adjustment and low-pass filter attacks are reported as failures for the codec detector, though the authors argue these attacks also degrade clean prompts, partially neutralising them as practical bypass routes. No listening test compares watermark-rejected output against human expectations of what a "protection failure" looks and sounds like.

## Wiki Connections

This paper builds on [[2301.02111]] (VALL-E) as the target TTS system for its codec-level defense. Relevant concept pages: [[zero-shot-tts]], [[neural-codec]], [[autoregressive-codec-tts]], [[voice-conversion]], [[evaluation-metrics]].
