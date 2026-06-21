---
id: "interspeech-2025-0246"
title: "DC-Spin: A Speaker-invariant Speech Tokenizer for Spoken Language Models"
authors: ["Heng-Jui Chang", "Hongyu Gong", "Changhan Wang", "James Glass", "Yu-An Chung"]
organization: null
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-06-03
task: [codec]
architecture: [transformer-enc-dec]
conditioning: [speaker-conditioned]
training: [self-supervised, supervised, fine-tuning]
model_size: "150M params (uLM); HuBERT Base encoder"
codec_used: "none"
datasets_train: ["LibriSpeech", "Libri-Light (6k hours)", "124k-hour English corpus"]
datasets_eval: ["LibriSpeech", "Expresso"]
metrics:
  - name: WER
    value: 18.7
    system: SpinHuBERT + DC-Spin 500,4096 + PR
    testset: Expresso test
  - name: UTMOS
    value: 3.04
    system: SpinHuBERT + DC-Spin 500,4096 + PR
    testset: Expresso test
code_available: null
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/chang25_interspeech.html"
related_concepts: [self-supervised-speech, spoken-language-model, neural-codec, disentanglement, evaluation-metrics]
related_papers: []
field_significance:
  level: moderate
  type: [architectural-novelty, empirical-benchmark]
generation:
  date: 2026-06-21
  agent: speech-generation-review-agent
  model: claude-sonnet-4-6
  commit: "3538db5"
---

