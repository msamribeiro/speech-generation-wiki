---
id: "interspeech-2025-1726"
title: "Voice Reconstruction through Large-Scale TTS Models: Comparing Zero-Shot and Fine-tuning Approaches to Personalise TTS in Assistive Communication"
authors: ["Éva Székely", "Péter Mihajlik", "Máté Soma Kádár", "László Tóth"]
organization: null
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: "2025-08-17"
ingested_date: "2026-07-03"
task: ["TTS", "evaluation"]
architecture: ["hybrid"]
conditioning: ["text-conditioned", "speaker-conditioned", "zero-shot", "prompt-conditioned"]
training: ["supervised", "fine-tuning"]
model_size: "not reported"
codec_used: "not reported"
datasets_train: ["Hungarian spontaneous speech (13h lecture recordings)", "Hungarian voice banking corpus (195 sentences)"]
datasets_eval: ["Hungarian spontaneous speech (held-out)", "Hungarian voice banking corpus (195 sentences)", "Hungarian dysarthric speech recordings"]
metrics:
  - name: SPK-SIM
    value: 0.727
    system: FT-Spont-Long
    testset: Hungarian voice banking (held-out spontaneous)
  - name: SPK-SIM
    value: 0.635
    system: ZS-Typical
    testset: Hungarian voice banking (held-out spontaneous)
  - name: CER
    value: 2.3
    system: FT-Spont-Long + dysarthric prompt
    testset: Hungarian voice banking (195 sentences)
code_available: true
demo_available: true
url: "https://www.isca-archive.org/interspeech_2025/szekely25_interspeech.html"
related_concepts: [zero-shot-tts, speaker-adaptation, prosody-control, subjective-evaluation, evaluation-metrics]
related_papers: []
field_significance:
  level: "moderate"
  type: ["empirical-benchmark", "engineering-integration"]
generation:
  date: "2026-07-03"
  agent: speech-generation-ingest-agent
  model: claude-sonnet-4-6
  commit: "49d73f8"
---

