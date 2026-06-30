---
id: "iclr-2025-uxDFlPGRLX"
title: "FlowDec: A flow-based full-band general audio codec with high perceptual quality"
authors: ["Simon Welker", "Matthew Le", "Ricky T. Q. Chen", "Wei-Ning Hsu", "Timo Gerkmann", "Alexander Richard", "Yi-Chiao Wu"]
organization: "Meta (FAIR / Codec Avatar Labs) / University of Hamburg"
venue: ICLR
venue_type: conference
year: 2025
month: 4
published_date: "2025-04-24"
ingested_date: "2026-06-30"
task: [codec]
architecture: [flow-matching, GAN]
conditioning: []
training: [supervised]
model_size: "26M params (postfilter flow model)"
codec_used: "NDAC (non-adversarial DAC variant, 75Hz/25Hz feature rate)"
datasets_train: ["MSP-Podcast", "CommonVoice 13.0", "LibriTTS", "EARS", "VCTK", "LibriVox", "Expresso", "WavCaps-FreeSound"]
datasets_eval: ["VCTK", "EARS", "MUSDB18-HQ", "MusicCaps", "AudioSet"]
metrics:
  - name: MUSHRA
    value: not reported
    system: proposed
    testset: "Test A (mixed speech/music/sound, 21 clips)"
code_available: true
demo_available: true
url: "https://openreview.net/forum?id=uxDFlPGRLX"
related_concepts: [flow-matching, neural-codec, gan-vocoder, diffusion-tts, evaluation-metrics]
related_papers: ["2210.02747", "2210.13438"]
field_significance:
  level: "moderate"
  type: ["architectural-novelty", "empirical-benchmark"]
generation:
  date: "2026-06-30"
  agent: speech-generation-ingest-agent
  model: claude-sonnet-4-6
  commit: "f406e94"
---

