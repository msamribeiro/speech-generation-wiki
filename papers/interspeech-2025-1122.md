---
id: "interspeech-2025-1122"
title: "BitTTS: Highly Compact Text-to-Speech Using 1.58-bit Quantization and Weight Indexing"
authors: ["Masaya Kawamura", "Takuya Hasumi", "Yuma Shirahata", "Ryuichi Yamamoto"]
organization: LY Corporation
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-06-03
task: [TTS]
architecture: [GAN, transformer-enc-dec]
conditioning: [text-conditioned, speaker-conditioned]
training: [supervised]
model_size: "not reported"
codec_used: "none"
datasets_train: ["LibriTTS-R"]
datasets_eval: ["LibriTTS-R"]
metrics:
  - name: MOS
    value: 3.75
    system: 32-bit baseline
    testset: LibriTTS-R
  - name: MOS
    value: 3.09
    system: 1.58-bit (acoustic + vocoder quantized, weight indexing)
    testset: LibriTTS-R
  - name: MOS
    value: 3.30
    system: 1.58-bit (acoustic only quantized, weight indexing)
    testset: LibriTTS-R
  - name: MOS
    value: 1.20
    system: 32-bit small model (channel-reduced baseline)
    testset: LibriTTS-R
code_available: null
demo_available: true
url: "https://www.isca-archive.org/interspeech_2025/kawamura25_interspeech.html"
related_concepts: ["streaming-tts", "evaluation-metrics", "subjective-evaluation", "transformer-enc-dec-tts", "gan-vocoder"]
related_papers: []
field_significance:
  level: moderate
  type: [engineering-integration]
generation:
  date: 2026-06-03
  model: claude-sonnet-4-6
  commit: "55d0339"
---

