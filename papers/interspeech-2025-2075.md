---
id: "interspeech-2025-2075"
title: "Segmentation-Variant Codebooks for Preservation of Paralinguistic and Prosodic Information"
authors: ["Nicholas Sanders", "Yuanchao Li", "Korin Richmond", "Simon King"]
organization: "University of Edinburgh"
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-07-04
task: ["codec"]
architecture: []
conditioning: []
training: ["self-supervised", "supervised"]
model_size: "not reported"
codec_used: "not reported"
datasets_train: ["Naver Prosody Control", "Expresso", "IEMOCAP"]
datasets_eval: ["Naver Prosody Control", "Expresso", "IEMOCAP"]
metrics:
  - name: UTMOS
    value: 2.51
    system: SVCs (k=500)
    testset: Expresso
  - name: WER
    value: 17.46
    system: SVCs (k=500)
    testset: Expresso
code_available: null
demo_available: true
url: "https://www.isca-archive.org/interspeech_2025/sanders25_interspeech.html"
related_concepts: [neural-codec, self-supervised-speech, emotion-synthesis]
related_papers: []
field_significance:
  level: "moderate"
  type: ["architectural-novelty"]
generation:
  date: 2026-07-04
  agent: speech-generation-ingest-agent
  model: claude-sonnet-4-6
  commit: "318f1b4"
---

