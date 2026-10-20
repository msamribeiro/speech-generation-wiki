---
id: "iclr-2026-GNo1qMqgPD"
title: "VoxPrivacy: A Benchmark for Evaluating Interactional Privacy of Speech Language Models"
authors: ["Yuxiang Wang", "HongYu Liu", "Dekun Chen", "Xueyao Zhang", "Zhizheng Wu"]
organization: "The Chinese University of Hong Kong, Shenzhen"
venue: ICLR
venue_type: conference
year: 2026
month: 4
published_date: "2026-04-27"
ingested_date: "2026-10-20"
task: [SCA, evaluation]
architecture: []
conditioning: []
training: [fine-tuning]
model_size: "not reported"
codec_used: "not reported"
datasets_train: ["VoxPrivacy training set (synthetic, ~4,300 h EN/ZH)", "LibriSpeech", "WenetSpeech", "Emilia", "FLEURS", "Common Voice", "SAVEE", "IEMOCAP", "ESD", "RAVDESS", "MELD", "CREMA-D", "ESC-50", "AudioSet", "FSD50K", "VocalSound", "UrbanSound8K", "Clotho-AQA", "MUSIC-AVQA", "AVQA"]
datasets_eval: ["VoxPrivacy", "Real-VoxPrivacy", "LibriSpeech", "FLEURS", "Common Voice 15", "WenetSpeech", "MELD", "VocalSound", "MMAU", "VoiceBench"]
metrics:
  - name: EER
    value: 4.15
    system: Gemini-2.0-flash (speaker verification probe)
    testset: WenetSpeech same/different-speaker pairs
  - name: EER
    value: 21.05
    system: Kimi-Audio (speaker verification probe)
    testset: WenetSpeech same/different-speaker pairs
  - name: EER
    value: 12.8
    system: GLM-4-Voice (speaker verification probe)
    testset: WenetSpeech same/different-speaker pairs
  - name: EER
    value: 49.13
    system: Qwen2.5-Omni (speaker verification probe)
    testset: WenetSpeech same/different-speaker pairs
