---
id: "interspeech-2025-1098"
title: "GST-BERT-TTS: Prosody Prediction Without Accentual Labels For Multi-Speaker TTS Using BERT With Global Style Tokens"
authors: ["Tadashi Ogura", "Takuma Okamoto", "Yamato Ohtani", "Erica Cooper", "Tomoki Toda", "Hisashi Kawai"]
organization: "National Institute of Information and Communications Technology, Japan"
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-07-03
task: [TTS]
architecture: [transformer-enc-dec]
conditioning: [speaker-conditioned, text-conditioned]
training: [supervised, fine-tuning]
model_size: "not reported"
codec_used: "none"
datasets_train: ["internal Japanese multi-speaker corpus (51,510 utterances, 170 speakers)"]
datasets_eval: ["Hi-Fi-CAPTAIN"]
metrics:
  - name: MOS
    value: "not reported (figure only)"
    system: proposed
    testset: Hi-Fi-CAPTAIN
code_available: null
demo_available: true
url: "https://www.isca-archive.org/interspeech_2025/ogura25_interspeech.html"
related_concepts: [prosody-control, evaluation-metrics, subjective-evaluation, transformer-enc-dec-tts]
related_papers: []
field_significance:
  level: moderate
  type: [architectural-novelty]
generation:
  date: 2026-07-03
  agent: speech-generation-ingest-agent
  model: claude-sonnet-4-6
  commit: "d8ecfd4"
---

