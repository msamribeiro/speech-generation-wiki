---
id: "interspeech-2025-1940"
title: "Investigating Stochastic Methods for Prosody Modeling in Speech Synthesis"
authors: ["Paul Mayer", "Florian Lux", "Alejandro Pérez-González-de-Martos", "Angelina Elizarova", "Lindsey Vanderlyn", "Dirk Väth", "Ngoc Thang Vu"]
organization: null
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-07-04
task: ["TTS"]
architecture: ["transformer-enc-dec", "flow-matching"]
conditioning: ["text-conditioned", "speaker-conditioned"]
training: ["supervised"]
model_size: "not reported"
codec_used: "none"
datasets_train: ["LibriTTS"]
datasets_eval: ["RAVDESS", "ADEPT"]
metrics: []
code_available: true
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/mayer25_interspeech.html"
related_concepts: [prosody-control, flow-matching, subjective-evaluation, evaluation-metrics, transformer-enc-dec-tts]
related_papers: ["2301.02111"]
field_significance:
  level: "moderate"
  type: ["empirical-benchmark", "architectural-novelty"]
generation:
  date: 2026-07-04
  agent: speech-generation-ingest-agent
  model: claude-sonnet-4-6
  commit: "318f1b4"
---

> [!abstract] Interspeech · 2025 · Conference
> **Mayer et al.** · [→ Paper](https://www.isca-archive.org/interspeech_2025/mayer25_interspeech.html) · Demo: ? · Code: ✓
>
> Systematic comparison of normalizing flows, conditional flow matching, and rectified flows as stochastic variance predictors for explicit pitch, energy, and duration modeling in TTS, demonstrating that rectified flows best balance prosodic naturalness and diversity, with sampling temperature enabling runtime control over that trade-off.

## Problem

TTS systems that model prosody explicitly through parameters such as pitch, energy, and duration offer interpretability and fine-grained controllability, making them valuable for applications including speaker anonymization, literary text reading, and content dubbing. However, conventional deterministic variance predictors collapse the inherent one-to-many mapping of prosody onto a single most-likely realization, producing speech that is consistent but insufficiently varied. While stochastic methods have been explored piecemeal (VITS for duration prediction using normalizing flows; VarianceFlow for pitch and energy; a 2024 preprint applying CFM to duration), no comprehensive comparison across methods and prosodic parameters existed.

## Method

The system is built on ToucanTTS, which follows the FastSpeech 2 architecture with a Conformer-based encoder and decoder and articulatory input features. Speaker identity is conditioned via a pre-trained ECAPA-TDNN speaker embedding from SpeechBrain. A CFM postnet (adapted from Matcha-TTS) refines the output spectrogram, but with sampling temperature fixed to 0.0 during inference so all prosodic variance originates from the dedicated predictors.

Three stochastic approaches replace the standard convolutional variance predictors. Normalizing flows (NF) map Gaussian noise to the prosody distribution through a series of invertible transformations in a single step. Conditional flow matching (CFM) learns a time-dependent vector field transporting noise to prosodic realizations via ODE integration (12 Euler steps at inference). Rectified flows (RF) extend CFM by applying the ReFlow post-training procedure, straightening the probability paths toward optimal transport to yield a deterministic noise-to-data coupling.

Two structural configurations are evaluated: cascaded prediction (each predictor conditioned on previous outputs, energy then pitch then duration) and joint prediction (all three parameters from a single module with three-dimensional output). Parameter counts are matched across configurations to ensure fair comparison.

## Key Results

Objective evaluation using Jensen-Shannon divergence of kernel-density-estimated prosodic distributions (Table 1) shows cascaded prediction substantially outperforms joint prediction for duration (JS 0.481 vs. 0.562 for the best cascade ordering), while pitch and energy show no significant difference between cascade orderings. This finding holds across all stochastic methods.

Subjective evaluation with 40 Prolific raters on the ADEPT dataset (Kruskal-Wallis test with Dunn post-hoc) reveals three key results. RF at 0.4 and CFM at 0.4 both achieve naturalness scores on par with the human baseline; the deterministic baseline scores highest on naturalness but lowest on diversity. RF at 0.8 achieves the best naturalness-diversity trade-off: highest diversity after the human baseline while maintaining naturalness parity with human recordings. Naturalness and perceived diversity are inversely correlated across all conditions, including human speech itself.

Temperature control is effective for all stochastic predictors, with pitch contour variance and duration variance increasing monotonically with temperature. The relationship follows a near-exponential trend, and the authors note that a log-transformed temperature axis would provide more intuitive user control.

## Novelty Assessment

The architectural contribution is incremental. Applying NF and CFM to prosody prediction is not new; this paper's addition is RF and a systematic three-way comparison under matched conditions. The evaluation methodology is the stronger contribution: using RAVDESS for KDE-based JS divergence against human reference distributions, isolating prosodic perception from voice quality through prosody cloning, and measuring both naturalness and diversity as separate subjective axes are methodologically careful choices that go beyond what prior comparisons have done. The finding that the naturalness-diversity trade-off holds even for human speech is a practically useful insight for systems exposing temperature as a user-facing parameter.

## Field Significance

Moderate — this paper provides useful comparative evidence for explicit prosody modeling, addressing a gap where stochastic methods had been applied piecemeal across different predictors and papers. The finding that rectified flows offer the best naturalness-diversity trade-off, and the quantification of that trade-off for human speech itself, gives practitioners a principled basis for choosing and configuring prosody predictors in controllable TTS pipelines. The contribution is primarily empirical; the architecture extends ToucanTTS by substituting variance predictor modules without altering the broader pipeline.

## Claims

- **supports:** Stochastic prosody predictors achieve naturalness parity with human recordings while enabling prosodic diversity that deterministic predictors cannot produce.
  > *Evidence:* RF at 0.4 temperature shows no statistically significant difference from human naturalness ratings (Kruskal-Wallis, Dunn post-hoc); the deterministic baseline achieves highest naturalness but lowest diversity across 40 Prolific raters on ADEPT. *(§3.4, Figure 5, Figure 6)*

- **refines:** For explicit prosody modeling, cascaded prediction of prosodic features outperforms joint prediction for duration but the ordering of pitch and energy within the cascade has negligible impact.
  > *Evidence:* Table 1 shows joint prediction yields 17% higher duration JS divergence (0.562) vs. cascading (0.481), while pitch-first and energy-first orderings differ by less than 0.005 across all prosodic parameters. *(§3.3, Table 1)*

- **complicates:** Naturalness and prosodic diversity are inversely correlated, constraining simultaneous optimization of both properties in TTS.
  > *Evidence:* Subjective ratings show no system achieves both maximal naturalness and maximal diversity; even human speech exhibits this trade-off, with the human baseline scoring highest on diversity but not on naturalness. *(§3.4, Figure 5, Figure 6)*

- **supports:** Sampling temperature in generative prosody predictors enables effective runtime control over prosodic variability without retraining.
  > *Evidence:* Pitch contour variance and duration variance increase monotonically with temperature for all stochastic methods (NF, CFM, RF), following a near-exponential relationship that allows a log-linear temperature axis for more intuitive user control. *(§3.3, Figure 2, Figure 3)*

## Limitations and Open Questions

Experiments are restricted to read speech (LibriTTS), with conversational speech explicitly deferred to future work. Prior work found significant benefits for stochastic duration prediction specifically in conversational scenarios, so the present findings may not generalize to that more demanding setting. The stochastic models do not yet capture the full diversity of human prosodic distributions, with synthetic KDE distributions consistently narrower and more unimodal than human reference distributions from RAVDESS; the authors hypothesize that larger predictor modules could close this gap.

## Wiki Connections

- [[prosody-control|Prosody Control]] — introduces stochastic variance predictors (NF, CFM, RF) for explicit pitch, energy, and duration control with temperature-based diversity tuning at inference time.
- [[flow-matching|Flow Matching]] — applies conditional flow matching and rectified flows as the core generative mechanism for prosody predictors, extending their use from full TTS synthesis to modular prosody modules.
- [[subjective-evaluation|Subjective Evaluation]] — employs 40 Prolific raters to measure prosodic naturalness and perceived diversity via 5-point Likert scales, with statistical testing via Kruskal-Wallis and Dunn post-hoc tests.
- [[evaluation-metrics|Evaluation Metrics]] — uses Jensen-Shannon divergence of KDE-estimated distributions as an objective measure for comparing synthetic prosodic variability against human reference distributions.
- [[transformer-enc-dec-tts|Transformer Encoder-Decoder TTS]] — builds on the FastSpeech 2 / ToucanTTS Conformer-based architecture, replacing the standard convolutional variance predictors with stochastic generative modules.
- [[2301.02111|VALL-E]] — cited as an example of black-box autoregressive TTS that lacks fine-grained prosody control, motivating the need for explicit prosody modeling approaches.
- *Towards Controllable Speech Synthesis in the Era of Large Language Models: A Systematic Survey* (2412.06602, not in corpus) — cited as broader context for controllable speech synthesis applications including dubbing and content generation.
