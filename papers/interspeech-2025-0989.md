---
id: interspeech-2025-0989
title: "HiFiTTS-2: A Large-Scale High Bandwidth Speech Dataset"
authors: ["Ryan Langman", "Xuesong Yang", "Paarth Neekhara", "Shehzeen Hussain", "Edresson Casanova", "Evelina Bakhturina", "Jason Li"]
organization: NVIDIA
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: "2025-08-17"
ingested_date: "2026-05-30"
task: [TTS, evaluation]
architecture: [autoregressive-LM]
conditioning: [text-conditioned, zero-shot, speaker-conditioned]
training: [supervised]
model_size: "Koel-TTS: ~378M params (validation model)"
codec_used: "not reported (Koel-TTS uses audio tokens; codec unspecified in this paper)"
datasets_train: ["HiFiTTS-2 (22 kHz subset: 36.7k hrs, 5,013 speakers; 44.1 kHz subset: 31.7k hrs, 4,631 speakers)", "LibriVox (source)", "MLS English (processing input)"]
datasets_eval: ["LibriTTS test-clean (unseen speakers)", "LibriTTS train-clean-360 held-out (seen speakers)"]
metrics:
  - name: WER
    value: 1.21
    system: Koel-TTS trained on HiFiTTS-2
    testset: LibriTTS seen speakers
  - name: WER
    value: 1.42
    system: Koel-TTS trained on HiFiTTS-2
    testset: LibriTTS unseen speakers
  - name: SPK-SIM
    value: 0.714
    system: Koel-TTS trained on HiFiTTS-2 (SSIM)
    testset: LibriTTS seen speakers
  - name: SPK-SIM
    value: 0.731
    system: Koel-TTS trained on HiFiTTS-2 (SSIM)
    testset: LibriTTS unseen speakers
  - name: WER
    value: 1.72
    system: Koel-TTS trained on LibriTTS+HiFiTTS
    testset: LibriTTS seen speakers
  - name: WER
    value: 1.67
    system: Koel-TTS trained on LibriTTS+HiFiTTS
    testset: LibriTTS unseen speakers
code_available: null
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/langman25_interspeech.html"
related_concepts: [zero-shot-tts, autoregressive-codec-tts, neural-codec, evaluation-metrics]
related_papers: []
---

# HiFiTTS-2: A Large-Scale High Bandwidth Speech Dataset

