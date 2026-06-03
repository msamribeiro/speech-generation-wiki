---
id: interspeech-2025-2447
title: "Accelerating Autoregressive Speech Synthesis Inference With Speech Speculative Decoding"
authors: ["Zijian Lin", "Yang Zhang", "Yougen Yuan", "Yuming Yan", "Jinjiang Liu", "Zhiyong Wu", "Pengfei Hu", "Qun Yu"]
organization: Tsinghua University / Tencent
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-06-03
task: [TTS]
architecture: [autoregressive-LM]
conditioning: [text-conditioned, zero-shot, speaker-conditioned]
training: [fine-tuning, supervised]
model_size: "draft model: ~8-layer Transformer subset of CosyVoice 2 (0.5B base)"
codec_used: "not reported"
datasets_train: ["LibriTTS (580h, 2306 speakers)"]
datasets_eval: ["LibriTTS test-clean (500 utterances, 37 speakers)"]
metrics:
  - name: WER
    value: 5.7
    system: SSD (β=0.4)
    testset: LibriTTS test-clean
  - name: WER
    value: 3.67
    system: CosyVoice 2 (baseline)
    testset: LibriTTS test-clean
  - name: MOS
    value: 3.925
    system: SSD (β=0.4)
    testset: LibriTTS test-clean
  - name: MOS
    value: 3.930
    system: CosyVoice 2 (baseline)
    testset: LibriTTS test-clean
  - name: SPK-SIM
    value: 0.63
    system: SSD (β=0.4)
    testset: LibriTTS test-clean
  - name: SPK-SIM
    value: 0.62
    system: CosyVoice 2 (baseline)
    testset: LibriTTS test-clean
code_available: null
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/lin25h_interspeech.html"
related_concepts: [autoregressive-codec-tts, streaming-tts, neural-codec, evaluation-metrics, zero-shot-tts]
related_papers: ["2301.02111", "2406.02430", "2407.05407", "2412.10117", "2410.06885"]
field_significance:
  level: moderate
  type: [architectural-novelty, engineering-integration]
generation:
  date: 2026-06-03
  model: claude-sonnet-4-6
  commit: "55d0339"
---

