---
id: "interspeech-2025-0355"
title: "Probing the Robustness Properties of Neural Speech Codecs"
authors: ["Wei-Cheng Tseng", "David Harwath"]
organization: University of Texas at Austin
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-07-02
task: [codec, evaluation]
architecture: []
conditioning: []
training: []
model_size: "not reported"
codec_used: "not applicable (evaluation study)"
datasets_train: ["not applicable (evaluation study)"]
datasets_eval: ["LibriSpeech test-clean", "RAVDESS"]
metrics:
  - name: PESQ
    value: "not reported (per-codec, per-noise-condition)"
    system: multiple
    testset: LibriSpeech test-clean
  - name: WER
    value: "not reported (per-codec, per-noise-condition)"
    system: multiple
    testset: LibriSpeech test-clean
  - name: EER
    value: "not reported (per-codec, per-noise-condition)"
    system: multiple
    testset: LibriSpeech test-clean
code_available: null
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/tseng25_interspeech.html"
related_concepts: [neural-codec, evaluation-metrics, autoregressive-codec-tts]
related_papers: ["2301.02111"]
field_significance:
  level: moderate
  type: [empirical-benchmark, evaluation-contribution]
generation:
  date: 2026-07-02
  agent: speech-generation-ingest-agent
  model: claude-sonnet-4-6
  commit: "d8ecfd4"
---

