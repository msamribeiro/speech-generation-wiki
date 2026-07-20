---
id: "interspeech-2025-1081"
title: "Speaker Normalization and Content Restoration for Zero-Shot Voice Conversion with Attention-Enhanced Discriminator"
authors: ["Desheng Hu", "Yang Xiang", "Jian Lu", "Xinhui Hu", "Xinkang Xu"]
organization: "Hithink RoyalFlush AI Research Institute"
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: "2025-08-17"
ingested_date: "2026-07-03"
task: [VC]
architecture: [GAN]
conditioning: [speaker-conditioned, zero-shot]
training: [supervised]
model_size: "50M params"
codec_used: "DAC encoder (content encoding only)"
datasets_train: ["LibriLight medium (~5k hours)"]
datasets_eval: ["VCTK"]
metrics:
  - name: MOS
    value: 3.86
    system: proposed
    testset: VCTK
  - name: SMOS
    value: 3.52
    system: proposed
    testset: VCTK
  - name: WER
    value: 3.78
    system: proposed
    testset: VCTK
  - name: SPK-SIM
    value: 0.25
    system: proposed
    testset: VCTK
code_available: null
demo_available: true
url: "https://www.isca-archive.org/interspeech_2025/hu25i_interspeech.html"
related_concepts: [voice-conversion, disentanglement, zero-shot-tts, gan-vocoder, evaluation-metrics, subjective-evaluation]
related_papers: ["2406.02430"]
field_significance:
  level: moderate
  type: [architectural-novelty]
generation:
  date: "2026-07-03"
  agent: speech-generation-ingest-agent
  model: claude-sonnet-4-6
  commit: "d8ecfd4"
---

