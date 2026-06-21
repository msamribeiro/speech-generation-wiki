---
id: "interspeech-2025-0406"
title: "Zero-Shot Mono-to-Binaural Speech Synthesis"
authors: [Alon Levkovitch, Julian Salazar, Soroosh Mariooryad, RJ Skerry-Ryan, Nadav Bar, Bastiaan Kleijn, Eliya Nachmani]
organization: Google
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: "2025-08-17"
ingested_date: "2026-05-30"
task: [TTS]
architecture: [GAN]
conditioning: [zero-shot]
training: [supervised]
model_size: "not reported"
codec_used: "none"
datasets_train: [LibriLight (60k hours, WaveFit vocoder pretraining)]
datasets_eval: [Binaural Speech Dataset (BSD), TUT Mono-to-Binaural (TMB)]
metrics:
  - name: MOS
    value: 4.07
    system: ZeroBAS
    testset: BSD
  - name: MOS
    value: 3.98
    system: ZeroBAS
    testset: TMB
  - name: MOS
    value: 3.86
    system: WarpNet (supervised)
    testset: BSD
  - name: MOS
    value: 4.01
    system: BinauralGrad (supervised)
    testset: BSD
  - name: MOS
    value: 3.60
    system: WarpNet (supervised)
    testset: TMB
  - name: MOS
    value: 3.27
    system: BinauralGrad (supervised)
    testset: TMB
code_available: null
demo_available: true
url: "https://www.isca-archive.org/interspeech_2025/levkovitch25_interspeech.html"
related_concepts: [zero-shot-tts, gan-vocoder, evaluation-metrics, subjective-evaluation]
related_papers: []
field_significance:
  level: "moderate"
  type: [engineering-integration, dataset-contribution]
generation:
  date: 2026-06-21
  agent: speech-generation-review-agent
  model: claude-sonnet-4-6
  commit: "3538db5"
