---
id: "interspeech-2025-2536"
title: "The Text-to-speech in the Wild (TITW) Database"
authors: ["Jee-weon Jung", "Wangyou Zhang", "Soumi Maiti", "Yihan Wu", "Xin Wang", "Ji-Hoon Kim", "Yuta Matsunaga", "Seyun Um", "Jinchuan Tian", "Hye-jin Shim", "Nicholas Evans", "Joon Son Chung", "Shinnosuke Takamichi", "Shinji Watanabe"]
organization: null
venue: "Interspeech"
venue_type: "conference"
year: 2025
month: 8
published_date: "2025-08-17"
ingested_date: "2026-07-05"
task: ["TTS", "evaluation"]
architecture: []
conditioning: []
training: []
model_size: "not reported"
codec_used: "not reported"
datasets_train: ["TITW-Easy", "TITW-Hard"]
datasets_eval: ["TITW-KSKT", "TITW-KSUT"]
metrics:
  - name: UTMOS
    value: 3.08
    system: MQTTS
    testset: TITW-KSKT
  - name: DNSMOS
    value: 2.83
    system: MQTTS
    testset: TITW-KSKT
  - name: WER
    value: 23.3
    system: MQTTS
    testset: TITW-KSKT
code_available: null
demo_available: false
url: "https://www.isca-archive.org/interspeech_2025/jung25c_interspeech.html"
related_concepts: ["evaluation-metrics", "diffusion-tts", "autoregressive-codec-tts"]
related_papers: ["2301.02111"]
field_significance:
  level: "moderate"
  type: ["dataset-contribution", "evaluation-contribution"]
generation:
  date: "2026-07-05"
  agent: speech-generation-ingest-agent
  model: claude-sonnet-4-6
  commit: "2829243"
---

