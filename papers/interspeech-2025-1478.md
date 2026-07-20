---
id: "interspeech-2025-1478"
title: "LightL2S: Ultra-Low Complexity Lip-to-Speech Synthesis for Multi-Speaker Scenarios"
authors: ["Yifan Liang", "Kang Yang", "Fangkun Liu", "Andong Li", "Xiaodong Li", "Chengshi Zheng"]
organization: null
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: "2025-08-17"
ingested_date: "2026-07-03"
task: ["TTS"]
architecture: ["hybrid"]
conditioning: ["speaker-conditioned"]
training: ["supervised"]
model_size: "not reported"
codec_used: "none"
datasets_train: ["LRS3"]
datasets_eval: ["LRS3"]
metrics:
  - name: UTMOS
    value: 2.93
    system: proposed
    testset: LRS3
  - name: WER
    value: 64.8
    system: proposed
    testset: LRS3
  - name: SECS
    value: 0.72
    system: proposed
    testset: LRS3
code_available: null
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/liang25d_interspeech.html"
related_concepts: [self-supervised-speech, evaluation-metrics, subjective-evaluation, speaker-adaptation]
related_papers: []
field_significance:
  level: "moderate"
  type: ["engineering-integration"]
generation:
  date: "2026-07-03"
  agent: speech-generation-ingest-agent
  model: claude-sonnet-4-6
  commit: "f9e71e3"
---

