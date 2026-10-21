---
id: "iclr-2026-wbttgzp7MT"
title: "EmotionThinker: Prosody-Aware Reinforcement Learning for Explainable Speech Emotion Reasoning"
authors: ["Dingdong Wang", "Shujie Liu", "Tianhua Zhang", "Youjun Chen", "Jinyu Li", "Helen M. Meng"]
organization: "The Chinese University of Hong Kong / Microsoft"
venue: ICLR
venue_type: conference
year: 2026
month: 4
published_date: "2026-04-27"
ingested_date: "2026-10-21"
task: [SCA]
architecture: [autoregressive-LM]
conditioning: []
training: [supervised, fine-tuning, RLHF]
model_size: "7B params (Qwen2.5-Omni-7B policy); 3B reasoning reward model (Qwen2.5-Omni-3B)"
codec_used: "none"
datasets_train: ["EmotionCoT-35K (from IEMOCAP, MELD, Expresso, MEAD, EARS)", "Stress-17K", "GigaSpeech (prosodic attribute and comparative augmentation tasks)", "LibriSpeech (ASR replay)", "Reward-model set of 101,400 GPT-4o-synthesized reasoning variants"]
datasets_eval: ["IEMOCAP test", "MELD test", "RAVDESS (zero-shot)", "SAVEE (zero-shot)", "Prosody perception test set (1,000 held-out utterances)"]
metrics:
  - name: Emotion Recognition Accuracy (unweighted, %)
    value: 68.89
    system: EmotionThinker
    testset: "Average over IEMOCAP/MELD/RAVDESS/SAVEE (Table 2)"
  - name: Emotion Recognition Accuracy (unweighted, %)
    value: 77.68
    system: EmotionThinker
    testset: "IEMOCAP (Table 2)"
  - name: Emotion Recognition Accuracy (unweighted, %)
    value: 65.41
    system: BLSP-Emo
    testset: "Average over IEMOCAP/MELD/RAVDESS/SAVEE (Table 2)"
  - name: Emotion Recognition Accuracy (unweighted, %)
    value: 50.83
    system: Qwen2.5-Omni-7B
    testset: "Average over IEMOCAP/MELD/RAVDESS/SAVEE (Table 2)"
  - name: Emotion Reasoning Score (GPT-4o judge, 1-5)
    value: 3.98
    system: EmotionThinker
    testset: "Overall test set, mean of four criteria (Table 2)"
  - name: Emotion Reasoning Score (human raters, 1-5)
    value: 4.4
    system: EmotionThinker
    testset: "100-output human evaluation set (Table 3)"
