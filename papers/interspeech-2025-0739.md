---
id: "interspeech-2025-0739"
title: "FD-Bench: A Full-Duplex Benchmarking Pipeline Designed for Full Duplex Spoken Dialogue Systems"
authors: ["Yizhou Peng", "Yi-Wen Chao", "Dianwen Ng", "Yukun Ma", "Chongjia Ni", "Bin Ma", "Eng Siong Chng"]
organization: null
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-06-03
task: [SCA, evaluation]
architecture: []
conditioning: []
training: []
model_size: "not reported"
codec_used: "not reported"
datasets_train: []
datasets_eval: ["FD-Bench (generated corpus, 40 hours, 293 conversations)"]
metrics:
  - name: WER
    value: 5.3
    system: Moshi (Easy interruptions)
    testset: FD-Bench CosyVoice2
  - name: WER
    value: 5.0
    system: Moshi (Hard interruptions)
    testset: FD-Bench CosyVoice2
code_available: true
demo_available: true
url: "https://www.isca-archive.org/interspeech_2025/peng25b_interspeech.html"
related_concepts: [spoken-language-model, speech-to-speech, evaluation-metrics, subjective-evaluation]
related_papers: ["2410.06885", "2412.10117"]
field_significance:
  level: moderate
  type: [evaluation-contribution, empirical-benchmark]
generation:
  date: 2026-06-03
  model: claude-sonnet-4-6
  commit: "55d0339"
---