> [!abstract] Interspeech · 2025 · Conference
> **Jung et al.** · [→ Paper](https://www.isca-archive.org/interspeech_2025/jung25c_interspeech.html) · Demo: ✗ · Code: ?
>
> Introduces TITW (Text-to-Speech in the Wild), a publicly available dataset derived from VoxCeleb1 through a fully automated pipeline, providing the first standardized public benchmark for training and evaluating TTS systems on in-the-wild speech.

## Problem

TTS training has historically depended on studio recordings, which are acoustically homogeneous, expensive to produce, and scale poorly to underrepresented languages. Research into noisy-TTS training — using naturally occurring in-the-wild audio — has made incremental progress, but without a standardised public dataset and evaluation protocol, results are largely not reproducible: most studies use private data or artificially degraded clean recordings. The absence of a shared benchmark also makes it difficult to track whether the field is actually solving the problem of training from raw social media audio.

## Method

TITW is constructed from VoxCeleb1, a 1,251-speaker corpus of YouTube interview footage originally designed for automatic speaker recognition. The processing pipeline requires no manual annotation. In a first stage, the entire corpus is transcribed at word-level using WhisperX (OpenAI Whisper Large v2 with forced phoneme-based alignment), with OWSMv3 applied in parallel to cross-check accuracy. Voice Activity Detection embedded in WhisperX then segments each recording wherever non-speech silence exceeds 500ms, yielding approximately 280,000 candidate segments.

A heuristic data selection stage filters on four criteria: utterance duration (1–8 seconds), per-word duration below 500ms (removing very slow or disfluent speech), automatic language identification to English only, and non-empty transcript. The result of transcription, segmentation, and this selection stage alone is TITW-Hard: approximately 248,000 segments totaling 173 hours (Table 1).

TITW-Easy extends the pipeline with two additional stages applied to the same base data: background noise reduction using DEMUCS, followed by DNSMOS-based quality filtering that removes any segment scoring below 3.0. Despite additional filtering, TITW-Easy contains approximately 283,000 segments totaling 189 hours (Table 1), reflecting that enhancement raises many borderline segments above the quality threshold. TITW-Easy reaches UTMOS 3.32 and DNSMOS 2.78, compared to UTMOS 3.00 and DNSMOS 2.38 for TITW-Hard, though both remain substantially noisier than standard benchmarks such as VCTK (DNSMOS 3.20) and EMILIA (DNSMOS 3.26) (Table 2).

Two evaluation protocols are defined. TITW-KSKT (Known Speaker, Known Text) generates speech for speakers and text present in the training data, targeting 40 speakers aligned with the VoxCeleb1-O speaker verification protocol, yielding 9,113 evaluation segments. TITW-KSUT (Known Speaker, Unknown Text) uses the same 40 speakers but 200 unseen text prompts (Rainbow Passage and Semantically Unpredictable Sentences), requiring 8,000 generated utterances.

## Key Results

Four TTS systems are benchmarked, all trained on TITW-Easy: TransformerTTS with ParallelWaveGAN, GradTTS-DiffWave, VITS, and MQTTS. MQTTS achieves the best quality on TITW-KSKT with UTMOS 3.08, DNSMOS 2.83, and WER 23.3%. Performance degrades across all systems under TITW-KSUT, with MQTTS WER rising to 67.1%, reflecting the added challenge of unseen text (Table 3).

Training on TITW-Hard produces substantially worse outcomes. GradTTS-DiffWave UTMOS drops from 2.18 (Easy) to 1.29 (Hard), DNSMOS from 2.39 to 1.47, and WER increases from 11.9% to 26.2%. TransformerTTS and MQTTS fail to converge at all on TITW-Hard, while VITS degrades only modestly (UTMOS 2.77 vs. 2.48) (Table 4). A copy-synthesis upper bound using ground-truth mel-spectrograms with DiffWave achieves UTMOS 2.63 and DNSMOS 2.68 on TITW-Easy, showing that the vocoder itself is not the primary quality bottleneck.

## Novelty Assessment

The primary contribution is a dataset and evaluation protocol, not a new architecture or training method. TITW is, to the authors' knowledge, the first public dataset specifically designed for noisy-TTS training that uses fully in-the-wild audio rather than artificially degraded studio recordings. The choice of VoxCeleb1 as a source is deliberate: its speaker verification provenance ensures single-speaker segments and creates a natural bridge to speech deepfake detection research. The automated pipeline is reproducible and language-agnostic in principle, though the current release is English only. The evaluation protocols (KSKT and KSUT) are straightforward adaptations of speaker verification methodology to TTS, which is a sensible design choice rather than a novel contribution on its own. The negative result demonstrating that TITW-Hard is too difficult for current TTS systems is practically informative and motivates the dual-tier design.

## Field Significance

Moderate — TITW addresses a genuine reproducibility gap in noisy-TTS training research by providing the first public, standardised in-the-wild TTS dataset and evaluation benchmark. It demonstrates that automated pipelines applied to speaker recognition corpora can yield usable TTS training material, and that quality filtering (DNSMOS-based selection after enhancement) is necessary with current model families. The deepfake detection angle strengthens the dataset's broader utility beyond its primary TTS role.

## Claims

- **supports:** Speech enhancement and automated quality filtering are necessary preprocessing steps for training current-generation TTS systems on in-the-wild audio.
  > *Evidence:* All four TTS systems trained on TITW-Easy (enhanced, DNSMOS-filtered) converge and produce intelligible speech, while TransformerTTS and MQTTS fail to converge on TITW-Hard (unenhanced); GradTTS-DiffWave UTMOS falls from 2.18 to 1.29 and DNSMOS from 2.39 to 1.47 when switching from Easy to Hard training data. *(§3.3, §5.3, Table 4)*

- **supports:** Training TTS models directly on raw social media audio without enhancement remains too unstable for most current model architectures.
  > *Evidence:* TITW-Hard data (DNSMOS 2.38) caused divergence in two of four tested systems despite heuristic data selection; only VITS and GradTTS-DiffWave converged, and both showed degraded quality relative to their TITW-Easy counterparts. *(§3.2, §5.3, Table 4)*

- **complicates:** Automated perceptual quality metrics such as UTMOS and DNSMOS do not consistently predict intelligibility performance at inference time.
  > *Evidence:* TITW-Easy and TITW-Hard training sets differ modestly in WER (9.1% vs. 9.3%), but GradTTS-DiffWave trained on Hard exhibits WER of 26.2% vs. 11.9% for Easy; the DNSMOS gap (2.38 vs. 2.78) understates the intelligibility collapse in the generated speech. *(§5.2, §5.3, Table 2, Table 4)*

- **supports:** Automatic speaker recognition datasets derived from in-the-wild audio provide a scalable and ethically beneficial source for TTS corpus construction.
  > *Evidence:* VoxCeleb1 (1,251 speakers, YouTube-sourced) yields 173–189 hours of TTS-trainable speech through a fully automated pipeline, and its single-speaker-per-segment guarantee enables direct pairing of synthetic and genuine speech for deepfake detection benchmarks. *(§3, Table 1)*

## Limitations and Open Questions

> [!warning]
> Table 1 and the paper's introduction text report contradictory sample counts: the introduction states TITW-Hard contains 189 hours and TITW-Easy 173 hours, while Table 1 shows the opposite (Easy 189h, Hard 173h). The Key Results and Claims sections above use Table 1 values, which are consistent with Table 2 quality scores (Easy > Hard). Readers consulting the paper should verify against the released dataset directly.

The dataset is currently English-only; the authors leave multilingual extension to future work, which limits utility for underrepresented language TTS. Evaluation uses only automated metrics (UTMOS, DNSMOS, MCD, WER) without human listening tests, so the perceptual quality of models trained on TITW relative to studio-trained counterparts is not directly measured. The DNSMOS threshold of 3.0 for TITW-Easy was set empirically and may not generalise to other downstream models or tasks.

## Wiki Connections

- [[evaluation-metrics|Evaluation Metrics]] — the paper adopts UTMOS, DNSMOS, MCD, and WER as a multi-dimensional quality battery and introduces TITW-KSKT and TITW-KSUT as evaluation protocols for in-the-wild TTS.
- [[diffusion-tts|Diffusion TTS]] — GradTTS-DiffWave (a diffusion-based acoustic model paired with DiffWave) is one of the four benchmarked systems, providing a reference point for how diffusion TTS handles in-the-wild training data.
- [[autoregressive-codec-tts|Autoregressive Codec TTS]] — MQTTS (a vector-quantized autoregressive TTS model) achieves the strongest quality on both TITW protocols among the benchmarked systems; VALL-E-style codec LM developments ([[2301.02111|VALL-E]]) are cited as the motivation for studying noisy-TTS training at scale.
- [[2301.02111|VALL-E]] — cited as a representative modern TTS system that motivates the need for diverse in-the-wild training data and scales zero-shot synthesis; TITW is positioned as a resource to enable this class of system to train outside controlled studio conditions.