---
> [!abstract] Interspeech · 2025 · Conference
> **Alon Levkovitch et al.** (Google) · [→ Paper](https://www.isca-archive.org/interspeech_2025/levkovitch25_interspeech.html) · Demo: ✓ · Code: ?
>
> ZeroBAS is the first zero-shot neural method to synthesize binaural speech from monaural audio and 3D positional information without any binaural training data, achieving perceptual quality on par with or exceeding supervised methods.

## Problem

Existing mono-to-binaural synthesis methods require position-annotated binaural audio datasets, which are costly to collect due to specialized equipment (HRTF recording systems, binaural microphones). Supervised models overfit to specific rooms, speakers, and languages in their small training sets; the standard BSD contains only two hours of data. This makes supervised systems brittle when evaluated on out-of-distribution room conditions or unseen speakers. A zero-shot approach would bypass the data collection bottleneck and generalize across environments.

## Method

ZeroBAS is a three-stage, parameter-free-at-inference pipeline operating on a mono waveform and 3D source/listener positions.

**Stage 1 — Geometric Time Warping (GTW):** The mono signal is warped into two channels by computing an interaural time delay (ITD) warpfield based on the Euclidean distance from the source to each ear at each timestep. Values are interpolated to produce left/right preliminary channels.

**Stage 2 — Amplitude Scaling (AS):** Exploiting the inverse-square law, the amplitude of each channel is attenuated according to the ratio of ear-to-source distances. This simulates the interaural level difference (ILD) without learned parameters.

**Stage 3 — Iterative Denoising Vocoder:** The GTW+AS output is treated as a degraded binaural estimate. A pretrained WaveFit vocoder (GAN-based, trained on LibriLight's 60k hours) is applied iteratively to each channel independently. Rather than starting from Gaussian noise (standard vocoder inference), the corrupted binaural signal is fed in at a low noise level (last denoising timestep k=1), and the iteration is repeated N=3 times. Log-mel features of each channel serve as conditioning. The model has never seen binaural data; its learned speech distribution implicitly imputes a generalized low-RIR room and an implicit HRTF.

The absence of room modeling or head-shape modeling is an acknowledged limitation — the method produces audio consistent with a low-reverberant environment and a generic HRTF.

## Key Results

On the in-distribution BSD dataset, ZeroBAS achieves MOS 4.07 vs. WarpNet's 3.86 and BinauralGrad's 4.01 (both supervised). MUSHRA shows no statistically significant preference between ZeroBAS and the supervised methods on BSD. On the out-of-distribution TUT Mono-to-Binaural (TMB) dataset (different rooms, French speakers, studio conditions), ZeroBAS scores MOS 3.98 vs. WarpNet 3.60, BinauralGrad 3.27, NFS 3.79. MUSHRA reveals a statistically significant preference for ZeroBAS over all supervised methods on TMB. Supervised methods fail on TMB due to (a) voice-quality degradation and (b) incorrect spatialization outside their training range; BinauralGrad produces Gaussian noise artifacts on OOD samples. Objective improvements of ZeroBAS over the DSP baseline are significant on both sets, with W-ℓ2 falling from 0.812 to 0.440 on BSD and from 1.134 to 0.293 on TMB. Ablations confirm that AS and GTW are both critical; removing GTW degrades MOS to 3.64, removing AS to 2.93.

## Novelty Assessment

The core novelty is repurposing a pretrained monaural denoising vocoder for the mono-to-binaural task without any binaural training data. The geometric warping and amplitude scaling components are known from prior DSP and neural work; the contribution is combining them as an initialization for iterative vocoder denoising, plus demonstrating that the vocoder's speech distribution generalizes to binauralize mono audio. The new TUT Mono-to-Binaural dataset is a methodological contribution enabling OOD evaluation. The approach is positioning-dependent — it still requires 3D positional metadata at inference — so it is not zero-shot with respect to position conditioning, only with respect to binaural training data.

## Field Significance

Moderate — ZeroBAS demonstrates that combining parameter-free geometric preprocessing with a pretrained monaural vocoder can match supervised binaural synthesis without any domain-specific training data. The paper also introduces the TUT Mono-to-Binaural dataset, which provides the first benchmark for out-of-distribution generalization in mono-to-binaural synthesis and reveals a significant brittleness in existing supervised models that standard in-distribution evaluations conceal.

## Claims

- Pretrained monaural denoising vocoders can refine geometrically warped speech into perceptually natural binaural audio without exposure to any binaural training data. *(§3.3, §4.4)*
- Supervised mono-to-binaural synthesis models trained on small room-specific datasets degrade significantly under out-of-distribution acoustic conditions, including different rooms and languages. *(§4.5, Table 2)*
- Geometric interaural time delay warping and amplitude scaling based on the inverse-square law are both necessary components for zero-shot binaural synthesis; removing either one substantially degrades perceptual quality. *(§5, Table 3)*
- Standard in-distribution benchmarks for binaural synthesis are insufficient to assess generalisation; evaluating on held-out room conditions reveals large performance gaps that in-distribution results conceal. *(§4.2, §4.5)*

## Limitations and Open Questions

The method does not model room impulse responses or listener head shape (HRTF is implicit and generic). Phase accuracy (P-ℓ2) remains comparable to DSP baselines but does not approach supervised methods. The vocoder processes each channel independently, which could introduce cross-channel inconsistencies. Quality plateaus at 3 WaveFit iterations and does not benefit from more. Future work could integrate personalized HRTFs or light room estimation modules to close the remaining gap on objective metrics.

## Wiki Connections

This paper extends the zero-shot paradigm — established in [[zero-shot-tts]] concept — to a spatial rendering task rather than speaker generalization. The WaveFit vocoder used here connects to the [[gan-vocoder]] concept. The subjective evaluation protocol (MOS + MUSHRA) is documented in [[subjective-evaluation]]. No corpus papers are cited in-corpus; primary baselines are WarpNet (ICLR 2021), BinauralGrad (NeurIPS 2022), and NFS (ICASSP 2023), all out-of-corpus.
