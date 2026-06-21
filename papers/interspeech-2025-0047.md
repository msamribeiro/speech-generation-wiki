---
id: "interspeech-2025-0047"
title: "Revival with Voice: Multi-modal Controllable Text-to-Speech Synthesis"
authors: ["Minsu Kim", "Pingchuan Ma", "Honglie Chen", "Stavros Petridis", "Maja Pantic"]
organization: Meta AI
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: "2025-08-17"
ingested_date: "2026-05-30"
task: [TTS]
architecture: [autoregressive-LM]
conditioning: [speaker-conditioned, instruction-conditioned, zero-shot]
training: [supervised, self-supervised]
model_size: "not reported"
codec_used: "DAC / RVQ (9-level, codebook 1024)"
datasets_train: ["LRS3", "VoxCeleb2", "LibriTTS-R"]
datasets_eval: ["LRS3 test set", "artistic portraits (20 copyright-free images)"]
metrics:
  - name: MOS
    value: 4.14
    system: RV-TTS (proposed)
    testset: LRS3 + artistic portraits
  - name: MOS
    value: 1.84
    system: FaceTTS (baseline)
    testset: LRS3 + artistic portraits
  - name: MOS
    value: 2.78
    system: YourTTS (baseline)
    testset: LRS3 + artistic portraits
code_available: null
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/kim25b_interspeech.html"
related_concepts: [zero-shot-tts, instruction-conditioned-tts, prosody-control, speaker-adaptation]
related_papers: ["2301.02111"]
field_significance:
  level: "moderate"
  type: ["engineering-integration"]
generation:
  date: 2026-06-21
  agent: speech-generation-review-agent
  model: claude-sonnet-4-6
  commit: "3538db5"
