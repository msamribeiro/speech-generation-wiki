---
id: "iclr-2026-0oXyMbPMtP"
title: "MambaVoiceCloning: Efficient and Expressive Text-to-Speech via State-Space Modeling and Diffusion Control"
authors: ["Sahil Kumar", "Namrataben Patel", "Honggang Wang", "Youshan Zhang"]
organization: "Yeshiva University, Chuzhou University"
venue: ICLR
venue_type: conference
year: 2026
month: 4
published_date: "2026-04-27"
ingested_date: "2026-10-20"
task: [TTS]
architecture: [diffusion]
conditioning: [text-conditioned, speaker-conditioned, zero-shot, prompt-conditioned]
training: [supervised]
model_size: "21M params (conditioning encoder stack); full system not reported"
codec_used: "none"
datasets_train: ["LJSpeech", "LibriTTS"]
datasets_eval: ["LJSpeech", "LibriTTS (unseen speakers)", "VCTK", "CSS10 (ES/DE/FR)", "Gutenberg long-form passages"]
metrics:
  - name: MOS
    value: 4.22
    system: proposed
    testset: LibriTTS unseen speakers (Table 1)
  - name: SMOS
    value: 4.07
    system: proposed
    testset: LibriTTS unseen speakers (Table 1)
  - name: MOS
    value: 4.15
    system: StyleTTS2
    testset: LibriTTS unseen speakers (Table 1)
  - name: SMOS
    value: 4.03
    system: StyleTTS2
    testset: LibriTTS unseen speakers (Table 1)
  - name: MOS
    value: 4.18
    system: proposed
    testset: VCTK zero-shot (Table 16)
  - name: SMOS
    value: 4.09
    system: proposed
    testset: VCTK zero-shot (Table 16)
  - name: WER
    value: 6.52
    system: proposed
    testset: LJSpeech (Table 4, %)
  - name: WER
    value: 6.50
    system: StyleTTS2
    testset: LJSpeech (Table 4, %)
  - name: F0-RMSE
    value: 0.653
    system: proposed
    testset: LJSpeech (Table 4)
  - name: PESQ
    value: 3.85
    system: proposed
    testset: LJSpeech (Table 4)
  - name: PESQ
    value: 3.79
    system: StyleTTS2
    testset: LJSpeech (Table 4)
code_available: true
demo_available: null
url: "https://openreview.net/forum?id=0oXyMbPMtP"
related_concepts: [diffusion-tts, zero-shot-tts, streaming-tts, subjective-evaluation]
related_papers: ["2105.06337", "2403.03100", "2505.17589", "2006.04558", "1712.05884", "1703.10135", "1904.02882"]
field_significance:
  level: "moderate"
  type: [architectural-novelty]
generation:
  schema_version: 2
  date: "2026-10-20"
  runtime: "claude-code"
  provider: "anthropic"
  agent: speech-generation-ingest-agent
  model: "claude-opus-5-5"
  commit: "fb8dafe"
---