> [!abstract] Interspeech · 2025 · Conference
> **Yifan Liang et al.** · [→ Paper](https://www.isca-archive.org/interspeech_2025/liang25d_interspeech.html) · Demo: ? · Code: ?
>
> LightL2S proposes an ultra-low-complexity lip-to-speech synthesis system for edge devices, combining MoViNet visual encoding, Zipformer sequence modelling, and a DDSP vocoder to achieve 0.8 GMacs — roughly 40x below the computational cost of AV-HuBERT-based methods — without requiring text supervision.

**Note on task tagging:** Lip-to-speech synthesis converts silent video (lip movements) to speech; the input modality is visual, not text. The canonical vocabulary does not include a dedicated term for this task, so it is tagged `TTS` for lack of a better fit. This is the same vocabulary gap flagged for interspeech-2025-1334; see review_flags below.

## Problem

Lip-to-speech synthesis systems competitive with the state of the art rely heavily on AV-HuBERT, a large audio-visual self-supervised model whose visual encoder is a ResNet-18 with Transformer blocks. AV-HuBERT-based methods such as IntelligibleL2S and NaturalL2S require 32–34 GMacs to process one second of video, placing them far outside the budget of edge devices. Earlier systems without AV-HuBERT (VCA-GAN, Multi-task) are cheaper but score substantially lower on naturalness and intelligibility. No prior work has demonstrated lip-to-speech synthesis in unconstrained, multi-speaker conditions below 1 GMac.

## Method

LightL2S is a three-stage pipeline: a lightweight visual frontend, a Zipformer-based sequence backbone, and a DDSP vocoder, trained end-to-end without paired text annotations.

The visual frontend is MoViNet-A0, a mobile video recognition network that replaces the ResNet-18 typically used in lip-to-speech systems. MoViNet-A0 uses 3D convolutions with an architecture optimised by neural architecture search (TuNAS), processing 88x88 grayscale mouth-region crops at 25 Hz and projecting to a 192-dimensional feature sequence.

The Zipformer backbone (using the Zipformer-S configuration with 6 stacks of 2 blocks) replaces the Conformer used in prior systems. Zipformer adopts a U-Net-like structure that downsamples intermediate frame rates, enabling multi-scale temporal modelling. Two outputs are produced jointly: (1) discrete SSL units, obtained by predicting K-means cluster assignments (200 clusters, derived from the sixth layer of HuBERT-BASE) via a cross-entropy loss with label smoothing; and (2) a fundamental frequency (F0) sequence at 100 Hz, predicted through a convolution layer conditioned on a reference speaker embedding and supervised with an L1 loss. Speaker identity is incorporated via a pre-extracted speaker embedding (ESPnet) injected as a conditioning signal.

The DDSP vocoder receives the predicted F0, speaker embedding, and decoded SSL-unit representations to generate speech through an explicit source-filter decomposition: a harmonic excitation signal is synthesised from F0, combined with a Gaussian noise aperiodic component, and passed through Conformer blocks that estimate harmonic amplitudes, harmonic phases, noise magnitude, and noise phase parameters. The vocoder is trained jointly with the rest of the system using a multi-resolution STFT loss (six window sizes from 64 to 2048 samples) and an adversarial loss from six multi-resolution spectrogram discriminators.

Training is done on LRS3 (400+ hours of TED/TEDx video) in two stages: 80 epochs without the discriminator, followed by 20 epochs with discriminator fine-tuning, on a single A100 40 GB GPU using BF16 mixed precision.

## Key Results

On the LRS3 test set (Table 1), LightL2S achieves 0.8 GMacs — far below all baselines (VCA-GAN 29.6, Multi-task 14.6, IntelligibleL2S 32.3, NaturalL2S 34.2 GMacs). Despite this reduction, it reaches UTMOS 2.93, exceeding both VCA-GAN (1.31) and IntelligibleL2S (2.70), though below NaturalL2S (3.66). The intelligibility cost is substantial: WER is 64.8% versus 27.7% for IntelligibleL2S and 30.4% for NaturalL2S; only methods that lack AV-HuBERT score comparably or worse on WER. Audio-visual synchronisation (LSE-C 8.01) and speaker similarity (SECS 0.72) are competitive.

In the 10-participant subjective listening test (Table 2), LightL2S achieves naturalness MOS 3.63 ± 0.45 (between IntelligibleL2S 3.29 and DiffV2S 3.69) and the highest speaker similarity MOS among all methods (3.72 ± 0.25). Intelligibility MOS is 3.20 ± 0.34, below NaturalL2S (3.98) and IntelligibleL2S (3.72). The ablation (Table 3) shows the spectrogram discriminator is critical: removing it collapses UTMOS from 2.93 to 1.58. Replacing Zipformer with a Conformer backbone raises GMacs from 0.80 to 1.09 while worsening UTMOS (2.74) and WER (66.3%).

## Novelty Assessment

The contribution is primarily engineering integration: MoViNet, Zipformer, and DDSP are all established methods from adjacent fields (video recognition, ASR, and signal-processing-based vocoding respectively), adapted here for the first time as a unified, edge-deployable lip-to-speech pipeline. The DDSP vocoder for lip-to-speech was introduced in the concurrent NaturalL2S system by the same first author, so LightL2S is best understood as an efficiency variant of that framework. The SSL-unit supervision approach (using HuBERT K-means targets) follows ReVISE and IntelligibleL2S. Genuine novelty lies in the specific combination of MoViNet-A0 and Zipformer for sub-1-GMac operation, which is the first demonstration in this task. The intelligibility gap versus AV-HuBERT-based systems (WER 64.8% vs. 27–35%) is a real and unresolved cost of removing the large pretrained visual-acoustic model.

## Field Significance

Moderate — LightL2S demonstrates that lip-to-speech synthesis in unconstrained multi-speaker conditions can be made viable at edge-device complexity by substituting heavyweight SSL visual encoders with mobile video networks, and Conformer backbones with Zipformer. The efficiency reduction is dramatic, and the subjective naturalness and speaker similarity remain competitive. The approach primarily shows what can be sacrificed (intelligibility) versus maintained (naturalness, speaker preservation) under tight resource constraints, providing useful empirical evidence for this design trade-off space in lip-to-speech research.

## Claims

- **supports:** Replacing ResNet-18/AV-HuBERT visual encoders with mobile video networks reduces lip-to-speech system complexity by an order of magnitude at the cost of intelligibility degradation.
  > *Evidence:* LightL2S uses MoViNet-A0 instead of ResNet-18-based AV-HuBERT, reducing inference from 32–34 GMacs to 0.8 GMacs on LRS3 while WER increases from 27–30% (AV-HuBERT-based methods) to 64.8%. *(§4.1, Table 1)*

- **supports:** Multi-resolution spectrogram discriminators substantially improve speech naturalness in DDSP-based lip-to-speech synthesis, beyond spectral regression alone.
  > *Evidence:* Removing the adversarial GAN loss from LightL2S collapses UTMOS from 2.93 to 1.58 and SECS from 0.72 to 0.63 on LRS3, while computational cost remains identical at 0.8 GMacs. *(§4.3, Table 3)*

- **complicates:** Objective quality metrics can diverge from human perceptual judgments in lip-to-speech synthesis, making WER and UTMOS insufficient as sole quality signals.
  > *Evidence:* NaturalL2S exceeds Ground Truth on UTMOS (3.66 vs. 3.59) but scores lower in subjective naturalness MOS (4.10 vs. 4.51); LightL2S achieves the highest speaker similarity MOS (3.72) despite being outranked on SECS by several baselines. *(§4.2, Table 2)*

- **refines:** Efficient transformer variants (Zipformer) can substitute standard Conformer backbones in visual speech modelling with simultaneous improvements in quality and computational cost.
  > *Evidence:* Replacing Zipformer with a Conformer backbone in LightL2S increases GMacs from 0.80 to 1.09 while reducing UTMOS from 2.93 to 2.74 and raising WER from 64.8% to 66.3% on LRS3. *(§4.3, Table 3)*

## Limitations and Open Questions

> [!warning]
> Intelligibility remains a major limitation: LightL2S achieves WER 64.8% on LRS3, more than double the 27–30% of AV-HuBERT-based methods. This gap is attributed to the absence of large-scale pre-trained visual-acoustic features and the lack of text supervision, and is explicitly left as an open problem. At this WER level, practical deployment for communication assistance — the paper's stated motivation — requires further work.

Evaluation is confined to English TED/TEDx video from LRS3; generalisation to other languages, accents, spontaneous speech, and noisy real-world conditions is untested. The system uses a reference speaker embedding extracted at inference time, meaning an enrolment clip must be available, which may be impractical for truly in-the-wild edge scenarios. Parameter count is not reported, making direct comparison with published efficient TTS systems difficult.

## Wiki Connections

- [[self-supervised-speech|Self-Supervised Speech]] — the system uses K-means clusters from HuBERT-BASE (layer 6) as the primary content supervision signal during training, removing the need for text annotation.
- [[evaluation-metrics|Evaluation Metrics]] — the paper evaluates across UTMOS, WER, SECS, and LSE-C, illustrating the multidimensional trade-offs specific to lip-to-speech synthesis.
- [[subjective-evaluation|Subjective Evaluation]] — a 10-participant listening test rates naturalness, intelligibility, and speaker similarity simultaneously, revealing discrepancies with objective rankings (particularly UTMOS vs. Ground Truth naturalness MOS).
- [[speaker-adaptation|Speaker Adaptation]] — multi-speaker synthesis is enabled via a reference speaker embedding injected into the F0 predictor and DDSP vocoder, achieving the highest speaker similarity MOS among compared methods.
