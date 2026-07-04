---
id: "interspeech-2025-2283"
title: "Pairwise Evaluation of Accent Similarity in Speech Synthesis"
authors: ["Jinzuomu Zhong", "Suyuan Liu", "Dan Wells", "Korin Richmond"]
organization: null
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-07-04
task: ["TTS", "evaluation"]
architecture: []
conditioning: []
training: []
model_size: "not applicable"
codec_used: "not applicable"
datasets_train: ["not applicable"]
datasets_eval: ["VCTK", "LJSpeech"]
metrics: []
code_available: null
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/zhong25c_interspeech.html"
related_concepts: ["evaluation-metrics", "subjective-evaluation", "zero-shot-tts", "voice-conversion"]
related_papers: []
field_significance:
  level: "moderate"
  type: ["evaluation-contribution"]
generation:
  date: 2026-07-04
  agent: speech-generation-ingest-agent
  model: claude-sonnet-4-6
  commit: "318f1b4"
---

> [!abstract] Interspeech · 2025 · Conference
> **Zhong et al.** · [→ Paper](https://www.isca-archive.org/interspeech_2025/zhong25c_interspeech.html) · Demo: ? · Code: ?
>
> Proposes refinements to the XAB accent similarity listening test and introduces pronunciation-based objective metrics — vowel formant RMSE and DTW-aligned phonetic posteriorgram distances — that correlate reliably with accent quality rankings for underrepresented accents where WER and UTMOS fail.

## Problem

Accent similarity evaluation in speech synthesis lacks consensus on both subjective and objective methods. Existing subjective approaches either treat accents as one-dimensional intensity scales or apply generic MOS/AB tests not validated for accent-specific assessment. On the objective side, most systems rely on accent identification embedding cosine similarity, whose correlation with actual accent similarity perception is untested. More critically, common proxies for quality — especially WER and UTMOS — carry built-in biases toward well-represented accents, making them unreliable for evaluating systems on underrepresented varieties such as Scottish English. The paper addresses both gaps together: how to design statistically efficient subjective listening tests for accent similarity, and which objective metrics align with human accent similarity judgements.

## Method

The subjective evaluation builds on the XAB listening test format, where listeners choose which of two candidate utterances (A, B) is more similar in accent to a reference (X). Three refinements are proposed and systematically ablated. First, reference transcriptions of the utterances are displayed to listeners, directing attention toward pronunciation details rather than global acoustic impression. Second, listeners are asked to highlight the specific sub-word regions that influenced their decision (adapted from the Rapid Prosody Transcription paradigm), providing an auxiliary task that deepens engagement with accent-related cues. Third, a two-part screening procedure rejects submissions from listeners who either fail attention-check trials or cannot identify the reference accent region (using an open-ended identification question). Test stimuli are drawn from VCTK utterances 001-023 (the rainbow passage read by all speakers), with Edinburgh accent speakers as reference and target; copysyn (HiFi-GAN vocoder applied directly to ground-truth mel-spectrograms) is contrasted against XTTS as a zero-shot TTS baseline.

For objective evaluation, two new pronunciation-distance metrics are introduced. Vowel formant RMSE (VF RMSE) uses Montreal Forced Aligner to obtain phone-level alignments, extracts F1 and F2 formants at vowel midpoints via Fast Track, and computes pairwise RMSE between reference and generated utterances. PPG-based distances use high-fidelity neural phonetic posteriorgrams (PPGs) aligned with Dynamic Time Warping to handle length differences, then measure either cosine similarity distance (PPG CosSim) or Jensen-Shannon divergence (PPG JS) between the aligned distributions. Crucially, the standard PPG normalisation step is dropped because it introduces bias toward General American phonemes. These metrics are benchmarked against a broad suite including accent and speaker embedding cosine similarities, WER/CER, UTMOS, MCD, and F0 statistics by computing Spearman rank correlation with a hypothesised quality ranking derived from the corruption protocol: five variants of XTTS progressively fine-tuned on LJSpeech (30k–150k steps) with increasingly severe accent degradation (corrupt30k through corrupt150k).

## Key Results

Among objective metrics, PPG CosSim and PPG JS achieve SRCC of 0.9643 with the hypothesised accent quality ranking (p=0.0005); VF RMSE reaches SRCC=0.9286 (p=0.0025); speaker similarity via WavLM achieves SRCC=1.0 (p=0), though this may conflate accent and general speaker modelling effects under catastrophic forgetting. By contrast, WER achieves SRCC=0.6429 (p=0.1194, not significant), and UTMOS shows SRCC=0.4643 (p=0.2939, not significant) — both recommended against for accent evaluation. F0 metrics also fail to correlate with the ranking (SRCC=0.1071-0.4643), consistent with the hypothesis that the corruption scheme affects acoustic quality broadly rather than F0 specifically. MCD correlates well (SRCC=0.9643, p=0.0005), suggesting it captures overall acoustic degradation.

For subjective evaluation, the baseline XAB test with 15 listeners yields no statistically significant preference for copysyn over xtts; adding transcriptions drops the required sample to around 10 valid submissions for significance; adding highlights further reduces this to as few as 5, with the full XAB+trans+highlight+screen design achieving 64.1±6.7% preference for copysyn. The cost: completion time nearly doubles from ~17 minutes to ~31 minutes, making the highlight task most valuable when the available listener pool is small (as is typical for underrepresented accent communities).

## Novelty Assessment

The objective metrics are the stronger contribution. VF RMSE and DTW-aligned PPG distances are well-motivated adaptations of existing phonetic analysis tools to the pairwise accent evaluation setting; the decision to drop PPG normalisation is a specific improvement over prior work. The subjective test refinements are incremental — each component is individually motivated by existing evaluation literature — but the comparative ablation across six test configurations on a concrete low-resource accent scenario adds genuine empirical value. The finding that WER is statistically unreliable for evaluating underrepresented accents is confirmatory (bias in ASR models is widely noted) but is here demonstrated systematically in a TTS evaluation context. The study is narrowly scoped: one accent (Edinburgh Scottish), one ZS-TTS system (XTTS), and VCTK speakers, limiting immediate generalisability.

## Field Significance

Moderate — This paper addresses a real practical gap: the absence of validated evaluation protocols for accent similarity in speech synthesis, particularly for accents underrepresented in mainstream TTS benchmarks. The finding that WER and UTMOS are unreliable for this task provides a useful caution for evaluation practice. The proposed pronunciation metrics offer accessible alternatives that can be applied to any accent evaluation setting without subjective test infrastructure.

## Claims

- **supports:** Pronunciation-based objective metrics correlate more reliably with accent quality rankings than general intelligibility or naturalness proxies.
  > *Evidence:* PPG CosSim and PPG JS both achieved SRCC=0.9643 with hypothesised accent ranking (p=0.0005); VF RMSE achieved SRCC=0.9286 (p=0.0025). WER SRCC was 0.6429 and not statistically significant (p=0.1194). *(§5.2, Table 2)*

- **contradicts:** WER is a reliable quality metric for evaluating TTS systems on underrepresented accents.
  > *Evidence:* WER ranked copysyn worse than xtts and xtts worse than corrupt30k, despite the hypothesised reverse quality ordering; WER's Spearman rank correlation with the quality ranking was non-significant (p=0.1194), attributed to accent bias in ASR models trained on standard varieties. *(§5.2, Table 2)*

- **supports:** Providing reference transcriptions to listeners during accent similarity evaluation significantly improves statistical discriminability with fewer participants.
  > *Evidence:* Baseline XAB with 15 listeners failed to reach p<0.05; XAB+trans+screen achieved significance within 10 valid submissions; XAB+trans+highlight+screen required as few as 5 valid submissions, reaching 64.1±6.7% preference for copysyn. *(§5.1, Figure 1)*

- **complicates:** Auxiliary highlight tasks in listening tests improve statistical efficiency but nearly double completion time, limiting benefit to settings with constrained listener pools.
  > *Evidence:* XAB+trans+highlight took ~30.6 minutes vs. ~16.9 minutes for baseline XAB; for accents with large listener pools the cost is not offset by efficiency gains, but for underrepresented accents with few available participants the highlight design remains advantageous. *(§5.1)*

## Limitations and Open Questions

The study evaluates a single accent variety (Edinburgh Scottish English) using two speakers from VCTK. Generalisability to other underrepresented or L2 accents, other ZS-TTS systems, and accent conversion tasks is asserted but not demonstrated. The high SRCC for speaker similarity (WavLM) is acknowledged as potentially confounded by the catastrophic forgetting corruption scheme affecting both accent and broader speaker modelling simultaneously. The XAB setup enforces different-gender reference and target speakers to prevent confusing speaker and accent similarity, but this design choice may not hold for all accent evaluation scenarios. Future work is noted to explore XMOS/XCMOS alternatives, broader accent coverage, and additional synthesis systems.

## Wiki Connections

- [[evaluation-metrics|Evaluation Metrics]] — introduces VF RMSE and DTW-aligned PPG distance as new pronunciation-based metrics for accent similarity evaluation, and demonstrates where WER and UTMOS fail for underrepresented accents.
- [[subjective-evaluation|Subjective Evaluation]] — proposes three refinements to the XAB listening test (transcription display, highlight annotation, meticulous screening) and validates them through a systematic ablation with real human raters.
- [[zero-shot-tts|Zero-Shot TTS]] — uses XTTS as a representative ZS-TTS baseline to demonstrate accent hallucination and evaluate the proposed metrics against a system known to misrepresent non-standard accents.
- [[voice-conversion|Voice Conversion]] — the proposed evaluation framework (PPG distances, XAB refinements) is designed to apply to accent conversion tasks as well as TTS, given that DTW alignment handles the length mismatches common in VC outputs.