code_available: null
demo_available: null
url: "https://openreview.net/forum?id=wbttgzp7MT"
related_concepts: [spoken-language-model, rlhf-speech]
related_papers: ["2503.20215", "2508.09600", "2504.18425", "2508.07273", "2025.acl-long.388", "2402.03300", "2501.12948", "2308.05725", "2505.22765", "2412.02612", "2407.10759", "2311.07919", "2310.13289", "2503.01743", "2504.08528", "2411.13577", "2506.02863"]
field_significance:
  level: "moderate"
  type: [architectural-novelty, dataset-contribution]
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
> **Dingdong Wang et al.** (The Chinese University of Hong Kong / Microsoft) · [→ Paper](https://openreview.net/forum?id=wbttgzp7MT) · Demo: ? · Code: ?
>
> Reframes speech emotion recognition in a speech LLM as reinforcement-learned reasoning, combining prosody-centric fine-tuning with a GRPO reward that gates a learned reasoning-quality score by its agreement with answer correctness.

## Problem

Speech LLMs, like conventional speech emotion recognition (SER) systems, mostly treat emotion as a categorical label, so their predictions come with no grounded justification. Earlier attempts at explainability (emotion captioning via supervised fine-tuning) produce surface descriptions of acoustics without linking them causally to the emotional judgment. The authors identify three obstacles to doing better with reinforcement learning: emotion corpora lack fine-grained prosodic annotations to supervise reasoning, current speech LLMs perceive prosody (stress, intonation, pitch) poorly, and outcome-only rule-based rewards cannot stop a model from producing plausible but ungrounded reasoning that happens to reach the right label.

## Method

This is a speech understanding system: audio goes in, and text (a `<think>` reasoning trace followed by an `<answer>` emotion label) comes out. No speech is generated. Training has three stages on top of Qwen2.5-Omni-7B.

**EmotionCoT-35K.** An automatic pipeline annotates about 200 hours (35,000 utterances) drawn from IEMOCAP, MELD, Expresso, MEAD and EARS over nine emotion classes. Signal processing tools give pitch, energy and speaking-rate levels. WhiStress tags stressed words. Savitzky-Golay-smoothed pitch and energy trajectories are classified into a coarse style (expressive vs. flat) and an intonation contour (rising, falling, rising-falling, falling-rising). A wav2vec 2.0 classifier supplies gender and age group. These attributes, the transcript and the gold label are passed to GPT-4o, which writes step-by-step reasoning traces that are told to reach the emotion "naturally" from the cues *(§3.1, Appendix B)*.

**Prosody-centric SFT (EmotionThinker-Base).** About 500 hours of auxiliary supervision target prosody perception: word-level stress from Stress-17K (from the accepted but not yet ingested StressTest paper, 2505.22765), categorical pitch/energy/speed/intonation questions on GigaSpeech, and comparative tasks in which pitch-, energy- or speed-shifted copies of one utterance are concatenated and the model must report the ordering. Stage I updates the audio encoder, adapter and LLM with full-parameter training, with 20% text-only and 20% ASR data mixed in to retain existing abilities. Stage II is a LoRA cold start on 5K EmotionCoT samples to teach the reasoning format *(§3.2, Appendix C)*.

**GRPO-PTR.** Reinforcement learning follows GRPO with a format reward and a binary outcome-accuracy reward, plus a reasoning reward from a Qwen2.5-Omni-3B reward model. The reward model is fine-tuned on 101,400 tuples in which GPT-4o rewrote gold traces to hit target 1-5 scores on four criteria: factual alignment, interpretative quality, caption completeness, and fluency/structure. Its weighted, normalised scores form the reasoning reward. Two mechanisms make up "progressive trust-aware" (PTR). First, a group-level trust weight τ is 1 when correct responses in the sampled group have a higher mean reasoning reward than incorrect ones, and decays as exp of the (negative) gap otherwise, which damps the learned reward when it disagrees with correctness. Second, the reasoning reward is switched on only after outcome and format rewards have pushed accuracy past a threshold (about 50% in the paper's example). Final weights are 1.0 for accuracy, 0.3 for format and 0.5 for reasoning, with K = 8 rollouts, KL coefficient 0.04 and 3,000 steps on 30K EmotionCoT samples *(§3.3, §4.1)*.

![Figure 3: Architecture of EmotionThinker with the proposed GRPO-PTR framework. The upper part depicts the high-level GRPO-PTR training pipeline, where only the policy model is optimized. The lower part details PTR strategy, which progressively introduces reasoning reward to stabilize training and enhance reasoning.](assets/iclr-2026-wbttgzp7MT/figure-3.png)

## Key Results

Across 16 open speech LLMs and omni models, EmotionThinker has the highest average unweighted emotion accuracy (68.89% vs. 65.41% for BLSP-Emo and 50.83% for its own Qwen2.5-Omni-7B backbone). It also has the highest GPT-4o-judged reasoning score (3.98/5 vs. 3.04 for the next best, MERaLiON2) *(Table 2)*. Its lead is not uniform. BLSP-Emo is slightly ahead on RAVDESS (72.00 vs. 71.56), and the MELD gain over Kimi-Audio is small (59.71 vs. 59.13). The comparison also favours the proposed system on two benchmarks: EmotionCoT-35K is built partly from IEMOCAP and MELD, so those test sets are in-domain for EmotionThinker. Only RAVDESS and SAVEE are zero-shot.

A human study (four raters, 100 outputs, 20 per model across five models) gives EmotionThinker a 4.4 average against 3.5 for Qwen2.5-Omni, and the model ranking matches the GPT-based scores *(Table 3)*. Prosody-centric SFT raises the backbone's accuracy on a 1,000-utterance prosody perception test from 25.71% to 75.11% on pitch and from 30.24% to 71.5% on stress *(Table 5)*.

The ablations in Table 4 are the most informative part of the paper. SFT on EmotionCoT reaches 53.91% accuracy and a 3.78 reasoning score. Plain GRPO raises accuracy to 62.91% but lowers the reasoning score to 3.45. Full GRPO-PTR reaches 68.89% and 3.98. Using an untrained reward model gives 66.67% and 3.36, below plain GRPO on reasoning. Removing τ keeps accuracy (67.71%) but lowers reasoning to 3.74. Turning off progressive scheduling drops accuracy to 62.80%. Raising the reasoning-reward weight to 1.0 lowers average accuracy to 65.52% *(Table 7)*. Rollout count K barely matters between 4 and 16 *(Table 6)*.

## Novelty Assessment

The pipeline (an omni-LLM backbone, GPT-4o-distilled chain-of-thought data, SFT cold start, then GRPO) follows the R1-style recipe that the paper's related work traces through vision-language reasoning models, and EmotionCoT-35K is an automatic relabelling of existing emotion corpora. The genuinely new piece is the reward design. Gating a learned process reward at the group level by its agreement with a verifiable outcome is a simple, general idea, and the ablations show that each part (trained reward model, trust weight, delayed introduction) contributes. Prosody-centric SFT with comparative augmentation is a sensible engineering step with a large measured effect on prosody perception. However, the reasoning-quality evaluation is close to circular: the training traces, the reward model's training data and the evaluation's reference explanations all come from GPT-4o prompted with the same automatically extracted cues, and a GPT-4o judge does the scoring. The small human study only partly offsets this.

## Field Significance

Moderate — This is a speech understanding paper with no speech output. Its contribution to speech generation is indirect. It shows that a speech LLM can be trained to explain emotion judgments in terms of pitch, energy, rate, stress and intonation, which makes a model of this kind a candidate emotion or prosody evaluator, or a reward signal, for expressive TTS and spoken dialogue. The paper itself does not test either use. Its outcome-gated reasoning reward is a transferable data point for RL on speech models where a learned judge risks reward hacking.

## Claims

- **complicates:** Outcome-only rule-based rewards in reinforcement learning for speech LLMs can raise task accuracy while degrading the quality of the intermediate reasoning, compared with supervised fine-tuning on the same data.
  > *Evidence:* On identical EmotionCoT data, plain GRPO raises average SER accuracy from 53.91% (SFT) to 62.91% but lowers the GPT-judged reasoning score from 3.78 to 3.45, and the case study shows GRPO-only outputs with hallucinated acoustic descriptions. *(§4.3, Table 4, Figure 4)*
- **supports:** Gating a learned reward model's contribution by its agreement with a verifiable outcome signal mitigates reward hacking when RL-training speech models with learned judges.
  > *Evidence:* The trust weight τ shrinks the reasoning reward when incorrect rollouts in a group score higher than correct ones. Removing τ leaves accuracy similar (67.71% vs. 68.89%) but lowers reasoning quality from 3.98 to 3.74, and an untrained reward model gives a lower reasoning score (3.36) than no reasoning reward at all. *(§3.3.2, §4.3, Table 4)*
- **refines:** Combining learned and rule-based rewards in speech-model RL is sensitive to scheduling and weighting: introducing or over-weighting the learned reward too early destabilises optimisation.
  > *Evidence:* Adding all rewards from the start lowers accuracy from 68.89% to 62.80%, and raising the reasoning-reward weight from 0.5 to 1.0 lowers average accuracy from 68.89% to 65.52%. *(§3.3.2, §4.3, Table 4, Table 7)*
- **supports:** General-purpose speech LLMs perceive fine-grained prosody (pitch, energy, rate, intonation, stress) poorly, and targeted prosody-centric fine-tuning with synthetic comparative augmentation substantially improves it.
  > *Evidence:* On 1,000 held-out utterances, prosody-centric SFT raises Qwen2.5-Omni-7B accuracy from 25.71% to 75.11% (pitch), 29.94% to 68.7% (speed), 27.67% to 69.42% (energy), 25.83% to 60.25% (intonation) and 30.24% to 71.5% (stress). *(§3.2, §4.3, Table 5)*
- **complicates:** LLM-judge scores of paralinguistic reasoning quality are only as independent as their references, and they agree with human rankings mainly in small samples.
  > *Evidence:* The GPT-4o judge scores explanations against GPT-4o-generated reference explanations built from the same extracted cues used to create the training traces. The human check covers only 100 outputs (20 per model) from four raters, although model rankings agree. *(§4.2, Table 3, Appendix E)*

## Limitations and Open Questions

- The comparison is not fully like-for-like. IEMOCAP and MELD contribute training data to EmotionCoT-35K and are also test sets, while most baselines see them zero-shot. Baselines without native reasoning output are scored on reasoning elicited by a unified prompt.
- The "factual" grounding is factual only relative to automatically extracted labels (DSP-based levels, WhiStress stress, wav2vec 2.0 gender and age). Errors in that pipeline propagate into training traces, reward-model data and evaluation references alike. The paper reports no direct accuracy for these annotators beyond a statement that they were validated externally.
- Reasoning and label can still diverge. In the appendix case study (gold label sad), EmotionThinker's own reasoning text concludes "frustration or irritation" *(Appendix F, Table 9)*.
- The nine-class emotion inventory includes "Whisper", which is a phonation style rather than an emotion, so the label space mixes emotion and vocal style.
- Whether a prosody-aware emotion reasoner of this kind works as an evaluator or reward model for expressive speech *synthesis* is untested. All evaluation data is natural speech.

## Wiki Connections

- [[spoken-language-model|Spoken Language Model]] — Adapts an omni speech LLM to consume speech and output emotion reasoning traces, and benchmarks 16 other speech LLMs on emotion recognition and explanation.
- [[rlhf-speech|RLHF Speech]] — Applies GRPO with a trust-gated learned reasoning reward to a speech LLM, with ablations isolating reward-model training, trust gating and progressive scheduling.
- [[2503.20215|Qwen2.5-Omni]] — Serves as the 7B policy backbone and 3B reward-model backbone, and as the main baseline that prosody-centric SFT and RL improve on.
- [[2508.09600|OSUM-EChat]] — Compared as an emotion-focused empathetic dialogue baseline, which scores well below the proposed system on both accuracy and reasoning.
- [[2504.18425|Kimi-Audio]] — The strongest general-purpose speech LLM baseline on accuracy, nearly matching EmotionThinker on MELD.
- [[2508.07273|Contextual Paralinguistic Speech-LLM]] — Cited as prior work that adds paralinguistic and emotion descriptions through supervised data, the descriptive approach this paper extends with RL-trained reasoning.
- [[2025.acl-long.388|DIVA]] — Included as a general speech LLM baseline for emotion accuracy and reasoning quality.
- [[2402.03300|DeepSeekMath]] — Source of the GRPO algorithm that GRPO-PTR extends with a trust-gated process reward.
- [[2501.12948|DeepSeek-R1]] — Provides the rule-based-reward RL reasoning paradigm whose outcome-only supervision the paper argues is insufficient for emotion reasoning.
- [[2308.05725|Expresso]] — One of five expressive speech corpora relabelled to build EmotionCoT-35K.
