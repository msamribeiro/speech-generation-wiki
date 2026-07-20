---
id: "interspeech-2025-1289"
title: "Unlocking Temporal Flexibility: Neural Speech Codec with Variable Frame Rate"
authors: ["Hanglei Zhang", "Yiwei Guo", "Zhihan Li", "Xiang Hao", "Xie Chen", "Kai Yu"]
organization: Shanghai Jiao Tong University
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-06-04
task: [codec]
architecture: [hybrid]
conditioning: []
training: [supervised]
model_size: "not reported"
codec_used: "DAC 75Hz (backbone)"
datasets_train: ["LibriTTS"]
datasets_eval: ["LibriTTS"]
metrics:
  - name: UTMOS
    value: 3.994
    system: DAC+TFC
    testset: LibriTTS (6kbps)
  - name: UTMOS
    value: 3.948
    system: DAC baseline
    testset: LibriTTS (6kbps)
  - name: UTMOS
    value: 3.895
    system: DAC+TFC
    testset: LibriTTS (3kbps)
  - name: UTMOS
    value: 3.628
    system: DAC baseline
    testset: LibriTTS (3kbps)
  - name: WER
    value: 2.845
    system: DAC+TFC
    testset: LibriTTS (6kbps)
  - name: WER
    value: 4.125
    system: DAC baseline
    testset: LibriTTS (6kbps)
  - name: STOI
    value: 0.970
    system: DAC+TFC
    testset: LibriTTS (6kbps)
  - name: STOI
    value: 0.967
    system: DAC baseline
    testset: LibriTTS (6kbps)
code_available: null
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/zhang25k_interspeech.html"
related_concepts: [neural-codec, autoregressive-codec-tts, streaming-tts, spoken-language-model]
related_papers: ["2412.17048"]
field_significance:
  level: moderate
  type: [architectural-novelty]
generation:
  date: 2026-06-04
  model: claude-sonnet-4-6
  commit: "6312840"
---