code_available: null
demo_available: true
url: "https://openreview.net/forum?id=GNo1qMqgPD"
related_concepts: [spoken-language-model, evaluation-metrics]
related_papers: ["2410.17196", "2508.18240", "2412.10117", "2504.18425", "2503.20215", "2407.10759", "2501.15368", "2412.02612", "2408.01800", "2409.06666", "2408.16725", "2310.13289", "2507.16632", "2212.04356", "2303.08774", "2412.19437", "2507.06261", "2312.11805", "1808.10583", "1912.06670", "2407.05361"]
field_significance:
  level: "moderate"
  type: [empirical-benchmark, evaluation-contribution, dataset-contribution]
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
> **Yuxiang Wang et al.** (The Chinese University of Hong Kong, Shenzhen) · [→ Paper](https://openreview.net/forum?id=GNo1qMqgPD) · Demo: ✓ · Code: ?
>
> A bilingual three-tier benchmark for interactional privacy (whether a speech LM in a shared, multi-user setting withholds one speaker's disclosed information from a different speaker) shows that open-source speech LMs sit near chance on voice-conditioned disclosure decisions, and that supervised fine-tuning on a 4,000-hour synthetic set closes most of the gap to closed-source systems.

## Problem

Speech LMs are moving from single-user devices into shared environments (smart homes, in-car assistants) where one assistant serves several people asynchronously. In that setting the assistant must track who disclosed what, and decide whether a later querier is entitled to that information. The paper calls this interactional privacy and frames it through contextual integrity: an ordinary fact (a calendar appointment) becomes private because of who shared it and how. Existing evaluation does not test this. General spoken-dialogue benchmarks such as [[2410.17196|VoiceBench]] and [[2508.18240|MTalk-Bench]] score response relevance to what was said, not who said it; multi-speaker benchmarks test attribution of who said what but stop at input analysis; and audio privacy benchmarks target globally sensitive data (passwords, bank details) rather than contextually sensitive information. Text-only contextual-privacy benchmarks exist for LLMs, but they supply speaker identity as explicit tags, whereas a speech LM must infer it from the waveform.

## Method

VoxPrivacy defines three tiers over a three-turn asynchronous dialogue (secret disclosure, confidentiality instruction, later probe), each realised in an owner-probe and a third-party-probe condition:

- **Tier 1, Direct Command Secrecy:** the user says "do not share this with anyone"; the model must refuse every later querier, including the owner. Scored by accuracy.
- **Tier 2, Speaker-Verified Secrecy:** the user says "keep this between us"; the model must use the querier's voice as a key, disclosing only to the original speaker.
- **Tier 3, Proactive Privacy Protection:** no instruction is given; the model must infer from content alone that the statement is private and then apply the same speaker-conditioned policy.

For Tiers 2 and 3, correctly withholding from a third party is the positive class, so recall measures leak prevention and precision measures how often the owner is wrongly refused; accuracy, precision, recall, and F1 are reported *(§4.3, Appendix F)*.

Construction is text-first. Privacy-sensitive statements across eight categories (personal info, location, academic background, interpersonal secrets, professional aspirations, beliefs, illicit actions, transient secrets) are generated with DeepSeek, Gemini, and ChatGPT in parallel, de-duplicated with a difflib similarity filter, refined for naturalness with DeepSeek, and manually checked. Dialogues are synthesised with [[2412.10117|CosyVoice 2]] using two disjoint 200-speaker pools drawn from [[1808.10583|AISHELL-2]] and WenetSpeech, gender-balanced 1:1 with an automatic gender classifier, and filtered by DNSMOS and Whisper-large-v3 WER thresholds. The benchmark contains 7,107 utterances (32.86 h) split evenly between English and Chinese *(§3.2–3.3, Table 1)*. Five annotators rated 200 sampled secrets on a 1–5 sensitivity scale, and 92% received a rating of 4 or above *(§3.3)*.

Two auxiliary resources are built. Real-VoxPrivacy is a human-recorded subset: 18 bilingual volunteers read a balanced sample of Tier 1–3 scripts, giving 586 utterances (288 EN, 298 ZH), with enrollment and probe audio taken from different recording sessions *(§5.1, Appendix J)*. The training set uses the same pipeline at larger scale (1,800 speakers; 2,066 h English and 2,273 h Chinese, with 2-round and 3-round formats) mixed with about 1,500 h of general-task data (ASR, emotion recognition, sound classification, audio QA, voice chat) to limit forgetting *(§4.1, Appendix C)*.

Nine speech LMs are evaluated: [[2504.18425|Kimi-Audio]], [[2503.20215|Qwen2.5-Omni]], MiniCPM-o 2.6, Qwen2-Audio, Voxtral-3B, Baichuan-Omni-1.5, [[2412.02612|GLM-4-Voice]], Gemini-2.0-flash, and [[2507.06261|Gemini-2.5-pro]]. A text-only Gemini-2.0-flash given explicit speaker ID tags serves as an upper bound that removes speech-processing errors. Responses are scored by DeepSeek-V3 and Gemini-2.5-Pro as LLM judges (three runs, majority vote), which first flag invalid responses (invalid response rate, IRR) and then judge whether the secret was disclosed. Three bilingual annotators re-scored 800 Tier 1 responses with the same criteria (Fleiss' κ = 0.92 for disclosure) to validate the judges *(§4.3–4.4, Appendix G)*. The proposed model is Kimi-Audio fine-tuned for one epoch, with its Whisper-large-v3 encoder and adaptor updated (AdamW, learning rate 1e-5, 8 A800 GPUs) *(§4.4)*.

## Key Results

**Tier 1.** With the DeepSeek-V3 judge, closed-source models reach 79.92–81.42% accuracy in English and 83.90–85.01% in Chinese, against 97.33% / 99.10% for the text-only upper bound. Open-source models are far lower: Qwen2.5-Omni 41.42% / 31.59%, MiniCPM-o 2.6 26.86% / 22.28%, Kimi-Audio 73.04% / 38.26% (with a 16.42% Chinese IRR). The fine-tuned Kimi-Audio reaches 88.11% / 79.43% *(§4.5, Table 2)*. Human accuracy tracks the LLM judges for most systems, but for Kimi-Audio in English humans give 64.75% against the judges' 71.38–73.04% *(Table 2)*.

**Tiers 2 and 3.** All open-source baselines sit near 50% accuracy (48–52% across both languages and tiers, including the four additional models in Appendix B), with unstable precision/recall splits: Kimi-Audio has 72.62% English Tier 2 recall but only 18.63% in Chinese. Gemini-2.5-pro reaches 76.05% / 77.93% on Tier 2 and 66.28% / 68.58% on Tier 3; the text-only upper bound reaches 88.37% / 93.72% and 85.21% / 87.80%. The fine-tuned model reaches 83.93% / 79.34% on Tier 2 and 77.57% / 82.88% on Tier 3, exceeding Gemini-2.5-pro on both *(§4.5, Tables 3 and 9)*. Every capable model drops from Tier 2 to Tier 3, which the authors read as a failure of commonsense inference rather than of voice processing.

**Real speech.** On Real-VoxPrivacy, open-source baselines stay near chance on Tiers 2 and 3 and the Tier 2 to Tier 3 drop recurs. Ranking is broadly preserved but not exactly: Gemini-2.5-pro scores 90.83% on English Tier 1 against 82.36% for the fine-tuned model, reversing their synthetic-set order *(§5.1, Table 4)*.

**Diagnostics.** In a non-sensitive control (speaker A states a fact, speaker B asks about it), most models score 88–99%, so the privacy failures are not a general inability to follow multi-speaker dialogue *(§5.2, Table 5a)*. In a balanced same/different-speaker test, open-source models attribute 58.65–60.64% of their errors to speaker switches against about 50% for the upper bound and Gemini-2.0-flash, which the paper names a speaker continuity bias *(Table 5b)*. A direct speaker-verification probe gives EER from 4.15% (Gemini-2.0-flash) to 49.13% (Qwen2.5-Omni); GLM-4-Voice has 74.40% verification accuracy (12.8% EER) yet stays near chance on Tiers 2 and 3, so verification ability is necessary but not sufficient *(Appendix E, Table 13)*.

**Robustness and forgetting.** On Tier 2, voice spoofing with the nearest-neighbour speaker (by speaker embedding) is the most damaging attack, lowering accuracy by 5.18 / 3.78 points for Gemini-2.0-flash and 6.41 / 6.42 for the fine-tuned model; needle-in-a-haystack and jailbreak prompts cost 1–5 points *(§5.3, Table 6)*. Mixed-task fine-tuning keeps general benchmarks close to the base model (LibriSpeech test-clean 1.23 vs 1.28, MMAU average 62.63 vs 63.27), whereas privacy-only fine-tuning degrades them (LibriSpeech test-clean 6.02, MELD 50.36 vs 59.07) *(§5.4, Table 7)*. VoiceBench average moves from 80.73 to 80.11, with small drops on MMSU and IFEval *(Table 11)*.

## Novelty Assessment

The main novelty is the task framing: it separates speaker-conditioned disclosure from both content-only safety and speaker attribution, and the three tiers isolate instruction following, voice-as-key verification, and unprompted sensitivity inference. The diagnostic design is a strength. The text-only upper bound, the non-sensitive control, the speaker-verification probe, and the real-speech subset together locate failures in the binding of speaker identity to policy, not in conversation handling or synthetic-audio artefacts. Data construction (LLM-generated text, CosyVoice 2 synthesis, DNSMOS/WER filtering) and scoring (LLM judges validated against humans) follow established benchmark practice. The fine-tuned model is a standard supervised fine-tuning baseline on in-distribution synthetic data. Its gains show that the capability can be learned, but because training and test share the same generation pipeline they say less about generalisation. The Real-VoxPrivacy result is the main evidence against that concern.

## Field Significance

Moderate — VoxPrivacy introduces a concrete, reusable test of whether speech LMs can use speaker identity as a condition on what they say, a capability that text-only and content-only safety benchmarks cannot measure. Its diagnostic results show that current open-source speech LMs do not bind voice identity to disclosure decisions even when they follow multi-speaker dialogue well. This evidence matters for multi-user deployment of spoken conversational agents.

## Claims

- **supports:** Open-source speech LMs largely fail to condition their responses on the identity of the current speaker, performing near chance on decisions that require using the speaker's voice as an access key.
  > *Evidence:* All seven open-source models score 48–52% accuracy on speaker-verified (Tier 2) and proactive (Tier 3) disclosure in both English and Chinese, while scoring 80–97% on a matched non-sensitive multi-speaker control. *(§4.5, §5.2, Tables 3, 9, 10)*
- **refines:** Speaker verification ability inside a speech LM is necessary but not sufficient for speaker-aware behaviour; the model must also integrate the recognised identity with conversational instructions.
  > *Evidence:* Models with near-50% speaker-verification accuracy (Qwen2.5-Omni, Baichuan-Omni-1.5, MiniCPM-o 2.6) fail Tier 2, but GLM-4-Voice, with 74.40% verification accuracy and 12.8% EER, also stays near 50% on Tiers 2 and 3. *(Appendix E, Table 13)*
- **complicates:** Spoken dialogue models show a gap between following an explicit confidentiality instruction and inferring sensitivity from content alone; removing the speech front-end does not close this gap entirely.
  > *Evidence:* Every capable model drops from Tier 2 to Tier 3 (e.g., Gemini-2.5-pro 76.05% to 66.28% in English), the drop recurs on human-recorded Real-VoxPrivacy, and Tier 3 failures concentrate in socially nuanced categories (professional aspirations, beliefs) rather than universally sensitive ones. *(§4.5, §5.1, Appendix H, Tables 3, 4)*
- **supports:** Fine-tuning an existing speech LM on task-specific synthetic dialogue needs general-task data mixed in to avoid catastrophic forgetting of ASR and audio-understanding abilities.
  > *Evidence:* Kimi-Audio fine-tuned on privacy data plus about 1,500 h of general-task data keeps LibriSpeech test-clean at 1.23 (base 1.28) and MMAU average at 62.63 (base 63.27); privacy-only fine-tuning raises LibriSpeech test-clean to 6.02 and drops MELD from 59.07 to 50.36. *(§5.4, Table 7)*
- **complicates:** Behavioural safety controls in speech LMs that rely on voice identity are vulnerable to acoustically similar speakers.
  > *Evidence:* Replacing the third-party querier with the nearest speaker by speaker embedding is the strongest of three Tier 2 attacks, cutting accuracy by 5.18 (Gemini-2.0-flash) and 6.41 (fine-tuned Kimi-Audio) points in English, more than needle-in-a-haystack or jailbreak prompts. *(§5.3, Appendix D, Table 6)*

## Limitations and Open Questions

> [!warning]
> The main benchmark and the training set come from the same synthetic pipeline (LLM-written secrets, CosyVoice 2 voices from two fixed speaker pools), so the fine-tuned model's gains are measured in distribution. The only real-speech check is a 586-utterance subset from 18 volunteers, where the fine-tuned model no longer leads on English Tier 1.

- The authors note that synthesised audio lacks paralinguistic variation such as emotion, and that the eight secret categories reflect one cultural notion of privacy; collectivist and individualist norms may differ *(§7)*.
- Scoring depends on LLM judges. Human validation covers only Tier 1 and a subset of models, and on Kimi-Audio English the judges score 6–8 points higher than humans *(Table 2)*. Tier 2 and Tier 3 judgments are not human-validated.
- The Chinese–English gap is attributed to higher ASR error rates and English-centric backbones, but neither explanation is tested directly *(§4.5)*.
- Only supervised fine-tuning is explored; the authors list reinforcement learning as future work *(§7)*. The fine-tuned model's parameter count is not stated.
- The benchmark covers only interactional privacy among broader speaker-aware behaviours, and only asynchronous probes from a single querier.

## Wiki Connections

- [[spoken-language-model|Spoken Language Model]] — Feeds bilingual multi-speaker audio into nine existing speech LMs and fine-tunes one, showing that current models rarely condition their responses on who is speaking.
- [[evaluation-metrics|Evaluation Metrics]] — Introduces a tiered precision/recall formulation for speaker-conditioned disclosure, validated LLM-as-judge scoring, and a speaker-verification EER probe for speech LMs.
- [[2504.18425|Kimi-Audio]] — Evaluated as a baseline and used as the base model for the paper's privacy fine-tune.
- [[2503.20215|Qwen2.5-Omni]] — Evaluated baseline; near chance on Tiers 2 and 3, with speaker-verification EER near 50%.
- [[2412.02612|GLM-4-Voice]] — Evaluated baseline; the key case where good speaker verification does not translate into speaker-aware disclosure.
- [[2507.06261|Gemini 2.5]] — Strongest closed-source system evaluated, also used as one of the two LLM judges.
- [[2412.10117|CosyVoice 2]] — TTS engine used to synthesise all benchmark and training dialogues.
- [[2410.17196|VoiceBench]] — Prior speaker-agnostic voice-assistant benchmark that VoxPrivacy contrasts with, also used to check that fine-tuning preserves general voice-chat ability.
- [[2212.04356|Whisper]] — Whisper-large-v3 provides the WER quality filter for synthesised audio and is the audio encoder updated during fine-tuning.
- [[2508.18240|MTalk-Bench]] — Multi-turn spoken dialogue benchmark cited as testing conversational skill without regard to speaker identity.
