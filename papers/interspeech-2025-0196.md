---
id: "interspeech-2025-0196"
title: "SPCODEC: Split and Prediction for Neural Speech Codec"
authors: ["Liang Wen", "Lizhong Wang", "Yuxing Zheng", "Weijing Shi", "Kwang Pyo Choi"]
organization: Samsung
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-06-02
task: [codec]
architecture: [GAN, VAE]
conditioning: []
training: [supervised]
model_size: "not reported"
codec_used: "none"
datasets_train: ["VCTK", "LibriTTS"]
datasets_eval: ["ITU-T P.501 test signals", "EVS test sequences"]
metrics:
  - name: MOS
    value: 4.63
    system: SPCODEC (16kHz, 10.66kbps)
    testset: ITU-T P.501 / EVS test sequences
  - name: MOS
    value: 4.68
    system: SPCODEC (32kHz, 16kbps)
    testset: ITU-T P.501 / EVS test sequences
  - name: PESQ
    value: 4.5
    system: SPCODEC (16kHz, 10.66kbps)
    testset: POLQA objective MOS (MOS-LQO)
code_available: null
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/wen25_interspeech.html"
related_concepts: ["neural-codec", "disentanglement", "evaluation-metrics"]
related_papers: []
field_significance:
  level: moderate
  type: [architectural-novelty]
generation:
  date: 2026-06-21
  agent: speech-generation-review-agent
  model: claude-sonnet-4-6
  commit: "3538db5"
---