> [!abstract] Interspeech · 2025 · Conference
> **Chang et al.** (MIT CSAIL / Meta AI) · [→ Paper](https://www.isca-archive.org/interspeech_2025/chang25_interspeech.html) · Demo: ? · Code: ?
>
> DC-Spin extends speaker-invariant clustering to a dual-codebook design and pairs it with a new HuBERT pre-training variant (SpinHuBERT) to produce discrete speech tokens that are simultaneously phonetically rich, speaker-invariant, and amenable to spoken language model training at limited resource budgets.

## Problem

Speech tokenizers for spoken language models (SLMs) face competing demands: tokens must encode phonetic content for recognition and language modelling tasks, yet retain enough acoustic detail to support high-intelligibility resynthesis. Existing approaches fall into two camps — SSL-based K-means units preserve phonetics but lose acoustic texture, while neural codecs preserve acoustics but encode little phonetic structure. Prior fine-tuning approaches like Spin partially close this gap but face a trade-off: small codebooks perform well on SLM tasks but large ones are needed for good phonetic alignment. No prior open-source tokenizer satisfies both qualifications simultaneously with limited training resources. A secondary gap is the absence of principled proxy metrics that correlate reliably with downstream SLM performance, making tokenizer development largely trial-and-error.

## Method

DC-Spin builds on Speaker-invariant Clustering (Spin), a fine-tuning method that trains an SSL encoder by feeding perturbed (speaker-shifted) and original utterances through the model and minimising cross-entropy between their frame-level codeword distributions. This encourages the encoder to suppress speaker identity and focus on phonetic content.

The paper contributes two linked components. First, **SpinHuBERT** replaces HuBERT's usual K-means pseudo-labels with Spin codeword IDs during the standard masked-prediction pre-training loop. Because Spin units are more phonetically aligned than K-means clusters, SpinHuBERT learns a stronger initial representation before any fine-tuning. Optionally, supervised fine-tuning (SFT) with CTC-based ASR or phoneme recognition (PR) is applied to further enforce content focus.

Second, **DC-Spin** (Double-Codebook Spin) resolves the codebook size tension by adding an auxiliary large codebook (e.g. 4096 entries) alongside the small primary codebook (50–500 entries). Both share the same encoder; the auxiliary codebook encourages the encoder to capture finer phonetic distinctions, which indirectly improves the quality of units from the smaller primary codebook. Discrete tokens for downstream use come exclusively from the primary codebook.

![The proposed speech tokenizer training. Stage (I) pre-trains a speech encoder with pseudo labels from K-means or Spin units, where the latter is the proposed SpinHuBERT (Section 2.2). The optional stage (II) fine-tunes the encoder with CTC-based ASR or phoneme recognition (PR). In stage (III), the encoder is fine-tuned with the DC-Spin objective (Section 2.3).](assets/interspeech-2025-0246/figure-2.png)

The downstream evaluation uses a 150M-parameter unit language model (uLM) for zero-shot SLM tasks and a HiFi-GAN vocoder conditioned on speaker and style IDs for speech resynthesis; these serve as standard, comparable proxies. The complete pipeline trains on a 124k-hour English corpus for SpinHuBERT pre-training and 6k hours of Libri-Light for SLM training — substantially less than competing high-resource systems.

## Key Results

On zero-shot SLM benchmarks (Table 1, comparable baselines using HuBERT Base + LibriSpeech), DC-Spin consistently outperforms plain Spin, K-means, NAST, and augmentation-invariant representations. DC-Spin 100,4096 achieves 78.47 sWUGGY (in-vocab) and 61.35 sBLIMP, compared to NAST's 73.35/55.86 and K-means' 67.75/51.96 at the same unit count. Against unconstrained high-resource SLMs (Table 2), SpinHuBERT + DC-Spin 500,4096 with a 150M SLM and 6k training hours exceeds the sBLIMP and sWUGGY scores of models using up to 13B parameters and 150k+ hours (e.g. TWIST: 59.2 sBLIMP; proposed: 65.9 sBLIMP with PR SFT).

On speech resynthesis from the Expresso dataset (Table 3), SpinHuBERT + DC-Spin + PR achieves 18.7% WER (test) and UTMOS 2.92, compared to 24.4% WER for HuBERT + K-means 500 — a relative improvement of ~23% in intelligibility. The comparison to EnCodec (27.5% WER at 1.5k bitrate) shows SSL-based tokens can surpass codec-based tokens on intelligibility at lower bitrate (448 bps vs. 1.5k bps for EnCodec RVQ1:2).

The proxy analysis (Section 3.5, Figure 3b) shows that n-gram predictability, phoneme normalised mutual information (PNMI), and character normalised mutual information (CNMI) are the strongest predictors of downstream SLM performance, whereas the commonly used ABX error rate negatively correlates with TSC and sBLIMP — a finding with practical implications for tokenizer development workflows.

## Novelty Assessment

The core architectural idea (dual-codebook Spin) is an incremental but principled extension of the existing Spin fine-tuning framework. The use of SpinHuBERT as a better HuBERT initialisation is also modest in concept — substituting Spin IDs for K-means pseudo-labels — but empirically effective. The more distinctive contribution is the proxy task analysis, which provides actionable guidance for tokenizer design and explicitly challenges the field's reliance on ABX as a proxy metric. The paper does not introduce a new codec architecture or a new SSL pre-training paradigm; it works entirely within the HuBERT/Spin ecosystem. The competitive performance at low resource budgets is noteworthy but relies on not matching the resource conditions of the strongest baselines, so the comparison in Table 2 should be read as resource-efficient rather than state-of-the-art in absolute terms.

## Field Significance

Moderate — DC-Spin contributes a practical improvement to the SSL-based speech tokenizer family with clear benefits for SLM tasks, and the proxy task analysis offers concrete design guidance that could influence how the field evaluates and selects tokenizers going forward. The paper is primarily an incremental engineering advance within an established paradigm rather than a shift in direction.

## Claims

- Speaker-invariant discrete speech tokens, trained with a dual-codebook objective, can improve both spoken language model performance and speech resynthesis intelligibility simultaneously. *(§3.2, §3.3, Tables 1–3)*
- N-gram predictability and phoneme/character mutual information are stronger proxies for SLM downstream performance than the widely-used ABX error rate. *(§3.5, Figure 3b)*
- SSL-based tokenizers can match or exceed neural codec tokenizers on speech intelligibility (WER) at substantially lower bitrates when the encoder is fine-tuned to suppress speaker variation. *(§3.3, Table 3)*
- Supervised fine-tuning with phoneme recognition targets provides consistent gains over ASR targets for speech resynthesis intelligibility, suggesting phoneme alignment is more directly beneficial than word-level transcription for unit-based vocoders. *(§3.3, Table 3)*
- Scaling SLM model size has diminishing returns on tasks that require sentence-level semantic coherence when the tokenizer quality is the primary bottleneck. *(§3.2, Table 2)*

## Limitations and Open Questions

> [!warning]
> The unconstrained comparison in Table 2 pits a 150M SLM trained on 6k hours against models using 13B parameters and 150k+ hours. While the framing as "limited-resource" is accurate, direct comparisons against high-resource systems risk being misleading — the TSC gap (70.7 vs. 82.9 for SPIRIT LM) is large and likely attributable to scale rather than tokenizer quality.

The evaluation is English-only, and the authors acknowledge multilinguality as future work. The HiFi-GAN vocoder receives speaker and style IDs externally, so the speaker-invariance of the tokens is not tested in an open-vocabulary, zero-shot resynthesis scenario. The Expresso WER figures (>10%) are elevated by the expressive speaking styles (laughter, whisper), making absolute comparisons with non-expressive benchmarks impractical. The paper also does not report wall-clock training times or resource costs in sufficient detail for full reproducibility assessment.

## Wiki Connections

This paper informs: [[self-supervised-speech]], [[spoken-language-model]], [[neural-codec]], [[disentanglement]], [[evaluation-metrics]].
