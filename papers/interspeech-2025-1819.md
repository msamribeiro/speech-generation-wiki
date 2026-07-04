---
id: "interspeech-2025-1819"
title: "Comparative Analysis of Fast and High-Fidelity Neural Vocoders for Low-Latency Streaming Synthesis in Resource-Constrained Environments"
authors: ["Reo Yoneyama", "Masaya Kawamura", "Ryo Terashima", "Ryuichi Yamamoto", "Tomoki Toda"]
organization: null
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-07-04
task: [TTS]
architecture: [GAN]
conditioning: [text-conditioned]
training: [supervised]
model_size: "0.332M params (MS-Wavehax)"
codec_used: "none"
datasets_train: ["JVS corpus", "JSUT corpus"]
datasets_eval: ["JVS corpus", "JSUT corpus"]
metrics:
  - name: MOS
    value: "not reported"
    system: MS-Wavehax (non-causal)
    testset: JVS corpus (analysis-synthesis)
  - name: MOS
    value: "not reported"
    system: MS-Wavehax (non-causal)
    testset: JSUT corpus (TTS)
code_available: null
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/yoneyama25_interspeech.html"
related_concepts: [gan-vocoder, streaming-tts, evaluation-metrics, subjective-evaluation]
related_papers: []
field_significance:
  level: moderate
  type: [architectural-novelty, empirical-benchmark]
generation:
  date: 2026-07-04
  agent: speech-generation-ingest-agent
  model: claude-sonnet-4-6
  commit: "318f1b4"
---