> [!abstract] Interspeech · 2025 · Conference
> **Tseng and Harwath** (University of Texas at Austin) · [→ Paper](https://www.isca-archive.org/interspeech_2025/tseng25_interspeech.html) · Demo: ✗ · Code: ?
>
> A systematic empirical evaluation of five neural speech codecs under ambient noise, white noise, and reverberation conditions, revealing significant and non-obvious differences in robustness across codecs and identifying linearity violations and spectral distortions as explanatory mechanisms.

## Problem

Neural speech codecs have been widely adopted both as audio compression tools and as tokenization backends for speech language models, but almost all evaluations assume clean speech. Real-world deployments involve background noise, reverberation, and amplitude variation, yet no prior work had systematically characterized how codec noise robustness varies across models or identified which design properties predict robust behavior. The opacity of deep-learning-based codecs makes failure modes difficult to anticipate without empirical probing.

## Method

The study evaluates five neural speech codecs (EnCodec, DAC, SpeechTokenizer, HiFi-Codec, and FreqCodec) under three noise corruption types: ambient noise (from WHAM), white noise (Gaussian), and reverberation (room impulse responses from the DNS Challenge). Clean speech is mixed with each noise source at multiple signal-to-noise levels. Codecs operate at their highest available bitrate for the primary comparison, with additional bitrate sweeps to probe how robustness varies with compression level.

Evaluation uses five objective metrics covering complementary information axes: Mel Distance and PESQ for perceptual quality, Whisper-Large WER for phonetic content preservation, ECAPA-TDNN EER for speaker identity retention, and emotion2vec accuracy on RAVDESS for emotional content fidelity. Test data are drawn from LibriSpeech test-clean (perceptual quality, intelligibility, speaker) and the RAVDESS speech subset (emotion).

Beyond noise robustness, the study probes two mathematical properties of codec input-output mappings. The linearity analysis tests additivity (whether encoding overlapping signals equals encoding them separately and summing) and homogeneity (whether scaling the input scales the output proportionally) across bitrates and gain levels. The frequency response analysis uses sine sweep stimuli to characterize spectral distortions introduced by each codec, validated across multiple amplitude levels.

## Key Results

At the highest available bitrate, DAC consistently demonstrates the strongest noise robustness, exhibiting the least degradation in WER, EER, and emotion accuracy across all three noise types. EnCodec and FreqCodec are competitive, while SpeechTokenizer and HiFi-Codec degrade substantially even under moderate noise. However, this ordering is not preserved at lower bitrates: at 3 kbps, EnCodec outperforms DAC in preserving downstream-relevant information across most conditions, indicating that noise robustness is bitrate-dependent and not an intrinsic property of the codec architecture.

The linearity analysis shows that higher bitrates generally improve additivity across all codecs, and that codecs with better additivity (particularly DAC) also exhibit stronger noise robustness. DAC and FreqCodec maintain relatively stable homogeneity across gain variations, while HiFi-Codec and SpeechTokenizer amplify unevenly at extreme gains, which aligns with their poorer performance in noisy conditions.

Frequency response characterization reveals that all codecs exhibit low-frequency boosting below 100 Hz, likely a byproduct of time-domain waveform losses (L1/L2), most pronounced in EnCodec. In the mid-range (100 Hz to 2 kHz), DAC, EnCodec, and FreqCodec maintain stable responses; HiFi-Codec and SpeechTokenizer show significant fluctuations that correspond to observed intelligibility degradation. Above 2 kHz, all codecs attenuate, with DAC and EnCodec showing smoother roll-offs than HiFi-Codec and SpeechTokenizer.

## Novelty Assessment

The contribution is primarily empirical: the paper applies standard signal analysis techniques (linearity testing, frequency response via sine sweeps) to neural codecs as black-box systems, and evaluates them under noise conditions not previously studied in a controlled, cross-codec comparative setup. The choice of metrics is comprehensive and multi-dimensional, covering perceptual, phonetic, speaker, and emotional axes simultaneously. No new codec or training method is proposed. The linearity framing is conceptually simple but the application to neural codecs is genuinely informative, providing a mechanistic partial explanation for the observed robustness differences. The finding that bitrate-ranked robustness at high bitrates does not predict bitrate-ranked robustness at low bitrates is non-obvious and practically relevant.

## Field Significance

Moderate — This paper fills a gap in codec benchmarking by demonstrating that noise robustness is a distinct, undercharacterized property that does not simply follow from clean-speech reconstruction quality or overall bitrate. The linearity and frequency response analyses provide interpretable, design-actionable diagnostics rather than just rankings, and the multi-metric evaluation covering speaker, emotion, and intelligibility is a more complete picture than standard perceptual quality scores alone. The findings are directly relevant to codec selection for speech LM pipelines deployed in noisy environments.

## Claims

- **supports:** Codec noise robustness does not reliably predict across bitrate regimes, and clean-speech quality metrics are a poor proxy for noisy-condition performance.
  > *Evidence:* At the highest bitrate, DAC outperforms EnCodec in noise robustness; at 3 kbps, EnCodec surpasses DAC on most downstream metrics under noise. HiFi-Codec matches EnCodec in clean-speech perceptual quality but degrades far more under noise. *(§4, Figure 1 in paper)*

- **supports:** Neural speech codecs exhibit non-linear input-output behavior (violations of additivity and homogeneity) that partially explains their robustness failures under noise and signal overlap.
  > *Evidence:* Additivity improves with bitrate across all evaluated codecs, and codecs with better additivity (DAC) show stronger noise robustness. HiFi-Codec and SpeechTokenizer exhibit uneven homogeneity at extreme gains, aligning with their higher degradation in speaker and emotion metrics. *(§5, Figure 2 in paper)*

- **complicates:** Codec evaluation using only clean-speech reconstruction metrics provides an incomplete characterization of codec behavior for real-world speech processing pipelines.
  > *Evidence:* Perceptual quality (PESQ) converges to similar values across codecs under severe noise, masking large differences in phonetic content (WER), speaker identity (EER), and emotional fidelity (SER-ACC). *(§4)*

- **supports:** Low-frequency spectral emphasis introduced by time-domain training losses is a consistent artifact across neural speech codecs and can cause unintended spectral coloration.
  > *Evidence:* All five evaluated codecs exhibit low-frequency boosting below 100 Hz under sine sweep analysis, most pronounced in EnCodec; the effect is attributed to waveform-level L1/L2 losses used in training. *(§6, Figure 3 in paper)*

## Limitations and Open Questions

The evaluation is limited to five codecs, all of which use residual vector quantization. More recent codecs with different quantization strategies (e.g., fully transformer-based or finite scalar quantization approaches) are not included. The noise conditions, while diverse, use fixed SNR ranges and do not cover all real-world degradation types (e.g., codec artifacts, packet loss, or non-stationary noise sources). All test data are English; whether these robustness rankings generalize across languages and speaking styles is unexamined. The emotion preservation evaluation relies on a single emotion recognizer (emotion2vec) trained on limited data, and speaker identity assessment uses a single ECAPA-TDNN model, which may introduce evaluator-specific biases. No human perceptual evaluation is included.

## Wiki Connections

- [[neural-codec|Neural Audio Codec]] — This paper provides the most systematic multi-dimensional noise robustness characterization of neural codecs published at the time, adding diagnostics (linearity, frequency response) not present in prior benchmarks.
- [[evaluation-metrics|Evaluation Metrics]] — The study demonstrates that perceptual quality metrics (PESQ, Mel Distance) are insufficient indicators of codec robustness, motivating multi-metric evaluation covering downstream task performance across phonetic, speaker, and emotional axes.
- [[autoregressive-codec-tts|Autoregressive Codec TTS]] — Robustness findings are directly relevant to codec-based TTS and speech LM systems that may operate over noisy input audio, as codec failures propagate to downstream generation quality.
- [[2301.02111|VALL-E]] — Cited as a motivating application of neural codecs as speech tokenizers; robustness findings in this paper directly bear on how VALL-E-style systems perform in noisy real-world conditions.
