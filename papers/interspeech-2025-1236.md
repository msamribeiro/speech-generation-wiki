---
id: "interspeech-2025-1236"
title: "Accelerating Diffusion-based Text-to-Speech Model Training with Dual Modality Alignment"
authors: ["Jeongsoo Choi", "Zhikang Niu", "Ji-Hoon Kim", "Chunhui Wang", "Joon Son Chung", "Xie Chen"]
organization: null
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-07-03
task: [TTS]
architecture: [flow-matching]
conditioning: [text-conditioned, zero-shot, prompt-conditioned]
training: [supervised]
model_size: "159M params"
codec_used: "none"
datasets_train: ["LibriTTS"]
datasets_eval: ["LibriSpeech-PC"]
metrics:
  - name: WER
    value: 1.97
    system: "F5-TTS + A-DMA (low-resource)"
    testset: LibriSpeech-PC test-clean
  - name: SPK-SIM
    value: 0.62
    system: "F5-TTS + A-DMA (low-resource)"
    testset: LibriSpeech-PC test-clean
code_available: true
demo_available: true
url: "https://www.isca-archive.org/interspeech_2025/choi25c_interspeech.html"
related_concepts: [flow-matching, diffusion-tts, zero-shot-tts, self-supervised-speech]
related_papers: ["2025.acl-long.313", "2301.02111", "2406.02430", "2407.05407"]
field_significance:
  level: "moderate"
  type: [architectural-novelty]
generation:
  date: 2026-07-03
  agent: speech-generation-ingest-agent
  model: claude-sonnet-4-6
  commit: "f9e71e3"
---