> [!abstract] ICLR · 2025 · Conference
> **Welker et al.** (Meta FAIR / University of Hamburg) · [→ Paper](https://openreview.net/forum?id=uxDFlPGRLX) · Demo: ✓ · Code: ✓
>
> A flow matching-based stochastic postfilter extends generative audio codec postfiltering from speech-only, high-bitrate, slow inference to full-band general audio at sub-8 kbit/s with a 10x reduction in postfilter network evaluations.

## Problem

Neural audio codecs that rely on adversarial training to recover natural-sounding output at low bitrates require complex multi-discriminator losses that are difficult to tune and lack interpretability. A prior non-adversarial alternative, ScoreDec, replaces adversarial training with a diffusion-based postfilter applied to a deterministic codec's output, but it is restricted to speech, was only validated at the relatively high bitrate of 24 kbit/s, and has prohibitively slow inference (around 60 network evaluations, real-time factor 1.7). No existing non-adversarial postfilter approach had been shown to generalize to general audio (speech, music, and sound) at low bitrates with practical inference cost.

## Method

FlowDec separates codec design into two stages: a deterministic underlying codec trained without adversarial losses, and a stochastic postfilter that maps the codec's (potentially distorted) output to a sample from the conditional distribution of clean audio given that output. The underlying codec, called NDAC, is a non-adversarial variant of DAC (Descript Audio Codec) with adversarial losses removed and a multiscale constant-Q transform loss plus an L1 waveform loss added to address low-frequency modeling failures observed when training without discriminators. Two NDAC variants are trained: a 75Hz feature-rate model for general coding (0.75-7.5 kbit/s) and a 25Hz, lower-bitrate variant (0.25-4.0 kbit/s) intended for downstream generative audio modeling.

![Method overview: Codecs such as DAC employ adversarial training, using multiple specialized discriminator networks trained jointly with the decoder. Our method FlowDec is trained in a non-adversarial two-stage fashion, removing these discriminators and instead adding a stochastic postfilter that can produce multiple enhanced estimates of the pretrained decoder.](assets/iclr-2025-uxDFlPGRLX/figure-1.png)

The postfilter is trained with a novel adaptation of conditional flow matching (CFM). Rather than sampling the flow's source distribution independently of the target as in standard CFM, FlowDec couples the source distribution to the codec's initial decoder output `y` by centering it at `y` with a heuristically chosen variance `σ_y`. This construction yields a closed-form, per-batch-optimal coupling without requiring a minibatch optimal-transport solver, and the resulting flow field reaches the target signal exactly during inference (σ at t=1 is zero), unlike a competing CFM-for-audio-enhancement formulation that uses a constant noise scale and leaves residual noise in the estimate. The postfilter operates on amplitude-compressed complex STFT features (following the SGMSE line of work) and uses a slightly modified NCSN++ U-Net (26M parameters) with doubled early-layer channel width to suppress high-frequency artifacts in music, reduced depth, and attention layers removed for linear-time inference. At inference, the flow ODE is solved with a Midpoint solver using 6 function evaluations by default, a 10x reduction from ScoreDec's ~60. Training data spans 2,778 hours of paired (codec output, clean target) audio across speech, music, and sound, constructed by mixing the three audio types in random Dirichlet-sampled proportions to expose the postfilter to complex mixed scenes.

## Key Results

At a matched inference budget of 6 network evaluations, FlowDec achieves a Fréchet Audio Distance (FAD×100) of 1.62 versus 145.3 for ScoreDec and roughly 29 for a constant-σ flow matching ablation, with substantially better SI-SDR and fwSSNR than ScoreDec at this budget (Table 4). At 50 evaluations, ScoreDec catches up on SI-SDR but FlowDec still produces markedly better FAD (1.34 vs 5.73). Against the GAN-based state-of-the-art codec DAC, FlowDec achieves the best FAD scores of all compared methods across bitrates and the best SIGMOS naturalness score for speech, while DAC generally retains an edge on intrusive spectral metrics (SI-SDR, fwSSNR), consistent with the known perception-distortion tradeoff. In MUSHRA-style listening tests with 11-15 expert raters, FlowDec-75m and DAC-75 show no statistically significant difference in score distributions at matched bitrates (4.5-7.5 kbit/s), with FlowDec outperforming EnCodec (6.0 kbit/s) and the Opus low anchor. A per-audio-type breakdown (Appendix) suggests FlowDec is rated better than DAC for speech, slightly worse for environmental sound, and comparable for music. Real-time factor at the default 6-evaluation setting is 0.23, an 8x improvement over ScoreDec's 1.7 and the first such postfilter system to run faster than real time.

## Novelty Assessment

The core contribution is architectural: a new conditional flow matching formulation for generative postfiltering that couples the noise distribution to the initial decoder's output rather than to a fixed Gaussian prior, yielding a contractive, exactly-converging flow field with a single hyperparameter. This is a genuine theoretical refinement over both score-based postfiltering (ScoreDec) and an independent concurrent CFM-based audio enhancement formulation, and the paper provides a derivation alongside the empirical comparison. The extension from speech-only to general full-band audio, and the order-of-magnitude reduction in postfilter inference cost without fine-tuning or distillation, are also concrete advances over ScoreDec. The underlying non-adversarial codec (NDAC) itself is a moderate engineering modification of DAC (added CQT and waveform losses) rather than a new architecture. Overall the paper's claim of being "competitive" with GAN-based DAC is well-supported by the listening test results (no significant difference at matched bitrate) but the objective spectral metrics show DAC retains advantages in distortion-based measures, a tradeoff the paper itself foregrounds rather than obscures.

## Field Significance

Moderate — this paper demonstrates that a non-adversarial, flow matching-based postfilter can match a strong GAN-based codec (DAC) in subjective quality while improving perceptual distance metrics (FAD), providing evidence that adversarial training is not strictly necessary to reach competitive perceptual quality in general-audio neural coding. Its main limitations are that the codec is non-streaming (noncausal architecture) and that improvements come at a measurable cost in intrusive distortion metrics relative to DAC.

## Claims

- supports: A flow matching-based stochastic postfilter conditioned on a deterministic codec's decoder output can replace adversarial training while achieving comparable subjective quality to a GAN-based codec.
  Evidence: MUSHRA listening tests (Test A, 11 expert listeners) show no significant difference between FlowDec-75m and DAC-75 score distributions at matched bitrates of 4.5 and 7.5 kbit/s. *(§5.2, Figure 6)*
- supports: Coupling the flow matching source distribution to the conditioning signal (rather than sampling it independently) removes the need for minibatch optimal-transport solvers and improves postfilter sample quality at low inference budgets.
  Evidence: At NFE=6, the proposed coupled formulation achieves FAD×100 of 1.62 versus 145.3 for the diffusion-based ScoreDec baseline and approximately 29 for an alternative constant-σ flow matching formulation, on the same underlying codec and test set. *(§5.1, Table 4)*
- complicates: Improving perceptual distance metrics (FAD) via generative postfiltering trades off against intrusive distortion metrics relative to discriminator-trained codecs.
  Evidence: Retrained non-adversarial DAC (NDAC) generally outperforms FlowDec on SI-SDR and fwSSNR even though FlowDec achieves better FAD, consistent with the perception-distortion tradeoff; the gap is small in the perceptually weighted fwSSNR but clear in SI-SDR. *(§5.1, Figure 4, Figure 5)*
- complicates: Generative postfilters trained with vanilla score- or flow-matching formulations using a fixed-variance or independent prior can fail to converge to the target signal or require expensive multi-step inference to reach acceptable quality.
  Evidence: ScoreDec (diffusion-based postfilter) produces unusable audio quality at a reduced inference budget of 6 function evaluations (FAD×100 = 145.3, SI-SDR = -27.23), only becoming competitive at roughly 50 evaluations; a constant-σ flow matching variant also underperforms the proposed coupled formulation at NFE=6. *(§5.1, Table 4)*

## Limitations and Open Questions

> [!warning]
> The proposed codec, like the DAC baseline it builds on, uses a noncausal architecture and is not streaming-capable, limiting applicability to real-time communication settings where low-latency, causal processing is required.

The authors note this could be addressed with a causal DNN design in future work, but no such variant is evaluated in this paper. Additionally, the postfilter and underlying codec are trained in two separate stages; joint training is identified as a possible quality improvement but is left unexplored due to potential training instability. The NCSN++ backbone used for the postfilter was originally designed for images, and the authors suggest audio-specific architectures could further improve quality, indicating the current network design is not optimized end-to-end for the audio domain. Listening test sample sizes (11 and 10 raters) and clip counts (21 ten-second clips) are modest for the audio-type breakdown analysis reported in the appendix.

## Wiki Connections

- [[flow-matching|Flow Matching]] — introduces a conditional flow matching variant for generative postfiltering, coupling the source distribution to the codec output to ensure exact convergence and stable low-step inference.
- [[neural-codec|Neural Audio Codec]] — proposes a non-adversarial codec design (postfilter-based) as an alternative to the GAN-dominated codec training paradigm, extended to full-band general audio.
- [[gan-vocoder|GAN Vocoder]] — directly compares against and aims to replace adversarial discriminator-based training used in state-of-the-art codecs like DAC.
- [[diffusion-tts|Diffusion TTS]] — builds on and is compared against ScoreDec, a diffusion (score-based) postfilter, showing flow matching achieves better quality at far fewer inference steps.
- [[evaluation-metrics|Evaluation Metrics]] — uses Fréchet Audio Distance, SI-SDR, fwSSNR, and SIGMOS alongside MUSHRA listening tests to characterize the perception-distortion tradeoff in codec quality.
- [[2210.02747|Flow Matching for Generative Modeling]] — applies and extends the conditional flow matching framework introduced in this foundational paper to the audio codec postfiltering setting.
- [[2210.13438|EnCodec]] — uses the official 48kHz EnCodec checkpoint as a listening-test comparison baseline.
