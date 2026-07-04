---
id: "interspeech-2025-2328"
title: "A Watermark for Auto-Regressive Speech Generation Models"
authors: ["Yihan Wu", "Ruibo Chen", "Georgios Milis", "Junfeng Guo", "Heng Huang"]
organization: "University of Maryland"
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-07-05
task: ["TTS", "evaluation"]
architecture: ["autoregressive-LM"]
conditioning: []
training: []
model_size: "not reported"
codec_used: "not reported"
datasets_train: ["not reported"]
datasets_eval: ["C4 (subset)", "Dolly CW", "MMW Story", "WaterBench (LongForm QA)"]
metrics:
  - name: FAD
    value: 0.0067
    system: DiP-reweight (α=0.4)
    testset: C4 subset
  - name: FAD
    value: 0.0051
    system: no watermark
    testset: C4 subset
code_available: null
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/wu25k_interspeech.html"
related_concepts: ["autoregressive-codec-tts", "spoken-language-model", "evaluation-metrics"]
related_papers: []
field_significance:
  level: "moderate"
  type: ["engineering-integration", "evaluation-contribution"]
generation:
  date: 2026-07-05
  agent: speech-generation-ingest-agent
  model: claude-sonnet-4-6
  commit: "318f1b4"
---

> [!abstract] Interspeech · 2025 · Conference
> **Wu et al.** (University of Maryland) · [→ Paper](https://www.isca-archive.org/interspeech_2025/wu25k_interspeech.html) · Demo: ? · Code: ?
>
> Introduces the first statistical watermarking framework for autoregressive speech generation models, adapting token-distribution reweighting from text LLM watermarking to the speech token vocabulary and characterising the re-encoded mismatch as the primary practical obstacle to reliable post-hoc detection.

## Problem

Autoregressive speech generation models produce highly realistic synthetic speech that can be misused for impersonation, fabrication of audio evidence, and automated voice scams. Existing post-hoc audio watermarking methods embed signals directly into the waveform, which degrades output quality and provides no statistical confidence measure that detection is accurate. Statistical watermarking, which operates on the token distribution during generation and yields rigorous hypothesis tests, had been developed for text language models but had never been applied to speech generation systems. Speech introduces a unique complication absent in text: the audio must be re-encoded into tokens for detection, and this encode-decode-encode round-trip does not perfectly recover the original token sequence.

## Method

The framework adapts two statistical watermarking methods from the text LLM literature (KGW and DiPmark) to autoregressive speech token generation. Both methods work by manipulating the token probability distribution at each generation step rather than post-processing the waveform.

At generation time, a secret key and the preceding n-gram of generated speech tokens are used to pseudo-randomly partition the token vocabulary into red and green lists. The KGW reweight strategy boosts the logit of green-list tokens by a fixed increment delta, increasing their selection probability; higher delta values produce stronger statistical signals but greater distortion. The DiP (DiPmark) reweight strategy is distortion-free: it reorders token probabilities within the [0,1] interval according to their position in a permutation, raising the cumulative probability mass of green-list tokens without altering the original sampling distribution. This preserves the generative model's output quality at the cost of a weaker statistical bias. Both algorithms are described in Algorithms 1 and 2 in the paper.

For detection, the generated audio is re-encoded into speech tokens using the model's encoder. A hypothesis test then counts the fraction of green-list tokens in the re-encoded sequence and applies a z-score statistic based on a Hoeffding concentration bound, rejecting the null hypothesis at a calibrated false positive rate. The paper evaluates this framework using SpiritLM (a speech-text LM that generates interleaved HuBERT-based speech units) as the underlying generation model, testing across four datasets: a C4 subset, Dolly CW, an MMW Story set, and a LongForm QA task from WaterBench. (§3, §4)

The key practical finding is the re-encoded mismatch: because the audio encoder does not perfectly invert the decoder, the token sequence recovered during detection diverges from the sequence generated under the watermark. This mismatch degrades the statistical signal, often substantially. (§3.2)

## Key Results

KGW reweight achieves the highest detection rates. At δ=2.0 on the C4 subset, TPR@1%FPR reaches 98.9%; at δ=1.5 it reaches 92.7% (Table 1). On the LongForm QA dataset, δ=2.0 reaches 99.5% TPR@1%FPR (Table 3). However, the FAD score rises from 0.0051 (no watermark) to 0.0494 at δ=2.0 on C4, indicating meaningful distortion.

DiP reweight is distortion-free by design. Its FAD scores remain close to the no-watermark baseline (0.0067 vs. 0.0051 on C4 at α=0.4), but peak detection is weaker, reaching only 47.5% TPR@1%FPR on C4 with α=0.4. On the Dolly-CW dataset, which shows generally lower detection rates, DiP-reweight at α=0.5 achieves 54% TPR@1%FPR.

The re-encoded mismatch effect is severe. Before re-encoding, KGW (δ=1.0) achieves 97.1% TPR@1%FPR on C4; after re-encoding, this drops to 54.3% (Table 5). On Dolly-CW, the same configuration falls from 99% to 17% (Table 6). DiP-reweight shows similar degradation: α=0.4 drops from 80.0% to 47.5% before and after re-encoding on C4, and from 99% to 53% on Dolly-CW. The mismatch is thus the dominant practical bottleneck for deployed detection. (§5.2)

## Novelty Assessment

The primary contribution is engineering integration: KGW and DiPmark are established text LLM watermarking methods, and this paper applies them to speech tokens without structural modification to either the watermarking algorithms or the speech generation model. The interface between the two is straightforward (treat speech tokens as text tokens for the purpose of reweighting). What is genuinely novel is the identification, formalisation, and empirical characterisation of the re-encoded mismatch phenomenon, which is specific to speech and has no direct analogue in text LLM watermarking. The experiments are conducted on a single speech LM (SpiritLM) and cover only HuBERT-based discrete unit representations; robustness to neural codec-based models (EnCodec, Mimi, etc.) is untested. No mitigation strategy for the re-encoded mismatch is proposed.

## Field Significance

Moderate — This paper provides the first published application of statistical watermarking to the speech generation domain, establishing a proof of concept and identifying the re-encoded mismatch as the main technical obstacle. The contribution is primarily framing and evaluation rather than algorithmic novelty. It enables future work to directly address the mismatch problem with targeted solutions, and provides the field with a principled, quality-preserving alternative to post-hoc waveform-level watermarking.

## Claims

- **supports:** Statistical watermarking techniques developed for text language models can be applied to autoregressive speech generation models by treating speech tokens as text tokens and applying token-distribution reweighting during generation.
  > *Evidence:* KGW and DiPmark are adapted to SpiritLM's HuBERT-based speech token vocabulary with no architectural modification to either system, achieving TPR@1%FPR up to 98.9% on C4. *(§4, Table 1)*

- **complicates:** Statistical watermark detection in autoregressive speech systems is degraded by the lossy round-trip of audio decode followed by re-encode, which disrupts the token-level statistical signal.
  > *Evidence:* On Dolly-CW, KGW (δ=1.0) drops from 99% to 17% TPR@1%FPR after re-encoding; DiP-reweight (α=0.4) drops from 99% to 53%, demonstrating that the mismatch is the dominant failure mode regardless of watermark method. *(§3.2, §5.2, Table 6)*

- **supports:** Distortion-free watermarking methods preserve generated audio quality at the cost of reduced detection power compared to logit-bias methods.
  > *Evidence:* DiP-reweight maintains FAD scores near the no-watermark baseline (0.0067 vs. 0.0051 on C4 at α=0.4) while KGW (δ=2.0) raises FAD to 0.0494; correspondingly, DiP-reweight peaks at 47.5% vs. 98.9% TPR@1%FPR. *(§5.1, Tables 1–4)*

- **complicates:** The strength of a statistical watermark signal before audio rendering is not a reliable predictor of detection accuracy in deployed conditions, because re-encoding introduces a new and substantial source of signal loss.
  > *Evidence:* Before re-encoding, KGW (δ=1.0) achieves 97.1% TPR@1%FPR on C4; after re-encoding this falls to 54.3%, illustrating that near-perfect pre-decoding detection does not guarantee practical detectability. *(§5.2, Table 5)*

## Limitations and Open Questions

> [!warning]
> The framework is evaluated only on SpiritLM, which uses HuBERT-based discrete speech units. Applicability to neural codec-based AR speech models (e.g., those using EnCodec or Mimi) is untested, and re-encoded mismatch characteristics may differ substantially across codec types.

No mitigation for the re-encoded mismatch is proposed; the paper frames this as an open problem without a solution pathway. The evaluation uses Frechet Audio Distance to assess distortion but does not include subjective listening tests, leaving the perceptual impact of the KGW-reweight quality degradation unverified. Robustness against adversarial attacks on the watermark (e.g., re-synthesis, noise injection) is not evaluated.

## Wiki Connections

- [[autoregressive-codec-tts|Autoregressive Codec TTS]] — the watermarking framework targets autoregressive speech generation models and is specifically designed to work at the token distribution level of such systems.
- [[spoken-language-model|Spoken Language Model]] — the primary test model is SpiritLM, a speech-text LM that generates interleaved discrete speech units, providing the experimental setting for all watermark evaluations.
- [[evaluation-metrics|Evaluation Metrics]] — the paper uses TPR at controlled FPR thresholds as the primary detection metric and FAD score to measure distortion, establishing an evaluation protocol for speech watermark assessment.
