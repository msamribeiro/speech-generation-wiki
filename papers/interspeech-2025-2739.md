---
id: "interspeech-2025-2739"
title: "AF-Vocoder: Artifact-Free Neural Vocoder with Global Artifact Filter"
authors: ["Zhuangqi Chen", "Xianjun Xia", "Xiaohuai Le", "Siyu Sun", "Chuanzeng Huang"]
organization: "ByteDance"
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: "2025-08-17"
ingested_date: "2026-07-05"
task: ["TTS"]
architecture: ["GAN"]
conditioning: []
training: ["supervised"]
model_size: "112.56M params (V1/SAN); 98.17M (V2); 13.44M (V3)"
codec_used: "none"
datasets_train: ["LibriTTS"]
datasets_eval: ["LibriTTS", "LJSpeech", "VCTK", "MUSDB18-HD"]
metrics:
  - name: PESQ
    value: 4.051
    system: AF-Vocoder V1
    testset: LibriTTS-dev
  - name: UTMOS
    value: 3.6499
    system: AF-Vocoder V1
    testset: LibriTTS-dev
  - name: PESQ
    value: 4.058
    system: AF-Vocoder V2
    testset: LibriTTS-dev
code_available: null
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/chen25q_interspeech.html"
related_concepts: ["gan-vocoder", "evaluation-metrics", "subjective-evaluation"]
related_papers: []
field_significance:
  level: "moderate"
  type: ["architectural-novelty"]
generation:
  date: "2026-07-05"
  agent: speech-generation-ingest-agent
  model: claude-sonnet-4-6
  commit: "2829243"
---