> [!abstract] Interspeech · 2025 · Conference
> **Hu et al.** (Hithink RoyalFlush AI Research Institute) · [→ Paper](https://www.isca-archive.org/interspeech_2025/hu25i_interspeech.html) · Demo: ✓ · Code: ?
>
> A zero-shot voice conversion system that trains a content encoder from scratch using phoneme supervision with speaker normalization and content restoration (SNCR) modules, combined with a consistency-loss speaker encoder and an attention-enhanced GAN discriminator that integrates self-attention into a convolutional backbone.

## Problem

Zero-shot voice conversion requires cleanly disentangling linguistic content from speaker identity — a notoriously difficult problem. Supervised approaches based on phonetic posteriorgrams (PPGs) tend to produce neutral-sounding speech by discarding accent and expression. Unsupervised methods using information bottleneck, instance normalization, or vector quantization often degrade linguistic content while removing speaker traits. SSL-based approaches retain speaker information in their representations and require additional filtering passes, while their large parameter footprints slow inference. Meanwhile, CNN-based discriminators in GAN vocoders struggle to capture long-range spectral dependencies, limiting reconstruction quality. This paper addresses the content-purity, speaker-robustness, and generation-quality problems together without relying on SSL feature extractors.

## Method

The system uses a DAC encoder as the content encoder backbone, trained from scratch on a phoneme-level prediction task rather than initialized from a pretrained model. A pitch-shift transformation removes fundamental frequency speaker cues before the signal enters the encoder. After each DAC block, a Speaker Normalization and Content Restoration (SNCR) module applies instance normalization (IN) to strip speaker-dependent statistics, then recovers content-relevant information via a channel-attention mechanism. The SNCR module decomposes the residual between the pre- and post-IN features into content-relevant and content-irrelevant parts using a dual restoration loss based on phoneme-prediction entropy: the content-relevant branch is pushed toward lower prediction entropy (more discriminative), while the content-irrelevant branch is pushed toward higher entropy. This dual constraint encourages the encoder to retain precisely the phonetic information that IN erases.

Speaker embeddings are extracted by ECAPA-TDNN with a consistency loss: two random segments from the same utterance are encoded independently, and cosine similarity is maximized between them. This encourages the speaker encoder to produce stable, content-agnostic representations without relying on a pretrained speaker verification model or contrastive mining across utterances.

The decoder follows the Vocos generator architecture, reconstructing mel-spectrograms from the concatenated content, pitch, and speaker representations. A HiFi-GAN vocoder converts mel-spectrograms to waveforms. For adversarial training, an attention-enhanced discriminator extends the GANSpeech architecture by replacing its second and third convolutional layers with residual convolutional blocks followed by L2-distance self-attention layers. Using L2 distance in place of dot-product attention avoids Lipschitz violations that cause training collapse with standard self-attention in discriminators. The discriminator operates both unconditionally and conditionally on a speaker embedding.

The full training loss combines GAN loss, feature matching loss, mel reconstruction loss (L1), phoneme prediction loss, SNCR dual restoration loss, and speaker consistency loss.

![Figure 1: (a) The overall framework of our proposed model. L c denotes consistency loss for learning accurate and robust speaker representations. (b) SNCR module. The branch with a dashed line is only used for inference. (c) Attention-enhanced discriminator. SA denotes self-attention layers, and conv represents 1-D convolution.](assets/interspeech-2025-1081/figure-2.png)

## Key Results

On a zero-shot VC task converting to 20 unseen VCTK speakers (10 male, 10 female), the proposed method achieves nMOS 3.86, sMOS 3.52, WER 3.78%, and SECS 0.25. Compared to DDDM-VC (the strongest baseline), the system matches speaker similarity (SECS 0.25 vs. 0.27) while substantially improving naturalness (nMOS 3.86 vs. 3.52) and word error rate (WER 3.78% vs. 10.61%). The proposed system uses 50M parameters versus DDDM-VC's 366M and runs at RTF 0.11 versus 0.29 on an NVIDIA 3090 GPU, a 7x parameter reduction and approximately 2.6x speed improvement at comparable speaker similarity. (Table 1)

VQMIVC achieves competitive SECS (0.23) but at much higher WER (21.4%), likely because its training set (VCTK) overlaps with the test speakers, inflating similarity scores without reflecting genuine zero-shot generalization. (§3.2)

The ablation study shows each component contributes positively: removing the dual restoration loss increases WER by 1.03 points; removing both dual restoration loss and instance normalization further degrades sMOS by 0.69; removing consistency loss additionally reduces SECS; and replacing the attention-enhanced discriminator with a standard CNN discriminator reduces nMOS and increases WER. (Table 2)

## Novelty Assessment

The three core contributions each combine existing ideas in ways that are not individually new but work together effectively. Instance normalization for speaker removal is established (AGAIN-VC, related work). The idea of restoring content lost by normalization via a disentangled residual with an entropy-based loss is the most novel element, drawing inspiration from domain-generalization techniques (Jin et al., 2021) applied to the VC content-speaker disentanglement problem. The speaker consistency loss is straightforward (cosine similarity on random utterance segments), simpler than contrastive approaches but functional. The attention-enhanced discriminator addresses a real limitation of CNN-only discriminators; using L2-distance self-attention for Lipschitz stability is a meaningful design choice borrowed from the GAN image generation literature (Kang et al., 2023). Overall, this is solid applied engineering within the GAN-based VC paradigm rather than a new paradigm, but the combination delivers a favorable accuracy-efficiency trade-off over heavier diffusion-based competitors.

## Field Significance

Moderate — The system demonstrates that a from-scratch trained content encoder with phoneme supervision and an instance-normalization-based SNCR module can match the speaker similarity of a much larger SSL-dependent diffusion model (DDDM-VC) while running 2.6x faster with 7x fewer parameters. This provides evidence that GAN-based approaches remain competitive with diffusion for zero-shot VC on standard benchmarks, particularly when parameter efficiency is a concern. The SNCR module's dual restoration loss mechanism introduces a useful design pattern for recovering content information lost during speaker normalization that may generalize to other disentanglement contexts.

## Claims

- **supports:** Explicit content-restoration mechanisms can compensate for linguistic information degraded by instance normalization in speaker disentanglement, preserving intelligibility without SSL feature extractors.
  > *Evidence:* The SNCR dual restoration loss, which decomposes the IN residual into content-relevant and content-irrelevant parts via phoneme-entropy constraints, reduces WER from 5.57% (without SNCR) to 3.78% compared to a baseline trained with neither IN nor dual restoration loss. *(§2.1, Table 2)*

- **supports:** GAN-based zero-shot voice conversion systems can match the speaker similarity of diffusion-based systems at substantially lower parameter count and faster inference.
  > *Evidence:* The proposed 50M-parameter GAN system achieves SECS 0.25 versus DDDM-VC's SECS 0.27, while running at RTF 0.11 versus 0.29 and using 366M fewer parameters; naturalness (nMOS 3.86 vs. 3.52) also favors the GAN system. *(§3.2, Table 1)*

- **supports:** Integrating self-attention layers into CNN-based discriminators improves adversarial training quality for mel-spectrogram reconstruction in voice conversion.
  > *Evidence:* Ablation Base4 (discriminator without self-attention) degrades nMOS and WER compared to the full system, confirming that attention-enhanced discrimination contributes to speech quality gains beyond the convolutional baseline. *(§3.3, Table 2)*

- **complicates:** Speaker similarity metrics in zero-shot VC evaluations can be inflated when the test set speakers overlap with a baseline system's training set, making fair cross-system comparison difficult.
  > *Evidence:* VQMIVC achieves SECS 0.23 (close to the proposed 0.25) but was trained on VCTK, which overlaps with the VCTK test speakers; its WER of 21.4% reveals its conversions are not actually intelligible, suggesting its SECS reflects in-distribution behavior rather than genuine zero-shot capability. *(§3.2)*

## Limitations and Open Questions

The system trains and evaluates on English only (LibriLight and VCTK). Whether the SNCR mechanism generalizes to cross-lingual voice conversion or languages with different phonological properties is untested. The ablation evaluates components one at a time, leaving interactions between SNCR and consistency loss unexplored. The mel-spectrogram reconstruction approach limits the system to fixed-rate synthesis, and no streaming or low-latency deployment scenario is discussed. Inference speed is reported on GPU only; embedded or CPU deployment characteristics are not assessed.

## Wiki Connections

- [[voice-conversion|Voice Conversion]] — proposes a complete zero-shot VC system targeting the content-speaker disentanglement problem with a from-scratch trained encoder.
- [[disentanglement|Disentanglement]] — introduces the SNCR module as an explicit mechanism for decomposing instance-normalized residual features into content-relevant and content-irrelevant components.
- [[zero-shot-tts|Zero-Shot TTS]] — addresses zero-shot generalization to unseen speakers at inference time using a reference embedding, bridging to zero-shot synthesis paradigms.
- [[gan-vocoder|GAN Vocoder]] — uses GAN adversarial training with an attention-enhanced discriminator extending the GANSpeech architecture for high-fidelity mel-spectrogram generation.
- [[evaluation-metrics|Evaluation Metrics]] — reports nMOS, sMOS, WER, SECS, and RTF across compared systems, providing a multi-dimensional view of zero-shot VC quality.
- [[subjective-evaluation|Subjective Evaluation]] — uses MOS listening tests (naturalness and speaker similarity) to compare the proposed method against three baseline systems with 95% confidence intervals.
- [[2406.02430|Seed-TTS]] — cited as a representative high-quality speech generation system in the adversarial training literature context.
