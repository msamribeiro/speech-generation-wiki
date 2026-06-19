---
id: "interspeech-2025-0815"
title: "Towards Better Disentanglement in Non-Autoregressive Zero-Shot Expressive Voice Conversion"
authors: ["Seymanur Akti", "Tuan-Nam Nguyen", "Alexander Waibel"]
organization: Karlsruhe Institute of Technology
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-06-03
task: [VC]
architecture: [VAE, GAN]
conditioning: [speaker-conditioned, zero-shot, emotion-conditioned, prompt-conditioned]
training: [self-supervised]
model_size: "not reported"
codec_used: "none"
datasets_train: ["LibriTTS-100", "ESD", "GigaSpeech (subset)", "Expresso"]
datasets_eval: ["ESD", "Expresso", "LibriTTS", "RAVDESS", "EmoDB"]
metrics:
  - name: MOS
    value: 3.37
    system: proposed
    testset: RAVDESS (zero-shot)
  - name: MOS
    value: 3.72
    system: proposed
    testset: RAVDESS (eMOS, zero-shot)
  - name: WER
    value: 7.98
    system: proposed
    testset: ESD
  - name: WER
    value: 8.84
    system: proposed
    testset: LibriTTS
  - name: EER
    value: 7.4
    system: proposed
    testset: LibriTTS
code_available: null
demo_available: true
url: "https://www.isca-archive.org/interspeech_2025/akti25_interspeech.html"
related_concepts: ["voice-conversion", "disentanglement", "self-supervised-speech", "emotion-synthesis", "zero-shot-tts", "prosody-control"]
related_papers: []
field_significance:
  level: moderate
  type: [architectural-novelty]
generation:
  date: 2026-06-03
  model: claude-sonnet-4-6
  commit: "55d0339"
---