> [!abstract] Interspeech · 2025 · Conference
> **Yoneyama et al.** · [→ Paper](https://www.isca-archive.org/interspeech_2025/yoneyama25_interspeech.html) · Demo: ? · Code: ?
>
> MS-Wavehax extends the Wavehax vocoder with input-side multi-stream decomposition via analysis filters, achieving the highest throughput under low-latency streaming conditions (<80 ms) at only 2.4% of HiFi-GAN V1's parameter count while reaching top MOS in both analysis-synthesis and TTS evaluations.

## Problem

Neural vocoder research predominantly optimises for batch inference, reporting real-time factors averaged over seconds-long utterances on a GPU. This evaluation paradigm misses the performance realities of low-latency streaming deployment: limited parallelism within small processing chunks, cache and parameter loading overhead that scales with model complexity, and the stricter causality requirements of live applications such as voice assistants and real-time voice conversion. Faster batch inference does not translate directly to lower streaming latency or higher streaming throughput, and most vocoders are designed without considering these CPU-only, resource-constrained scenarios.

## Method

MS-Wavehax (multi-stream Wavehax) modifies the Wavehax architecture by introducing an analysis-filter-based input decomposition before the convolutional processing stage. The input waveform is first split into four subscale signals by a learned analysis filter (length 63). Each subscale undergoes a short-time Fourier transform to produce a two-channel complex spectrogram. These four subscale spectrograms are then concatenated with projected mel-spectrogram features, forming a 12-channel input representation. A series of ConvNeXt-based 2D convolutional layers and residual blocks processes this representation, using 64 hidden channels (twice the standard Wavehax) while reducing the frequency dimension to one-quarter. The final pointwise Conv2D layer outputs eight spectrograms (real and imaginary parts for four subscale signals), which are converted back to the time domain via iSTFT and merged by a synthesis filter.

This approach differs from prior multi-stream vocoders (such as MS-HiFi-GAN) which decompose only the output. Decomposing the input instead partitions full-band spectrograms into sub-spectrograms with locally consistent spatial structure, enabling convolutional kernels to capture within-stream and cross-stream dependencies more efficiently. The result is approximately half the computational cost of standard Wavehax at 0.332M parameters.

Two key streaming adaptations are applied universally: caching mechanisms using ring buffers for all convolutional, STFT, and iSTFT layers (eliminating redundant computation across chunks), and replacement of layer normalization with batch normalization. Layer normalization requires computing statistics over the entire sequence and is incompatible with incremental processing; batch normalization has no such requirement and also improved objective metrics (PESQ, UTMOS) in internal ablations.

The paper also proposes MS-Vocos, which applies multi-stream synthesis to Vocos using a channel-split final layer, but this variant underperforms standard Vocos in quality evaluations. Both MS-Wavehax and MS-Vocos are evaluated against HiFi-GAN, iSTFTNet, Vocos, and Wavehax (including their multi-stream variants), all adapted for streaming on a single-thread AMD EPYC CPU.

![Figure 1: An overview of multi-stream Wavehax. The harmonic prior waveform is generated from the input fundamental frequencies via signal processing. The lengths of the analysis and synthesis filters are both 63. The frame length and frame shift for STFT/iSTFT are 20 ms and 10 ms, respectively. The numbers on the right represent the tensor shapes at each processing stage.](assets/interspeech-2025-1819/figure-2.png)

## Key Results

MS-Wavehax achieves the lowest streaming RTF (highest throughput) among all evaluated vocoders at small chunk sizes corresponding to latencies below 80 ms. At this regime it benefits from having the fewest parameters (0.332M) and lowest multiply-accumulate operations in ONNX-exported form (2.291B MACs for one second of audio, vs. 28.02B for HiFi-GAN V1). Wavehax places second; the RTF ordering reverses at larger chunk sizes, where Vocos overtakes because its 1D convolutional filters have constant matrix size and higher cache reuse compared to Wavehax's 2D convolutional kernels, which require growing im2col matrices as chunk size increases.

In subjective quality evaluation, non-causal MS-Wavehax and causal MS-Wavehax with a single-frame lookahead (LA=1) both achieve the highest MOS across all models in both the analysis-synthesis task (JVS corpus, 100 Japanese speakers) and the TTS task (JSUT corpus, single Japanese female speaker). Specific MOS values are shown in Figure 4 of the paper and are not reproduced in the prose; differences across systems are smaller in TTS than in A/S, likely because the single-speaker setting reduces the difficulty of vocoder conditioning.

Causal MS-Wavehax with LA=0 shows a quality gap relative to LA=1 in the multi-speaker A/S task, but this gap disappears entirely in TTS. The multi-stream variants of HiFi-GAN and iSTFTNet also outperform their non-MS counterparts, consistent with prior work. MS-Vocos is the exception, underperforming standard Vocos; the authors attribute this to a latent representation that is narrower than the output complex spectrum, making inter-stream dependency modeling difficult.

## Novelty Assessment

The principal novelty in MS-Wavehax is input-side decomposition: prior multi-stream vocoders (MS-HiFi-GAN, the MS variant of iSTFTNet) decompose the output signal, while MS-Wavehax decomposes the input waveform via a learned analysis filter before the STFT stage. This produces sub-spectrograms with locally consistent frequency structure, which is a meaningful inductive bias for 2D convolutional processing. The combination of this decomposition with Wavehax's harmonic prior and 2D ConvNeXt backbone is not simply a repackaging of existing components.

The second contribution is the streaming latency-throughput analysis using im2col matrix sizes to explain the chunk-size dependence of RTF. This analytical framework is practical and transferable to vocoder design decisions in other contexts. The experimental setup on a single-thread CPU reflects realistic edge-deployment constraints that are underrepresented in vocoder benchmarks.

The work is incremental in the sense that each component (multi-stream decomposition, Wavehax, ConvNeXt, caching mechanisms) exists in prior literature. The originality lies in the analysis-filter input decomposition and the empirical framework for streaming evaluation.

## Field Significance

Moderate — this paper addresses a genuine gap between how vocoders are typically evaluated and how they perform in real streaming deployments. The latency-throughput analysis using im2col matrix size as an explanatory variable provides a principled design framework that extends to other streaming vocoder selection decisions. MS-Wavehax's combination of top quality and extreme compactness (0.332M params) makes it a practically relevant system for resource-constrained and edge applications.

## Claims

- **supports:** Input-side multi-stream decomposition via analysis filters can improve both quality and efficiency in harmonic-prior 2D convolutional vocoders.
  > *Evidence:* MS-Wavehax splits the input waveform into four subscale signals before STFT, producing sub-spectrograms with locally consistent spatial structure; it achieves the highest MOS in both analysis-synthesis and TTS tasks while requiring roughly half the MACs of standard Wavehax and only 0.332M parameters. *(§2.2, §4.2)*

- **supports:** Batch-inference real-time factor is a poor proxy for streaming throughput; architecture choices interact with chunk size and cache behaviour in ways that can reverse the efficiency ranking.
  > *Evidence:* At small chunk sizes MS-Wavehax achieves the lowest RTF among all vocoders; at larger chunk sizes Vocos overtakes it because 1D filter matrices are constant-sized and cache-resident, while Wavehax's 2D im2col matrices grow with chunk size T, causing more frequent main-memory access. *(§3.2, Table 2, Figure 2)*

- **complicates:** Multi-stream decomposition does not uniformly improve quality across all frame-level vocoder architectures.
  > *Evidence:* MS-Vocos underperforms standard Vocos in MOS evaluation; the authors attribute this to a latent representation narrower than the output complex spectrum, which makes estimating inter-stream dependencies particularly difficult. *(§4.2)*

- **supports:** Replacing layer normalization with batch normalization in GAN vocoders enables streaming compatibility and can also yield objective quality improvements.
  > *Evidence:* Layer normalization computes statistics over the full sequence and is incompatible with incremental streaming; switching to batch normalization removed this constraint and improved PESQ and UTMOS scores in internal experiments on all tested vocoder configurations. *(§2.3)*

- **supports:** Causal vocoder processing with minimal lookahead can match non-causal quality in single-speaker TTS settings.
  > *Evidence:* Causal MS-Wavehax with a one-frame lookahead achieves nearly identical MOS to non-causal MS-Wavehax on the JSUT TTS task; the gap observed in the multi-speaker A/S task disappears, suggesting that causal constraints function as an effective regularizer in single-speaker conditions. *(§4.2, Figure 4)*

## Limitations and Open Questions

Evaluations are conducted exclusively on Japanese speech corpora (JVS and JSUT) on a single CPU architecture (AMD EPYC), so generalisation to multilingual or GPU-accelerated deployment is not demonstrated. All models were conditioned on mel-spectrograms extracted from clean studio recordings; behaviour on noisy or spontaneous speech is not assessed. The paper does not include a real-time demo or end-to-end latency measurement with an acoustic model frontend, so the practical time-to-first-audio-byte in a complete TTS pipeline remains uncharacterised.

## Wiki Connections

- [[gan-vocoder|GAN Vocoder]] — MS-Wavehax inherits from the HiFi-GAN vocoder family and is trained with the UnivNet GAN discriminator; this paper extends the line of GAN vocoders toward streaming and edge-deployment efficiency.
- [[streaming-tts|Streaming TTS]] — the central topic; provides a latency-throughput analytical framework and proposes a streaming-optimized vocoder architecture with explicit chunk-size-aware design decisions.
- [[evaluation-metrics|Evaluation Metrics]] — introduces RTF-vs-chunk-size profiling as a practical streaming evaluation methodology, complementing the standard batch-RTF metric and explaining performance inversions across architectures.
- [[subjective-evaluation|Subjective Evaluation]] — conducts five-point MOS listening tests with 30 native Japanese speakers across nine vocoder configurations in both analysis-synthesis and TTS conditions.