> [!abstract] Interspeech · 2025 · Conference
> **Hanglei Zhang et al.** (Shanghai Jiao Tong University) · [→ Paper](https://www.isca-archive.org/interspeech_2025/zhang25k_interspeech.html) · Demo: ? · Code: ?
>
> This paper introduces Temporally Flexible Coding (TFC), the first variable frame rate strategy for neural speech codecs, which dynamically allocates temporal resolution based on signal entropy to reduce sequence length and improve reconstruction quality at equivalent bitrates.

## Problem

Existing neural speech codecs operate at a Constant Frame Rate (CFR), allocating the same temporal granularity to silence and to dense phoneme transitions alike. This creates two problems for downstream generative models: first, it wastes capacity on low-information segments like silence; second, it produces long token sequences — up to 75 tokens per second for systems such as EnCodec or DAC — that are orders of magnitude longer than text token streams (~3–5 tokens/second), creating inefficiency in autoregressive speech generation. While prior work has introduced Variable Bitrate (VBR) strategies to adjust codebook depth per frame, the number of frames itself has remained fixed, and sequence length reduction has remained unaddressed.

## Method

TFC is a plug-and-play module built on top of the DAC codec backbone (24kHz, 75Hz frame rate). The core mechanism consists of three temporal resolutions derived from the encoder output: fine (75Hz), medium (37.5Hz), and coarse (18.75Hz). Medium and coarse representations are obtained by applying successive 2× CNN downsampling blocks to the fine encoder output, expanding the effective receptive field of each frame accordingly.

Frame rate allocation is determined by non-parametric temporal entropy computed over each resolution's receptive field window. Amplitude values of the speech signal are projected onto a set of N uniformly spaced histogram bins via Gaussian affinities, and Shannon entropy over the resulting distribution is computed per temporal segment. Low-entropy segments — silence, near-steady tones — receive coarse frames; high-entropy phonetic transitions receive fine frames. The allocation is controlled by user-specified granularity ratios $(r_f, r_m, r_c)$ that sum to 1, enabling seamless tuning of the average frame rate from 18.75Hz to 75Hz at inference time without retraining.

![Proposed framework of Temporally Flexible Coding. (b) VFR allocation examples under different granularity ratios.](assets/interspeech-2025-1289/figure-1.png)

After entropy-guided masking, each resolution's continuous representations are quantized independently via RVQ. The three quantized streams are fused into a unified sequence aligned to the finest resolution through mask-gated summation with upsampling. The decoder uses a conditional hierarchical design that progressively refines representations from coarse to fine, adding upsampled coarser features before each finer level, mitigating error propagation from pure sequential reconstruction. The model is trained on LibriTTS (960h) with a heuristic batch allocation of $r_f{=}0.4$, $r_m{=}0.3$, $r_c{=}0.3$ to ensure coverage across all resolutions. Inference uses at most 8 codebooks from the 32-codebook RVQ, yielding a practical upper bound of 6kbps.

## Key Results

At matched bitrates, DAC+TFC consistently outperforms the DAC baseline across most metrics (Table 1). The largest gains appear in WER: at 3kbps (DAC with 4 codebooks vs. DAC+TFC at 37.5Hz with 8 codebooks), WER drops from 2.44 to 2.19 and UTMOS improves from 3.63 to 3.90. At 6kbps, the WER advantage is more pronounced (4.13 → 2.85), suggesting that sequence-length reduction — rather than just bitrate — also benefits intelligibility. STOI improvements are consistent but modest.

In the VFR regime (Section 4.3), mixing granularities at an average 37.5Hz outperforms both pure-37.5Hz CFR and the 3kbps DAC baseline on Mel Distance, UTMOS, and STOI. Notably, certain mixed-granularity configurations at lower average frame rates (below 75Hz) achieve higher UTMOS scores than the 75Hz DAC+TFC configuration, indicating that forced fine-grained coding of low-entropy regions carries an unnecessary reconstruction cost.

> [!note]
> Evaluations are on LibriTTS test data only, with no human listening study. UTMOS, STOI, WER, and spectral distances are used as proxies for perceptual quality. Results are limited to a single codec backbone; behaviour on other architectures is not reported.

## Novelty Assessment

The VFR contribution is genuinely novel for neural acoustic codecs. Prior codec work has varied codebook depth per frame (VBR) but not the temporal span each frame covers. The entropy-based allocation heuristic draws on established methods from image compression and prior ASR research; its adaptation to acoustic tokens is the technical contribution rather than the underlying algorithm. The hierarchical encoder–decoder design is similarly adapted from image compression literature. The paper is honest about its scope limitations: it tests only the DAC backbone, uses no human evaluation, and defers ablations over architecture choices to future work. The impact on sequence length reduction is demonstrated cleanly at matched bitrates, which is the central claim and is fairly evaluated.

## Field Significance

Moderate — TFC addresses a structural limitation of current neural codec design that has direct downstream consequences for autoregressive speech generation: excessively long token sequences. The approach is principled, cleanly separates bitrate control (codebook count) from sequence-length control (frame rate), and is designed to compose orthogonally with other low-frame-rate codec techniques. Its significance depends on whether the codec community adopts VFR as a design axis, which the single-backbone evaluation cannot yet establish.

## Claims

- Constant-frame-rate coding introduces temporal redundancy in neural speech codecs by allocating equal resolution to silence and phonetically dense regions alike. *(§1, §2.1)*
- Dynamically allocating coarser temporal frames to low-entropy speech regions reduces token sequence length without proportional degradation in reconstruction quality. *(§4.2, Table 1)*
- Reducing the number of encoded frames at equivalent bitrate can improve intelligibility, suggesting that sequence compactness benefits autoregressive downstream models independently of bitrate. *(§4.2, Table 1)*
- Variable frame rate allocation and variable bitrate control are orthogonal axes in neural codec design and can be combined additively. *(§3, §5)*

## Limitations and Open Questions

> [!warning]
> Evaluation is restricted to a single codec backbone (DAC) and a single dataset (LibriTTS). No human listening study is reported; all quality judgements rest on UTMOS, STOI, WER, and spectral distances. Generalisability to other architectures or acoustic conditions is untested.

The paper does not report downstream TTS or speech LM experiments, so the claimed benefit of shorter token sequences for generation quality and latency is prospective rather than demonstrated. The entropy-based frame allocation heuristic uses fixed hyperparameters (bin count N, smoothing σ) without ablation; sensitivity to these choices is unknown. Training with mixed granularity ratios uses a fixed heuristic allocation that may not be optimal. The granularity ratios at inference must be set by the user; no automatic target-rate optimisation is described.

## Wiki Connections

Concept pages: [[neural-codec]] · [[autoregressive-codec-tts]] · [[streaming-tts]] · [[spoken-language-model]]

In-corpus papers cited: [[2412.17048]]