> [!abstract] Interspeech · 2025 · Conference
> **Sanders et al.** (University of Edinburgh) · [→ Paper](https://www.isca-archive.org/interspeech_2025/sanders25_interspeech.html) · Demo: ✓ · Code: ?
>
> Segmentation-Variant Codebooks (SVCs) quantize speech at multiple linguistic granularities (frame, phone, word, utterance) using separate KMeans codebooks on pooled HuBERT features, substantially improving the preservation of paralinguistic and prosodic information over single-resolution discrete speech units at comparable bitrates.

## Problem

Self-supervised learning models such as HuBERT generate powerful discrete speech units (DSUs) that capture phonetic and lexical structure well, but their quantization process systematically discards prosodic and paralinguistic information (emotion, prominence, speaking style). The dominant approach to improving expressivity retention is simply increasing the codebook vocabulary size, but this raises bitrates inefficiently: a k=2000 codebook costs more than double the bits of k=500 while yielding only marginal gains in emotion and prominence classification. No existing DSU method exploits the multi-granularity structure of speech to encode segment-level prosodic context independently of frame-level phonetic content.

## Method

SVCs build on frozen HuBERT-large representations. Frame-wise continuous embeddings from HuBERT are pooled across segmentation boundaries derived from forced alignment: mean pooling is applied over each phone, word, and utterance segment, while frame-level representations remain unpooled. Four separate KMeans codebooks (each with vocabulary size k=500) are trained on the representations at their respective granularities. At inference, each representation is assigned to the nearest centroid in its corresponding codebook, yielding four parallel DSU streams.

The key design choice is pre-pooling: representations are pooled in continuous space before KMeans quantization, rather than quantizing frame-by-frame first and pooling the resulting tokens. This preserves segment-level prosodic and paralinguistic features that discretization at the frame level would otherwise discard. For downstream use, the four streams are merged back to frame resolution by averaging all coarser-level DSUs with the frame-level stream based on their overlapping segmentation windows.

![Segmentation-Variant Codebooks processing with pre-pooling overview. h_n refers to continuous hidden representations at the nth frame, S_n refers to the resulting frame-level stream obtained from mean pooling multiple streams of DSUs based on overlapping segmentation.](assets/interspeech-2025-2075/figure-2.png)

Resynthesis uses a HiFi-GAN vocoder trained on the Expresso dataset for 100,000 steps, conditioned on SVC inputs and speaker labels. Probing tasks use linear classifiers trained on DSU representations for speech emotion recognition (SER) on IEMOCAP and prominence classification on Naver Prosody Control.

## Key Results

Pre-pooling versus post-pooling: pooling continuous representations before KMeans consistently outperforms pooling discrete tokens after quantization. On utterance-level SER, pre-pooling achieves 0.5074 accuracy versus 0.2834 for post-pooling; on word-level prominence, 0.3423 versus 0.1210 F-score (Table 2). This confirms that discretization erases fine-grained prosodic detail that segment-level pooling cannot recover after the fact.

Frame-level probing: SVCs outperform both k=500 and k=2000 baselines across all emotion sub-categories and prominence on the full frame-level probing benchmarks (Table 3). For example, angry speech F1 rises from 0.272 (k=500) and 0.298 (k=2000) to 0.614 with SVCs, approaching the continuous feature top-line of 0.660. The k=2000 baseline, despite a higher bitrate than SVCs on Expresso (548 vs. 544 bits/s), still underperforms SVCs across all categories, indicating that multi-level granularity is more bitrate-efficient than simply expanding a single codebook.

Expressive resynthesis on Expresso: SVCs achieve 41.22% style classification accuracy versus 24.53% (k=500) and 31.63% (k=2000), with WER of 17.46% and UTMOS of 2.5135 (Table 4). A large gap remains compared to continuous features (74.72% style accuracy, UTMOS 3.0759), confirming that DSU-based expressive resynthesis remains a hard problem even with improved codebooks.

## Novelty Assessment

The multi-level codebook design is a genuine contribution to the DSU literature. The core insight, that pooling at distinct linguistic granularities before KMeans captures segment-level prosodic structure inaccessible to frame-level quantization, is well-motivated and clearly validated. The comparison between pre- and post-pooling (§5.1, Table 2) is the most instructive experiment in the paper and adds a concrete design principle to the field.

The method has two practical constraints worth noting. First, it requires forced alignments at phone, word, and utterance boundaries, which limits applicability to paired speech-text settings and complicates use with noisy or spontaneous speech. Second, the multi-stream representation must be collapsed back to frame resolution before use, meaning downstream systems see a merged DSU stream rather than the richer factorized representation. Exploring architectures that consume the four streams directly is flagged as future work.

The resynthesis evaluation does not include human listening tests; UTMOS and style classifier accuracy are the only quality proxies. The authors acknowledge this limitation explicitly.

## Field Significance

Moderate — SVCs address a well-known weakness of DSU-based speech representations: the systematic loss of prosodic and paralinguistic information during quantization. The multi-granularity approach provides a principled and measurable improvement over single-resolution baselines at comparable bitrates. The paper introduces a useful design principle (pre-pool before discretization) that can generalize to other SSL-based codec workflows.

## Claims

- **supports:** Multi-granularity quantization codebooks encode paralinguistic and prosodic features more efficiently than single-resolution codebooks at equivalent or higher bitrates.
  > *Evidence:* SVCs (k=500 per codebook, ~544 bits/s on Expresso) outperform frame-level k=2000 baselines (~548 bits/s) on all emotion sub-categories and prominence classification, with angry speech F1 rising from 0.298 to 0.614. *(§6.2, Table 3)*

- **supports:** Pooling continuous speech representations before discretization retains more paralinguistic and prosodic information than pooling after discretization.
  > *Evidence:* Pre-pooling consistently outperforms post-pooling in both utterance-level SER (0.5074 vs 0.2834 accuracy) and word-level prominence classification (0.3423 vs 0.1210 F-score) across single-level and multi-level codebook conditions. *(§6.1, Table 2)*

- **complicates:** DSU-based expressive speech resynthesis retains a large gap in style fidelity relative to continuous-feature resynthesis, even with improved codebook designs.
  > *Evidence:* SVCs achieve 41.22% style classification accuracy on Expresso versus 74.72% for continuous features and 88.42% for ground truth, a 33-point gap despite SVCs outperforming all other DSU baselines tested. *(§6.2, §6.3, Table 4)*

- **complicates:** Forced-alignment requirements constrain multi-granularity DSU methods to text-paired speech settings, limiting their applicability to spontaneous or unlabelled data.
  > *Evidence:* Phone, word, and utterance boundaries for SVC segmentation are derived from forced alignments using the Montreal Forced Aligner and HTK; no unsupervised segmentation is evaluated, and the authors list automatic segmentation as future work. *(§4.1, §7)*

## Limitations and Open Questions

The method depends on forced alignments at three linguistic levels (phone, word, utterance), which requires text transcriptions and limits use in low-resource or spontaneous speech domains. The authors acknowledge this and identify unsupervised or automatic segmentation as the most promising avenue for removing this constraint.

Evaluation uses automated quality proxies only (UTMOS, style classifier accuracy, WER). The authors note that human listening tests and qualitative error analysis are needed and explicitly defer them to future work.

The merged frame-level DSU representation discards part of the factorized structure SVCs provide. Whether architectures that natively consume multi-stream DSUs would yield further gains is unanswered.

Finally, all codebook vocabulary sizes are fixed at k=500. The paper suggests that per-level vocabulary tuning could improve task adaptation, but this is not explored empirically.

## Wiki Connections

- [[neural-codec|Neural Audio Codec]] — SVCs extend the discrete speech unit literature by proposing a multi-level KMeans quantization method designed to retain prosodic and paralinguistic content that standard codec-style quantization discards.
- [[self-supervised-speech|Self-Supervised Speech]] — HuBERT-large is the frozen backbone that provides the continuous frame-wise representations from which all SVC codebooks are trained; the entire method builds on SSL pretraining.
- [[emotion-synthesis|Emotion Synthesis]] — emotion preservation in DSU-based resynthesis is one of two primary evaluation axes, assessed via SER probing on IEMOCAP and style classification accuracy on Expresso.
