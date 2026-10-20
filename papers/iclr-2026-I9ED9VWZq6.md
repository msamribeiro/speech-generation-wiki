---
id: "iclr-2026-I9ED9VWZq6"
title: "SpeechJudge: Towards Human-Level Judgment for Speech Naturalness"
authors: ["Xueyao Zhang", "Chaoren Wang", "Huan Liao", "Ziniu Li", "Yuancheng Wang", "Li Wang", "Dongya Jia", "Yuanzhe Chen", "Xiulin Li", "Zhuo Chen", "Zhizheng Wu"]
organization: "CUHK Shenzhen / ByteDance Seed"
venue: ICLR
venue_type: conference
year: 2026
month: 4
published_date: "2026-04-27"
ingested_date: "2026-10-20"
task: [evaluation, TTS]
architecture: [autoregressive-LM]
conditioning: [text-conditioned, zero-shot, prompt-conditioned]
training: [fine-tuning, RLHF]
model_size: "7B params (SpeechJudge-GRM on Qwen2.5-Omni-7B Thinker, LoRA rank 128 SFT / 64 RL); 0.5B base TTS for post-training experiments"
codec_used: "DualCodec speech tokenizer (Qwen2.5-0.5B-TTS post-training base only; frame rate not reported); none for the reward model"
datasets_train: ["SpeechJudge-Data (train, 42K preference pairs)", "Emilia (TTS base pre-training)", "INTP (DPO post-training pairs and prompts)"]
datasets_eval: ["SpeechJudge-Eval (1,000 pairs)", "Human vs. SeedTTS clone OOD set (500 pairs)", "SeedTTS-Eval", "Amphion-TTS-Eval"]
metrics:
  - name: Accuracy
    value: 77.2
    system: SpeechJudge-GRM (SFT+RL)
    testset: SpeechJudge-Eval (Table 3)
  - name: Accuracy
    value: 79.4
    system: SpeechJudge-GRM (SFT+RL) w/ Voting@10
    testset: SpeechJudge-Eval (Table 3)
  - name: Accuracy
    value: 75.3
    system: SpeechJudge-GRM (SFT)
    testset: SpeechJudge-Eval (Table 3)
  - name: Accuracy
    value: 72.7
    system: SpeechJudge-BTRM (baseline)
    testset: SpeechJudge-Eval (Table 3)
  - name: Accuracy
    value: 69.1
    system: Gemini-2.5-Flash (best zero-shot AudioLLM)
    testset: SpeechJudge-Eval (Table 2)
  - name: Accuracy
    value: 53.7
    system: UTMOS
    testset: SpeechJudge-Eval (Table 2)
  - name: Accuracy
    value: 62.4
    system: SpeechJudge-GRM (SFT+RL)
    testset: Human vs. SeedTTS clone OOD set (Table 11)
  - name: Accuracy
    value: 50.4
    system: SpeechJudge-BTRM (baseline)
    testset: Human vs. SeedTTS clone OOD set (Table 11)
  - name: CMOS
    value: 0.25
    system: Qwen2.5-0.5B-TTS w/ SpeechJudge-GRM (online DPO), naturalness CMOS vs. base
    testset: SeedTTS-Eval + Amphion-TTS-Eval (Figure 6a)
  - name: CMOS
    value: 0.18
    system: Qwen2.5-0.5B-TTS w/ INTP (baseline), naturalness CMOS vs. base
    testset: SeedTTS-Eval + Amphion-TTS-Eval (Figure 6a)
  - name: WER
    value: 11.73
    system: Qwen2.5-0.5B-TTS (base, average)
    testset: SeedTTS-Eval + Amphion-TTS-Eval (Table 14)
  - name: WER
    value: 8.85
    system: Qwen2.5-0.5B-TTS w/ SpeechJudge-GRM (online, average)
    testset: SeedTTS-Eval + Amphion-TTS-Eval (Table 14)
  - name: WER
    value: 8.51
    system: Qwen2.5-0.5B-TTS w/ SpeechJudge-GRM (offline, average)
    testset: SeedTTS-Eval + Amphion-TTS-Eval (Table 14)
  - name: SPK-SIM
    value: 0.661
    system: Qwen2.5-0.5B-TTS w/ SpeechJudge-GRM (online, average)
    testset: SeedTTS-Eval + Amphion-TTS-Eval (Table 14)
  - name: SPK-SIM
    value: 0.668
    system: Qwen2.5-0.5B-TTS (base, average)
    testset: SeedTTS-Eval + Amphion-TTS-Eval (Table 14)