---
> [!abstract] Interspeech · 2025 · Conference
> **Minsu Kim et al.** (Meta AI) · [→ Paper](https://www.isca-archive.org/interspeech_2025/kim25b_interspeech.html) · Demo: ? · Code: ?
>
> RV-TTS generates speech whose voice is inferred from a face image (including artistic portraits) while allowing fine-grained control of speech characteristics via natural descriptive text, addressing the triple gap of low audio quality, domain mismatch between real faces and artwork, and inconsistent voice generation.

## Problem

Face-driven TTS systems suffer from three interrelated problems: (1) publicly available audio-visual corpora (LRS3, VoxCeleb2) contain noisy, reverberant recordings incompatible with high-quality TTS training; (2) models trained on photorealistic face images degrade when applied to portraits or old photographs; and (3) face-to-voice mapping is inherently one-to-many, but prior methods do not model this ambiguity and produce inconsistent voice outputs across inferences with the same face.

## Method

RV-TTS is a speech language model based on the MusicGen architecture — a 24-layer Transformer with cross-attention, trained to auto-regressively predict 9-level RVQ codes (codebook size 1024). Three conditioning streams are fused at inference:

1. **Voice conditioning (face image):** A ResNet50 face encoder (initialized from ArcFace) is contrastively pre-trained against an ECAPA-TDNN audio encoder using InfoNCE loss on VoxCeleb2, so that face embeddings and audio embeddings share a 256-dimensional latent space. During TTS training, the model is alternated between face-driven and audio-driven voice embeddings depending on whether the sample comes from an audio-visual or audio-only dataset. This allows LibriTTS-R (clean) to be mixed into training without requiring paired face data.

2. **Style augmentation:** During training, input face images are randomly passed through CAST neural style transfer and augmented with gray-scaling and blurring (50% chance). This reduces the train/test domain gap when applying the model to portraits or old photographs.

3. **Descriptive text conditioning (speech characteristics):** A pre-trained T5 LM encodes natural descriptive text (pace, noise level, distance, tone, place). Labels for LibriTTS-R are adapted from Parler-TTS with gender-specific words neutralized; labels for LRS3 and VoxCeleb2 are generated via Data-Speech.

For diverse but consistent voice generation, the model uses sampling-based decoding to produce multiple candidate voices from the same face, then accepts a user-selected sample as an in-context prompt to enforce voice consistency across subsequent utterances.

## Key Results

On LRS3 and artistic portraits (human evaluation, 30–78 samples):

| Method | MOS | FMS | VCS |
|--------|-----|-----|-----|
| FaceTTS | 1.84 | 2.31 | 2.85 |
| FVTTS | 1.60 | 2.38 | 3.45 |
| YourTTS (audio-driven) | 2.78 | 3.09 | 4.42 |
| Parler-TTS (text-driven) | 3.68 | 1.88 | — |
| **RV-TTS** | **4.14** | **3.86** | 3.96 |

RV-TTS achieves the highest MOS (+2.30 vs. FaceTTS) and FMS (+1.45 vs. FaceTTS). On a speaker identification test (78 samples, 15 raters), RV-TTS achieves 76% accuracy on LRS3 and 73% on artistic portraits.

Ablation confirms that mixing high-quality LibriTTS-R audio data contributes the largest MOS gain (+0.50), followed by style augmentation (+0.26 in MOS), with contrastive learning primarily benefiting face-voice association (SIM metric) rather than naturalness.

Controllability is verified using objective metrics (speaking rate, SI-SDR, C50, pitch std) that shift predictably with descriptive text changes.

## Novelty Assessment

The combination of face-driven voice synthesis with natural-language style control is novel and practically motivated (historical figure revival). The key technical contributions — alternating audio-visual/audio-only embeddings via contrastive alignment, face-image style augmentation with neural transfer — are straightforward but effective engineering choices rather than architectural innovations. The base speech LM (MusicGen-style) and RVQ codec are prior work. The speaker identification test is a stronger and more realistic evaluation than typical MOS-only reporting for face-driven TTS.

## Field Significance

Moderate — RV-TTS demonstrates that combining audio-visual corpora with high-quality audio-only data via contrastive modality alignment is an effective strategy for closing the speech quality gap in face-driven TTS. The paper's primary contribution is engineering integration: established components (MusicGen, RVQ, InfoNCE contrastive learning, neural style transfer) are assembled into a coherent system, with the artistic portrait application providing a plausible and underexplored use case.

## Claims

- Mixing high-quality audio-only data with audio-visual corpora via contrastive modality alignment substantially improves speech naturalness in face-driven TTS systems. *(§2.3, Table 1)*
- Neural style transfer augmentation of training face images reduces the domain gap between photorealistic faces and artistic portraits, improving both naturalness and face-matching scores. *(§2.2, Table 1)*
- In-context prompting with a user-selected speech sample enables consistent voice generation across multiple utterances from the same face, addressing the inherent one-to-many ambiguity of face-to-voice mapping. *(§2.4)*
- Contrastive pre-training of face and audio encoders primarily improves face-voice association rather than overall speech naturalness. *(§3.4.1, Table 1)*
- Objective acoustic metrics (speaking rate, SI-SDR, C50, pitch standard deviation) shift predictably with natural language descriptors, confirming that text-driven style control transfers to measurable acoustic properties. *(§3.4.3, Table 3)*

## Limitations and Open Questions

- Voice consistency score (VCS) for RV-TTS (3.96) is below the audio-driven YourTTS (4.42), indicating residual inconsistency in pure face-conditioned generation.
- The artistic portrait test uses only 20 copyright-free images; broader and more controlled evaluation is needed.
- Descriptive text label generation for LRS3/VoxCeleb2 relies on Data-Speech auto-annotation rather than human labels.
- Face encoder is ResNet50; more powerful vision-language models may improve face-voice matching.
- No multilingual evaluation.

## Wiki Connections

- [[zero-shot-tts]] — RV-TTS clones voice from a face image reference, which functions analogously to audio-prompt zero-shot cloning
- [[instruction-conditioned-tts]] — natural descriptive text controls pace, noise, distance, tone, and place at inference time
- [[speaker-adaptation]] — contrastive alignment between face and audio encoders is the core adaptation mechanism
- [[prosody-control]] — text-driven controllability is verified with objective prosody metrics (pitch std, speaking rate)
- [[2301.02111|VALL-E]] — in-context learning via audio prompting, on which RV-TTS's voice consistency mechanism directly builds
