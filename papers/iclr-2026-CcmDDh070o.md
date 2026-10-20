---
id: "iclr-2026-CcmDDh070o"
title: "ParaS2S: Benchmarking and Aligning Spoken Language Models for Paralinguistic-aware Speech-to-Speech Interaction"
authors: ["Shu-wen Yang", "Ming Tu", "Andy T. Liu", "Xinghua Qu", "Hung-yi Lee", "Lu Lu", "Yuxuan Wang", "Yonghui Wu"]
organization: "National Taiwan University / ByteDance Seed"
venue: ICLR
venue_type: conference
year: 2026
month: 4
published_date: "2026-04-27"
ingested_date: "2026-10-20"
task: [SCA, evaluation]
architecture: [autoregressive-LM]
conditioning: []
training: [supervised, fine-tuning, RLHF]
model_size: "not reported"
codec_used: "not reported"
datasets_train: ["ParaS2S synthetic training prompts (LLM-generated queries, TTS-synthesized; 100 h SFT demonstrations, up to 100k RL prompts)", "LibriSpeech (speaker prompts)", "CommonVoice (speaker prompts)", "NNCES (child speaker prompts)"]
datasets_eval: ["ParaS2SBench (synthetic: emotion, sarcasm, age, gender)", "IEMOCAP", "MELD", "VoiceBench"]
metrics:
  - name: "ParaS2SBench score (1-5, automatic judge)"
    value: 4.382
    system: "proposed (Kimi-Audio + SFT warm-up + GRPO)"
    testset: "ParaS2SBench overall average (Table 4)"
  - name: "ParaS2SBench score (1-5, automatic judge)"
    value: 3.955
    system: "Kimi-Audio SFT"
    testset: "ParaS2SBench overall average (Table 4)"
  - name: "ParaS2SBench score (1-5, automatic judge)"
    value: 2.35
    system: "Kimi-Audio (base)"
    testset: "ParaS2SBench overall average (Table 4)"
  - name: "ParaS2SBench score (1-5, automatic judge)"
    value: 3.552
    system: "Gemini (best existing model)"
    testset: "ParaS2SBench overall average (Table 4)"
  - name: "ParaS2SBench score (1-5, automatic judge)"
    value: 3.176
    system: "Whisper-GPT-TTS cascade baseline (style-blind)"
    testset: "ParaS2SBench overall average (Table 4)"
  - name: MOS
    value: 4.303
    system: "proposed (Kimi-Audio + SFT warm-up + GRPO)"
    testset: "ParaS2SBench subset, 30 prompts per category, 10 crowd-sourced raters (Table 7)"
  - name: MOS
    value: 3.996
    system: "Kimi-Audio SFT"
    testset: "ParaS2SBench subset, 30 prompts per category, 10 crowd-sourced raters (Table 7)"
  - name: MOS
    value: 3.564
    system: "GPT-4o voice mode"
    testset: "ParaS2SBench subset, 30 prompts per category, 10 crowd-sourced raters (Table 7)"
  - name: "Pearson correlation with expert scores (average)"
    value: 0.776
    system: "multi-stage judge with tone description (O2)"
    testset: "200 prompts per category, TTS and S2S responses (Table 2)"
  - name: "Pearson correlation with expert scores (average)"
    value: 0.618
    system: "gpt-audio end-to-end judge (B1)"
    testset: "200 prompts per category, TTS and S2S responses (Table 2)"