> [!abstract] Interspeech · 2025 · Conference
> **Kawamura et al.** (LY Corporation) · [→ Paper](https://www.isca-archive.org/interspeech_2025/kawamura25_interspeech.html) · Demo: ✓ · Code: ?
>
> BitTTS applies 1.58-bit quantization-aware training and a novel weight indexing scheme to a lightweight JETS-based TTS model, achieving 83% model size reduction while exceeding the quality of a naive channel-reduced baseline of comparable size.

## Problem

On-device TTS applications — car navigation, conversational bots — require models that fit within tight memory and storage constraints on low-end hardware. Naive model shrinkage via channel reduction degrades quality severely, while prior work on quantization in TTS has focused mostly on 8-bit or 4-bit precision. Extremely low-precision quantization (1–1.58 bit), which has shown promise in language modelling and computer vision, had not been investigated for TTS. A further practical obstacle is that hardware uniformly addresses values in 8-bit units, so ternary weights stored naively still consume 8 bits each, eliminating most of the theoretical size benefit.

## Method

BitTTS builds on JETS (Jointly training FastSpeech2 and HiFi-GAN end-to-end), replacing the transformer encoder and decoder blocks with four 1D convolutional layers of kernel size 5 to reduce compute. The HiFi-GAN vocoder is retained. Channels are set to 256 (encoder), 192 (decoder), and 128 (HiFi-GAN initial channel). Quantization-aware training (QAT) is applied following the BitNet b1.58 formulation: weights are scaled by their average absolute value, then clipped and rounded to ternary values {-1, 0, 1}. Activations are normalised with layer normalisation before scaling to the quantisation range. Gradients flow through the non-differentiable rounding and clipping via the straight-through estimator.

The key engineering contribution is weight indexing. Because hardware stores values in 8-bit units, each ternary weight naively occupies 8 bits despite holding only 1.58 bits of information. Weight indexing addresses this by grouping every 5 consecutive ternary weights (L*=5) and encoding the combination as a single int8 index: since 3^5 = 243 < 256 = 2^8, every five weights map to one byte with negligible wastage. At inference, the original ternary weights are reconstructed from the index table before computation. Quantisation is applied only to 1D convolutional layers (>90% of model weight) and the final waveform-output layer in the vocoder is excluded, as preliminary experiments showed that layer is particularly sensitive to precision loss. Three configurations are evaluated: quantising both acoustic model and vocoder, acoustic model only, and vocoder only.

Training uses LibriTTS-R (475.61 hours, 2,456 speakers at 24 kHz) with AdamW, an exponential learning rate schedule, and dynamic batching for 2,000K steps on an NVIDIA A100.

## Key Results

The 32-bit base model (25.66 MB) scores 3.75 MOS. A channel-reduced 32-bit small model at 12.78 MB collapses to 1.20 MOS, confirming that naive parameter reduction is ineffective at this scale. 1.58-bit quantization with weight indexing applied to both the acoustic model and vocoder reaches 4.39 MB (83% reduction from the base) at 3.09 MOS — substantially higher than the same-size 32-bit small model and comparable to 4-bit quantization in most conditions. Quantising only the acoustic model with weight indexing yields 7.60 MB at 3.30 MOS and RTF of 0.040 on Apple M1 Pro CPU, nearly matching the base model's RTF of 0.042. The vocoder is consistently more sensitive to quantisation than the acoustic model: 4-bit vocoder quantisation (2.57 MOS) performs worse than 1.58-bit vocoder quantisation (3.18 MOS), likely reflecting training instability in the 4-bit vocoder run rather than a principled ordering.

> [!note]
> The 15-rater MOS listening test on 30 utterances is small by field standards; confidence intervals are ±0.09–0.11 MOS, so modest differences between quantisation configurations should be treated with caution. All systems are trained on LibriTTS-R with a single architecture; generalisability to other TTS architectures and on-device hardware beyond Apple M1 Pro is not demonstrated.

## Novelty Assessment

The contribution is primarily engineering integration. Quantisation-aware training and ternary weights are established in language modelling (BitNet b1.58) and computer vision; this paper is the first to systematically evaluate them for lightweight TTS. The weight indexing scheme is a simple but practical fix to the hardware-alignment problem specific to ternary weights, requiring only an index lookup at inference. There is no new TTS architecture, training objective, or conditioning mechanism. The evaluation is honest about trade-offs between model size, quality, and inference speed, and the comparisons are fair (same architecture, same training data). The finding that naive channel reduction is far worse than quantisation at similar model sizes is the clearest actionable result.

## Field Significance

Moderate — This paper provides the first systematic evidence that 1.58-bit QAT is applicable to lightweight TTS without catastrophic quality loss, extending a trend from NLP and vision. The weight indexing technique directly solves a practical deployment barrier for ternary models on standard 8-bit hardware. The work is incremental relative to the quantisation literature at large, but fills a gap specific to the TTS compression space.

## Claims

- Quantisation-aware training is more effective than parameter reduction via channel shrinkage for achieving compact TTS models at equivalent model sizes.
- The vocoder stage of a TTS pipeline is more sensitive to aggressive weight quantisation than the acoustic model, making it a disproportionate source of quality degradation.
- Extremely low-bit (1–2 bit) weight quantisation can preserve intelligible and reasonably natural speech synthesis when applied selectively to convolutional layers.
- Hardware-aware packing schemes are necessary to realise the theoretical storage savings of sub-byte quantisation on devices that operate in 8-bit units.

## Limitations and Open Questions

> [!warning]
> All experiments use a single TTS architecture (JETS with convolutional blocks) trained on a single corpus (LibriTTS-R). Whether these findings generalise to transformer-heavy or autoregressive architectures, or to lower-resource languages, is not addressed.

The subjective evaluation uses only 15 raters and 30 utterances, which is at the lower end of statistical power for MOS comparisons. The index distribution is heavily non-uniform (indices 0, 121, 242 — representing all-zero, all-one, all-minus-one blocks — are overrepresented), suggesting that further compression via entropy coding (e.g. Huffman) is feasible but unexplored. Inference speed does not improve relative to the full 32-bit base model because weight reconstruction from indices adds overhead; achieving both compact storage and fast inference simultaneously requires dedicated hardware or kernel-level support that is not demonstrated.

## Wiki Connections

Related concepts: [[transformer-enc-dec-tts]], [[gan-vocoder]], [[streaming-tts]], [[evaluation-metrics]], [[subjective-evaluation]]
