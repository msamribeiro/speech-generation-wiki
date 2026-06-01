---
id: interspeech-2025-0596
title: "Facilitating Personalized TTS for Dysarthric Speakers Using Knowledge Anchoring and Curriculum Learning"
authors: [Yejin Jeon, Solee Im, Youngjae Kim, Gary Geunbae Lee]
organization: POSTECH
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-05-30
task: [TTS]
architecture: [transformer-enc-dec]
conditioning: [speaker-conditioned, zero-shot]
training: [supervised, distillation]
model_size: "33M params"
codec_used: "none"
datasets_train: [LibriSpeech]
datasets_eval: [UASpeech]
metrics:
  - name: MOS
    value: 3.601
    system: proposed
    testset: UASpeech (MOS-Nat, 19 AMT workers)
  - name: MOS
    value: 3.909
    system: proposed
    testset: UASpeech (MOS-Spk, 19 AMT workers)
  - name: MOS
    value: 3.371
    system: Hybrid (FastPitch + FreeVC)
    testset: UASpeech (MOS-Nat)
  - name: MOS
    value: 3.731
    system: Hybrid (FastPitch + FreeVC)
    testset: UASpeech (MOS-Spk)
code_available: null
demo_available: null
url: https://www.isca-archive.org/interspeech_2025/jeon25_interspeech.html
related_concepts: [zero-shot-tts, speaker-adaptation, disentanglement]
related_papers: []
---
> [!abstract] Interspeech · 2025 · Conference
> **Yejin Jeon et al.** (POSTECH) · [→ Paper](https://www.isca-archive.org/interspeech_2025/jeon25_interspeech.html) · Demo: ? · Code: ?
>
> A teacher-student knowledge anchoring framework with curriculum learning enables zero-shot multi-speaker TTS to generate intelligible, speaker-consistent speech from short dysarthric reference audio, achieving a 50+ point reduction in phoneme error rate over previous baselines.

## Problem

Personalized TTS for dysarthric speakers faces two compounding difficulties. First, dysarthric speech contains articulation errors (slurring, incorrect phoneme production) that, if used as training data directly, cause models to reproduce those errors in synthesis. Second, dysarthric speakers can typically only produce very short utterances (single words) due to the physical demands of speech production, whereas standard TTS training relies on long, well-articulated sentences. This creates a domain gap between training data (normal speech) and inference conditions (short, dysarthric reference audio), both in articulation quality and in utterance duration. Prior hybrid approaches (single-speaker TTS followed by voice conversion) address intelligibility at the cost of speaker identity.

## Method

The system uses a FastSpeech2 backbone acoustic model with HiFi-GAN vocoder for mel-to-waveform conversion. The key innovation is in the speaker encoder, which is split into a teacher-student pair:

**Knowledge anchoring.** Both teacher (θ_t) and student (θ_s) speaker encoders share an identical architecture: a pretrained speaker verification network computes per-utterance embeddings, which are averaged into a speaker centroid, then passed through a linear projection, FC block with Mish activation, and a GLU-gated convolution stack. The teacher receives full-length reference audio and produces a speaker representation that conditions the backbone TTS (text encoder and decoder via adaptive instance normalization). The student receives shorter (cropped) audio and is trained to match the teacher's output via an MAE loss (L_MAE between teacher and student style vectors). The teacher thus acts as an anchor: the student learns to extract timbre-specific features robust to articulation distortions, while being guided by the cleaner teacher representation as a filter.

**Curriculum learning.** The student is progressively trained on shorter audio segments over the course of training. With C=3 cropping stages over S=500,000 steps, each stage of ~160,000 steps trains on audio cropped to (1 - k/(C+1)) of the full length, transitioning from 75% to 50% to 25% of the full reference length. This gradual exposure bridges the gap from training-time long utterances to inference-time single-word inputs. Random cropping is shown to be inferior to progressive cropping in ablations. At inference, only the student encoder is used.

The total training loss combines mel reconstruction MAE and the teacher-student MAE loss on style vectors. All training uses LibriSpeech (normal speech); zero-shot synthesis is then applied to UASpeech dysarthric recordings.

## Key Results

Evaluated on UASpeech against three baselines: Adaptive (style prototype/episodic training), Conditional (pretrained speaker verification encoder), and Hybrid (FastPitch + FreeVC).

The proposed method achieves PER 14.254% vs. 31.017% for the best baseline (Hybrid), a >50% relative reduction. Speaker similarity is 0.619 (comparable to Conditional at 0.647, better than Hybrid at 0.534). MOS-Nat: 3.601 vs. 3.371 (Hybrid); MOS-Spk: 3.909 vs. 3.731 (Hybrid).

Across dysarthric severity groups (Very Low / Low / Middle / High intelligibility), the proposed model consistently achieves the lowest PER and comparable or superior speaker similarity. The benefit is most pronounced for Low intelligibility speakers (PER drops from ~33 to ~15).

Ablation confirms both knowledge anchoring and curriculum learning are necessary: removing the student model raises PER from 14.254 to 22.846; removing curriculum learning raises it further to 26.428.

## Novelty Assessment

The application of teacher-student knowledge distillation to speaker encoder learning for dysarthric TTS is new. The framing of dysarthric TTS as a dual domain-transfer problem (articulation distortion + utterance length mismatch) is clear and well-motivated. The curriculum learning strategy — progressive shortening of student inputs — is a practical and effective solution to the length mismatch problem. The backbone (FastSpeech2 + HiFi-GAN + speaker conditioning) is standard; the novelty lies entirely in the training strategy for the speaker encoder. The 33M-parameter model is compact and trains on a single GPU.

## Limitations and Open Questions

The system is English-only. The backbone is non-autoregressive (FastSpeech2) with mel-spectrogram output, which imposes a quality ceiling compared to codec-based or flow-matching systems. The approach requires labeled phoneme data to compute PER, which is available for UASpeech but may not generalize. Speaker similarity (0.619) remains below what might be needed for truly personalized assistive use. The method has not been tested on languages with non-Latin scripts or very different phoneme inventories. Future work could explore curriculum learning with codec-based or diffusion-based TTS backbones.

## Wiki Connections

This paper extends [[zero-shot-tts]] into the assistive technology domain, where the input reference audio is pathological rather than normal. The knowledge anchoring approach is related to the [[disentanglement]] concept — separating timbre from articulation quality. The speaker adaptation concern (learning a robust speaker representation from minimal data) connects to [[speaker-adaptation]]. It builds on prior POSTECH work on zero-shot multi-speaker TTS with negated speaker representations (Jeon et al., AAAI 2024, not yet in corpus).
