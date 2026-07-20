---
id: "interspeech-2025-0115"
title: "Bringing Interpretability to Neural Audio Codecs"
authors: ["Samir Sadok", "Julien Hauret", "Eric Bavu"]
organization: null
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-06-03
task: [codec]
architecture: [transformer-enc-dec]
conditioning: []
training: [supervised, self-supervised]
model_size: "not reported"
codec_used: "SpeechTokenizer; BigCodec; DAC; Mimi"
datasets_train: ["LibriSpeech-100-clean"]
datasets_eval: ["LibriSpeech test-clean", "PTDB"]
metrics:
  - name: STOI
    value: 0.75
    system: AnCoGen-SpeechTokenizer
    testset: LibriSpeech test-clean
  - name: STOI
    value: 0.72
    system: AnCoGen-BigCodec
    testset: LibriSpeech test-clean
  - name: SPK-SIM
    value: 0.74
    system: AnCoGen-SpeechTokenizer
    testset: LibriSpeech test (voice conversion)
  - name: SPK-SIM
    value: 0.68
    system: AnCoGen-BigCodec
    testset: LibriSpeech test (voice conversion)
code_available: null
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/sadok25_interspeech.html"
related_concepts: [neural-codec, disentanglement, self-supervised-speech, voice-conversion, evaluation-metrics]
related_papers: []
field_significance:
  level: moderate
  type: [conceptual-contribution, empirical-benchmark]
generation:
  date: 2026-06-20
  agent: speech-generation-review-agent
  model: claude-sonnet-4-6
  commit: "bfd00ba"
---