> [!abstract] ICLR · 2026 · Conference
> **Sahil Kumar et al.** (Yeshiva University, Chuzhou University) · [→ Paper](https://openreview.net/forum?id=0oXyMbPMtP) · Demo: ? · Code: ✓
>
> Replaces every attention and recurrent module in the inference-time conditioning path of a StyleTTS2-style diffusion TTS system (text, rhythm/duration, and prosody) with selective state-space (Mamba) blocks, keeping the diffusion decoder and vocoder fixed so that the effect of the conditioning stack can be isolated.

## Problem

Conditioning stacks in non-autoregressive diffusion TTS typically rely on transformer attention or recurrent modules for text encoding, duration, and style. Attention costs quadratic time and memory and mixes global context, which complicates chunked streaming; recurrent modules drift over long inputs. Earlier Mamba-based TTS encoders (Speech Slytherin, "Mamba in Speech") remain hybrid at inference, keeping attention-based duration or style modules. The paper asks whether the whole conditioning path can be made SSM-only at inference without losing quality, and it tests this under a controlled setup where only the conditioning stack changes.

## Method

MambaVoiceCloning (MVC) takes phonemized text (phonemizer with espeak-ng, language tags for non-English tests) and an 80-bin log-mel reference at 24 kHz. A shallow conv/GRU module averages over the reference mel to give a global style embedding that is shared by all encoders. The conditioning stack has three SSM modules, all with state dimension 96, depthwise convolution kernel 5, and gating temperature 1.0:

- **Gated Bi-Mamba text encoder.** Forward and backward Mamba scans are combined through a learned sigmoid gate over the concatenated directions (earlier bi-Mamba TTS encoders used plain concatenation), then style-modulated with adaptive layer normalization (AdaLN). The default depth is 6 layers.
- **Expressive Mamba encoder.** Operates on the reference mel features after a gated AdaLN transform and models slow prosodic dynamics.
- **Temporal Bi-Mamba encoder.** Models rhythm and duration with forward/backward Mamba blocks plus a local Conv1D, using linear (ungated) fusion.

During training only, a 2-layer, 4-head transformer aligner (hidden size 256) with a monotonic alignment loss maps text encodings to frame-level features. It is discarded at inference, so the deployed conditioning path runs in linear time without attention maps. Pitch is predicted by gate-fusing the expressive and temporal streams, and a Conv1D+SSM speech-dynamics stage produces an F0 trajectory and a residual noise vector that condition the diffusion decoder.

![Figure 1: Overview of MambaVoiceCloning (MVC). The framework uses Bi-Mamba Text Encoders for phoneme modeling, a Temporal Bi-Mamba for rhythmic alignment, and an Expressive Mamba for prosodic control. A lightweight aligner (dotted box) provides phoneme-frame supervision only during training, ensuring an SSM-only encoder at inference. Conditioning features drive a diffusion decoder and vocoder for waveform synthesis.](assets/iclr-2026-0oXyMbPMtP/figure-1.png)

The decoder is the unmodified StyleTTS2 mel diffusion model with a fixed 5-step schedule, followed by corpus-matched vocoders (iSTFTNet for LJSpeech, HiFi-GAN for LibriTTS). Training combines L1 mel reconstruction, least-squares adversarial losses with multi-period and multi-resolution discriminators, and the alignment loss. The conditioning encoder stack has 21M parameters; the size of the full system is not reported. For streaming, the bidirectional text encoder is swapped for a causal Uni-Mamba that carries SSM state across chunks and sees a finite look-ahead of L seconds.

All baselines (StyleTTS2, VITS, JETS, a Hybrid-Mamba that keeps attention in duration/style modules, and an SSM-only Bi-Mamba with concat fusion) are retrained in the authors' codebase with identical front-end, decoder, vocoder, optimizer, and schedule. Mamba baselines are matched for encoder parameter count within ±5%.

## Key Results

- **Unseen LibriTTS speakers (Table 1):** MOS-N 4.22 and MOS-S 4.07, against 4.15/4.03 for StyleTTS2 and 3.69/3.54 for VITS (paired t-test vs. StyleTTS2, p < 0.01).
- **LJSpeech objective metrics (Table 4):** MCD 4.91 and PESQ 3.85 are best, while F0 RMSE (0.653 vs. 0.651) and WER (6.52% vs. 6.50%) match StyleTTS2 rather than improve on it. RTF is 0.0169 vs. 0.0174.
- **Protocol-matched Mamba baselines (Table 12):** the ordering is Hybrid-Mamba < Bi-Mamba concat-only < MVC on every metric, but the margins are small (e.g., WER 6.68% → 6.58% → 6.52%; PESQ 3.79 → 3.82 → 3.85).
- **Zero-shot and cross-lingual (Tables 16, 17):** on VCTK, MOS-N 4.18 / MOS-S 4.09 vs. 4.12 / 4.01 for StyleTTS2 (significant only for MOS-S). On CSS10, the English-trained model gains slightly on ES and FR and matches StyleTTS2 on DE.
- **OOD and long-form (Tables 2, 3):** MOS on Gutenberg OOD text holds steady (3.87 ID → 3.88 OOD) while VITS and JETS degrade. On >60 s segments, MOS-long is 4.16 vs. 3.91 for StyleTTS2, which is the largest gap the paper reports.
- **Efficiency (Tables 9, 15):** the encoder has 21M parameters vs. 42M for StyleTTS2, with 1.6× encoder throughput and 72% of StyleTTS2's peak encoder memory. The diffusion decoder still takes 54.2% of per-utterance runtime against 31.4% for the encoder stack, so end-to-end RTF gains are small.
- **Streaming (Table 5):** on 2-6 minute passages, look-ahead of 0.25 s gives 11.2% WER / 3.74 MOS, and 2.0 s gives 7.3% / 3.91.
- **Ablations (Tables 6, 8):** removing any one encoder costs 0.36-0.41 CMOS-N on OOD text, with the Expressive Mamba the largest. On long-form, concat-only fusion with neither gating nor AdaLN drops MOS-long from 4.16 to 3.64.

The comparisons are fair in the sense that decoder, vocoder, and schedule are shared, but every baseline is a re-implementation inside the authors' codebase, so absolute baseline numbers may differ from the original releases. Industrial-scale systems are discussed only qualitatively (Table 22).

## Novelty Assessment

The contribution is architectural but narrow. Mamba encoders for TTS already exist; what is new is removing attention from duration and style modeling as well (with attention kept only as a training-time teacher), plus the gated forward/backward fusion with AdaLN. The authors describe MVC as "an encoder-side refinement rather than a paradigm shift", and the numbers agree. Against StyleTTS2 the subjective gains are about 0.04-0.08 MOS on short utterances, WER and F0 RMSE are at parity, and the real differences are encoder memory, encoder throughput, and long-form MOS. The protocol-matched Hybrid-Mamba and concat-only Bi-Mamba controls are the most informative part of the design, because they separate "remove inference-time attention" from "add gated fusion and AdaLN". The fusion ablation suggests most of the long-form gain comes from gating and AdaLN rather than from being SSM-only.

## Field Significance

Moderate. The paper gives a controlled, decoder-matched check that the conditioning path of a diffusion TTS system can run entirely on state-space models at inference while matching or slightly beating an attention-based conditioner, with clear savings in encoder memory and throughput. It also shows that end-to-end latency stays bounded by the diffusion decoder, so encoder-side efficiency alone yields only small real-time-factor gains.

## Claims

- **supports:** State-space sequence models can replace attention throughout the inference-time conditioning path of non-autoregressive TTS without loss of naturalness or intelligibility.
  > *Evidence:* Under an identical StyleTTS2 diffusion decoder and vocoder, the SSM-only conditioner matches StyleTTS2 on LJSpeech WER (6.52% vs. 6.50%) and F0 RMSE and slightly exceeds it on MOS-N for unseen LibriTTS speakers (4.22 vs. 4.15), with a 21M-parameter encoder against 42M. *(§5.1, Table 1, Table 4; §A.1, Table 9)*
- **complicates:** Efficiency gains in the text and prosody conditioning encoders yield only small end-to-end speed improvements when an iterative diffusion decoder remains in the pipeline.
  > *Evidence:* The diffusion decoder accounts for 54.2% of per-utterance runtime against 31.4% for the Mamba encoder stack, so a 1.6× encoder throughput gain translates to an LJSpeech RTF change of only 0.0174 → 0.0169. *(§5, §D.4, Table 15, Table 4)*
- **refines:** Simply swapping attention for a bidirectional SSM is not enough to sustain long-form prosody; how the two scan directions are fused and how style modulates the encoder matter more than removing attention.
  > *Evidence:* On long-form LJSpeech, concat-only fusion reaches MOS-long 3.64 against 4.16 for gated fusion plus AdaLN, with pitch RMSE 2.89 vs. 1.92 Hz. Removing inference-time attention alone (Hybrid-Mamba → concat-only Bi-Mamba) gives much smaller changes. *(§5.4, Table 8; §B.6, Table 12)*
- **supports:** Linear-time, state-carrying encoders allow chunked streaming synthesis whose quality approaches non-streaming quality once a short look-ahead is available.
  > *Evidence:* With a causal Uni-Mamba carrying state across chunks on 2-6 minute passages, WER falls from 11.2% at 0.25 s look-ahead to 7.3% at 2.0 s, and MOS rises from 3.74 to 3.91. Boundary artifacts mostly disappear at L ≥ 0.5 s. *(§5.3, Table 5)*
- **supports:** Long-form naturalness separates TTS conditioning architectures more clearly than short-utterance evaluation does.
  > *Evidence:* The MOS gap between the SSM-only system and StyleTTS2 is 0.07 on segments of 10 s or less but 0.25 on segments over 60 s (4.16 vs. 3.91). *(§5.2, Table 3)*

## Limitations and Open Questions

> [!warning]
> All baselines are re-trained inside the authors' own codebase and most subjective gains over StyleTTS2 fall within roughly 0.1 MOS. Zero-shot (VCTK) and cross-lingual (CSS10) tables compare only against VITS and StyleTTS2, not the protocol-matched Mamba controls. The improvements therefore rest on small margins under one implementation.

- The training data is small and English-only (LJSpeech plus LibriTTS, about 270 hours). Cross-lingual CSS10 results come from an English-trained model with language-tagged phonemization, not from multilingual training, and stress errors on long German compounds remain.
- The diffusion decoder is the main latency source and is left unchanged. Whether SSM-only conditioning helps with flow-matching or single-step decoders is not tested.
- AdaLN supplies a global style vector only. The paper states that MVC does not target fine-grained emotion or prosody control.
- Pitch error is reported on different scales across tables: F0 RMSE around 0.65 in Tables 4 and 12, pitch RMSE around 23 Hz in Table 7, and around 1.9 Hz in Table 8 (long-form utterances, a different test condition, though the tenfold gap is not explained by it). The units and normalization behind Tables 4 and 12 are not stated. MVC's LJSpeech RTF also appears as 0.0169 in Table 4 and 0.0177 in Table 12 alongside otherwise identical metric values. Cross-table comparisons of these figures should be read with care.
- Subjective tests use 5-10 MTurk raters per clip on fairly small sets (for example, 80 LibriTTS clips and 20-30 CSS10 clips per language). No demo page is reported.

## Wiki Connections

- [[diffusion-tts|Diffusion TTS]] — Keeps a StyleTTS2 mel diffusion decoder fixed and redesigns only its conditioning path, showing that the decoder still dominates inference latency.
- [[zero-shot-tts|Zero-Shot TTS]] — Evaluates reference-audio conditioning on unseen VCTK speakers and tests sensitivity to reference length (2-8 s).
- [[streaming-tts|Streaming TTS]] — Characterizes chunked streaming with a causal Uni-Mamba and finite look-ahead, measuring the WER and MOS trade-off from 0.25 to 2.0 s.
- [[subjective-evaluation|Subjective Evaluation]] — Reports crowd-sourced MOS-N, MOS-S, and CMOS with 95% confidence intervals and Holm-Bonferroni-corrected paired tests.
- [[2105.06337|Grad-TTS]] — Cited as the diffusion TTS lineage behind the decoder and as the reference point for choosing the number of diffusion steps.
- [[2403.03100|NaturalSpeech 3]] — Discussed as an industrial-scale contextual reference that is excluded from numeric comparison because of its data scale.
- [[2505.17589|CosyVoice 3]] — Likewise treated only as a qualitative contextual reference, not as a decoder-matched baseline.
- [[2006.04558|FastSpeech 2]] — Cited as the non-autoregressive duration-modeling basis of the JETS baseline.
- [[1904.02882|LibriTTS]] — Provides the multi-speaker training data and the unseen-speaker subjective test set.
- [[1712.05884|Tacotron 2]] — Cited as an attention-based TTS pipeline representing the quadratic-cost conditioning the paper aims to remove.
