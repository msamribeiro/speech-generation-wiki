---
id: interspeech-2025-2765
title: "The State Of TTS: A Case Study with Human Fooling Rates"
authors: ["Praveen Srinivasa Varadhan", "Sherry Thomas", "Sai Teja M S", "Suvrat Bhooshan", "Mitesh M. Khapra"]
organization: IIT Madras
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-06-03
task: ["TTS", "evaluation"]
architecture: []
conditioning: ["zero-shot"]
training: ["fine-tuning"]
model_size: "not reported"
codec_used: "not reported"
datasets_train: ["Expresso"]
datasets_eval: ["LJSpeech", "LibriTTS", "LibriSpeech", "Expresso"]
metrics:
  - name: MUSHRA
    value: 85.37
    system: PlayHT
    testset: Expresso
  - name: MUSHRA
    value: 80.39
    system: ElevenLabs
    testset: Expresso
  - name: MUSHRA
    value: 74.78
    system: Human reference
    testset: Expresso
code_available: null
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/srinivasavaradhan25_interspeech.html"
related_concepts: ["evaluation-metrics", "subjective-evaluation", "zero-shot-tts", "speaker-adaptation"]
related_papers: ["2410.06885"]
field_significance:
  level: moderate
  type: ["empirical-benchmark", "evaluation-contribution"]
generation:
  date: 2026-06-03
  model: claude-sonnet-4-6
  commit: "55d0339"
---

