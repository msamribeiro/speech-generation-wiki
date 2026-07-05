---
id: "interspeech-2025-2595"
title: "Harnessing Text-to-Speech Voice Cloning Models for Improved Audiological Speech Assessment"
authors: ["Lidea Shahidi", "Erdem Baha Topbas", "Thu Ngan Dang", "Tobias Goehring"]
organization: "University of Cambridge"
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-07-05
task: ["TTS", "evaluation"]
architecture: []
conditioning: []
training: []
model_size: "not reported"
codec_used: "not reported"
datasets_train: ["not applicable (evaluates pre-trained models)"]
datasets_eval: ["BKB (Bamford-Kowal-Bench)"]
metrics: []
code_available: null
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/shahidi25_interspeech.html"
related_concepts: ["zero-shot-tts", "evaluation-metrics", "subjective-evaluation"]
related_papers: []
field_significance:
  level: "low"
  type: ["empirical-benchmark"]
generation:
  date: 2026-07-05
  agent: speech-generation-ingest-agent
  model: claude-sonnet-4-6
  commit: "2829243"
---

> [!abstract] Interspeech · 2025 · Conference
> **Shahidi et al.** (University of Cambridge) · [→ Paper](https://www.isca-archive.org/interspeech_2025/shahidi25_interspeech.html) · Demo: ? · Code: ?
>
> Three open-source zero-shot TTS voice cloning models are evaluated on their ability to replicate a standard UK clinical speech corpus, using a large-scale perceptual study that characterises psychometric intelligibility functions, speech quality, and speaker identity discrimination to identify which models produce stimuli suitable for audiological assessment.

## Problem

Conventional audiological speech assessments rely on small, carefully recorded corpora (such as the UK's BKB sentences) that are fixed in terms of speaker identity and number of available stimuli. The scarcity of stimuli introduces statistical confounds from repeated exposure, limits experimental design, and constrains assessments to the vocal characteristics of a narrow set of speakers. This under-represents speakers from diverse linguistic and social backgrounds, despite evidence that accent familiarity affects comprehension, particularly for older listeners with hearing loss. Prior work using TTS to generate synthetic assessment stimuli either used proprietary models incompatible with open scientific inquiry, or did not account comprehensively for the mismatch in speaker characteristics between original and synthetic speech, confounding psychometric comparisons.

## Method

The study evaluates three open-source zero-shot TTS voice cloning models: VALL-E X, XTTSv2, and E2 TTS. All three were used in pre-trained, zero-shot mode without any fine-tuning. Reference recordings from BKB lists 1-5 were provided to each model to specify the target speaker's acoustic characteristics (a male Southern Standard British English speaker); the models then generated new utterances for BKB lists 6-18 with equivalent semantic content.

Seventy-three native British English listeners with no hearing loss were recruited via an online platform in a pre-registered study. Each completed three perceptual tasks: (1) speech intelligibility measured as word recall percentage across nine signal-to-noise ratios (-12 to +4 dB) in multi-talker babble noise, with logistic psychometric functions fitted to obtain threshold and slope; (2) speech quality rated on the MOS-X2 scale across four dimensions (intelligibility, naturalness, prosody, social impression), with vocoded speech and an HMM-based synthesiser (Festival) included as reference anchors; and (3) speaker identity assessed via a two-alternative forced-choice discrimination task in both quiet and degraded (bandpass-filtered, noise-added) listening conditions. Statistical analysis used repeated-measures ANOVA with Bonferroni-corrected post hoc paired t-tests.

## Key Results

On speech intelligibility, psychometric function thresholds for E2 (difference of 0.41 dB, p=0.053) and VALL-E X (0.05 dB, p=1.00) did not differ significantly from the original BKB corpus. XTTSv2 yielded a significantly lower threshold (-0.58 dB, p<0.001), indicating higher intelligibility than the original. Slope differences were non-significant for all models, confirming that measurement precision was not degraded. VALL-E X, however, showed notable deviations in the psychometric function above -6 dB SNR, a range particularly relevant for hearing-impaired listeners.

On speech quality, MOS-X2 scores ranked: Original > XTTSv2 > E2 > VALL-E X > Festival > Vocoded. All synthetic conditions received significantly lower ratings than the original speech. XTTSv2 and E2 were rated as similarly intelligible to the original, though E2 scored lower on naturalness, prosody, and social impression.

On speaker identity, E2 achieved the highest replication fidelity: listeners judged E2 samples to be more similar to the original speaker than the original samples themselves in quiet conditions. XTTSv2 and VALL-E X received significantly fewer "same speaker" responses when compared to original recordings. For within-model consistency, E2 > XTTSv2 > VALL-E X in both quiet and degraded conditions.

## Novelty Assessment

This paper applies existing zero-shot TTS voice cloning models to a clinical application domain rather than proposing new models or training methods. The study design is its primary contribution: a pre-registered, psychometrically rigorous perceptual evaluation that goes beyond aggregate MOS scores to include psychometric function characterisation and speaker identity discrimination in both quiet and degraded conditions. The inclusion of degraded listening conditions as a proxy for everyday hearing scenarios adds ecological validity beyond standard TTS evaluation practice. The claim to novelty is primarily the domain transfer (audiological assessment) and the comprehensive multi-dimensional evaluation framework, not any advance in synthesis technology.

## Field Significance

Low -- This paper is a careful applied evaluation of existing zero-shot TTS voice cloning models in a clinical domain that is underserved by the TTS literature. It contributes a multi-dimensional perceptual evaluation methodology relevant to audiological research and provides useful evidence that E2 TTS and XTTSv2 can replicate clinically validated speech stimuli with sufficient intelligibility. Its relevance to core TTS research is limited; the primary audience is audiological researchers rather than the speech synthesis community.

## Claims

- **supports:** Zero-shot TTS voice cloning models can replicate clinical speech assessment stimuli with psychometrically comparable intelligibility functions, meeting a necessary condition for their use as synthetic alternatives to recorded corpora.
  > *Evidence:* Psychometric function thresholds for E2 (0.41 dB above original, p=0.053) and VALL-E X (0.05 dB, p=1.00) did not significantly differ from the original BKB corpus, and slopes were statistically indistinguishable across all three models. *(§3.1)*

- **complicates:** High naturalness and speech quality in zero-shot TTS voice cloning do not guarantee accurate speaker identity replication, revealing a trade-off invisible to standard aggregate quality metrics.
  > *Evidence:* XTTSv2 achieved the highest MOS-X2 ratings and lowest intelligibility thresholds but received significantly fewer "same speaker" responses when compared with original recordings; E2 achieved the best speaker replication consistency but lower naturalness, prosody, and social impression scores than XTTSv2. *(§3.2, §3.3)*

- **complicates:** Aggregate TTS quality scores can mask domain-critical performance failures that only appear at specific signal-to-noise operating points.
  > *Evidence:* VALL-E X produced comparable psychometric function thresholds to the original BKB corpus but exhibited significant intelligibility deviations above -6 dB SNR, the range most critical for hearing-impaired listeners, a failure that MOS-X2 ratings did not capture. *(§3.1, §4)*

## Limitations and Open Questions

The study evaluates models on a single target speaker with a specific accent (Southern Standard British English male), and the authors acknowledge that speaker replication fidelity must be assessed across a wider range of accents and speaking styles before deployment in diverse clinical settings. The study population consisted exclusively of normal-hearing British English speakers, meaning that the perceptual results may not generalise to hearing-impaired or non-native English listeners, who are the primary target of audiological assessments. All three models were evaluated in their off-the-shelf pre-trained state; fine-tuning to the target speaker was not explored and may substantially alter the trade-off profiles observed.

## Wiki Connections

- [[zero-shot-tts|Zero-Shot TTS]] -- the three evaluated models (VALL-E X, XTTSv2, E2 TTS) are all zero-shot voice cloning systems that condition on reference audio rather than fine-tuning to a speaker.
- [[evaluation-metrics|Evaluation Metrics]] -- the study employs MOS-X2 quality ratings and psychometric function characterisation as complementary evaluation dimensions, showing their complementary diagnostic value over single-score evaluations.
- [[subjective-evaluation|Subjective Evaluation]] -- the pre-registered large-scale online study (N=73) with real human listeners across three perceptual tasks demonstrates how rigorous perceptual evaluation can reveal model limitations invisible to automated metrics.