> [!abstract] Interspeech · 2025 · Conference
> **Akti et al.** (Karlsruhe Institute of Technology) · [→ Paper](https://www.isca-archive.org/interspeech_2025/akti25_interspeech.html) · Demo: ✓ · Code: ?
>
> A non-autoregressive zero-shot expressive voice conversion system built on a conditional VAE that reduces source timbre leakage through multilingual discrete speech units, mix-style layer normalisation, and a perturbation-based similarity loss, while improving expressivity transfer via cross-attention F0 injection and style embeddings enriched with global pitch and energy.

## Problem

Expressive voice conversion requires simultaneously transferring speaker identity and expressive attributes — emotion, pitch contour, intensity — while suppressing residual timbre from the source speaker. Prior VAE-based approaches suffer from source style leakage, where acoustic characteristics of the source audio bleed through into the converted output, undermining both speaker similarity to the target and fidelity of the emotional transfer. Continuous self-supervised representations compound this problem because they retain non-linguistic speaker cues that then contaminate the content embedding. A further challenge is the train-inference mismatch: during training, source and target utterances often share the same content, but this alignment does not hold at inference time, making the system's style encoder content-dependent in ways that hurt generalisation.

## Method

The system follows the FreeVC / VITS architecture — a conditional variational autoencoder with a HiFi-GAN waveform synthesiser — extended with several disentanglement-oriented modifications. Content representations are derived from mHuBERT-147 discrete speech units rather than continuous features; quantisation suppresses speaker-dependent information that continuous representations would carry, and the multilingual nature of mHuBERT-147 enables zero-shot cross-lingual conversion.

Content embeddings are processed through a Mix Encoder that applies mix-style layer normalisation (Mix-LN): the normalisation affine parameters are computed as a convex combination of the sample's own style embedding and a batch-shuffled counterpart, controlled by a Beta-distributed mixing coefficient. This injects noise from other speakers' styles, reducing the content representation's dependence on the source speaker. An additional perturbation-based similarity loss ensures that content embeddings extracted from original and pitch-augmented audio remain close in cosine space, further suppressing style information from the content path.

Local F0 contours are extracted at the same frame rate as the content embeddings (320 samples/frame) and fused with them via multi-head cross-attention, with content as query and F0 as key/value. This differs from prior work that re-normalises and additively injects F0, and allows the target F0 to be directly used at inference regardless of length differences between source and target. A transformer-based normalising flow maps the posterior distribution to the prior during training, with the reverse flow used at inference.

The style encoder extends ECAPA-TDNN by computing separate 512-dimensional embeddings from mel spectrograms, F0 contours, and energy contours, then fusing them with a trainable weighted sum. This enriched global embedding conditions all encoders and carries both speaker identity and expressive attributes.

![Overall architecture of the proposed system.](assets/interspeech-2025-0815/figure-1.png)

## Key Results

On the multi-dataset setting (Table 1), the proposed model outperforms Hierspeech++ on all emotion transfer metrics: ECA improves from 37.0% to 78.9% overall, SECS from 73.1% to 81.2% on Expresso, EECS from 82.0% to 85.3%, and EER drops from 14.6% to 7.4% on LibriTTS (lower is better, indicating less source leakage). These gains come at a WER cost: 7.98% vs. 5.01% on ESD and 8.84% vs. 3.48% on LibriTTS. Subjective evaluation on RAVDESS shows eMOS of 3.72 vs. 2.94 for Hierspeech++, and sMOS of 3.65 vs. 2.97, while nMOS is comparable (3.37 vs. 2.68).

In the ESD-only comparison, the proposed model achieves ECA (O→N) of 78.8%, surpassing X-E-Speech (74.1%) and Consistency-VC (72.6%), though WER is substantially higher (10.44% vs. 4.71% and 5.69%). This intelligibility deficit is attributed to the discrete speech unit representation, which removes speaker information but can also discard fine phonetic detail.

Ablation results (Table 2) confirm that all four proposed components contribute: removing F0 cross-attention degrades SECS by 0.6pp and ECA by 12.5pp; removing Mix-LN causes the most severe WER degradation on LibriTTS (19.83%), consistent with the hypothesis that Mix-LN reduces the train-inference content-style mismatch; removing the similarity loss weakens unseen-speaker conversion; and replacing mHuBERT with MMS features lowers WER but substantially reduces emotion transfer.

Cross-lingual evaluation (Table 4) shows the system achieves 76.1% ECA for English→German conversion vs. 61.7% for Hierspeech++ and 45.3% for Consistency-VC, though German→English WER is 30.84%, indicating intelligibility degrades when the source language is absent from training.

## Novelty Assessment

The individual components — discrete speech unit inputs, VAE/VITS backbone, ECAPA-TDNN style encoding, HiFi-GAN vocoder, mix-style layer normalisation — are each drawn from existing work. The novelty lies in the particular combination and its application to expressive VC: using mHuBERT-147 (first reported use in a VC system per the authors) as a multilingual discrete content encoder, the F0 cross-attention fusion instead of additive injection, and the perturbation-based similarity loss. Mix-LN is adopted from prior work but the analysis of its role in reducing train-inference content mismatch provides new insight. The paper is an engineering integration with genuine ablation support for each design choice, though the intelligibility trade-off introduced by discrete units is a known and unresolved limitation rather than a novel finding.

## Field Significance

Moderate — This paper advances the sub-problem of source timbre leakage in expressive voice conversion, showing that discrete multilingual speech units combined with mix-style normalisation and a perturbation-based similarity loss can substantially improve emotion transfer at the cost of intelligibility. It provides solid ablation evidence for each component, and the cross-lingual capability without multilingual training data is a practically useful by-product. The intelligibility regression limits immediate deployment applicability.

## Claims

- Discrete speech unit representations reduce source speaker leakage in voice conversion but introduce pronunciation artefacts that degrade intelligibility compared to continuous feature counterparts.
- Mix-style layer normalisation mitigates the train-inference mismatch caused by content-style dependence in style encoders, improving zero-shot generalisation on unseen speakers.
- Enriching global style embeddings with explicit pitch and energy features improves emotion transfer fidelity in expressive voice conversion beyond mel-spectrogram-only style encoding.
- Cross-attention fusion of local F0 contours with content embeddings produces stronger prosodic alignment to the target than additive F0 injection in non-autoregressive voice conversion.
- Zero-shot cross-lingual voice conversion is achievable with a monolingual training corpus when content representations are extracted from a multilingual speech model, though intelligibility degrades for unseen source languages.

## Limitations and Open Questions

> [!warning]
> The proposed system incurs a substantial WER penalty relative to baselines: 7.98% vs. 5.01% (ESD) and 8.84% vs. 3.48% (LibriTTS) for the full model, with discrete units identified as the cause. This intelligibility regression is acknowledged but not resolved; future work is deferred.

The evaluation uses a small subjective panel (15 listeners, 10–15 samples per model), limiting the statistical power of MOS comparisons. The cross-lingual results are restricted to English and German; how performance degrades for more distant language pairs is untested. The model is trained on English-only data, and German-to-English conversion shows a 30.84% WER, suggesting significant cross-lingual generalisation limits. The model size and computational cost are not reported, making it difficult to assess deployment feasibility.

## Wiki Connections

This paper advances [[disentanglement]] in expressive [[voice-conversion]], specifically targeting source timbre leakage in the [[zero-shot-tts]] generalisation setting. It builds on [[self-supervised-speech]] representations (mHuBERT-147 discrete units) for content encoding, and its design is closely related to [[prosody-control]] through the F0 cross-attention and enriched [[emotion-synthesis]] style embeddings.
