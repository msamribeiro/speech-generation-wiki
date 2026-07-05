---
id: "interspeech-2025-2787"
title: "Towards Adaptable and Intelligible Speech Synthesis in Noisy Environments"
authors: ["Lubos Marcinek", "Jonas Beskow", "Joakim Gustafsson"]
organization: "KTH Royal Institute of Technology"
venue: "Interspeech"
venue_type: conference
year: 2025
month: 8
published_date: "2025-08-17"
ingested_date: "2026-07-05"
task: ["TTS", "evaluation"]
architecture: ["autoregressive-LM"]
conditioning: ["zero-shot", "prompt-conditioned"]
training: ["supervised"]
model_size: "not reported"
codec_used: "not reported"
datasets_train: ["DEMAND corpus (noise mixing)", "custom TTS-in-noise corpus"]
datasets_eval: ["DEMAND corpus (noise mixing)", "custom TTS-in-noise corpus"]
metrics: []
code_available: null
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/marcinek25_interspeech.html"
related_concepts: ["zero-shot-tts", "evaluation-metrics", "subjective-evaluation"]
related_papers: []
field_significance:
  level: "low"
  type: ["empirical-benchmark"]
generation:
  date: "2026-07-05"
  agent: speech-generation-ingest-agent
  model: claude-sonnet-4-6
  commit: "2829243"
---