> [!abstract] Interspeech · 2025 · Conference
> **Ogura et al.** (National Institute of Information and Communications Technology, Japan) · [→ Paper](https://www.isca-archive.org/interspeech_2025/ogura25_interspeech.html) · Demo: ✓ · Code: ?
>
> GST-BERT-TTS extends BERT-based prosody prediction to multi-speaker Japanese TTS by injecting Global Style Token embeddings into BERT's token embedding layer, enabling speaker-aware fundamental frequency, energy, and duration prediction without accent labels.

## Problem

Prosody prediction for pitch-accent languages such as Japanese has traditionally depended on accent dictionaries and hand-crafted labels, which are incomplete, context-insensitive, and expensive to maintain. Data-driven alternatives reduce this reliance but typically require large labeled datasets. The fo-BERT approach achieves label-free mora-level pitch prediction using BERT's linguistic context, but it is restricted to single-speaker TTS and cannot account for speaker-dependent prosodic characteristics. In multi-speaker settings, where each speaker exhibits distinct pitch dynamics and speaking rates, a speaker-agnostic prosody predictor produces inadequate results.

## Method

GST-BERT-TTS builds on the fo-BERT framework in three training phases. In the first phase, a multi-speaker acoustic model is trained with a GST module that extracts utterance-level style embeddings from mel-spectrograms; these embeddings are added to the encoder's hidden representations to inform a variance adaptor predicting log f0, energy, and duration. The ConvNeXt-based acoustic model is paired with an MS-FC-HiFi-GAN vocoder and uses monotonic alignment search for duration modelling.

In the second phase, a BERT model is trained to predict prosodic parameters directly from text and katakana input. Speaker-specific style embeddings are derived by averaging all utterance-level GST vectors for each speaker. Because BERT's embedding dimension differs from the GST embedding dimension (256), a trainable linear adaptor projects the GST vector before adding it element-wise to every token embedding, ensuring that speaker identity influences all subsequent transformer layers uniformly. The BERT model's output at each katakana character position is mapped to log f0, energy, and duration via dedicated linear heads, trained jointly with a token classification objective under a weighted multi-task loss.

During inference, the BERT module generates prosodic parameters that replace the variance adaptor's own predictions in the TTS module, decoupling prosody prediction from the acoustic model's internal dynamics. The architecture retains fo-BERT's label-free property while adding speaker conditioning through the GST branch.

![Proposed GST-BERT-TTS framework which integrates GST into BERT-based prosody prediction, allowing shared style embeddings across multiple speakers to enhance multi-speaker prosody modeling. Step 1: Train the TTS module. Step 2: Train the BERT module using features extracted from the TTS module. Step 3: Perform inference using both modules.](assets/interspeech-2025-1098/figure-2.png)

## Key Results

On an internal Japanese multi-speaker corpus (170 speakers, 51,510 utterances), GST-BERT-TTS (full params.) achieves an MSE of 0.266 for log f0, compared to 0.773 for fo-BERT (full params.) and 0.33 for the TTS variance adaptor baseline (Table 2). For energy and duration prediction, GST-BERT (full params.) also outperforms fo-BERT and approaches or surpasses the TTS-internal predictor.

In accent correctness listening tests on Hi-Fi-CAPTAIN, GST-BERT-TTS reaches 65% at the highest rating level (no discernible accent errors), compared to 47.5% for fo-BERT and 34% for label-free TTS alone (Table 3). For the combined top-two categories (scores 4+3), GST-BERT-TTS reaches 94%, slightly above the TTS+Accent dictionary baseline (88%). However, TTS+Accent achieves 73.5% at the strictest rating level, indicating that explicit accent labels still outperform the label-free approach for the most demanding accent accuracy criterion.

MOS results (Figure 3) confirm that GST-BERT-TTS improves subjectively rated quality over the label-free TTS baseline, but remains below TTS+Accent. The authors attribute the overall low MOS scores to high pitch and rate variability in the training corpus and vocoder generalisation issues. Comparisons are somewhat constrained by the proprietary dataset, and the accent test was conducted on only two speakers (one male, one female).

## Novelty Assessment

The architectural contribution is the injection of GST-derived speaker embeddings into BERT's token embedding layer via a linear adaptor, which is a targeted and straightforward extension of fo-BERT. The integration is technically clean and the motivation is sound: GST provides a continuous speaker representation that scales to arbitrary speakers without requiring speaker ID labels at inference time. Extending the prosody predictor from log f0 alone to log f0, energy, and duration adds practical value. The contribution is primarily engineering integration of two known components (BERT-based prosody prediction and Global Style Tokens) within an established acoustic model framework, rather than a new architecture. The evaluation on a proprietary dataset limits reproducibility, and the accent correctness test is conducted on only two speakers. The finding that GST embeddings degrade prosody for truly unseen speakers (flat, monotonous prosody) is a notable and honest limitation.

## Field Significance

Moderate — This paper addresses a practical gap in multi-speaker TTS for pitch-accent languages: enabling speaker-aware prosody prediction without accent labels. The approach introduces a simple but effective mechanism for injecting speaker style information into BERT-based prosody predictors, extending label-free prosody modelling from single-speaker to multi-speaker settings. The contribution is methodologically incremental; it primarily demonstrates that GST embeddings provide a useful speaker conditioning signal within a BERT prosody predictor, rather than offering a new training paradigm or benchmark.

## Claims

- **supports:** BERT-based prosody predictors for pitch-accent languages can be extended to multi-speaker settings through speaker style embedding injection without requiring accent label annotation.
  > *Evidence:* GST-BERT-TTS integrates averaged GST vectors into BERT token embeddings via a linear adaptor, reducing log f0 MSE from 0.773 (fo-BERT) to 0.266 and raising top-level accent correctness from 47.5% to 65% on Hi-Fi-CAPTAIN. *(§2.1, §3.4, Table 2, Table 3)*

- **complicates:** Speaker style embeddings derived from seen-speaker averages do not generalise well to unseen speakers in label-free prosody prediction.
  > *Evidence:* The authors report that using the GST embedding of an unseen speaker during inference produced flat, monotonous prosody, attributing the failure to insufficient speaker diversity in the training corpus and limited coverage of GST's latent space. *(§4.1)*

- **complicates:** Improvements in prosody prediction accuracy and accent correctness do not automatically translate into higher overall MOS in multi-speaker TTS.
  > *Evidence:* GST-BERT-TTS outperforms fo-BERT on MSE and accent correctness, yet MOS scores remain lower than TTS+Accent; the authors attribute this partly to vocoder artefacts on a high-variability multi-speaker corpus and to the fact that MOS conflates naturalness and audio clarity with prosodic correctness. *(§3.4, Figure 3)*

- **supports:** Multi-task prosody prediction covering fundamental frequency, energy, and duration jointly improves over pitch-only prediction when speaker conditioning is present.
  > *Evidence:* Adding energy and duration prediction to fo-BERT (full params.) had minimal impact on log f0 MSE (0.773 vs. 0.776), whereas GST-BERT (full params.) showed a synergistic improvement over GST-BERT (log f0 only), reducing log f0 MSE from 0.302 to 0.266. *(§3.4, Table 2)*

## Limitations and Open Questions

> [!warning]
> Evaluation uses a proprietary internal Japanese corpus; the accent correctness listening test involves only two speakers. Generalisation of the approach to other pitch-accent languages, larger speaker sets, or publicly available benchmarks is untested.

GST embeddings fail to provide useful conditioning for genuinely unseen speakers, limiting the method's zero-shot applicability. Addressing this may require a larger and more diverse speaker pool or a different style representation strategy. The authors acknowledge a tension between BERT's strength in contextual linguistic modelling and its suitability for constrained, character-level prosodic parameter regression, and identify integration with large language models as a direction requiring future methodological work.

## Wiki Connections

- [[prosody-control|Prosody Control]] — this paper proposes a speaker-aware mechanism to predict prosodic parameters (f0, energy, duration) from text without accent labels, contributing to data-driven prosody control in pitch-accent languages.
- [[transformer-enc-dec-tts|Transformer Encoder-Decoder TTS]] — the TTS backbone uses a ConvNeXt-based acoustic model with a variance adaptor and monotonic alignment search, situated within the encoder-decoder TTS paradigm.
- [[evaluation-metrics|Evaluation Metrics]] — the paper reports MSE for log f0, energy, and duration prediction, a four-level accent correctness listening test, and a MOS subjective quality evaluation.
- [[subjective-evaluation|Subjective Evaluation]] — MOS ratings from 20 subjects on 120 samples are used to assess synthesis quality, alongside the accent correctness listening test following the fo-BERT protocol.
