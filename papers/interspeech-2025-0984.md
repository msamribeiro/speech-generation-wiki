---
id: "interspeech-2025-0984"
title: "Benchmarking Neural Speech Codec Intelligibility with SITool"
authors: ["Anna Leschanowsky", "Kishor Kayyar Lakshminarayana", "Anjana Rajasekhar", "Lyonel Behringer", "Ibrahim Kilinc", "Guillaume Fuchs", "Emanüel A. P. Habets"]
organization: null
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-06-04
task: [codec, evaluation]
architecture: []
conditioning: []
training: []
model_size: "not reported"
codec_used: "not reported"
datasets_train: []
datasets_eval: ["DRT word lists (ANSI/ASA S3.2-2020)", "MRT word lists"]
metrics:
  - name: STOI
    value: 0.93
    system: DAC-S (female)
    testset: DRT benchmark
  - name: WER
    value: 0.19
    system: Original (male reference)
    testset: DRT benchmark
code_available: null
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/leschanowsky25_interspeech.html"
related_concepts: [neural-codec, evaluation-metrics, subjective-evaluation]
related_papers: []
field_significance:
  level: moderate
  type: [evaluation-contribution, empirical-benchmark]
generation:
  date: 2026-06-04
  model: claude-sonnet-4-6
  commit: "6312840"
---