> [!abstract] Interspeech · 2025 · Conference
> **Varadhan et al.** (IIT Madras) · [→ Paper](https://www.isca-archive.org/interspeech_2025/srinivasavaradhan25_interspeech.html) · Demo: ? · Code: ?
>
> The paper introduces Human Fooling Rate (HFR), a binary forced-choice deception metric for TTS evaluation, and shows through a large-scale crowdsourced study that CMOS/MUSHRA-based claims of human parity do not hold under direct human deception testing — with commercial models approaching but not matching human listeners, and open-source systems falling significantly short.

## Problem

Existing TTS evaluation methods — MOS, CMOS, MUSHRA — measure preference and relative quality but do not directly test whether listeners can distinguish synthetic from human speech. As TTS systems have reported CMOS gains that nominally exceed human reference scores, a disconnect has emerged between these evaluation signals and real-world perceptual indistinguishability. Furthermore, standard benchmarks such as LJSpeech and LibriSpeech contain recordings with low expressive variation, causing even human reference recordings to score poorly on deception; systems trained or evaluated against these benchmarks can appear successful by matching inherently limited references rather than achieving genuine naturalness.

## Method

HFR is defined as the percentage of trials in which a TTS sample is misclassified as human speech in a binary forced-choice test. Listeners are presented with single recordings and must decide: human or machine? They are guided to attend to specific perceptual cues — digital voice quality, unnatural pauses, pitch variation, prosodic appropriateness, word skips, and artifacts. No relative comparison or preference rating is involved.

The evaluation is conducted on SAFFRON, a crowdsourcing-compatible platform the authors developed to standardise HFR and MUSHRA tests with strict listening controls (forced full-sample playback, response-time tracking, integration with the Prolific recruitment platform). Ten TTS systems are evaluated: five open-source (StyleTTS 2, XTTS, GPT-SoVITS, F5-TTS, VoiceCraft) and two commercial (ElevenLabs, PlayHT), across three standard benchmarks (LJSpeech, LibriTTS, LibriSpeech) and the Expresso dataset of professional expressive speech. A fine-tuning sub-experiment adapts the best and worst open-source systems on Expresso (40 hours) and re-evaluates HFR. A granular variant of HFR asks raters to also label which perceptual markers led them to identify speech as synthetic.

In total, 135 native US-English speakers recruited from Prolific provided over 30,300 ratings; each system receives responses from at least 30 participants across 30 utterances.

## Key Results

On standard benchmarks (zero-shot, open-source systems), no system approaches the human HFR baseline of 74.11%. The best-performing model, StyleTTS 2, achieves a mean HFR of 50.89% versus the human baseline — despite claiming CMOS +0.28 on LJSpeech. F5-TTS, which reports CMOS +0.31 on Seed-TTS test-en, scores 46.78% mean HFR.

On the Expresso benchmark, the gap between commercial and open-source systems is sharper. PlayHT (HFR 71.49) and ElevenLabs (HFR 69.85) match the human reference rate of 70.68, while the best open-source system (F5-TTS, HFR 50.26) remains 20 points behind. MUSHRA scores do not reflect this gap: XTTS scores MUSHRA 76.58 — higher than the human reference (74.78) — yet fools listeners only 41.8% of the time.

Fine-tuning on Expresso yields only modest gains: F5-TTS improves from 50.26 to 52.22 HFR; VoiceCraft improves more substantially (30.52 to 43.45), but neither system approaches the human level.

The granular HFR analysis identifies digital voice quality (36.1%), unnatural pauses (22.8%), and flat/monotonic delivery (20.6%) as the primary markers causing open-source systems to be identified as synthetic. Commercial systems show error rates on these markers comparable to human speech.

HFR tests are also more efficient: average rating time per sample is 24.3 s for standard HFR, 22.5 s for granular HFR, versus 42.5 s for MUSHRA.

## Novelty Assessment

The HFR metric itself is not architecturally novel — binary forced-choice discrimination tests have precedent in psychoacoustics and the NLP Turing test literature — but applying them systematically and at scale to modern TTS evaluation is a meaningful contribution to the field. The key insight that CMOS/MUSHRA scores can be inflated by reference-matching bias against low-expressivity benchmarks is empirically demonstrated here for the first time with direct evidence from deception-rate data. The evaluation scope (ten systems, four datasets, 135 participants, 30,000+ ratings) is substantial for a subjective study and adds credibility to the findings. The SAFFRON platform and the granular HFR variant are practical tools the field can adopt. The contribution is primarily empirical and methodological rather than architectural.

## Field Significance

Moderate — This paper provides direct empirical evidence that commonly used preference-based metrics (CMOS, MUSHRA) overestimate the naturalness of open-source TTS systems when measured against a human deception standard, and that the choice of evaluation benchmark materially affects conclusions about system quality. It reinforces a growing concern in the evaluation literature that the apparent near-parity results from recent systems are partly an artifact of evaluation design. The SAFFRON platform and HFR procedure offer a replicable, lower-cost alternative to traditional subjective evaluations.

## Claims

- CMOS and MUSHRA scores can overestimate naturalness relative to human deception rates, because listeners in preference tests match stimuli to reference recordings rather than assess absolute human-likeness.
- Evaluating TTS systems on benchmarks with low expressive variation sets an artificially low bar, as human recordings from such datasets are themselves infrequently identified as human.
- Commercial TTS systems achieve near-human deception rates in zero-shot speaker adaptation to expressive conversational speech, while leading open-source systems remain substantially below this threshold.
- Fine-tuning on high-quality expressive data improves but does not close the gap between open-source TTS naturalness and human speech in deception-based evaluations.
- Digital voice quality artifacts and flat prosody are the primary perceptual cues that allow listeners to reliably identify synthetic speech from open-source systems.

## Limitations and Open Questions

> [!warning]
> The study evaluates only US-English native listeners on English speech; findings about human deception rates may not generalise to other languages, accents, or listener populations with different familiarity with synthetic speech.

The evaluation is limited to ten systems due to budget constraints, excluding many recent strong models. HFR is a binary measure that does not distinguish degrees of unnaturalness, and — like all subjective tests — is subject to listener variability and session-level fatigue effects. The fine-tuning experiments use only 40 hours of Expresso; whether larger or more diverse fine-tuning data would close the open-source/commercial gap remains an open question. SAFFRON's reproducibility in other institutional contexts (ethics approval, crowdsourcing platform) has not been demonstrated.

## Wiki Connections

This paper directly informs [[evaluation-metrics]] and [[subjective-evaluation]], providing evidence that standard metrics are insufficient for deployment-readiness assessment. The zero-shot evaluation places [[zero-shot-tts]] and [[speaker-adaptation]] in context by demonstrating a large performance gap between commercial and open-source systems. The F5-TTS result is directly measured here: [[2410.06885]].