> [!abstract] Interspeech · 2025 · Conference
> **Sadok et al.** · [→ Paper](https://www.isca-archive.org/interspeech_2025/sadok25_interspeech.html) · Demo: ? · Code: ?
>
> A systematic analysis of how linguistic content, speaker identity, and pitch are encoded across RVQ scales in four contemporary neural audio codecs, paired with an adaptation of the AnCoGen masked-autoencoder framework to enable direct extraction and manipulation of those attributes from codec tokens.

## Problem

Neural audio codecs have displaced mel-spectrograms as the dominant intermediate representation in generative speech models, yet the relationship between their discrete tokens and perceptually meaningful speech attributes remains poorly understood. Unlike semantic tokens (e.g., HuBERT), acoustic codec tokens are optimised purely for reconstruction fidelity, leaving open the question of where — and how cleanly — content, identity, and pitch are encoded across the residual vector quantisation (RVQ) hierarchy. This opacity limits both interpretability and controllability: practitioners cannot easily predict which codec or which RVQ scale will respond to a given manipulation.

## Method

The paper pursues a two-stage framework applied to four codecs — DAC, SpeechTokenizer, Mimi, and BigCodec — chosen to span a range of design philosophies (vanilla RVQ, HuBERT-distilled, split-RVQ with semantic suppression, and single-VQ high-cardinality).

In the **analysis stage**, token co-occurrence matrices between codec tokens and HuBERT's 100-class semantic tokens are computed on the LibriSpeech test-clean set. The top-k HuBERT associations per codec token are summarised as a mapping precision metric and visualised via t-SNE over learned codebook embeddings — separately for linguistic content (colour-coded by HuBERT-to-sound class mapping), speaker identity (colour-coded by speaker), and pitch (filtered to a single vowel-associated HuBERT class). Mutual information between SpeechTokenizer RVQ scales and each attribute is estimated using the CLUB contrastive bound.

In the **synthesis stage**, an encoder-decoder Transformer (AnCoGen-Codec) is trained on LibriSpeech-100-clean to bridge codec tokens and four speech attributes: linguistic content (HuBERT encoder outputs), pitch contour (CREPE), loudness (RMS), and speaker identity (ECAPA-TDNN embeddings). During training, a masking strategy is applied to either the codec-token side or the attribute side; the decoder predicts the masked modality via cross-entropy over token indices. At inference, the model operates bidirectionally: codec tokens in → attributes out (analysis), or attributes in → codec tokens out (generation), with the frozen codec decoder performing the final token-to-waveform step. Two AnCoGen-Codec variants are trained — one for SpeechTokenizer and one for BigCodec.

## Key Results

The analysis reveals a consistent pattern across codecs: linguistic content concentrates in the first RVQ scale, speaker identity migrates toward higher scales, and pitch remains the most diffusely encoded attribute throughout. SpeechTokenizer's first scale shows strong HuBERT alignment (t-SNE clusters clearly separating phoneme classes) as expected from its distillation objective, while BigCodec's single-VQ structure yields only 8% mean mapping to the top HuBERT token — the weakest of the four. Surprisingly, even Mimi's split-RVQ design — intended to prevent semantic leakage into higher scales — still carries substantial linguistic content in upper scales, a finding the authors attribute to the codec's very low 12.5 Hz frame rate.

For the synthesis tasks (Table 1), AnCoGen-SpeechTokenizer achieves content accuracy of 82% and speaker cosine similarity (COS) of 0.74 on voice conversion, outperforming AnCoGen-BigCodec (73.58% content accuracy, COS 0.68). AnCoGen-BigCodec achieves the highest N-MOS resynthesis score (4.39 vs. 4.34 for SpeechTokenizer), but at the cost of poorer content reconstruction. The Melspectrogram-based AnCoGen baseline requires a 51.2 kbps bitrate; AnCoGen-SpeechTokenizer operates at 4 kbps and AnCoGen-BigCodec at 1.04 kbps with broadly comparable quality.

## Novelty Assessment

The contribution is primarily analytical: the paper does not introduce a new codec architecture but applies established probing and visualisation methods (t-SNE, mutual information estimation, token co-occurrence matrices) to characterise the internal structure of existing codecs. The AnCoGen-Codec adaptation is an extension of prior work (AnCoGen-Melspectrogram) to a codec-token input space, which is useful but engineering-incremental. The genuine value lies in the systematic, cross-codec comparison and in the finding that even disentanglement-oriented codecs (SpeechTokenizer, Mimi) fall short of clean separation — a result that is useful for practitioners choosing codecs for controllable generation tasks. The evaluation is fair but limited to LibriSpeech, which is acoustically homogeneous.

## Field Significance

Moderate — This paper adds useful empirical grounding to a question the field has largely taken on faith: that codec tokens encode the full acoustic signal but do so opaquely. The finding that pitch is the most interpretability-resistant attribute, and that explicit disentanglement objectives only partially succeed, is relevant to any downstream task that requires attribute control over codec-based generation. The work is confirmatory and diagnostic rather than directional; it does not change the dominant paradigm but sharpens understanding of its limitations.

## Claims

- Even codecs explicitly trained with disentanglement objectives fail to cleanly separate pitch from other speech attributes in their token embeddings. *(§2.3, §2.4)*
- Linguistic content in neural audio codec representations concentrates in the lowest RVQ scales regardless of whether distillation was used, but leaks into higher scales when the frame rate is very low. *(§2.1)*
- A masked-autoencoder framework can bridge codec tokens and perceptual speech attributes bidirectionally, enabling voice conversion at dramatically lower bitrates than spectrogram-based equivalents. *(§3.1, §3.2)*
- Post-hoc interpretability tools reveal systematic trade-offs between content accuracy and synthesis quality that differ by codec design, complicating the choice of codec for controllable speech generation. *(§3.2, Table 1)*

## Limitations and Open Questions

> [!warning]
> All experiments use LibriSpeech, a clean read-speech corpus with limited acoustic diversity. Whether the observed encoding patterns hold for spontaneous speech, expressive data, or noise-conditioned codecs is untested.

The study covers four specific codecs; the broader generalisation across the growing landscape of codec designs (including future multi-scale or end-to-end codec-LM systems) is an open question. Pitch estimation from codec tokens remains poor, and no remedy is proposed — it is unclear whether this is a fundamental limitation of RVQ-based representations or an artefact of the particular codecs studied. The bidirectional AnCoGen-Codec is compared only against the Melspectrogram baseline and not against dedicated disentanglement-oriented codec frameworks such as FreeCodec or SpeechFlow, which would provide stronger context for the synthesis results.

## Wiki Connections

- [[neural-codec]]
- [[disentanglement]]
- [[self-supervised-speech]]
- [[voice-conversion]]
- [[evaluation-metrics]]
