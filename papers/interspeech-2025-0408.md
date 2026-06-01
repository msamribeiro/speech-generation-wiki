---
id: interspeech-2025-0408
title: "Improving User Impression of Spoken Dialogue Systems by Controlling Para-linguistic Expression Based on Intimacy"
authors: [Shoki Kawanishi, Akinori Ito, Yuya Chiba, Takashi Nose]
organization: Tohoku University
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: "2025-08-17"
ingested_date: "2026-05-30"
task: [SCA, TTS]
architecture: [transformer-enc-dec, GAN]
conditioning: [emotion-conditioned, prosody-conditioned]
training: [supervised, fine-tuning]
model_size: "not reported"
codec_used: "none"
datasets_train: [SMOC corpus (Japanese, 4700 utterances), JSUT corpus (voice conversion target)]
datasets_eval: [SMOC test set (100 utterances), subjective dialogue experiments (33 participants)]
metrics:
  - name: MOS
    value: 3.36
    system: Proposed (Day 3)
    testset: VCTK
code_available: null
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/kawanishi25_interspeech.html"
related_concepts: [spoken-language-model, prosody-control, emotion-synthesis, subjective-evaluation]
related_papers: []
---
> [!abstract] Interspeech · 2025 · Conference
> **Shoki Kawanishi et al.** (Tohoku University) · [→ Paper](https://www.isca-archive.org/interspeech_2025/kawanishi25_interspeech.html) · Demo: ? · Code: ?
>
> A spoken dialogue system that progressively adapts both linguistic style and paralinguistic (prosodic) expression from formal to intimate as interactions accumulate, improving user satisfaction over three-day dialogue experiments compared to fixed-style baselines.

## Problem

LLM-based dialogue systems respond uniformly regardless of the history of interactions with a user, failing to replicate how humans adjust their speech style based on perceived social closeness. Prior work on intimacy-adaptive dialogue focused only on linguistic changes (e.g., switching from honorific to casual Japanese) while neglecting paralinguistic cues (prosody, speech rate, F0) that also covary with relationship intimacy. This gap limits how natural and appealing conversational agents feel over repeated interactions.

## Method

The system has two adaptation axes operating in concert.

**Linguistic adaptation:** GPT-4 generates responses under one of two prompts: a polite/formal prompt (Day 1) and a casual/intimate prompt (Day 3), with a 50/50 mix on Day 2. This follows prior work by Kageyama et al. (2018).

**Paralinguistic adaptation (new contribution):** A Tacotron 2-based TTS system, augmented with an intimacy embedding network replacing the speaker embedding, is trained on the SMOC Japanese spontaneous multimodal corpus. Utterances are labeled high-intimacy (acquaintances with subjective scores 4–5) or low-intimacy (first-meeting pairs). The intimacy embedding is a binary vector concatenated with the Tacotron 2 encoder output. Voice conversion (RVC tool) is applied first to reduce speaker variability in the training data to a single JSUT female speaker. HiFi-GAN serves as the neural vocoder. Binary intimacy labels switch in sync with linguistic style changes across the three experiment days.

**Experiment setup:** 33 participants (11 per condition) conducted 10-exchange daily spoken dialogues over 3 days with a CG agent (MMDAgent-EX, emotion/gesture fixed to neutral). Three conditions: Casual (always high-intimacy speech), Polite (always low-intimacy speech), and Proposed (gradual adaptation). Participants rated 11 items on 5-point scales.

## Key Results

Objective evaluation confirms the TTS model captures intimacy: speech rate RMSE and log-F0 RMSE are lower when synthesized speech intimacy level matches the ground-truth label. In subjective dialogue evaluation, the Proposed condition achieves Satisfaction (Q1) of 4.00 on Day 3, the highest of all conditions (Casual: 3.27, Polite: 3.45). Cognitive Demand (Q11) decreases most steeply in the Proposed condition, falling below the Casual condition by Day 3. Friendliness (Q2) is highest for Proposed on Day 3 (4.09), though differences are narrow. No statistically significant differences between conditions are observed on Day 3 for any individual item, but the Proposed condition receives the best rating in 7 of 11 items on the final day.

## Novelty Assessment

The core novelty is the integration of continuous paralinguistic (prosodic) intimacy control into a multi-session dialogue management loop. The individual components — Tacotron 2 TTS, HiFi-GAN, GPT-4 response generation, intimacy-label conditioning — are standard. The contribution is systemic: demonstrating that gradually adjusting both linguistic and paralinguistic style over days improves user impressions beyond linguistic-only adaptation. The effect is real but modest in magnitude; larger longitudinal studies with more participants are needed to reach statistical significance. The paper is primarily an application/systems contribution.

## Limitations and Open Questions

Only binary (high/low) intimacy labels are used; a continuous intimacy scale might enable finer-grained adaptation. The experiment spans only 3 days; longer interactions may reveal larger effects. The CG agent's gesture and emotion were frozen, which may have suppressed multimodal intimacy cues. The study is Japanese-language only, limiting generalizability. Future work intends to incorporate multimodal behaviors (gestures, facial expressions) and longer-term experiments.

## Wiki Connections

This paper informs [[spoken-language-model]] on the integration of adaptive speech style into multi-session SCA. The intimacy-conditioned TTS relates to [[prosody-control]] and [[emotion-synthesis]] — though the intimacy dimension is distinct from typical emotion categories. The evaluation methodology (multi-session subjective questionnaire) connects to [[subjective-evaluation]]. No in-corpus citations identified.