> [!abstract] Interspeech · 2025 · Conference
> **Székely et al.** · [→ Paper](https://www.isca-archive.org/interspeech_2025/szekely25_interspeech.html) · Demo: ✓ · Code: ✓
>
> Evaluates zero-shot and fine-tuned XTTS-v2 across five data availability scenarios for personalised TTS in assistive communication, including a novel interaction mode that uses dysarthric speech as an audio prompt for user-guided prosody adaptation.

## Problem

Augmentative and Alternative Communication (AAC) users who require personalised synthetic speech face a fundamental constraint: the quality and speaker fidelity of the voice depends heavily on how much unaffected pre-morbid speech is available. Voice banking, the proactive recording of speech before significant deterioration, is feasible for progressive conditions but inaccessible for sudden-onset events such as stroke. Prior work on personalised TTS has largely assumed access to moderate-to-large quantities of clean, scripted speech. This paper asks how far modern zero-shot and fine-tuned large-scale TTS models can extend the boundary, including for speakers whose only available recordings consist of dysarthric or otherwise atypical speech.

## Method

The study builds on XTTS-v2, an open-source multilingual autoregressive TTS model that combines a GPT-2-based encoder with a diffusion-based decoder. Speaker conditioning in its zero-shot mode operates through a conditioning encoder that extracts speaker characteristics from a reference audio clip of as little as 10 seconds, producing embeddings that guide the diffusion decoder. The paper defines five data availability scenarios structured around a single case study: a Hungarian stroke survivor who had pre-morbid recordings of 13 hours of spontaneous lecture speech and 195 voice-banking sentences, and who later re-recorded those sentences with dysarthric speech.

Five systems were evaluated: three fine-tuned variants (FT-Spont-Long on 13h of spontaneous speech, FT-Spont-Short on 1h, and FT-Read on 195 sentences totalling roughly 12 minutes), and two zero-shot systems using 10 seconds of either typical speech (ZS-Typical) or dysarthric speech (ZS-Atypical) as the reference audio. Fine-tuned models were trained on a single GPU until validation loss minimised.

Objective evaluation used Speaker Encoder Cosine Similarity (SECS) via ECAPA2 embeddings against held-out spontaneous speech. Perceptual evaluation used a MUSHRA-like listening test in a situational framing (listeners were briefed on the AAC context and asked to rate suitability for conversational use on a 1-5 scale), with 30 native Hungarian listeners and statistical analysis via one-way ANOVA with post-hoc Tukey tests.

A separate experiment introduced an interactive prosody adaptation mode: the zero-shot input pathway was modified so that the GPT-2 encoder receives the dysarthric audio prompt (for prosody information) while the diffusion vocoder receives a fixed speaker embedding from the pre-morbid corpus (for voice quality). Prosody transfer was validated by comparing prominence peaks in scalograms extracted with the Wavelet Prosody Toolkit, using the Structural Similarity Index (SSIM) between time-aligned scalograms of prompted and unprompted synthetic outputs versus the dysarthric source.

## Key Results

Fine-tuned systems outperformed zero-shot systems on both objective and perceptual measures. FT-Spont-Long achieved the highest SECS (0.727), followed by FT-Spont-Short (0.706) and FT-Read (0.643). ZS-Typical reached 0.635 despite not being fine-tuned, while ZS-Atypical scored substantially lower (0.324), reflecting the difficulty of extracting a reliable speaker embedding from 10 seconds of dysarthric speech. In the MUSHRA-like test, all three fine-tuned systems were significantly preferred over the two zero-shot systems (p < 0.001).

Within the fine-tuned group, FT-Spont-Long was significantly preferred over FT-Spont-Short (p < 0.001) and FT-Read (p = 0.003), but the difference between FT-Spont-Short and FT-Read was not significant (p = 0.213). This suggests that 12 minutes of read-aloud voice banking sentences yields perceptual quality comparable to one hour of spontaneous speech.

Crucially, ZS-Atypical and ZS-Typical showed no significant difference in MUSHRA-rated AAC suitability (p = 0.683), despite a large gap in SECS. This indicates that zero-shot TTS can serve speakers who lack typical pre-morbid recordings almost as well as those who have them, at least for perceptual AAC suitability.

For the dysarthric prompt experiment, the prompted system achieved a 29.3% higher average SSIM (0.0548 vs 0.0424), with a binomial test rejecting chance improvement (p = 0.0013). CER on Whisper-turbo ASR outputs was 2.3% for prompted synthesis versus 2.9% for unprompted, indicating that using dysarthric reference audio does not materially harm intelligibility.

A qualitative user study involving four conversations reported by the research participant confirmed a subjective sense of improved prosodic control, though the typing bottleneck for text input was identified as a significant usability barrier.

## Novelty Assessment

The primary novelty is the prosody adaptation mode that splits the audio prompt pathway in XTTS-v2: the GPT-2 encoder receives the dysarthric prompt while the vocoder receives a fixed speaker embedding. This is an engineering modification of an existing system, not a new architecture, but the resulting interactive capability has genuine clinical relevance. The five-scenario data availability framework and the MUSHRA-like evaluation with situational framing also represent a methodological contribution to AAC-focused TTS evaluation. The core comparison (zero-shot vs. fine-tuning) applies an established paradigm, and the use of XTTS-v2 as the backbone is a straightforward application of an existing model. The case study is a single speaker with a single language (Hungarian), which limits generalisability, and the paper acknowledges this directly.

## Field Significance

Moderate — This paper provides carefully structured evidence on how much speech data is needed to personalise large-scale TTS models for AAC users, a practical question with direct clinical implications. The finding that zero-shot TTS with dysarthric reference audio is perceptually comparable to zero-shot with typical speech is particularly useful for the field, as it opens TTS personalisation to users who have never had the opportunity to voice bank. The dysarthric audio prompt mechanism for prosody control demonstrates a viable path toward user-directed speech in text-based AAC, and the open-source interface enables participatory design workflows that extend beyond the specific models studied.

## Claims

- **supports:** Fine-tuning large-scale TTS models on minimal scripted read-aloud data (order of minutes) produces personalised voices with perceived AAC suitability comparable to fine-tuning on hour-scale spontaneous recordings.
  > *Evidence:* FT-Read (12 min of read-aloud sentences, SECS 0.643) and FT-Spont-Short (1h spontaneous, SECS 0.706) show no significant difference in MUSHRA-rated listener preference (p=0.213) for a Hungarian stroke survivor case study. *(§3.2, Table 1)*

- **supports:** Zero-shot TTS adaptation from dysarthric reference audio achieves perceptual suitability for assistive communication comparable to adaptation from typical speech, enabling personalisation for users who lack pre-morbid recordings.
  > *Evidence:* ZS-Atypical and ZS-Typical show no significant difference in MUSHRA-rated AAC suitability (p=0.683), despite a large objective speaker similarity gap (SECS 0.324 vs 0.635). *(§3.2)*

- **refines:** Embedding-based objective speaker similarity metrics may not reliably predict perceived suitability in assistive communication contexts, where other voice qualities matter more than timbre fidelity.
  > *Evidence:* ZS-Atypical achieves the lowest SECS (0.324) of all evaluated systems yet is rated comparably to ZS-Typical (SECS 0.635) by human listeners in the MUSHRA-like AAC suitability test (p=0.683). *(§3.1, §3.2, Table 1)*

- **supports:** Using dysarthric speech as a prosodic reference input for a fine-tuned TTS system yields measurable alignment with the speaker's intended prosodic pattern while preserving intelligibility.
  > *Evidence:* Dysarthric-prompted synthesis achieved 29.3% higher SSIM in scalogram comparison of prosodic peaks versus unprompted synthesis (0.0548 vs 0.0424, p=0.0013 by binomial test); CER increased only from 2.9% to 2.3% (actually improved) when switching to dysarthric-prompted synthesis. *(§5.2)*

## Limitations and Open Questions

> [!warning]
> The entire evaluation rests on a single research participant (one Hungarian stroke survivor). Generalisability across speakers, languages, and impairment types is untested, and the authors acknowledge this directly.

The interface requires typed text input, making it inaccessible for AAC users with motor impairments. The paper treats the interface as an evaluation platform rather than a functional device; eye-tracking compatibility is noted as future work. The prosody adaptation mechanism relies on the fact that for identical lexical content, prosodic patterns partially transfer through the audio prompt, which may not hold for diverse sentence types or severe dysarthria cases. The study also evaluates only XTTS-v2 and does not compare against other zero-shot or fine-tuning approaches that have emerged in parallel.

## Wiki Connections

- [[zero-shot-tts|Zero-Shot TTS]] — this paper systematically evaluates the zero-shot capabilities of XTTS-v2 for AAC personalisation, including the novel use of dysarthric reference audio as a zero-shot conditioning signal.
- [[speaker-adaptation|Speaker Adaptation]] — fine-tuning is the central comparison condition, and the paper provides quantitative evidence on how data type and quantity affect adaptation quality across five plausible scenarios.
- [[prosody-control|Prosody Control]] — the paper introduces an explicit mechanism to separate prosody conditioning (via dysarthric audio prompt to the GPT-2 encoder) from speaker identity conditioning (via fixed speaker embedding to the vocoder), enabling user-directed prosody adaptation.
- [[subjective-evaluation|Subjective Evaluation]] — the MUSHRA-like evaluation with situational framing (briefing listeners on the AAC context before rating) is a methodological contribution to how perceptual evaluations are conducted for assistive communication scenarios.
- [[evaluation-metrics|Evaluation Metrics]] — the paper applies SECS (Speaker Encoder Cosine Similarity via ECAPA2) alongside perceptual MUSHRA-like ratings and SSIM-based scalogram comparison for prosody assessment, demonstrating that objective and perceptual metrics can diverge substantially in this application domain.