> [!abstract] Interspeech · 2025 · Conference
> **Liang Wen et al.** (Samsung) · [→ Paper](https://www.isca-archive.org/interspeech_2025/wen25_interspeech.html) · Demo: ? · Code: ?
>
> SPCODEC introduces a latent split-and-prediction mechanism into neural speech coding that explicitly disentangles low- and high-frequency latent representations, using the quantized low-frequency component to predict and reduce redundancy in the high-frequency component, achieving state-of-the-art POLQA scores for wideband and super-wideband speech.

## Problem

Existing time-domain neural speech codecs — SoundStream, EnCodec, DAC — quantize a single unified latent embedding without imposing any spectral structure on it. This ignores the well-established asymmetry between low- and high-frequency speech: lower frequencies dominate perceptual quality and are more predictable, while higher frequencies are partially redundant given the lower-frequency content. Prior work on latent partitioning (HiFiCodec's group VQ, SRCodec's split-residual approach) divides embeddings mechanically without constraining what spectral content each group encodes, leaving redundancy unexploited. Frequency-domain subband codecs avoid this by operating explicitly in the spectral domain, but they introduce algorithmic latency that makes them unsuitable for real-time communication.

## Method

SPCODEC is a causal, time-domain, fully convolutional encoder-decoder codec built on the same backbone as DAC. The core innovation is the split-and-prediction (SP) module applied within the quantization stage. The encoder output is split along the channel dimension into a low-frequency embedding `e_l` (larger dimension) and a high-frequency embedding `e_h` (smaller dimension). Unlike group VQ approaches, the split is enforced semantically: `e_l` is supervised by reconstruction loss applied to a low-pass-filtered version of the input waveform, while the full reconstruction is supervised separately, ensuring that each partition captures its intended spectral content.

An attention-based prediction module operates between the two groups. During encoding, the quantized low-frequency embedding `ê_l` is used to predict a correction that is subtracted from `e_h` before quantization, removing the component predictable from low frequencies and thus reducing bitstream redundancy. During decoding, the same module adds back the predicted component to the quantized high-frequency representation before the decoder reconstructs the full waveform. The two groups use separate quantizers `Q_l` and `Q_h` with independent codebooks. At 16 kHz, the system uses eight 8-bit codebooks for `Q_l` and four 4-bit codebooks for `Q_h`, totalling 10.66 kbps. At 32 kHz, eight 10-bit and four 10-bit codebooks yield 16 kbps. The SP module itself adds only 33K parameters and 4.3 MMAC/s of compute.

![SPCODEC: a time domain encoder-quantizer-decoder codec architecture which is trained end-to-end with reconstruction loss, VQ loss, adversarial loss and feature matching loss for both reconstructed low frequency waveform and 'low+high' frequency waveform. Latent embedding is split into disentangled two parts, which are supervised to contain high and low frequency information, respectively. Prediction module: high frequency feature is predicted from input low and high frequency information for encoder's feature redundancy removal and decoder's feature fusion.](assets/interspeech-2025-0196/figure-2.png)

Training follows the DAC paradigm: adversarial loss using a multi-period discriminator and a complex multi-scale band-splitting STFT discriminator, feature matching loss, and multi-resolution spectrogram reconstruction loss, applied jointly to both the full-band and low-band outputs. The system is causal (zero algorithmic delay), achieving a real-time factor of 8.6× for WB and 3.9× for SWB on a single CPU core.

## Key Results

On a 30-sample multilingual (English, Chinese, Korean) test set drawn from ITU-T P.501 and EVS test sequences, SPCODEC outperforms all baselines on both objective POLQA and subjective MOS. At 10.66 kbps (16 kHz), it achieves POLQA 4.5 and MOS 4.63 ± 0.17, compared to Opus at 4.15 POLQA and 4.46 ± 0.18 MOS, EVS at 4.14 POLQA and 4.12 ± 0.21, and SoundStream at 3.68 POLQA and 3.49 ± 0.29. At 16 kbps (32 kHz), SPCODEC achieves POLQA 4.53 and MOS 4.68 ± 0.13, matching Opus at 4.63 MOS but with superior POLQA.

In the ablation study (all variants run on the DAC backbone for fairness), the SP module consistently improves over DAC, HiFiCodec, and SRCodec on MelDistance, STOI, and POLQA at both bitrate levels and both sampling rates. The performance advantage of the SP module is more pronounced at higher bitrates: at 32 kHz / 16 kbps, SPCODEC reaches POLQA 4.543 vs. DAC's 4.501, but at 5.33 kbps the gap narrows. The authors attribute this bitrate-scaling advantage to the predictive module deriving greater benefit from higher-quality low-frequency representations at higher bitrates.

> [!note]
> The subjective MOS evaluation used only 7 professional audio engineers as listeners, not a crowd-sourced panel. Confidence intervals are reported but the sample size is small relative to standard listening test practice.

## Novelty Assessment

The split-and-prediction mechanism is a genuine structural contribution to neural codec design. The key novelty is not the idea of splitting embeddings — that exists in HiFiCodec and SRCodec — but the combination of spectral supervision (explicitly training each partition to encode its designated frequency band) with inter-group prediction to remove cross-band redundancy. This is a principled application of classical bandwidth extension and predictive coding ideas within a learned latent space. The overall system architecture (convolutional encoder-decoder, RVQ, GAN discriminators) is standard; the contribution is entirely within the quantization module and training objective. The evaluation is fair in the ablation (same backbone, same training settings) though the baseline codec comparisons span different sampling rates and bitrates, making cross-system MOS comparisons less clean. The test set of 30 samples is small.

## Field Significance

Moderate — SPCODEC demonstrates that imposing spectral structure on learned codec embeddings — through supervised disentanglement and inter-group prediction — yields measurable gains over unstructured group VQ approaches, particularly at higher bitrates. The SP module is designed to be lightweight (33K parameters) and drop-in compatible with existing codecs, which gives it practical value as an architectural add-on. However, the gains are incremental over the strongest baselines, and the evaluation scale is limited.

## Claims

- Imposing spectral structure on codec latent embeddings through supervised disentanglement improves compression efficiency over unstructured group quantization. *(§2.2, §3.3, Table 3)*
- Inter-group prediction in neural codecs, where high-frequency representations are predicted from quantized low-frequency components, reduces bitstream redundancy without proportional increases in model size. *(§2.2, §3.1)*
- The performance advantage of latent-space spectral decomposition in neural codecs scales with bitrate, with larger gains at higher bitrates where low-frequency quality is sufficient to anchor the prediction. *(§3.3, Table 3)*
- Causal, time-domain neural speech codecs can match or exceed traditional codecs (EVS, Opus) on perceptual quality metrics at equivalent bitrates. *(§3.2, Table 2)*

## Limitations and Open Questions

> [!warning]
> The subjective evaluation uses only 7 listeners; this falls below the minimum typically required to draw statistically robust MOS conclusions. The test set of 30 samples, while drawn from standardised ITU-T and EVS material, is small for a codec evaluation claiming state-of-the-art status.

The evaluation covers only speech; generalisability of the SP approach to music or general audio is not addressed. The cross-codec comparisons in Table 2 mix sampling rates (16 kHz vs. 24 vs. 32 kHz), so direct MOS comparison between SPCODEC and Encodec requires care. The ablation holds the encoder-decoder fixed to DAC's architecture, which means the gains may differ when applied to other backbone designs. No streaming or latency analysis is provided beyond the real-time factor on one CPU configuration.

## Wiki Connections

Concept pages most relevant to this paper: [[neural-codec]], [[disentanglement]], [[evaluation-metrics]].

The split-and-prediction approach extends the group residual vector quantization line established by codecs like HiFiCodec and SRCodec, differentiating itself through explicit spectral supervision rather than structural partitioning alone.