code_available: null
demo_available: true
url: "https://openreview.net/forum?id=CcmDDh070o"
related_concepts: [spoken-language-model, speech-to-speech, rlhf-speech, emotion-synthesis, evaluation-metrics, subjective-evaluation]
related_papers: ["2504.18425", "2503.20215", "2412.02612", "2507.18119", "2025.acl-long.997", "2410.17196", "2407.05407", "2402.03300", "2501.12948", "2312.15185", "2212.04356", "2505.02625", "2411.00774", "2409.06666", "2408.16725", "2410.00037", "2502.11946", "2407.10759", "2311.07919", "2310.13289", "2402.07729", "2410.19168", "2305.11000", "2503.04721", "iclr-2026-I9ED9VWZq6"]
field_significance:
  level: "moderate"
  type: [evaluation-contribution, dataset-contribution, engineering-integration]
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
> **Shu-wen Yang et al.** (National Taiwan University / ByteDance Seed) · [→ Paper](https://openreview.net/forum?id=CcmDDh070o) · Demo: ✓ · Code: ?
>
> Introduces a benchmark that scores whether a speech-to-speech dialogue model's spoken reply fits the user's paralinguistic state (emotion, sarcasm, age, gender) in both content and speaking style, and shows that a judge distilled into a reward model can drive GRPO post-training that makes an end-to-end speech LM responsive to those cues.

## Problem

End-to-end speech-to-speech dialogue models inherit LLM reasoning and score well on spoken QA and instruction-following benchmarks, but those benchmarks ignore how the user spoke and, when they do vary the input style (StyleTalk, VoxDialogue, SD-Eval, ADU-Bench), they evaluate only the text of the response. No existing benchmark checks whether the response *speech* adapts its content and tone to the user's voice. The authors also find that current models tend to infer the user's state from the words alone, which produces tone-deaf replies when the words are neutral. Training such behaviour is limited by data: expressive paired demonstrations need style annotation and expressive response recordings, which are costly to collect.

## Method

**ParaS2SBench.** Test queries follow three principles: each spoken sentence is paired with two contrasting speaking styles that should elicit different replies (following StyleTalk); the text content is filtered by an LLM for neutrality (style cannot be inferred from the words), reasonability, and paralinguistic relevance (the two styles actually lead to different ideal responses); and evaluation is performed on the output speech. Synthetic queries cover emotion and sarcasm (synthesized with the instruction-driven gpt-4o-mini-tts) and age and gender (zero-shot cloning with CosyVoice from LibriSpeech, CommonVoice and NNCES voices), filtered by WER and an emotion2vec classifier and then manually verified by three annotators. A real-speech split is drawn from IEMOCAP and MELD. The test set totals 2,690 prompts, 3,890 utterances and 7.8 hours (§A.2, Table 6).

**Multi-stage automatic judge.** Rather than feeding the audio pair to an audio LLM, the judge projects input and output speech into text: Whisper-v3 transcriptions; emotion, sarcasm, age and gender labels from fine-tuned classifiers (emotion2vec for emotion and sarcasm, a wav2vec 2.0 age classifier, an ECAPA-TDNN gender classifier); and a free-form tone description of the response from AudioReasoner, prompted to ignore content. A text LLM (GPT-4.1) then scores the pair on a 1 to 5 "continuation fitness" scale using an expert-written rubric (§3.2, §A.3, §A.8.5). The classifiers are trained with **PolyTone**, a data recipe in which the same sentence is synthesized in many styles, so the classifier cannot shortcut through the lexical content (§3.2 Stage 1).

**ParaS2SAlign.** The base policy is Kimi-Audio, which predicts parallel audio and text token streams autoregressively and decodes audio tokens with a flow-matching decoder (§A.4). Training has three stages: (1) a short SFT warm-up (2 epochs) on LLM-written, gpt-4o-mini-tts-synthesized, manually selected expressive responses, because the base model rarely produces a high-scoring sample to learn from; (2) reward-model distillation, where the warm-up model samples 32 responses for each of up to 10k prompts, the full judge (using predicted rather than ground-truth input labels) scores the 320k pairs, and Qwen2.5-Omni is LoRA-fine-tuned to emit the score as a single token; (3) GRPO on unlabeled speech prompts with group size 8, query batch 32 and KL weight 0.2 (§4, §5, §A.4, §A.5). The paper does not report Kimi-Audio's parameter count or codec details.

## Key Results

- **Judge correlation (Table 2).** Against expert scores on 200 prompts per category, the multi-stage judge with ground-truth input labels and a tone description (O2) reaches an average Pearson correlation of 0.776, versus 0.618 for the end-to-end gpt-audio judge. Replacing ground-truth input labels with PolyTone-trained classifier predictions (O5, the setting used for reward distillation) lowers the average to 0.723; classifiers trained on data without shared content (O6) give 0.701, with the largest PolyTone gains on emotion and sarcasm. Free-form tone descriptions of the response beat discrete emotion tags (O2 0.776 vs. O3 0.731).
- **Ranking fidelity (Table 3).** The automatic judge and human experts order eight response systems almost identically (one adjacent swap, between gpt-4o-audio-preview and Qwen2.5-Omni). The absolute scores differ, with the automatic judge rating S2S systems on emotion about 1 point higher than humans.
- **Existing S2S models (Table 4).** Most current systems score near the style-blind Whisper-GPT-TTS cascade (3.176): Gemini 3.552, gpt-4o-audio-preview 3.403, Qwen2.5-Omni 3.369, GLM-4-Voice 3.034, Kimi-Audio 2.35. Kimi-Audio collapses on the real-speech split (1.365 IEMOCAP, 1.166 MELD).
- **Alignment (Table 4).** SFT lifts Kimi-Audio to 3.955 and GRPO to 4.382 (about 10.8% relative over SFT), still below a topline that feeds ground-truth style labels to an LLM and expressive TTS (4.725). The GRPO gain over SFT is largest on real MELD speech (3.307 to 3.927) and synthetic gender (3.53 to 4.239).
- **Human evaluation (Table 7).** Ten crowd-sourced raters, outside the expert group and unfamiliar with the rubric, scored 30 prompts per category. The ordering is preserved: GRPO 4.303, SFT 3.996 (7.6% relative gain), GPT-4o voice mode 3.564, Qwen2.5-Omni 3.538, Kimi-Audio 2.699. Crowd raters score higher overall than experts and the automatic judge.
- **Data efficiency (Figure 2a).** RL improves the SFT model across warm-up budgets from 10 to 100 hours, and a model warmed up on 20 hours and then RL-trained exceeds pure SFT on the full 100 hours (§5.2).
- **Capability preservation (Figure 2b, Figure 3c).** VoiceBench scores are largely preserved after warm-up and GRPO; without a KL penalty GRPO causes catastrophic forgetting, and too high a penalty limits ParaS2SBench gains. Pure SFT beyond 10 epochs degrades general ability and shortens responses (§5.2, §A.5).

Baselines are run on the authors' own benchmark, and the comparison with prior S2S models is fair only in the sense that none of them was trained for this task; the strongest evidence for the method is the SFT-vs-GRPO comparison on the same base model, which holds under both automatic and human scoring.

## Novelty Assessment

The main new artifact is the benchmark design: contrasting-style pairs with LLM-filtered neutral content, scored on the response waveform for both content and style. Prior style-aware dialogue benchmarks stop at the text response, so evaluating response speaking style is a genuine extension. PolyTone is a simple but sensible data recipe (parallel multi-style synthesis of identical text) for forcing paralinguistic classifiers to ignore lexical content. The alignment side is engineering integration: SFT warm-up, distilling an LLM-based judge into a learned reward model, and GRPO with a KL penalty are all established techniques, applied here to paralinguistic-aware S2S. Note that GOAT-SLM already targeted paralinguistic-aware S2S with a multi-stage SFT pipeline; the novelty here is the RL route and the data-efficiency comparison, not the task itself. The reward and the evaluation judge share most components (Whisper, the captioners, GPT-4.1 with the same rubric), so the human evaluation in Table 7 is the main independent check.

## Field Significance

Moderate — The paper provides the first benchmark that scores a speech LM's spoken reply for paralinguistic appropriateness, and documents that current end-to-end S2S models perform no better than a style-blind cascade on neutral-content queries. It also demonstrates that a decoupled, text-projected judge can be accurate enough to serve as an RL reward for speech-to-speech dialogue, with RL reducing the amount of expensive expressive demonstration data required.

## Claims

- **supports:** End-to-end speech-to-speech dialogue models largely fail to adapt response content and speaking style to the user's paralinguistic state when the lexical content is neutral.
  > *Evidence:* On ParaS2SBench most open and closed S2S models score close to a Whisper-GPT-TTS cascade that ignores input style (3.176), and Kimi-Audio scores 2.35 overall and below 1.4 on real IEMOCAP and MELD speech. *(§5.2, Table 4)*
- **complicates:** End-to-end audio LLMs used as judges of spoken dialogue tend to infer paralinguistic cues from the spoken words, making them less reliable than pipelines that score style separately from content.
  > *Evidence:* A multi-stage judge projecting transcripts, classifier labels and tone descriptions into text for an LLM reaches 0.776 average Pearson correlation with expert scores versus 0.618 for an end-to-end gpt-audio judge; analysis of the audio LLM's reasoning shows it hallucinates cues from content. *(§5.1, Table 2)*
- **supports:** Training paralinguistic classifiers on utterances that share linguistic content across styles improves their usefulness for judging emotion and attitude in dialogue.
  > *Evidence:* PolyTone-trained captioners (O5) give higher judge-human correlation than captioners trained on unique-content data (O6), with the clearest gains on emotion (0.684 vs. 0.645) and sarcasm (0.703 vs. 0.669) and minor gains on age and gender. *(§5.1, Table 2)*
- **supports:** Reinforcement learning against a learned AI-feedback reward can improve expressive, context-appropriate speech generation in speech LMs beyond supervised fine-tuning while reducing the amount of paired demonstrations needed.
  > *Evidence:* GRPO with a reward model distilled from the judge raises Kimi-Audio from 3.955 (SFT) to 4.382 on ParaS2SBench, with a 7.6% relative gain under crowd-sourced human MOS; a 20-hour warm-up plus RL exceeds pure SFT on 100 hours. *(§5.2, Table 4, Table 7, Figure 2a)*
- **complicates:** Post-training speech LMs for a narrow conversational behaviour risks degrading general dialogue ability unless the update is regularized.
  > *Evidence:* Without a KL penalty, GRPO causes catastrophic forgetting on VoiceBench; pure SFT for more than 10 epochs shortens responses and degrades general capability, while a 2-epoch warm-up with KL weight 0.2 preserves it. *(§5.2, §A.5, Figure 3c)*

## Limitations and Open Questions

- The reward used for RL and the benchmark judge share the same transcription model, classifiers, LLM and rubric, so automatic-score gains partly measure agreement with the training signal. The crowd-sourced human evaluation corroborates the ordering but covers only 30 prompts per category, and crowd raters were noticeably more lenient than experts (§A.6).
- The benchmark judge uses ground-truth input style labels (O2), so it applies only to labeled test sets; the label-free variant used for RL correlates less well with humans (0.723), and no judge reaches 0.9 correlation.
- Synthetic queries come from commercial and open TTS systems, and the training data is fully synthetic and not human-verified. The real-speech split covers only emotion (IEMOCAP, MELD); age, gender and sarcasm are evaluated on synthetic speech only, because suitable real queries could not be sourced.
- The scoring rubric encodes specific normative choices (e.g., gender-personalized recommendations, sharing the user's emotion) that may not match all users' expectations; crowd raters were more tolerant of tone-deaf replies than the rubric implies.
- Only one base model (Kimi-Audio) is aligned, and only GRPO is tested; PPO and DPO are left to future work. The paper states data, code and models will be released, but no repository is given in the text.

## Wiki Connections

- [[spoken-language-model|Spoken Language Model]] — benchmarks existing speech LMs on real and synthetic audio and post-trains Kimi-Audio for paralinguistic awareness.
- [[speech-to-speech|Speech-to-Speech]] — targets end-to-end spoken dialogue, evaluating the response waveform rather than the response text.
- [[rlhf-speech|RLHF Speech]] — uses GRPO with a reward model distilled from an LLM-based judge (AI feedback) for expressive spoken dialogue.
- [[emotion-synthesis|Emotion Synthesis]] — trains and scores the model on producing an emotionally appropriate speaking tone in response to the user's emotion and sarcasm.
- [[evaluation-metrics|Evaluation Metrics]] — proposes a multi-stage automatic judge and the PolyTone classifier training recipe, validated by correlation with expert scores.
- [[subjective-evaluation|Subjective Evaluation]] — collects expert Likert scores to validate the judge and crowd-sourced MOS to confirm the alignment gains.
- [[2504.18425|Kimi-Audio]] — serves as the base S2S policy for SFT warm-up and GRPO, and as the weakest existing baseline on the benchmark.
- [[2503.20215|Qwen2.5-Omni]] — evaluated as a baseline and LoRA-fine-tuned as the distilled reward model.
- [[2412.02612|GLM-4-Voice]] — evaluated as an existing S2S baseline in both the judge study and the benchmark.
- [[2507.18119|GOAT-SLM]] — the prior paralinguistic-aware S2S model, built with multi-stage SFT, which this paper contrasts with its RL approach.
- [[2025.acl-long.997|Align-SLM]] — prior RL-from-AI-feedback work for spoken language models, which targeted semantics rather than style.
- [[2410.17196|VoiceBench]] — used to check that general dialogue ability is preserved after alignment.
- [[2407.05407|CosyVoice]] — used for zero-shot synthesis of age and gender queries and as a TTS response system in the judge study.
- [[2402.03300|DeepSeekMath]] — source of the GRPO algorithm used for post-training.