> [!abstract] Interspeech · 2025 · Conference
> **Zijian Lin et al.** (Tsinghua University / Tencent) · [→ Paper](https://www.isca-archive.org/interspeech_2025/lin25h_interspeech.html) · Demo: ? · Code: ?
>
> Speech Speculative Decoding (SSD) adapts speculative decoding from text LLMs to autoregressive TTS, using a lightweight draft model initialised from the target model's upper layers to propose candidate speech tokens that the full target model then verifies in parallel, achieving 1.4× speedup on CosyVoice 2 with negligible quality degradation.

## Problem

Autoregressive LM-based TTS systems achieve strong naturalness and zero-shot speaker adaptation, but their token-by-token generation is latency-bound. The LM stage accounts for roughly 70% of total synthesis time in systems like CosyVoice 2. Prior speculative decoding methods for speech (e.g., those presented at ICASSP 2025) embed multiple draft heads directly into the target model, requiring fine-tuning of the target model itself. The absence of an independent, drop-in draft model that operates without modifying the frozen target constrains practical deployment flexibility.

## Method

SSD operates by first using a lightweight draft model to autoregressively generate a fixed-length sequence of candidate speech tokens (draft length L_d = 3 in the reported configuration), then passing those tokens to the frozen target model for a single parallel forward pass that scores all candidate positions simultaneously. Acceptance or rejection at each position uses a modified criterion: a tolerance factor β ≥ 0 relaxes the standard ratio-based threshold, admitting acoustically plausible token substitutions that strict token-level equality would reject.

![The Inference Architecture of proposed Speech Speculative Decoding (SSD) framework. The dashed line depicts autoregressive decoding in the draft model, followed by parallel verification by the target model. The prefix comprises all prompt tokens and previously generated tokens during inference. Green indicates accepted tokens, while red represents rejected tokens.](assets/interspeech-2025-2447/figure-3.png)

The draft model is constructed as a shallower Transformer (8 layers) sharing the architecture of CosyVoice 2 (24-layer Transformer from Qwen2.5 0.5B). Initialisation uses the pre-trained upper layers (layers 19–24) of the target, giving immediate vocabulary alignment. During training on LibriTTS (580h), only the bottom two Transformer layers and the classification head are updated; all inherited upper layers and embeddings remain frozen. This selective fine-tuning is roughly analogous to adapter tuning: the draft model learns speech-specific generation patterns in its trainable lower layers while the frozen upper layers preserve the semantic representations of the full target model. Cross-entropy loss on speech token sequences is the sole training objective. The tolerance factor β is set to 0.4 in the main evaluation, and L_d = 3 is chosen based on the trade-off analysis in Figure 3 (stored as figure-4.png in raw assets).

## Key Results

On the LibriTTS test-clean evaluation set (500 utterances, 37 unseen speakers), SSD with β = 0.4 reduces LM-RTF from 0.504 (CosyVoice 2 baseline) to 0.36, a 1.4× speedup. Subjective quality is preserved: NAT-MOS is 3.925 ± 0.139 vs. 3.930 ± 0.147 for the baseline, a difference well within rater variance. SIM-MOS is 3.784 ± 0.152 vs. 3.789 ± 0.148. WER rises from 3.67% to 5.70%, which the authors attribute to draft model training on data roughly 1/300th the size of CosyVoice 2's proprietary training corpus rather than to the speculative decoding mechanism itself — the draft model in isolation reaches 16.13% WER, whereas SSD's verification keeps it at 5.70%.

The tolerance factor ablation (Table 3) shows that varying β from 0 to 0.4 consistently improves LM-RTF while quality metrics (NMOS, SS) remain stable, suggesting that speech token identity is weakly coupled to perceptual quality — a key property that makes the relaxed acceptance criterion viable.

> [!note]
> The WER increase from 3.67% to 5.70% is notable and should be interpreted cautiously: the draft model is trained on open-source data only while CosyVoice 2 was trained on a large proprietary corpus. Whether the WER gap narrows with in-domain or larger-scale draft training is not tested.

## Novelty Assessment

The core algorithmic contribution is the adaptation of speculative decoding to the speech domain, specifically the tolerance-factor acceptance criterion that exploits the many-to-one mapping between speech token sequences and perceived speech quality. This is a genuine insight: because perceptually similar speech can arise from distinct token sequences, strict distributional acceptance is unnecessarily conservative.

The draft model construction strategy — initialising from the target's upper layers and fine-tuning only the lower layers — is an efficient engineering choice rather than a novel technique; selective layer fine-tuning is well established. The framing as an independent, non-invasive drop-in draft model is more practically useful than prior work requiring target model modification, though the acceleration is modest at 1.4×.

Overall, the contribution is primarily engineering integration with a targeted algorithmic adaptation (the tolerance factor). It does not propose a new architecture for speech generation itself; the improvement is to inference-time efficiency of existing autoregressive systems.

## Field Significance

Moderate — SSD demonstrates that speculative decoding can be applied to speech synthesis with only a small, independently-trained auxiliary model, achieving inference speedup without modifying the target system. The tolerance-factor finding — that speech tokens have a weaker perceptual one-to-one identity than text tokens, enabling relaxed acceptance criteria — is a useful insight for future work on efficient autoregressive speech generation. The speedup (1.4×) is real but modest, and the WER increase is a limitation that warrants further investigation.

## Claims

- Speculative decoding adapted for speech can reduce autoregressive inference latency without measurable degradation in subjective naturalness or speaker similarity.
- Speech token sequences exhibit many-to-one mappings to perceived quality, enabling relaxed acceptance criteria that improve decoding throughput over strict token-distribution matching.
- Initialising a lightweight draft model from the upper layers of the target model provides immediate vocabulary alignment and reduces the data requirements for draft model training.
- Inference-stage acceleration of autoregressive TTS is achievable without fine-tuning the target model, preserving deployment flexibility for frozen production systems.

## Limitations and Open Questions

> [!warning]
> The WER increase (3.67% → 5.70%) is unexplained beyond a data-scale hypothesis. The draft model's limited training data (LibriTTS, ~580h vs. CosyVoice 2's proprietary corpus) is identified as the likely cause, but this is not verified experimentally — e.g., by scaling draft training data or by ablating with matched data.

Evaluation is restricted to a single target model (CosyVoice 2) on a single English benchmark. Generalisation of SSD to multilingual systems, streaming inference contexts, or multi-codebook AR models (e.g., VALL-E-style RVQ decoding) is not explored. The tolerance factor β is treated as a fixed hyperparameter tuned on objective metrics; its interaction with speaker diversity and domain shift is unexamined. The reported 1.4× speedup measures LM-RTF only and does not account for the draft model's own compute overhead in the total pipeline time.

## Wiki Connections

This paper contributes evidence to [[autoregressive-codec-tts]] by demonstrating a practical inference-time acceleration strategy for LM-based TTS. The tolerance-factor criterion relates to [[neural-codec]] and how codec token discretisation affects perceptual quality. The zero-shot evaluation setup connects to [[zero-shot-tts]].

In-corpus papers this work builds on: [[2301.02111]] (VALL-E, the foundational neural codec LM for TTS), [[2406.02430]] (Seed-TTS), [[2407.05407]] (CosyVoice), [[2412.10117]] (CosyVoice 2, the target model), [[2410.06885]] (F5-TTS, cited as a flow-matching alternative context).
