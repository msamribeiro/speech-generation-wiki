---
id: "interspeech-2025-1536"
title: "Fairness in Dysarthric Speech Synthesis: Understanding Intrinsic Bias in Dysarthric Speech Cloning using F5-TTS"
authors: ["Anuprabha M", "Krishna Gurugubelli", "Anil Kumar Vuppala"]
organization: null
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-07-03
task: ["TTS", "evaluation"]
architecture: ["flow-matching"]
conditioning: ["zero-shot", "speaker-conditioned", "prompt-conditioned"]
training: ["supervised", "fine-tuning"]
model_size: "not reported"
codec_used: "not reported"
datasets_train: ["TORGO"]
datasets_eval: ["TORGO"]
metrics:
  - name: WER
    value: 36.66
    system: Wav2vec2.0 fine-tuned (reference + synthetic data)
    testset: TORGO (low severity)
  - name: WER
    value: 94.23
    system: Wav2vec2.0 fine-tuned (reference + synthetic data)
    testset: TORGO (high severity)
code_available: null
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/m25_interspeech.html"
related_concepts: ["flow-matching", "zero-shot-tts", "evaluation-metrics", "speaker-adaptation"]
related_papers: ["2025.acl-long.313"]
field_significance:
  level: "moderate"
  type: ["evaluation-contribution", "empirical-benchmark"]
generation:
  date: 2026-07-03
  agent: speech-generation-ingest-agent
  model: claude-sonnet-4-6
  commit: "f9e71e3"
---