> [!abstract] Interspeech · 2025 · Conference
> **Marcinek et al.** (KTH Royal Institute of Technology) · [→ Paper](https://www.isca-archive.org/interspeech_2025/marcinek25_interspeech.html) · Demo: ? · Code: ?
>
> A systematic empirical study showing that zero-shot TTS can modulate vocal effort via audio prompt selection, and that higher effort levels improve ASR-measured intelligibility by up to 30% in adverse acoustic conditions, with the largest benefit occurring in speech-like (informational masking) noise at low SNRs.

## Problem

Deploying TTS in real-world environments requires intelligibility under noise, yet most synthesis systems are designed and evaluated in controlled quiet conditions. Prior strategies to improve robustness, including time-frequency masking, formant enhancement, and linguistic paraphrasing, address downstream processing rather than the generation stage itself. The specific question of whether zero-shot TTS systems, which condition on audio prompts, can meaningfully exploit vocal effort variation to improve robustness, and how that benefit interacts with noise type and SNR, had not been systematically quantified.

## Method

The study uses Llasa, a zero-shot TTS system built on a Llama-based autoregressive architecture, to synthesize speech at six discrete vocal effort levels. Audio prompts were drawn from two movie scenes chosen for their natural progression of vocal intensity (Reese Witherspoon in "Legally Blonde" and Samuel L. Jackson in "Pulp Fiction"), yielding two speakers at three effort levels each. One hundred sentences of varying textual complexity were synthesized at all six effort levels, producing 1,200 clean samples. Only the 1,035 samples achieving WER = 0 under Whisper 3 Turbo ASR (verifying clean-condition intelligibility) were retained for noise mixing.

Environmental noise from the DEMAND corpus was added at twelve types spanning domestic, office, public, and transportation settings at nine SNR levels from -24 to 24 dB, producing 88,020 speech-in-noise files. Critically, the study deliberately avoids energy normalization before mixing, preserving the natural energy advantage of higher vocal effort levels rather than artificially removing it. Intelligibility is evaluated using wav2vec 2.0 ASR with WER as the metric.

To assess which features drive intelligibility, regression models (linear, decision tree, random forest, gradient boosting, XGBoost, and a feedforward neural network) were trained on acoustic and linguistic features including noise type, SNR, vocal effort level, speaker gender, and an Overall Textual Feature Score (OTFS) combining four text complexity metrics. An 80/20 train-validation split was used.

A supplementary human listening experiment recruited 35 native English speakers via Prolific to transcribe 40 speech samples each, covering five challenging SNR conditions and four noise environments.

## Key Results

Higher vocal effort improved ASR intelligibility by up to 30% (WER reduction) in adverse conditions, with the most pronounced gains in environments dominated by competing speech (meeting rooms and cafeterias at low SNRs). At SNR above 6 dB, effort level differences largely disappeared, with most environments converging below WER = 0.2 at SNR > 12 dB.

In the regression analysis, XGBoost achieved R² = 0.65 on all features; SNR emerged as the dominant predictor (removing it drops R² to 0.20), followed by noise type (R² drops to 0.40 without it). Removing vocal effort level had negligible effect on model performance (R² = 0.64), indicating effort contributes relatively little additional predictive variance once SNR and noise type are known. Text complexity (OTFS) similarly contributed little.

Human listeners showed meaningfully different patterns from the ASR system: sharper threshold effects at specific SNR values (for example, large improvements at -15 dB in bus noise and -10 dB in meeting room noise), non-monotonic responses in cafeteria noise at mid-range SNRs, and greater sensitivity to speech-like masking overall. This confirmed that wav2vec 2.0 WER does not fully capture perceptual intelligibility.

## Novelty Assessment

The contribution is primarily empirical and methodological. Using a zero-shot TTS for vocal effort control via audio prompt selection is not a new technique; the paper's novel aspect is the systematic evaluation framework: 88,020 speech-in-noise files across twelve noise environments and nine SNRs, with both ASR and human-listener assessment. The methodological decision to preserve natural energy differences across effort levels (no energy normalization) is well-justified and distinguishes this from prior intelligibility studies that remove the very acoustic advantage being studied.

The regression analysis is straightforward and the models are standard, but the feature-removal ablations produce actionable findings about which factors actually matter. The human listening experiment, though small (n = 35), usefully flags where ASR-based evaluation diverges from human perception.

## Field Significance

Low — This paper provides useful empirical evidence that zero-shot TTS can exploit vocal effort variation for noise robustness, but the finding is a validation of a known perceptual phenomenon (the Lombard effect) rather than a new capability. The result that SNR dominates predictive models more than vocal effort narrows the practical utility of effort-based adaptation to low-SNR regimes only. The human vs. ASR divergence finding has broader relevance for evaluation methodology but is not the paper's focus.

## Claims

- **supports:** Vocal effort modulation in zero-shot TTS via audio prompt selection can produce meaningful intelligibility gains in adverse acoustic conditions.
  > *Evidence:* Llasa-synthesized speech at higher vocal effort levels showed up to 30% WER reduction in meeting room and cafeteria noise at low SNRs; acoustic feature analysis confirmed systematic increases in F0, intensity, and high-frequency spectral energy consistent with authentic vocal effort adaptation. *(§2.1, §3.1)*

- **complicates:** The intelligibility benefit of vocal effort adaptation in TTS is noise-type and SNR-dependent, and becomes negligible in moderate-to-favorable acoustic conditions.
  > *Evidence:* WER differences across effort levels converged above 6 dB SNR in most environments; most environments reached WER < 0.2 at SNR > 12 dB regardless of effort level, indicating effort adaptation is useful only in adverse conditions. *(§3.1)*

- **complicates:** SNR and noise type dominate intelligibility prediction; speaker identity, vocal effort, and text complexity add minimal predictive variance beyond those two factors.
  > *Evidence:* XGBoost regression (R² = 0.65) dropped to R² = 0.20 when SNR was removed and to R² = 0.40 when noise type was removed, while removing vocal effort or voice type had near-zero effect (R² = 0.64 in both cases). *(§3.2, Table 2)*

- **complicates:** ASR-based WER underestimates the perceptual difficulty of noisy speech, particularly in speech-like masking conditions and at threshold SNR regions.
  > *Evidence:* Human listeners (n = 35) exhibited non-monotonic WER patterns in cafeteria noise and sharp threshold effects not observed in wav2vec 2.0 ASR, and showed greater sensitivity to meeting room noise; human WER ranged 0.1-1.2 with qualitatively different SNR-response shapes. *(§4)*

## Limitations and Open Questions

The study uses audio prompts drawn from two specific film scenes to represent vocal effort levels, which may not generalize to other speakers or to effort levels encountered in real deployments. The effort levels are ordinal (1-6 from prompt selection) rather than a controlled acoustic parametrization, making it difficult to disentangle effort from speaker-specific stylistic variation.

The human listening experiment is small (n = 35) and covers only a subset of the conditions evaluated by ASR, limiting statistical power for the human vs. ASR comparison. The regression models explain only 65% of variance (R² = 0.65) under all features, leaving substantial unexplained variance that likely reflects sentence-level and noise-clip-level variability not captured by the features used.

The paper's application scenario, a kitchen assistant for elderly users, is described in the conclusion but not evaluated, leaving the practical performance of the real-time system undemonstrated.

## Wiki Connections

- [[zero-shot-tts|Zero-Shot TTS]] — this paper uses a zero-shot TTS system (Llasa) and evaluates how its audio-prompt conditioning mechanism can be leveraged for vocal effort control without speaker-specific fine-tuning.
- [[evaluation-metrics|Evaluation Metrics]] — WER via wav2vec 2.0 ASR is the primary intelligibility metric, and the paper compares its behavior against human WER, surfacing cases where automatic and perceptual metrics diverge.
- [[subjective-evaluation|Subjective Evaluation]] — a listening experiment with 35 native English speakers transcribing 40 speech-in-noise samples each provides a direct human-ASR comparison across five SNR conditions and four noise types.
