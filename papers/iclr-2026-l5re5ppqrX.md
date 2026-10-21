---
id: "iclr-2026-l5re5ppqrX"
title: "EchoMind: An Interrelated Multi-level Benchmark for Evaluating Empathetic Speech Language Models"
authors: ["Li Zhou", "Lutong Yu", "You Lyu", "Yihang Lin", "Zefeng Zhao", "Junyi Ao", "Yuhao Zhang", "Benyou Wang", "Haizhou Li"]
organization: "The Chinese University of Hong Kong, Shenzhen"
venue: ICLR
venue_type: conference
year: 2026
month: 4
published_date: "2026-04-27"
ingested_date: "2026-10-21"
task: [SCA, evaluation]
architecture: []
conditioning: []
training: []
model_size: "not applicable (benchmark)"
codec_used: "not applicable (benchmark)"
datasets_train: ["not applicable (benchmark)"]
datasets_eval: ["EchoMind (TTS-generated, 3,356 audio inputs)", "EchoMind-Human (1,453 human-recorded audio inputs)"]
metrics:
  - name: WER
    value: 2.93
    system: Audio-Flamingo3
    testset: EchoMind (content understanding, target expression)
  - name: WER
    value: 3.97
    system: Qwen2.5-Omni-7B
    testset: EchoMind (content understanding, target expression)
  - name: WER
    value: 10.74
    system: GPT-4o-Audio
    testset: EchoMind (content understanding, target expression)
code_available: null
demo_available: null
url: "https://openreview.net/forum?id=l5re5ppqrX"
related_concepts: [spoken-language-model, speech-to-speech, evaluation-metrics, subjective-evaluation]
related_papers: ["2410.21276", "2503.20215", "2502.11946", "2412.02612", "2025.emnlp-demos.70", "2509.09174", "2504.18425", "2501.15368", "2505.03739", "2025.acl-long.912", "2507.08128", "2312.15185", "2204.02152", "2507.06261", "2025.findings-emnlp.933", "2508.18240", "2503.05085", "2410.17196", "2508.09600", "2402.07729", "2410.19168", "2025.findings-acl.115", "2025.acl-long.682", "2411.13577"]
field_significance:
  level: "moderate"
  type: [evaluation-contribution, dataset-contribution, empirical-benchmark]
generation:
  schema_version: 2
  date: "2026-10-21"
  runtime: "claude-code"
  provider: "anthropic"
  agent: speech-generation-ingest-agent
  model: "claude-opus-5-5"
  commit: "afa19d3"
---