> [!abstract] Interspeech · 2025 · Conference
> **Anuprabha M et al.** · [→ Paper](https://www.isca-archive.org/interspeech_2025/m25_interspeech.html) · Demo: ? · Code: ?
>
> Presents a fairness evaluation framework for zero-shot dysarthric speech cloning, showing that F5-TTS systematically over-normalizes intelligibility while better preserving speaker identity and prosody, with the bias worsening significantly for severe dysarthric cases.

## Problem

Dysarthric speech technologies suffer from acute data scarcity, making synthetic data augmentation via zero-shot voice cloning an appealing strategy. However, cloning systems trained predominantly on typical speech may treat dysarthric features as noise rather than speaker-identity signals, inadvertently correcting them. Prior work on synthetic dysarthric speech augmentation evaluated aggregate quality but did not examine whether cloning performance is uniform across dysarthric severity levels or gender groups. Without this analysis, augmenting training corpora with biased synthetic data can entrench disparities in downstream ASR and assistive communication systems, making them less useful precisely for the most severely impaired speakers.

## Method

The paper frames zero-shot dysarthric speech cloning as a fairness problem and proposes a two-stage evaluation framework. In the first stage, F5-TTS (a flow-matching, Diffusion-Transformer-based system) is used to clone speech from speakers in the TORGO database, spanning four dysarthric severity levels: healthy, low, mid, and high. F5-TTS generates each sample from a short audio reference prompt and the corresponding transcription, requiring no speaker-specific fine-tuning. Cloning quality is assessed with four objective metrics: WER and CER (intelligibility, via HuBERT-large ASR fine-tuned on LibriSpeech), SIM-o (speaker similarity, via cosine distance between WavLM-large speaker embeddings), and AutoPCP (prosody consistency, via a multilingual prosody comparator). To isolate how much the synthesized speech differs in intelligibility from the reference, the paper defines delta-WER (the difference between the audio prompt's WER and the generated audio's WER) and delta-CER analogously.

In the second stage, Parity Difference (PD) and Disparate Impact (DI) are computed for each objective measure across severity categories, comparing each dysarthric group against the healthy reference group. PD measures the absolute gap between groups; DI measures the ratio, with 1.0 indicating no disparity. For intelligibility metrics, softmax normalization is applied before computing DI to prevent inherently higher error rates in severe cases from distorting the ratio. The framework is then applied across gender to reveal additional bias axes.

A downstream impact experiment completes the study: Wav2vec2.0-Base is fine-tuned for ASR on TORGO, and a CNN classifier is trained for dysarthria detection, each under three data regimes: reference audio only, synthetic audio only, and the combination.

## Key Results

F5-TTS shows a pronounced intelligibility bias. For high-severity speakers, delta-WER DI drops to 0.59 and delta-CER DI to 0.75, both rated "poor" (Table 2), indicating that the synthesized speech is substantially more intelligible than the reference because the model smooths away the characteristic distortions of severe dysarthria. In contrast, speaker similarity (SIM-o DI 0.85) and prosody (AutoPCP DI 0.90) remain relatively stable across severity levels, suggesting the system preserves acoustic identity better than intelligibility content.

Gender analysis reveals non-uniform bias: male speakers face higher intelligibility disparity (delta-WER PD=0.37, DI=0.68), while female speakers show higher prosody disparity (AutoPCP PD=0.36). Low-severity speakers exhibit near-zero bias across all metrics (DI above 0.97).

In the downstream experiment, adding synthetic F5-TTS data alongside reference audio (model 3) improves dysarthria detection accuracy from 62.5% to 73.75% and reduces WER by 44.6% for low-severity speakers. However, it degrades ASR WER by 5.7% and 7.7% for mid- and high-severity speakers respectively, confirming that biased augmentation harms the populations most in need (Table 4, §4.3). Training on synthetic data alone (model 2) generalises poorly to real dysarthric speech across all severity levels.

## Novelty Assessment

The contribution is primarily methodological: adapting fairness metrics from algorithmic fairness (Disparate Impact, Parity Difference) to evaluate TTS cloning bias across speaker subgroups. Applying these metrics to dysarthric speech synthesis is new, and the TORGO-based evaluation framework provides a replicable setup for future fairness-aware work in this space. The underlying TTS system (F5-TTS) and all evaluation tools (WavLM, HuBERT, AutoPCP) are existing components; the paper's value lies in the diagnostic framework and its findings, not in architectural innovation. The finding that intelligibility is the primary failure mode for severe dysarthric cloning is the main empirical contribution and is consistent with what one would expect from a system trained on typical speech, though it had not been systematically measured before. The downstream experiment is illustrative but uses a small test set (TORGO restricted sentences), and results may not transfer to other dysarthric datasets or severity distributions.

## Field Significance

Moderate — This paper opens a principled fairness analysis direction for TTS applied to atypical speech, demonstrating that zero-shot voice cloning introduces severity-dependent biases that silently degrade augmentation quality for the most vulnerable users. The evaluation framework and TORGO-based benchmark can serve as a template for similar analyses of other voice cloning systems, and the findings motivate future work on fairness-aware training objectives or data curation for dysarthric speech synthesis.

## Claims

- **supports:** Zero-shot TTS systems trained on typical speech systematically over-normalise intelligibility when cloning dysarthric speech, producing cleaner output than the reference at the cost of dysarthric feature preservation.
  > *Evidence:* F5-TTS yields delta-WER DI of 0.59 for high-severity speakers and delta-CER DI of 0.75, both rated "poor," while SIM-o DI remains 0.85 and AutoPCP DI 0.90, indicating intelligibility divergence is the primary failure mode rather than loss of speaker or prosodic similarity. *(§4.1, §4.2, Table 3)*

- **complicates:** Augmenting dysarthric ASR training data with zero-shot voice-cloned samples improves performance for mild cases but degrades it for severe cases when the cloning system is biased toward intelligibility normalisation.
  > *Evidence:* Adding F5-TTS synthetic data alongside reference audio reduces WER by 44.6% for low-severity speakers but increases WER by 5.7% and 7.7% for mid- and high-severity speakers respectively, compared to the reference-only baseline. *(§4.3, Table 4)*

- **supports:** Fairness metrics (Disparate Impact and Parity Difference) applied to objective TTS evaluation measures reveal severity-level disparities that aggregate quality scores would obscure.
  > *Evidence:* Per-severity DI analysis on TORGO shows near-zero intelligibility bias for low severity (delta-WER DI=0.97) escalating to severe bias for high severity (DI=0.59), a pattern invisible in mean quality scores and only apparent through disaggregated fairness analysis. *(§2.3, §4.2, Table 3)*

- **complicates:** Bias in zero-shot dysarthric speech cloning varies across gender as well as severity, with male and female speakers affected on different dimensions.
  > *Evidence:* Male speakers exhibit higher intelligibility bias (delta-WER PD=0.37, DI=0.68) while female speakers show higher prosody bias (AutoPCP PD=0.36), demonstrating that severity-level aggregation masks a secondary gender-based disparity axis. *(§4.2, Table 3)*

## Limitations and Open Questions

> [!warning]
> The study uses a single TTS system (F5-TTS) and a single dataset (TORGO, restricted sentences only). Whether the intelligibility-bias pattern generalises to other zero-shot TTS systems, to other dysarthric datasets, or to connected speech is untested. Results from a small speaker pool (8 dysarthric, 7 healthy) should be interpreted with caution.

The fairness metrics (DI, PD) are borrowed from sociotechnical fairness literature and treat the healthy group as a reference. This framing may not be universally appropriate for disability-related speech variability, where defining a "privileged" reference group raises its own ethical questions. No human listening studies were conducted, so the relationship between the measured objective biases and perceptual quality from the perspective of dysarthric users remains unexamined. Future work on fairness-aware data augmentation, dysarthria severity-conditioned synthesis, or severity-adaptive training objectives is suggested but not pursued here.

## Wiki Connections

- [[flow-matching|Flow Matching]] — the paper evaluates F5-TTS, which uses flow matching via a Diffusion Transformer as its core speech generation mechanism.
- [[zero-shot-tts|Zero-Shot TTS]] — examines zero-shot voice cloning as a data augmentation strategy for dysarthric speech and characterises its fairness limitations across severity levels.
- [[evaluation-metrics|Evaluation Metrics]] — introduces a fairness evaluation framework applying Disparate Impact and Parity Difference to standard TTS quality metrics (WER, CER, SIM-o, AutoPCP) for subgroup analysis.
- [[speaker-adaptation|Speaker Adaptation]] — assesses how well a zero-shot cloning system preserves speaker identity across dysarthric severity levels, finding speaker similarity is better retained than intelligibility.
- [[2025.acl-long.313|F5-TTS]] — the zero-shot TTS system under study; this paper evaluates its intrinsic bias when applied to dysarthric speech cloning on the TORGO dataset.