> [!abstract] Interspeech · 2025 · Conference
> **Choi et al.** · [→ Paper](https://www.isca-archive.org/interspeech_2025/choi25c_interspeech.html) · Demo: ✓ · Code: ✓
>
> A-DMA (Accelerating training with Dual Modality Alignment) adds two training-only auxiliary losses to flow-matching TTS: a CTC-based text alignment loss at an intermediate transformer layer and a cosine-similarity loss against frozen HuBERT features at a deeper layer, doubling convergence speed and improving WER by 26% over the F5-TTS baseline in a low-resource setting without modifying the inference pipeline.

## Problem

Flow-matching and diffusion-based TTS models such as E2 TTS and F5-TTS have removed the need for external duration predictors and force-aligned text, but this simplification shifts the full burden of text-to-speech correspondence onto the diffusion network itself. Without explicit alignment supervision, these models rely on implicit guidance to discover how text tokens map to acoustic frames, which leads to slow convergence, high variance in early training, and poor performance when training data is limited. Prior work on accelerating diffusion training (FasterDiT, min-SNR weighting) addresses the denoising schedule but not the representational mismatch between text and speech modalities.

## Method

A-DMA extends the training of a flow-matching TTS backbone (F5-TTS Small, 159M parameters) with two auxiliary loss terms that are applied only during training and removed entirely at inference.

**Text-guided alignment** attaches a CTC head to an intermediate transformer layer. Given the hidden state at that layer conditioned on noisy speech, masked speech, and the text transcript, the CTC loss supervises the model to decode the input characters from those intermediate representations. This provides early-layer supervision for text-acoustic correspondence without requiring a force-aligned duration model.

**Speech-guided alignment** inserts a lightweight interpolation-plus-1D-convolution projection after a (different) intermediate layer, mapping the layer's hidden state to the temporal and feature dimensions of a frozen HuBERT-large model. The alignment loss is a negative cosine similarity between the projected TTS feature and the HuBERT last-layer output extracted from the reference speech. This regularises the TTS model's internal representations toward the rich acoustic-semantic features of the SSL encoder.

The overall loss combines the conditional flow matching (OT-CFM) objective with the two auxiliary terms, weighted by hyperparameters lambda_text = 0.1 and lambda_speech = 1.0. At inference, the CTC head and HuBERT branch are discarded; the inference pipeline is identical to the unmodified F5-TTS baseline.

![Figure 1: Overall training framework. The model uses masked speech, noisy speech, and transcript to predict the masked part of the speech via flow matching. The SSL model and CTC layer are utilized only during training, with the SSL parameters frozen throughout the process.](assets/interspeech-2025-1236/figure-2.png)

A key finding from layer-wise ablation is that the two modalities benefit from alignment at different depths: text alignment is most effective around the 8th transformer layer, while speech alignment is optimal at the 12th (of 18 total layers). Applying both at the same layer degrades performance relative to either single-modality configuration, suggesting a feature-space trade-off when both losses compete for the same representation. The final configuration applies text alignment at layer 8 and speech alignment at layer 12, yielding the best WER (2.226%) across all dual-alignment variants (Table 2). The model is trained on LibriTTS (585 hours, 24 kHz) with mel-spectrograms (100 banks, 94 Hz) and evaluated using the pre-trained Vocos vocoder.

## Key Results

On the LibriSpeech-PC test-clean benchmark in a low-resource setting (0.6kh training data), F5-TTS + A-DMA achieves WER 1.97% and SIM 0.62, compared to the F5-TTS baseline at WER 2.68% and SIM 0.60 (Table 1). E2 TTS + A-DMA similarly improves from WER 3.10% / SIM 0.63 to WER 2.25% / SIM 0.65. Both A-DMA variants outperform USLM (WER 6.11%) and match or surpass high-resource baselines (E2 TTS at 100kh: WER 2.95%; F5-TTS at 100kh: WER 2.42%) while using only 0.6% of the data.

UTMOS scores in the ablation remain stable across all conditions (4.01-4.12 range, vs. 4.04 baseline), confirming that the alignment objectives do not harm perceptual naturalness. The paper claims doubled convergence speed based on the WER-vs.-training-steps curve, with A-DMA reaching the baseline's final WER in roughly half the number of steps.

## Novelty Assessment

The contribution is a training-recipe innovation built on established components. CTC auxiliary losses applied to intermediate ASR encoder layers (InterCTC) are a known technique in speech recognition; SSL feature distillation into TTS models has been explored in NaturalSpeech 3 and related work. A-DMA's specific novelty is (1) applying these two distinct alignment signals to a generative flow-matching model rather than a discriminative encoder, (2) the empirical finding that text and speech alignments should occupy different layer depths, and (3) demonstrating that both can be combined additively without interference. The approach does not modify the architecture, codec, or vocoder, and the evaluation is entirely automated (no human listening tests), which limits the ability to assess naturalness gains beyond UTMOS predictions.

## Field Significance

Moderate — A-DMA addresses a genuine practical bottleneck in training alignment-free flow-matching TTS, offering a clean solution that requires no architectural change at inference. The layer-depth analysis provides a reusable design principle for future auxiliary-loss schemes in transformer-based generative speech models, and the low-resource results suggest the approach may be especially useful when training data is limited.

## Claims

- **supports:** Auxiliary alignment losses applied at intermediate transformer layers can substantially accelerate convergence in flow-matching TTS training without modifying the inference pipeline.
  > *Evidence:* A-DMA reduces WER from 2.68% to 1.97% and improves SIM from 0.60 to 0.62 for F5-TTS on LibriSpeech-PC test-clean in a low-resource (0.6kh) setting, with the convergence curve showing that A-DMA matches the baseline's terminal WER in roughly half the training steps. *(§2, §4.1, Table 1)*

- **refines:** In transformer-based generative models, text-semantic and speaker-acoustic alignment supervision are best applied at different network depths rather than at the same layer.
  > *Evidence:* Layer-wise ablation on F5-TTS Small (18 DiT layers) shows that applying CTC text alignment at layer 8 and HuBERT speech alignment at layer 12 (WER 2.226%) outperforms both same-layer dual alignment at layer 8 (WER 3.063%) and same-layer alignment at layer 12 (WER 2.688%), supporting a modality-specific depth hypothesis. *(§4.2, Table 2)*

- **supports:** Frozen self-supervised speech encoders such as HuBERT are effective perceptual teachers for improving speaker similarity in flow-matching TTS through cosine-similarity distillation.
  > *Evidence:* Negative cosine similarity to HuBERT-large last-layer features at the 12th transformer layer improves SIM from 0.578 to 0.609 and WER from 7.474% to 3.521%; WavLM achieves similar gains (SIM 0.612, WER 3.992%). L1 distance and BCE-sigmoid variants are less effective. *(§4.3, Table 3)*

- **complicates:** Training-time alignment acceleration for diffusion TTS primarily improves text intelligibility and speaker similarity rather than overall speech naturalness, as measured by automatic MOS prediction.
  > *Evidence:* UTMOS scores across all A-DMA ablation conditions (Table 2) range from 4.01 to 4.12, close to the 4.04 baseline, while WER and SIM show large improvements. The paper reports no human listening test, leaving perceptual naturalness gains unconfirmed. *(§4.2, Table 2, §6)*

## Limitations and Open Questions

The evaluation covers only automated metrics (WER via Whisper-large-V3, SIM via WavLM speaker verification, UTMOS as an automatic MOS proxy); no human listening evaluation is reported. All A-DMA experiments use the low-resource 0.6kh training regime; whether doubled convergence speed holds at the 100kh scale used by high-resource baselines is untested. The paper also does not address model efficiency at inference, explicitly leaving inference acceleration as future work.

## Wiki Connections

- [[flow-matching|Flow Matching]] — A-DMA's training improvements target flow-matching TTS (F5-TTS and E2 TTS) and the alignment strategy is designed for the CFM objective's intermediate representations.
- [[diffusion-tts|Diffusion TTS]] — the paper frames A-DMA as a general acceleration strategy for diffusion-based TTS training, applicable wherever implicit text-speech alignment is the convergence bottleneck.
- [[zero-shot-tts|Zero-Shot TTS]] — A-DMA is evaluated on zero-shot speaker adaptation using LibriSpeech-PC with reference speech prompts, achieving competitive results in the low-resource regime.
- [[self-supervised-speech|Self-Supervised Speech]] — frozen HuBERT-large serves as the speech alignment teacher; the quality of its representations directly determines how well speech-guided alignment improves SIM.
- [[2025.acl-long.313|F5-TTS]] — the primary baseline and backbone for A-DMA; all main experiments and ablations use F5-TTS Small, and A-DMA is designed to be compatible with its DiT architecture and OT-CFM training.
- [[2301.02111|VALL-E]] — cited as the representative AR TTS baseline that flow-matching NAR models aim to surpass in quality and inference speed; motivates the shift toward diffusion-based TTS.
- [[2407.05407|CosyVoice]] — high-resource AR baseline (300M params, 170kh) in Table 1; A-DMA in low-resource achieves WER 2.25% vs. CosyVoice's 3.59%, demonstrating data efficiency.
- [[2406.02430|Seed-TTS]] — cited as a representative large-scale AR system in the introduction, contextualising the limitations of AR approaches that A-DMA's NAR training efficiency aims to address.