> [!abstract] Interspeech · 2025 · Conference
> **Yizhou Peng et al.** · [→ Paper](https://www.isca-archive.org/interspeech_2025/peng25b_interspeech.html) · Demo: ✓ · Code: ✓
>
> FD-Bench is an automated benchmarking pipeline that evaluates full-duplex spoken dialogue systems on interruption handling and response quality, exposing substantial gaps in the three tested open-source systems under frequent interruptions and noisy conditions.

## Problem

Existing benchmarks for spoken dialogue systems measure performance turn-by-turn: one party finishes before the other responds. This evaluation protocol does not reflect real conversational dynamics, where users interrupt, backchannel, or shift topics mid-utterance. No standard benchmark existed for assessing how well full-duplex spoken dialogue systems (FDSDS) handle simultaneous speaking and listening, manage interrupt-induced latency, or recover coherent responses under noise.

## Method

FD-Bench is an end-to-end pipeline that generates, synthesises, routes, and scores dialogues without human annotation. The pipeline has four stages. First, GPT-4o generates multi-turn conversational scripts (up to 5 rounds) covering dozens of domains, with users designed to interrupt frequently — yielding 293 conversations with approximately 1,200 interruptions across five interruption types (Affirmative Acknowledgment, Denial/Discontent, Further Inquiry, Requiring Repeat, Topic Shift). Second, state-of-the-art TTS systems (ChatTTS, F5-TTS [[2410.06885]], CosyVoice 2 [[2412.10117]]) synthesise speaker-diverse user speech at 24kHz; noise from MUSAN is injected either into silence gaps (Noise-gap) or as continuous background (Noise-bg) at three SNR levels (0/10/20 dB), and three interruption difficulty levels (Easy: 6–10 s silence, Medium: 4–6 s, Hard: 2–4 s) simulate varying interrupt aggressiveness. Third, the generated corpus is streamed in real time to three open-source FDSDS (Moshi, Freeze-Omni, VITA-1.5) via a server-client architecture. Fourth, outputs are transcribed with Whisper-large-v3-turbo and timestamped with Silero-VAD, enabling a suite of objective metrics: interrupt success rates (SRRate, SIRate, SRIRate), timing delays (IRD: interrupt-response delay, FSED: first-speech-emit delay, ERT: early-reply time, EIT: early-interrupt time), early/noise interrupt rates, and WER. Subjective reply quality is scored by GPT-4o across six dimensions (relevance, encouragement, simplicity, flexibility, practicality, creativity), and conditioned perplexity (c-PPL) using Llama-3.3-70B provides a language-model-based coherence signal.

## Key Results

Across all three systems, performance degrades under frequent interruptions and noisy conditions. Moshi, which uses a built-in full-duplex mechanism, consistently outperforms VAD-based systems on interruption handling metrics — its SRRate under Easy interruptions reaches 61.7% vs. 11.3% for Freeze-Omni and 26.9% for VITA-1.5. Moshi's WER remains stable (5.0–5.3%) across interruption difficulties, whereas Freeze-Omni and VITA-1.5 could not produce reliable WER estimates. Moshi's subjective scores are also highest (4.42–4.55) and most consistent; VITA-1.5 scores degrade significantly under hard interruptions (from 2.68 to 2.27), partly due to unwanted English-Chinese code-switching.

For timing, Freeze-Omni achieves the lowest FSED (449–515 ms vs. 2,020–4,155 ms for Moshi), meaning it begins responding faster, even though it handles interruptions less reliably. Extreme IRD values exceeding 10 seconds for Freeze-Omni and VITA-1.5 are attributed to VAD misjudgments and server instability. Under background noise, FSED for Moshi drops markedly — more noise tokens are apparently treated as user input — and subjective scores degrade across all models at low SNR. Gap noise has comparatively little effect on most metrics, with Freeze-Omni's high VAD threshold (0.8) effectively blocking most spurious events.

> [!note]
> Results reported in the paper cover only the CosyVoice2-synthesised subset of the corpus (IDs 5–7 from Table 2). The ChatTTS and F5-TTS subsets are not reported; full-dataset results may differ.

## Novelty Assessment

FD-Bench's contribution is methodological rather than architectural. The pipeline itself — automated dialogue generation, TTS-based corpus synthesis, server-client streaming, and multi-metric scoring — assembles existing components (GPT-4o, TTS engines, Whisper, VAD, LLM-as-judge) into a reproducible evaluation harness. The individual metrics, particularly the timing-based interrupt handling rates, are novel in the FDSDS evaluation context: SRRate, SIRate, SRIRate, EIRate, NIRate, IRD, and FSED are defined and operationalised here, though conceptually they adapt standard dialogue responsiveness concepts to a continuous-audio setting. The result is a practical gap-filler rather than a theoretical advance; its value lies in enabling fair, automated comparison of FDSDS on the specific dimensions that distinguish full-duplex from turn-based operation.

## Field Significance

Moderate — FD-Bench addresses a genuine evaluation gap: the field had no standardised, automated way to benchmark interrupt handling in full-duplex spoken dialogue systems. The empirical findings confirm that current open-source FDSDS still struggle substantially with interruption robustness, which is informative baseline evidence as the sub-field matures. The pipeline's automation and planned data/code release improve reproducibility, but the scope is narrow (English only, three systems, generated-speech inputs), and the benchmark has not yet been widely adopted.

## Claims

- Full-duplex spoken dialogue systems that integrate interruption detection internally rather than relying on external VAD modules show systematically higher interrupt success rates under challenging conditions.
- Real-time responsiveness (first-speech-emit delay) and interruption robustness trade off against each other: systems optimised for low latency may not handle mid-utterance interruptions reliably.
- Background noise degrades response quality in full-duplex spoken dialogue systems more severely than discrete noise events inserted into silence gaps.
- Automated LLM-based scoring and conditioned perplexity provide complementary signals for evaluating spoken dialogue quality, but neither alone captures interruption-handling behaviour.
- Current full-duplex spoken dialogue systems remain far from human-level robustness under frequent interruptions, suggesting the capability remains an open engineering and research challenge.

## Limitations and Open Questions

> [!warning]
> All reported results use speech synthesised by CosyVoice2 only; the ChatTTS and F5-TTS subsets are present in the corpus but results are not reported in the paper. Claims about system behaviour may not generalise across TTS-synthesised voices, and no real human speech is used.

Evaluation is English-only, which limits applicability to multilingual FDSDS. The benchmark depends on GPT-4o for subjective scoring and Llama-3.3-70B for c-PPL, meaning evaluation costs are non-trivial and tied to specific external model versions. VAD threshold sensitivity is acknowledged as confounding for VAD-based systems but is not systematically ablated. Server instability for Freeze-Omni and VITA-1.5 is reported as a factor in extreme IRD values, which reduces confidence in those timing results. The pipeline does not yet test prosodic or paralinguistic aspects of interruption (e.g., overlapping speech, backchannels in the acoustic sense), focusing instead on lexical and timing dimensions.

## Wiki Connections

Concepts directly relevant to this paper: [[spoken-language-model]], [[speech-to-speech]], [[evaluation-metrics]], [[subjective-evaluation]].

In-corpus papers used as TTS engines for corpus synthesis: [[2410.06885]] (F5-TTS), [[2412.10117]] (CosyVoice 2).