> [!abstract] ICLR · 2026 · Conference
> **Li Zhou et al.** (The Chinese University of Hong Kong, Shenzhen) · [→ Paper](https://openreview.net/forum?id=l5re5ppqrX) · Demo: ? · Code: ?
>
> EchoMind builds one set of semantically neutral dialogue scripts, renders each in contrasting vocal styles, and runs speech LMs through linked understanding, reasoning, and spoken-response tasks on those same inputs, so that any change in a model's reply can be attributed to how the user spoke rather than to what was said.

## Problem

Benchmarks for speech LMs usually test one capability at a time: transcription and acoustic recognition, multi-hop audio reasoning, or dialogue. Each is typically built from repurposed corpora or narrow task-specific data, so the tasks share no common context. That makes it impossible to ask whether a model's failure to respond empathetically comes from failing to perceive a vocal cue (a sob, a whisper, a cough, background rain) or from failing to use a cue it did perceive. Most dialogue benchmarks also use a single expressive rendering per script (Table 1), so they cannot isolate the effect of delivery from the effect of lexical content.

## Method

EchoMind is organised around an empathy-oriented taxonomy of vocal cues: 3 coarse dimensions (speaker information, paralinguistic information, environmental information), 12 fine-grained categories (gender, age, physiological state, emotion, volume, speech rate, non-verbal expression, weather, location, background human sounds, sudden events, others), and 39 specific attributes (Table 2).

**Scripts.** GPT-4o generates one-turn dialogues over 17 topics, with user utterances deliberately written to carry no explicit emotional or contextual cue. For each utterance it generates a high-EQ response conditioned on a target vocal cue, a cue-agnostic response, and an empathetic response for an alternative cue. Three authors review every utterance and keep only unanimously approved ones, leaving 1,137 scripts; each response type is then expanded to five references (§3.1, Appendix B.1).

**Audio.** Each user utterance is rendered in three vocal styles: target, neutral, and alternative. Synthesis is tailored to the cue: Doubao TTS for speaker attributes; for paralinguistics, a mix of prompting the Doubao conversational agent (coughs), selecting a fatigued-sounding agent voice (vocal fatigue), Doubao voice cloning from naturally hoarse speakers (hoarseness), and attribute-specific prompts to GPT-4o-mini-TTS for the remaining cues. The authors state that paralinguistic clips were generated one at a time with immediate quality checks, sometimes requiring dozens of attempts. Environmental conditions mix clean Doubao TTS with AudioCaps background sounds. Reference response audio is synthesised by GPT-4o-mini-TTS from a GPT-4o-written voice profile (affect, tone, emotion, personality) for each utterance-voice pair. The full set contains 3,356 audio inputs (4.51 hours), 1,137 of them target-expression inputs (Tables 8 and 9). A parallel EchoMind-Human subset re-records 491 scripts with two trained voice actors (one male, one female), giving 1,453 inputs (§3.1, Appendix A.3).

**Tasks.** Level 1 (understanding) covers ASR on expressive and noisy speech plus voice-understanding multiple-choice questions (one coarse-grained and seven fine-grained sub-tasks). Level 2 (reasoning) has ten multiple-choice sub-tasks that require combining content and vocal cues, such as preceding event inference, speaker intent recognition, and empathy-aware response selection. Level 3 (conversation) asks for an open-ended spoken or text reply. All three levels use the same audio, so per-model performance can be correlated across levels (Table 3, §3.2).

**Metrics.** ASR uses WER and semantic similarity (SemSim, cosine similarity of Qwen3-Embedding-0.6B embeddings). Multiple-choice tasks use accuracy. Text responses get reference-based BLEU, ROUGE-L, METEOR, and BERTScore, plus four GPT-4o-judged 5-point ratings: context fit (C-CtxFit), response naturalness (C-RespNat), colloquialism degree (C-ColloqDeg), and speech information relevance (C-SpeechRel, the only text dimension that depends on vocal cues). Response audio is scored for quality with NISQA and an automatic MOS predictor, and for emotional alignment with two new measures. EmoAlign compares emotion2vec's prediction on the response audio with a GPT-4o-predicted reference emotion. The Vocal Empathy Score (VES) is a 5-point Gemini-2.5-Pro rating of whether the reply's vocal style mirrors the user's state (§3.3, Appendix B.2). Conversation prompts are run at three strengths: no system prompt (P-Zero), a basic prompt (P-Basic), and an enhanced prompt that explicitly asks the model to consider vocal cues (P-Enhance) (§4.1, Appendix B.3).

## Key Results

Twelve speech LMs are evaluated: GPT-4o-Audio, three Audio Flamingo 3 variants, DeSTA2.5-Audio, VITA-Audio, LLaMA-Omni2, Baichuan-Omni-1.5, GLM-4-Voice, OpenS2S, Qwen2.5-Omni-7B, Kimi-Audio, Step-Audio, and EchoX. Eight of them produce speech output (§4.1, Table 4).

- **Content understanding is near-solved, voice understanding is not.** SemSim exceeds 96.7 for every model that reports it, and WER ranges from 2.93 (Audio-Flamingo3) to 10.92 (EchoX). Voice-understanding accuracy ranges from 25.24% (VITA-Audio) to 66.25% (GPT-4o-Audio). Among open models, only Audio-Flamingo3, its Think variant, and Qwen2.5-Omni-7B exceed 60%. On integrated reasoning, GPT-4o-Audio reaches 68.04% and DeSTA2.5-Audio (63.04%) is the only open model above 60% (Table 4).
- **The cue-dependent response dimension is the weakest.** No model averages above 4 on C-SpeechRel (best: GPT-4o-Audio, 3.42). By contrast, six models exceed 4 on C-CtxFit, nine on C-RespNat, and eight on C-ColloqDeg (§4.2, Table 4).
- **Generated speech is clean but not empathically matched.** NISQA is 3.94 to 4.99 across speech-output models, but VES tops out at 3.34 (GPT-4o-Audio) and is as low as 1.40 for EchoX. EmoAlign ranges from 23.60 (Kimi-Audio) to 51.31 (GPT-4o-Audio) (Table 4).
- **Abilities correlate across levels, with outliers.** Voice-understanding accuracy broadly predicts reasoning accuracy and response quality. GLM-4-Voice and VITA-Audio give relatively good responses despite weak multiple-choice scores. Both do best with no system prompt, which the authors attribute to weak instruction following (§4.2, Figure 2, Figure 3).
- **Prompting matters, inconsistently.** Seven of the twelve models get their best C-SpeechRel under P-Enhance, while others are best with no prompt. Step-Audio shows the largest swing across prompt settings (§4.3 RQ1, Table 17).
- **Human recordings are harder than TTS inputs.** For the three strongest models, performance drops on EchoMind-Human relative to the TTS version of the same scripts at all three levels, most sharply in conversation (§4.3 RQ2, Figure 4). The gap is largest for non-verbal expressions (average -0.35) and smallest for volume (-0.06) (Appendix B.4, Table 18).
- **Ceiling with oracle cues.** When models also receive the vocal-cue label in text, C-SpeechRel rises by 0.76 to 1.10 and VES by 0.51 to 1.08 for Qwen2.5-Omni-7B, Step-Audio, and GPT-4o-Audio (§4.3 RQ3, Table 7). This locates much of the deficit in perception rather than in response generation.
- **Human evaluation.** Three raters scored six cases per vocal-cue type for three models (Table 5). Human C-SpeechRel scores are higher than GPT-4o's for all three models (e.g., 4.17 vs. 3.25 for Step-Audio). Human VES for GPT-4o-Audio is 2.49 against Gemini's 3.34; raters cited long, formal replies and a formal timbre. The human-only Response Difference score, which measures whether replies change when the same script is spoken differently, ranges from 3.10 to 3.50. Pairwise arena win rates are 0.42 (GPT-4o-Audio), 0.34 (Step-Audio), and 0.28 (Qwen2.5-Omni-7B) (Table 6). Gemini-2.5-Pro and GPT-4o judges correlate at 0.90, 0.85, 0.81, and 0.64 on the four text dimensions, lowest for C-SpeechRel (§4.2, Appendix C.1).

Comparisons are fair in that all models see identical inputs. However, Table 4 reports each model at its best-performing prompt (Table 17), so the response columns mix prompt conditions across models.

## Novelty Assessment

The contribution is evaluation design, not modelling. Two elements are new for speech LM benchmarking: content held constant across multiple vocal renderings, and understanding, reasoning, and conversation tasks linked on the same audio. Table 1 shows that SD-Eval offers multiple styles per script but no task linkage, while URO-Bench offers all task types but no interrelation. The 39-attribute taxonomy, which includes environmental and physiological cues, is broader than emotion-only empathy benchmarks. EmoAlign and VES are straightforward compositions of existing tools (emotion2vec, an LLM judge) rather than validated new metrics, and the response-generation evaluation leans heavily on model judges. The main input set is synthetic, and the hardest paralinguistic cues were produced through hand-curated commercial TTS and agent tricks, which limits regeneration or extension by others. The human-recorded subset is a useful control but comes from only two speakers.

## Field Significance

Moderate. EchoMind provides a controlled testbed for whether speech LMs use non-lexical vocal cues when replying. It contributes a concrete diagnosis: current systems transcribe well, perceive vocal cues unreliably, and rarely adapt their spoken replies to the user's state, and supplying the cue as text recovers much of the gap. Its synthetic-versus-human comparison is also a useful data point on how far TTS-built benchmarks overstate robustness to natural speech.

## Claims

- **supports:** Speech LMs that transcribe expressive speech accurately can still fail to perceive the paralinguistic and environmental cues carried by the same audio.
  > *Evidence:* Across 12 models, SemSim on ASR exceeds 96.7 wherever reported, while voice-understanding multiple-choice accuracy ranges from 25.24% to 66.25%. Background sound detection and voice-style (physiological state) detection are the hardest sub-tasks. *(§4.2, Table 4; Appendix C.2)*
- **supports:** In current speech LMs, the empathy gap in spoken responses comes more from failing to perceive vocal cues than from failing to generate an appropriate reply once the cue is known.
  > *Evidence:* Giving the vocal-cue label alongside the audio raises C-SpeechRel by 0.76 to 1.10 and VES by 0.51 to 1.08 for Qwen2.5-Omni-7B, Step-Audio, and GPT-4o-Audio. GPT-4o-Audio reaches 4.42 VES, against 3.34 without the label. *(§4.3 RQ3, Table 7)*
- **complicates:** Speech-output quality metrics do not capture whether a spoken reply's vocal delivery is emotionally matched to the interlocutor.
  > *Evidence:* NISQA for speech-output models clusters between 3.94 and 4.99, while the Vocal Empathy Score ranges from 1.40 to 3.34 and EmoAlign from 23.60 to 51.31 on the same responses. *(§4.2, Table 4)*
- **complicates:** Benchmarks built entirely from synthesised speech can overstate how well speech LMs handle vocal cues in natural human speech.
  > *Evidence:* The three strongest models score lower on the human-recorded EchoMind-Human inputs than on TTS renderings of the same scripts at all three task levels, most sharply in conversation. The largest gaps are for non-verbal expressions (average -0.35) and physiological state (-0.18). *(§4.3 RQ2, Figure 4; Appendix B.4, Table 18)*
- **complicates:** LLM-judge ratings of vocal-cue relevance and vocal empathy agree less with human listeners than LLM-judge ratings of content fit or fluency do.
  > *Evidence:* GPT-4o and Gemini-2.5-Pro judges correlate at 0.64 on C-SpeechRel, compared with 0.81 to 0.90 on the other text dimensions. Human-versus-model agreement is lowest on C-SpeechRel and VES, and human VES for GPT-4o-Audio is 2.49 against a Gemini score of 3.34. *(§4.2, Table 5; Appendix B.4; Appendix C.1)*

## Limitations and Open Questions

> [!warning]
> The response-level empathy scores (C-SpeechRel, VES, EmoAlign references) all depend on proprietary LLM judges or GPT-4o-generated labels. Human validation covers only three models on a small sample, and agreement with the judges is weakest on exactly these vocal-cue dimensions.

- The main benchmark audio is synthesised with commercial systems (Doubao TTS and app agents, GPT-4o-mini-TTS), and many paralinguistic clips were curated by hand. Some attributes (coughs, fatigue, hoarseness) depend on vendor-specific workarounds that may not be reproducible.
- EchoMind-Human uses two non-native speakers, which the authors justify as typical usage. It is not a sample of natural, spontaneous emotional speech.
- Dialogues are single-turn. Empathy in extended multi-turn conversation is not tested.
- Headline response results are reported at each model's best prompt setting, so cross-model rankings mix prompt conditions.
- Release of data and code is promised in the reproducibility statement, but the paper does not include a repository link beyond the project website.

## Wiki Connections

- [[spoken-language-model|Spoken Language Model]] — Feeds real and synthetic speech into 12 existing speech LMs and measures how well they perceive, reason about, and respond to non-lexical vocal cues.
- [[speech-to-speech|Speech-to-Speech]] — Evaluates the spoken replies of end-to-end speech-to-speech dialogue systems for audio quality and vocal empathy.
- [[evaluation-metrics|Evaluation Metrics]] — Introduces EmoAlign and the Vocal Empathy Score, which measure whether a reply's generated speech carries an emotion aligned with the user's state (current systems show large deficits), and reports how LLM-judge metrics agree with each other and with humans.
- [[subjective-evaluation|Subjective Evaluation]] — Includes a three-rater human study of text and audio responses, plus arena-style pairwise rankings, to validate the model-judge protocol.
- [[2410.21276|GPT-4o]] — GPT-4o-Audio is the strongest evaluated system on most levels and also serves as the script generator and text judge.
- [[2503.20215|Qwen2.5-Omni]] — Evaluated system and one of three models used in the human-evaluation, human-versus-TTS, and oracle-cue analyses.
- [[2502.11946|Step-Audio]] — Evaluated system with the largest sensitivity to prompt setting and the largest oracle-cue gain on C-SpeechRel.
- [[2412.02612|GLM-4-Voice]] — Evaluated system that gives relatively good responses despite low multiple-choice accuracy, which the authors attribute to weak instruction following.
- [[2025.emnlp-demos.70|OpenS2S]] — Evaluated empathetic speech LM that does not outperform general-purpose models on vocal empathy.
- [[2312.15185|emotion2vec]] — Used as the speech emotion recogniser underlying the EmoAlign metric.
- [[2025.findings-emnlp.933|URO-Bench]] — The closest prior benchmark covering understanding, reasoning, and conversation, but without interrelated tasks or multiple vocal styles per script.
- [[2508.18240|MTalk-Bench]] — Related speech-to-speech dialogue benchmark from overlapping authors; S2S-Arena, a paralinguistic instruction-following benchmark also from overlapping authors, is likewise cited.