> [!abstract] Interspeech · 2025 · Conference
> **Chen et al.** (ByteDance) · [→ Paper](https://www.isca-archive.org/interspeech_2025/chen25q_interspeech.html) · Demo: ? · Code: ?
>
> AF-Vocoder introduces GAFilter, a learnable global frequency-domain artifact filter integrated into a BigVGAN-style GAN vocoder, which suppresses both aliasing and blurring artifacts without the upsampling-downsampling anti-aliasing layers previously required.

## Problem

GAN-based vocoders dominate high-fidelity speech synthesis due to their speed and quality, but consistently produce two classes of audible artifacts: aliasing artifacts arising from imperfect upsampling layers, and blurring artifacts caused by loss of harmonic spectral detail. Prior approaches address these separately. Time-domain vocoders (HiFi-GAN, BigVGAN) use anti-aliasing upsampling-downsampling operations to reduce aliasing but leave blurring unaddressed. Frequency-domain vocoders (Vocos, FreeV) avoid aliasing through ISTFT-based synthesis but sacrifice periodic inductive bias and produce insufficient high-frequency harmonic detail. Enhanced discriminators and auxiliary losses improve global metrics but fail to eliminate blurring artifacts in practice.

## Method

AF-Vocoder is a GAN-based vocoder built on the BigVGAN backbone, retaining stacked ConvTranspose1D upsampling layers and a Multi-Periodicity Composition (MPC) module with Snake activations for periodic inductive bias. The key addition is GAFilter (Global Artifact Filter), a learnable frequency-domain module inserted after the final upsampling layer. GAFilter transforms the generator's output feature into the frequency domain via STFT, applies a temporally invariant learnable filter W (dimensions 1 x C x F x 1) through element-wise multiplication, and reconstructs the time-domain signal via ISTFT. This allows the network to autonomously suppress anomalous frequency components that manifest as artifacts, without requiring explicit upsampling-downsampling operations for aliasing suppression.

The final MPC block is replaced by an Artifact-Free MPC (AF-MPC) incorporating GAFilter, and post-processing applies Snake activation, GAFilter, Conv1D, and Tanh in sequence. The upsampling-downsampling anti-aliasing layers present in BigVGAN are removed, as GAFilter subsumes their function. AF-Vocoder is trained using BigVGAN discriminators (Multi-Period Discriminator and Multi-Resolution Discriminator) with least-squares GAN loss, feature matching loss, and mel-spectrogram loss. A SAN-framework variant (AF-Vocoder-SAN) is also evaluated using BigVSAN discriminators. Three complexity levels are presented: V1 (112.56M, matching BigVGAN channel configuration), V2 (98.17M, reduced channels), and V3 (13.44M, lightweight). All models train for 1M steps on LibriTTS at 24 kHz.

![Figure 1: Overall architecture of AF-Vocoder. AF-Vocoder consists of stacked upsampling layers for recovering speech waveform from mel-spectrogram, where the Snake function is utilized for providing periodic inductive bias and a global artifacts filter (GAFilter) is designed to remove unexpected artifacts. The GAFilter module is a frequency filter providing desired inductive bias of frequency control, which facilitates the synthesis of artifact-free speech.](assets/interspeech-2025-2739/figure-2.png)

## Key Results

On the LibriTTS-dev benchmark (Table 1), AF-Vocoder V1 achieves M-STFT 0.7844, PESQ 4.051, Periodicity 0.0946, V/UV F1 0.9641, and UTMOS 3.6499, outperforming the reproduced BigVGAN baseline (M-STFT 0.8056, PESQ 3.974, UTMOS 3.5139) on all metrics. The reduced-complexity V2 (98.17M) achieves PESQ 4.058 and UTMOS 3.6534, surpassing BigVGAN despite fewer parameters. The lightweight V3 (13.44M, comparable in size to Vocos) outperforms Vocos on PESQ (3.711 vs 3.613) and UTMOS (3.5628 vs 3.5488).

Applying GAFilter to BigVGAN directly (BigVGAN+GAFilter) also improves all metrics, confirming the module transfers to existing architectures. AF-Vocoder-SAN achieves the best objective scores on M-STFT (0.7713) and Periodicity (0.0886), but perceptual UTMOS (3.517) is lower than V1 under standard GAN training.

Subjective CMOS tests on MUSDB18-HD (Table 2) confirm perceptual improvements: AF-Vocoder V1 +0.328 CMOS over BigVGAN, and AF-Vocoder-SAN +0.559 CMOS over BigVSAN. Out-of-domain evaluations on VCTK, LJSpeech, and MUSDB18-HD (Table 3) all show consistent PESQ and periodicity improvements over BigVGAN, confirming cross-domain robustness.

## Novelty Assessment

The core contribution is GAFilter, a simple and well-motivated module: apply STFT, multiply by a learnable filter, apply ISTFT. The idea of learnable frequency-domain filtering is not new in signal processing but its integration as a post-generation inductive bias specifically to suppress GAN vocoder artifacts is a focused application that the paper validates thoroughly. The overall architecture remains BigVGAN-derived; GAFilter is an add-on module rather than a fundamental redesign. The engineering insight (removing upsampling-downsampling modules once GAFilter handles aliasing suppression) is a structural simplification that incidentally reduces parameter count in V2 while maintaining quality. The training stability improvement is a useful byproduct rather than a designed contribution. Evaluation is objective-heavy with a small (8-rater) CMOS test; a larger-scale subjective evaluation would strengthen the perceptual claims.

## Field Significance

Moderate — AF-Vocoder provides solid evidence that frequency-domain artifact filtering, implemented as a lightweight post-processing module, can address both aliasing and blurring artifacts in GAN vocoders simultaneously. The result that GAFilter can replace the upsampling-downsampling anti-aliasing layers from BigVGAN is practically useful, offering quality improvements at lower or equivalent parameter counts. The contribution is incremental relative to BigVGAN but demonstrates a clean and transferable design pattern.

## Claims

- **supports:** Frequency-domain learnable filters can simultaneously suppress aliasing and blurring artifacts in GAN vocoders, replacing specialised anti-aliasing upsampling modules.
  > *Evidence:* GAFilter, a temporally invariant STFT-domain multiplicative filter, improves all objective metrics over BigVGAN on LibriTTS-dev while allowing removal of the anti-aliasing upsampling-downsampling operations. *(§2.2, §3.3, Table 1)*

- **supports:** Frequency-domain artifact filtering in GAN vocoders generalises to out-of-distribution speakers and acoustic domains without domain-specific adaptation.
  > *Evidence:* AF-Vocoder, trained only on LibriTTS, outperforms BigVGAN in PESQ and periodicity error on VCTK, LJSpeech, and MUSDB18-HD. *(§3.3, Table 3)*

- **complicates:** Slicing adversarial network training frameworks can increase perceptual artifact severity in GAN vocoders despite achieving stronger objective periodicity scores.
  > *Evidence:* Under BigVSAN's SAN framework, the authors observe more audible artifacts; AF-Vocoder-SAN achieves UTMOS 3.517, below the 3.650 of AF-Vocoder V1 trained with standard least-squares GAN loss. *(§3.3, Table 1)*

- **supports:** Frequency-domain inductive biases improve gradient stability during GAN vocoder training, reducing early-stage gradient variance.
  > *Evidence:* Gradient curves show lower deviation and faster stabilisation for AF-Vocoder versus BigVGAN in early training steps, attributed to GAFilter filtering out anomalous spectral components. *(§3.3, Figure 3)*

## Limitations and Open Questions

The subjective evaluation uses only 8 raters and a single MUSDB18-HD test set, which limits the strength of the perceptual claims. Whether the perceptual gains hold across diverse speakers, languages, or acoustic conditions at larger rater scale is not established. The paper does not report inference latency or real-time factor, leaving practical throughput trade-offs relative to BigVGAN unquantified. GAFilter is evaluated only on mel-spectrogram vocoders; its interaction with neural codec decoders or codec-native TTS pipelines is not explored. The SAN framework's tendency to increase audible artifacts under GAFilter integration is noted but not explained mechanistically.

## Wiki Connections

- [[gan-vocoder|GAN Vocoder]] — AF-Vocoder extends the BigVGAN-family GAN vocoder line by introducing a frequency-domain artifact filter module that addresses both aliasing and blurring within the GAN training paradigm.
- [[evaluation-metrics|Evaluation Metrics]] — The paper evaluates using M-STFT, PESQ, Periodicity error, V/UV F1, and UTMOS as complementary objective metrics covering spectral distance, speech quality, and pitch characteristics.
- [[subjective-evaluation|Subjective Evaluation]] — A CMOS listening test with 8 raters on MUSDB18-HD confirms perceptual improvements beyond what objective metrics alone demonstrate.