> [!abstract] Interspeech · 2025 · Conference
> **Leschanowsky et al.** (Fraunhofer IIS / UC San Diego / Int'l Audio Labs Erlangen) · [→ Paper](https://www.isca-archive.org/interspeech_2025/leschanowsky25_interspeech.html) · Demo: ? · Code: ?
>
> SITool, a Flask-based web toolkit for Diagnostic Rhyme Test and Modified Rhyme Test, is used to benchmark 13 speech codecs on subjective intelligibility, revealing that STOI and ESTOI correlate with DRT scores but WER does not, and that objective metrics miss gender- and wordlist-specific variation.

## Problem

Codec evaluation has concentrated on overall speech quality (MOS, MUSHRA, DCR) rather than on speech intelligibility specifically. Intelligibility is a distinct perceptual dimension that does not necessarily correlate with quality, yet it is critical for communication applications such as digital telephony, broadcasting, and media streaming. Publicly available tools for standardised intelligibility testing — specifically the Diagnostic Rhyme Test (DRT) and Modified Rhyme Test (MRT) — are either outdated, platform-locked (e.g., Qualtrics only), or not adapted for crowdsourcing environments. The relationship between objective intelligibility metrics (STOI, ESTOI, WER) and subjective DRT scores for neural codecs has also not been systematically studied.

## Method

SITool is a standalone Flask web application that administers DRT and MRT tests in both laboratory and crowdsourcing (Amazon Mechanical Turk) settings. The tool supports consent forms, language proficiency screening (B1/B2 Cambridge English level), trap questions, a gold-standard reference condition, and a lower anchor. Score calculation follows the ANSI/ASA S3.2-2020 guessing-corrected formula.

The benchmark evaluates 13 codecs: three traditional CELP-based codecs (AMR, AMR-WB, EVS), the parametric Codec2, and nine neural codecs spanning GAN-based end-to-end autoencoders (Lyra V2, DAC-S, SNAC), transformer-bottleneck designs (Mimi at 0.55 and 1.1 kbps), and a diffusion-augmented semantic codec (SemantiCodec at 340 and 680 bps). Bitrates range from 0.34 kbps (SemantiCodec) to 8 kbps (EVS). Stimuli use English DRT word lists at 16 kHz with male and female talkers. Eighty-two crowdsourced participants were retained after post-screening. Objective metrics — STOI, ESTOI, and WER via Whisper — were computed on the same stimuli and compared to subjective DRT scores via correlation and linear regression analyses.

## Key Results

Subjective intelligibility results show that below 1,100 bps all neural codecs degrade significantly relative to the uncoded reference. DAC-S (speech fine-tuned, 1.5 kbps) matches or exceeds EVS (8 kbps) for both male and female talkers — a notable finding given the six-fold bitrate difference. SemantiCodec at 340 bps achieves only 76–78% DRT accuracy, the lowest among neural codecs tested.

A systematic gender gap exists: the female reference signal scores lower than the male (M=85 vs. M=91.87), and this gap propagates through codec conditions. Wordlist 2 consistently yields the lowest scores, driven by high error rates on word pairs testing /f/ and /θ/ consonants. These gender- and wordlist-specific effects are statistically significant in a Linear Mixed-Effects Model (p < .0001).

On the correlation between objective and subjective metrics: STOI achieves Pearson r = 0.958 and ESTOI r = 0.890 when averaged over gender and wordlist, but correlation drops substantially at the disaggregated level (STOI: 0.595, ESTOI: 0.499). WER shows no meaningful correlation with subjective DRT scores (r = -0.11 averaged, -0.15 disaggregated), questioning its validity as an intelligibility proxy for single-word codec evaluation.

> [!note]
> STOI/ESTOI and WER were computed on the same stimuli as subjective DRT scores, enabling direct metric-to-subjective comparison. However, WER was obtained via Whisper, which also shows high error rates on the original clean signal (WER 0.19–0.25), suggesting ASR-based intelligibility measurement may be confounded by the single-word utterance format.

## Novelty Assessment

The contribution is dual: an open-source evaluation tool (SITool) and a systematic benchmark of 13 codecs on intelligibility. Neither component is architecturally novel — DRT and MRT are decades-old standardised tests, and the codecs themselves are existing systems. The value lies in filling a tooling gap (no publicly available, crowdsourcing-ready DRT/MRT platform existed) and in providing the correlation analysis between objective metrics and DRT scores, which has practical consequences for codec developers who currently rely on WER as an intelligibility proxy. The finding that WER is uncorrelated with DRT outcomes — even at the aggregate level — is a concrete negative result with methodological implications.

## Field Significance

Moderate — SITool addresses a genuine tooling gap in codec evaluation methodology. The correlation analysis between DRT and objective metrics (STOI, ESTOI, WER) provides actionable guidance: STOI and ESTOI are reasonable proxies for aggregate intelligibility ranking but cannot substitute for subjective tests when gender- or wordlist-specific variation matters. The negative result on WER is particularly relevant for labs that use Whisper-based WER as a cheap intelligibility proxy in codec development loops.

## Claims

- Neural speech codecs operating below approximately 1,100 bps degrade in intelligibility relative to uncoded speech, even when subjective quality metrics suggest acceptable performance. *(§4.1)*
- WER derived from large-vocabulary ASR systems is not a valid proxy for subjective speech intelligibility in codec benchmarking on isolated word stimuli. *(§4.3)*
- STOI and ESTOI correlate well with subjective DRT intelligibility scores when averaged across talker gender and wordlists, but fail to capture the finer-grained variation that subjective tests reveal. *(§4.3)*
- Talker gender significantly affects subjective intelligibility scores independently of codec condition, and this variation is not reliably reflected in objective intelligibility metrics. *(§4.1, §4.2)*

## Limitations and Open Questions

> [!warning]
> The female reference signal shows consistently lower DRT scores than the male reference (M=85 vs. M=91.87), suggesting potential issues with the audio materials rather than codec effects. The authors flag this for further investigation, and it limits confidence in the codec-level conclusions for female talkers.

The study uses closed-set word tests (DRT/MRT), which are prone to ceiling effects at higher codec qualities — a limitation the authors acknowledge by suggesting future use of open-set SUS tests. The benchmark is limited to English and to 16 kHz stimuli (codecs designed for higher sampling rates were resampled), which may not reflect their intended operating conditions. The crowdsourced sample of 82 participants is distributed unevenly across eight test blocks, with some blocks approaching the recommended minimum of eight participants per condition. WER results were obtained via Whisper, which already yields 19–25% WER on clean single-word utterances, questioning whether the metric has sufficient headroom to distinguish codec conditions meaningfully.

## Wiki Connections

- [[neural-codec]] — directly benchmarks 13 neural and traditional codecs on a new intelligibility dimension
- [[evaluation-metrics]] — provides a systematic correlation analysis between STOI, ESTOI, WER, and subjective DRT/MRT scores, with a negative result on WER
- [[subjective-evaluation]] — introduces SITool as an open framework for crowdsourced DRT/MRT testing