**Paper:** [https://www.isca-archive.org/interspeech_2025/langman25_interspeech.html](https://www.isca-archive.org/interspeech_2025/langman25_interspeech.html)

**One-sentence contribution:** HiFiTTS-2 is a large-scale, commercially usable English speech dataset of 36.7k hours at 22.05 kHz (31.7k hours at 44.1 kHz) derived from LibriVox, enabling high-bandwidth zero-shot TTS training with substantially improved speaker similarity compared to prior datasets of comparable or larger scale.

## Problem

Modern zero-shot TTS systems (e.g., VALL-E family, VoiceCraft, Koel-TTS) are typically trained on large volumes of 16 kHz audio, which limits output quality to telephone-grade bandwidth. High-bandwidth datasets suitable for 22 kHz or 44 kHz training are either small (HiFiTTS: 292 hrs, 10 speakers), insufficiently diverse for zero-shot generalization (LJSpeech: 1 speaker), or not commercially licensable (Emilia, CommonVoice subsets). Existing large datasets (MLS English: 44.7k hrs) are stored at 16 kHz regardless of original recording bandwidth, include mixed-bandwidth audio that degrades codec and vocoder training, and lack punctuation/capitalization in transcripts — a significant deficit for TTS prosody modeling. There is no large-scale, high-bandwidth, commercially licensed dataset designed with TTS-specific pipeline requirements.

## Method

HiFiTTS-2 is built by processing the English subset of MLS (44.7k hrs from LibriVox, stored at 16 kHz) using a five-stage pipeline applied to the original 48 kHz LibriVox audio files:

**1. Text Preprocessing:** Punctuation and capitalization (PC) are restored either by matching MLS transcripts to original audiobook text (successful for 87%) or by predicting PC using NeMo DistilBERT (remaining 13%). HTML tags, formatting artifacts (e.g., "nbsp", "p p"), and abbreviations are normalized using NeMo text normalization.

**2. Audio Processing:** Original 48 kHz LibriVox FLAC files are downloaded, downsampled to 44.1 kHz, and energy-based silence trimming is applied (threshold: -50 dB, leaving ≤0.5s silence at boundaries).

**3. Bandwidth Estimation:** Following HiFiTTS methodology, the maximum frequency with ≥-50 dB power relative to peak is estimated from the first 30 seconds of each audiobook. For the 22.05 kHz subset, utterances with estimated bandwidth ≥11 kHz are retained (36.7k hrs). For 44.1 kHz, utterances with bandwidth ≥13 kHz are retained (31.7k hrs), effectively excluding all audio originally recorded at ≤24 kHz sampling rate. Estimated bandwidth metadata is released per utterance.

**4. Segmentation:** MLS utterances (10–20 s) are split at natural pause boundaries (period + ≥0.08s pause, excluding abbreviations) using NeMo Forced Aligner with Parakeet-TDT-CTC. The longest qualifying pause is selected per utterance, yielding a bell-curve duration distribution.

**5. Text Validation and Speaker Counting:** WER/CER are computed via Parakeet-TDT-CTC ASR; utterances with CER=100% (no transcription match or silence) are filtered. Speaker diarization using Sortformer identifies ~104.4 hrs of multi-speaker segments, tagged as metadata. Both WER/CER and speaker count are released as metadata for user-controlled filtering.

**Dataset statistics:** 22 kHz subset: 36.7k hrs, 13.1M utterances, 5,013 speakers. 44.1 kHz subset: 31.7k hrs, 11.3M utterances, 4,631 speakers (strict subset of 22 kHz). Commercial use permitted (public domain LibriVox source, speakers requesting exclusion removed).

**Experiments:** Koel-TTS (~378M params, AR transformer decoder with NAR encoder) is trained on individual and combined datasets. Training triplets: (5s context audio, transcript, target audio) from the same speaker; HiFiTTS-2 triplets further filtered to CER < 3% and speaker similarity > 0.6, yielding 30,400 hrs. Evaluated on LibriTTS seen/unseen speaker splits using CER, WER (Parakeet-TDT), SSIM (TitaNet-Small speaker embeddings), and SQUIM-MOS.

## Key Results

Koel-TTS trained on HiFiTTS-2 (30,400 hrs, 4,940 speakers) substantially outperforms models trained on LibriTTS (539 hrs) and HiFiTTS (283 hrs) for speaker similarity on unseen speakers: SSIM 0.731 vs. 0.494 (LibriTTS) and 0.059 (HiFiTTS alone). Intelligibility (WER) also improves: 1.42% (HiFiTTS-2) vs. 2.48% (LibriTTS) for unseen speakers. Combining all three datasets yields the best unseen-speaker WER (1.39%) and SSIM (0.739). SQUIM-MOS is similar across all conditions (4.38–4.46). The model trained solely on HiFiTTS (10 speakers) fails badly on unseen speakers (WER 10.38%, SSIM 0.059), confirming that speaker diversity rather than audio quality alone drives zero-shot performance.

## Novelty Assessment

The primary contribution is the dataset and processing pipeline, not a new model. The scale (36.7k hrs at 22 kHz, comparable to MLS English but at high bandwidth) is the main differentiator. The bandwidth estimation + multi-speaker detection pipeline is engineering-focused but carefully designed. The commercial licensing (LibriVox public domain source) is a meaningful practical advantage over Emilia and other large-scale datasets. The experiments with Koel-TTS convincingly demonstrate that scale and speaker diversity from HiFiTTS-2 substantially improve zero-shot speaker similarity, with the improvement attributable to scale + diversity rather than audio quality alone.

## Limitations and Open Questions

The dataset is English-only. Audio quality is not filtered by SNR, accepting noise present in LibriVox recordings; this is intentional (modern TTS can handle noise) but may affect some applications. The 44.1 kHz subset's bandwidth ranges from 13–22 kHz (mixed-bandwidth within the subset), which may complicate training for systems requiring uniform bandwidth. No listening-test based MOS evaluation is reported; evaluation relies on automatic metrics (SQUIM-MOS, SSIM). The Koel-TTS codec details are not specified in this paper. Future work on non-English and non-audiobook sources is mentioned but not pursued.

## Wiki Connections

HiFiTTS-2 directly advances [[zero-shot-tts]] by providing the large-scale, speaker-diverse training data that zero-shot systems require. The use of Koel-TTS as the validation model connects to [[autoregressive-codec-tts]]. The bandwidth estimation methodology and discussion of mixed-bandwidth problems is relevant to [[neural-codec]] training. The dataset pipeline design (punctuation restoration, speaker diarization, text validation) offers a model for future large-scale dataset construction. The comparison table (VCTK: 44 hrs, HiFiTTS: 292 hrs, LibriTTS: 586 hrs, MLS: 44.7k hrs, HiFiTTS-2: 36.7k hrs) is useful for [[evaluation-metrics]] discussions about training data requirements.