code_available: true
demo_available: true
url: "https://openreview.net/forum?id=I9ED9VWZq6"
related_concepts: [evaluation-metrics, subjective-evaluation, rlhf-speech, spoken-language-model, zero-shot-tts]
related_papers: ["2025.acl-long.598", "2503.20215", "2507.06261", "2025.acl-long.313", "2412.10117", "2409.00750", "2406.02430", "2204.02152", "2407.05407", "2407.05361", "2403.03100", "interspeech-2025-0468", "2508.16332", "2504.18425", "2512.23808", "2503.01743", "2503.19786", "2410.21276", "2412.15115", "2502.05139", "2212.04356", "2206.08317", "2507.12705", "2510.14664", "2508.20916", "2505.09558", "2412.19437", "2501.12948", "2402.03300"]
field_significance:
  level: "high"
  type: [dataset-contribution, evaluation-contribution, empirical-benchmark]
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
> **Xueyao Zhang et al.** (CUHK Shenzhen / ByteDance Seed) · [→ Paper](https://openreview.net/forum?id=I9ED9VWZq6) · Demo: ✓ · Code: ✓
>
> Releases a 99K-pair human preference corpus for TTS naturalness, a 1,000-pair judgment benchmark on which no existing metric or AudioLLM exceeds 70% agreement with humans, and a generative reward model trained with chain-of-thought SFT plus RL that reaches 77.2% and can drive preference optimization of a zero-shot TTS model.

## Problem

Naturalness is the most general-purpose subjective criterion for speech synthesis, yet the field has no large-scale pairwise human preference corpus centred on it. Existing human-rated speech corpora are MOS datasets with pointwise labels, built mostly from older TTS systems and at limited scale, while recent preference corpora target narrower attributes such as low-level acoustic quality or intelligibility ([[2025.acl-long.598|INTP]]). Papers therefore run their own, inconsistent listening tests or fall back on proxies such as [[2204.02152|UTMOS]], and it is unclear whether these proxies can rank outputs of modern zero-shot TTS systems the way listeners do. Without such data, reward models for aligning speech generators with human naturalness preferences cannot be trained or validated, and AudioLLM-as-judge approaches ([[2507.12705|AudioJudge]]) remain largely prompt-engineering exercises.

## Method

The work has three parts.

**SpeechJudge-Data.** Speech pairs are synthesized by six open zero-shot TTS models spanning three families: autoregressive (ARS, [[2412.10117|CosyVoice 2]], and the intelligibility-enhanced CosyVoice2-INTP and Ints-INTP), flow matching ([[2025.acl-long.313|F5-TTS]]), and masked generative modelling ([[2409.00750|MaskGCT]]). Reference prompts are either regular (Emilia-Large) or expressive (emotional ParaSpeechCaps, accented L2-ARCTIC and KeSpeech, in-house whispers, and Genshin Impact game-character voices). Target texts in Chinese, English, and Chinese-English code-switching are cleaned or generated with DeepSeek-V3, covering monolingual and cross-lingual (zh2en, en2zh, zh2mixed, en2mixed) settings, and both intra-model and inter-model pairs are formed. 69 professional annotators from a Chinese data-labelling firm gave each clip a binary text-accuracy (intelligibility) label and each pair a five-point naturalness CMOS; each pair received two annotations, with a third on disagreement (2.49 on average), yielding 99K annotated pairs over two months. Majority voting and removing ties gives 79K preference pairs; removing pairs whose absolute WER gap exceeds 12% (to stop intelligibility differences dominating naturalness judgments) gives a 44K high-quality subset.

**SpeechJudge-Eval.** Naturalness judgment is posed as binary pairwise classification (which of two clips for the same text is more natural), scored by accuracy against human labels. The 1,000 evaluation pairs are stratified samples of full-agreement (all annotators concur) pairs from the high-quality subset, split 400 regular and 600 expressive across languages. A same-sized dev set is held out, leaving 42K pairs for training. The benchmark compares objective metrics (WER, SIM, FAD), MOS predictors (DNSMOS, UTMOS, and the four [[2502.05139|Audiobox Aesthetics]] axes), deepfake detectors (AASIST, ADV), and nine open and closed AudioLLMs prompted zero-shot.

**SpeechJudge-GRM.** The reward model is Qwen2.5-Omni-7B (Thinker) fine-tuned with LoRA in two stages. Applying RL with verifiable reward directly failed because the base model follows reasoning instructions poorly (its accuracy drops from 60.6% to 52.9% when given a chain-of-thought prompt). In the SFT "cold start", [[2507.06261|Gemini-2.5-Flash]] generates chain-of-thought rationales scoring both clips on prosody and intonation, pacing and rhythm, articulation and clarity, and overall naturalness; the 25K training pairs where Gemini agrees with the human label become SFT targets (effectively rationale distillation from the teacher). The remaining 17K pairs where Gemini disagrees, treated as hard cases, are used for RL with DAPO (a GRPO variant) using 8 rollouts per prompt and a reward of +1 if the parsed preference matches the human label and -1 otherwise; only the final verdict is constrained. Because the model is generative, inference-time scaling by majority voting over multiple sampled judgments is possible. A Bradley-Terry baseline (SpeechJudge-BTRM: a scalar head on the same backbone, same data, LoRA) isolates the effect of the generative formulation.

For downstream use, the authors build Qwen2.5-0.5B-TTS, a two-stage autoregressive-plus-diffusion zero-shot TTS model on DualCodec tokens pre-trained on Emilia (not one of the data-generating systems), and post-train it four ways: offline DPO on INTP, offline DPO on SpeechJudge-Data, offline DPO on INTP pairs relabelled by SpeechJudge-GRM, and online DPO with SpeechJudge-GRM as the reward on INTP prompts.

## Key Results

**Benchmarking existing judges (Table 2).** Naturalness judgment among modern TTS outputs is hard for everything tested. Objective metrics sit near chance (WER 57.9%, SIM 44.5%, FAD 48.6%), as do MOS predictors (DNSMOS 57.9%, UTMOS 53.7%, Audiobox CE 60.8% best), and deepfake detectors fall below chance (AASIST 46.7%, ADV 38.3%). The best AudioLLM is Gemini-2.5-Flash at 69.1%, followed by GPT-4o Audio (67.4%) and Kimi-Audio-7B-Instruct (67.0%); several (Gemma-3n, GPT-4o mini Audio) are at chance. Chain-of-thought prompting helps Gemini slightly (69.1% to 70.5%) but hurts Qwen2.5-Omni (Table 6), and few-shot prompting drops Qwen2.5-Omni to 48 to 53% (Table 7).

**Reward model (Table 3).** SpeechJudge-BTRM reaches 72.7%. CoT SFT alone gives 75.3%, and adding RL gives 77.2% (79.0% regular, 76.0% expressive); majority voting over 10 samples raises these to 77.6% and 79.4%. The GRM's advantage over BTRM is largest on expressive prompts: on emotional and game-character styles BTRM drops to 69.8% and 66.8%, whereas GRM (SFT+RL, Voting@10) reaches 78.1% and 75.7% (Table 10). All judges are weakest on English-only pairs (GRM SFT+RL 69.8% en2en vs. 85.4% zh2zh, Table 8), tracking lower human agreement on English.

**Out-of-distribution generalization (Table 11).** On 500 pairs of native English voice-actor recordings vs. commercial SeedTTS clones (human always labelled more natural), deepfake detectors are near-perfect (98.6 to 99.8%), Gemini-2.5-Flash is at chance (50.8%), BTRM is at chance (50.4%), and the SFT-only GRM is worse than chance (40.8%). RL lifts the GRM to 62.4% (63.7% with voting), the best among the reward models but well below its in-distribution accuracy.

**Downstream use.** For best-of-100 sample selection from Qwen2.5-Omni (Talker), human raters prefer GRM-selected samples over random ones more often than BTRM-selected ones (Figure 5). In TTS post-training, all four methods improve intelligibility (average WER falls from 11.73 to between 8.51 and 9.55, Table 14) and naturalness; online DPO with the GRM reward gives the largest naturalness CMOS gain over the base (+0.25 ± 0.09 vs. +0.18 ± 0.07 for INTP and +0.16 ± 0.08 for offline DPO on SpeechJudge-Data, Figure 6a), though the confidence intervals overlap. Speaker similarity is roughly unchanged (average SPK-SIM 0.661 to 0.670 vs. 0.668 base), with ties above 40% in the subjective similarity test (Figure 6b).

The subjective evaluations in §5.3 and §5.4 use 70 to 100 samples per system rated by experienced speech-synthesis researchers (at least three ratings per sample), which is modest in size.

## Novelty Assessment

The primary contribution is data. A 99K-pair, multiply annotated naturalness preference corpus over current zero-shot TTS systems, with expressive and cross-lingual coverage and documented agreement statistics, did not previously exist for speech. The benchmark is a direct derivation of it and is useful mainly because it quantifies how poorly standard proxies (UTMOS, DNSMOS, WER, SIM) rank modern systems. The reward model is a careful transfer of the text-LLM generative-reward recipe (CoT SFT from a stronger teacher, then GRPO/DAPO with a verifiable reward, inference-time voting) to an AudioLLM, not a new architecture. The informative parts are empirical: the generative judge beats a matched Bradley-Terry head on the same data, and the RL stage, not SFT, is what gives out-of-distribution robustness. The downstream TTS alignment demonstration applies standard DPO and online DPO, and its naturalness gains over the intelligibility-only INTP baseline are small. Because the dataset, benchmark, and generators come from one group's toolchain, the benchmark is largely in-distribution for the trained judges.

## Field Significance

> [!tip]
> High — Provides the first large-scale human preference resource and matched benchmark for speech naturalness, and shows that widely used automatic proxies such as UTMOS and DNSMOS are near chance at ranking modern zero-shot TTS outputs. It also demonstrates a working pipeline from human naturalness preferences to an AudioLLM-based reward model to preference-optimized TTS, which can serve as a reference point for RLHF-style alignment of speech generators.

## Claims

- **complicates:** Common automatic proxies for speech quality, including MOS predictors, ASR error rate, and speaker similarity, are poor at ranking the naturalness of outputs from current zero-shot TTS systems.
  > *Evidence:* On 1,000 full-agreement human preference pairs, WER, SIM, and FAD reach 57.9%, 44.5%, and 48.6% pairwise accuracy, DNSMOS and UTMOS reach 57.9% and 53.7%, and the best zero-shot AudioLLM (Gemini-2.5-Flash) reaches 69.1%. *(§4.2, Table 2)*
- **supports:** Generative reward models that reason before judging, trained with supervised rationales followed by reinforcement learning on human preferences, predict human speech-quality preferences better than scalar Bradley-Terry reward models trained on the same data.
  > *Evidence:* On SpeechJudge-Eval, the Qwen2.5-Omni-7B Bradley-Terry head reaches 72.7%, CoT SFT reaches 75.3%, SFT+RL reaches 77.2%, and majority voting over 10 samples reaches 79.4%; on an out-of-distribution human-vs-SeedTTS-clone set, BTRM is at 50.4% and SFT-only at 40.8%, while SFT+RL reaches 62.4%. *(§5.2, Table 3; §G.1, Table 11)*
- **supports:** A learned naturalness reward model can be used for preference optimization of a TTS model to improve perceived naturalness beyond what intelligibility-focused preference data provides.
  > *Evidence:* Online DPO of Qwen2.5-0.5B-TTS with SpeechJudge-GRM as the reward yields naturalness CMOS +0.25 ± 0.09 over the base, versus +0.18 ± 0.07 for DPO on the intelligibility dataset INTP, while average WER falls from 11.73 to 8.85. *(§5.4, Figure 6a, Table 14)*
- **complicates:** Detecting synthetic speech and judging the relative naturalness of two synthetic utterances are distinct capabilities, so anti-spoofing models are not valid naturalness judges.
  > *Evidence:* AASIST and ADV score 46.7% and 38.3% on synthetic-vs-synthetic naturalness pairs but 98.6% and 99.8% on human-recording-vs-SeedTTS-clone pairs. *(§4.2, Table 2; §G.1, Table 11)*
- **complicates:** Human naturalness preferences between modern TTS outputs are only moderately consistent, and agreement is lower for expressive speech and for listeners judging a non-native language.
  > *Evidence:* Only 51.5% of pairs reach full annotator agreement; binary inter-annotator agreement is 69.0% overall, with 74.4% on Chinese but 61.7% on English and 62.5% on code-switched pairs from predominantly native-Mandarin annotators, and expressive subsets show lower agreement than regular ones. *(§3.2, Figure 3; §C.2, Table 5)*

## Limitations and Open Questions

- **Annotator population and language coverage.** All annotators are professional raters in China, mostly native Mandarin speakers with L2 English. Agreement on English and code-switched data is markedly lower, so the corpus mainly reflects Chinese and Chinese-English bilingual listeners' preferences, and the trained judges are weakest on English-only pairs.
- **In-distribution benchmark.** SpeechJudge-Eval is drawn from the same six open-source generators, prompt sources, and annotator pool as the training data, and contains only full-agreement pairs. The single out-of-distribution test (human vs. SeedTTS clone) drops the best GRM to 62.4%, so the 77 to 79% headline accuracies probably overstate performance on unseen generators or real-vs-synthetic comparisons.
- **Teacher dependence.** The CoT capability is bootstrapped from proprietary Gemini-2.5-Flash, and the RL set is defined by Gemini's errors. Rationale faithfulness was checked only on small samples (an LLM meta-judge for consistency and a small expert rating, Tables 12 and 13).
- **Read-speech, utterance-level scope.** The data is TTS-style read speech, not spontaneous conversation, and naturalness is judged per utterance with no localization of segment-level artifacts.
- **Residual failure modes.** The error analysis finds the GRM over-weights acoustic cleanness against lively prosody, prefers well-articulated flat readings over expressive ones with small errors, penalizes exaggerated but style-appropriate emotional prosody, and is near random when preference gaps are tiny.
- **Small downstream gains and secondary attributes.** Naturalness CMOS differences between post-training methods have overlapping confidence intervals, subjective sets are 70 to 100 samples rated by in-house researchers, and naturalness-focused post-training does not improve speaker similarity, which the authors leave to future alignment work.

## Wiki Connections

- [[evaluation-metrics|Evaluation Metrics]] — Benchmarks WER, SIM, FAD, DNSMOS, UTMOS, Audiobox Aesthetics, deepfake detectors, and AudioLLMs as naturalness judges and introduces pairwise judgment accuracy against human preference as the evaluation criterion.
- [[subjective-evaluation|Subjective Evaluation]] — Contributes a 99K-pair human CMOS preference corpus with inter-annotator agreement analysis, plus human listening tests validating sample selection and TTS post-training.
- [[rlhf-speech|RLHF Speech]] — Trains a naturalness reward model from human preferences and uses it as an offline annotator and online DPO reward to align a zero-shot TTS model.
- [[spoken-language-model|Spoken Language Model]] — Evaluates nine AudioLLMs on paired-audio naturalness judgment and fine-tunes Qwen2.5-Omni-7B with SFT and RL into an audio-consuming generative judge.
- [[zero-shot-tts|Zero-Shot TTS]] — Builds its preference data from six zero-shot TTS systems across expressive and cross-lingual prompts and post-trains a new zero-shot TTS model with the learned reward.
- [[2025.acl-long.598|INTP]] — Builds directly on this same-group intelligibility preference work: reuses its pair-construction strategy and two INTP-enhanced generators, and uses INTP DPO as the main post-training baseline and INTP prompts for online DPO.
- [[2503.20215|Qwen2.5-Omni]] — Serves as the backbone of both SpeechJudge-GRM and SpeechJudge-BTRM, and its Talker produces the candidates for best-of-100 selection.
- [[2507.06261|Gemini 2.5]] — Gemini-2.5-Flash is the strongest zero-shot judge on the benchmark (69.1%) and the teacher whose rationales seed SFT; its disagreements with humans define the RL subset.
- [[2025.acl-long.313|F5-TTS]] — One of the six data-generating systems, representing the flow-matching family.
- [[2412.10117|CosyVoice 2]] — One of the autoregressive data-generating systems, also used in an INTP-enhanced variant.
- [[2409.00750|MaskGCT]] — Represents the masked generative family among the data-generating systems and supplies the ARS autoregressive model.
- [[2204.02152|UTMOS]] — Evaluated as a naturalness judge and found near chance (53.7%) on modern TTS pairs.
- [[2406.02430|Seed-TTS]] — SeedTTS-Eval hard cases serve as evaluation texts, and SeedTTS voice clones form the out-of-distribution human-vs-clone test.
