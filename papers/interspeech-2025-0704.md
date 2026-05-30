---
id: interspeech-2025-0704
title: "Differentiable Reward Optimization for LLM based TTS system"
authors: [Changfeng Gao, Zhihao Du, Shiliang Zhang]
organization: Alibaba Group (Tongyi Lab)
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-05-30
task: [TTS]
architecture: [autoregressive-LM, flow-matching]
conditioning: [zero-shot, emotion-conditioned, instruction-conditioned]
training: [RLHF, fine-tuning, supervised]
model_size: "not reported"
codec_used: "CosyVoice 2.0 tokenizer (custom)"
datasets_train: [in-house industry ASR corpus (13,000+ hours for reward model), in-house TTS SFT data (4000 samples, 5 speakers)]
datasets_eval: [seed-tts-eval, CommonVoice (Japanese/Korean), MELD (emotion)]
metrics:
  - name: WER
    value: 0.78
    system: CosyVoice2.0 + DiffRO-ASR
    testset: seed-tts-eval zh
  - name: WER
    value: 1.56
    system: CosyVoice2.0 (baseline)
    testset: seed-tts-eval zh
  - name: WER
    value: 1.89
    system: CosyVoice2.0 + DiffRO-ASR
    testset: seed-tts-eval en
  - name: WER
    value: 5.58
    system: CosyVoice2.0 + DiffRO-ASR
    testset: seed-tts-eval hard
  - name: WER
    value: 6.91
    system: CosyVoice2.0 (baseline)
    testset: seed-tts-eval hard
  - name: WER
    value: 1.05
    system: CosyVoice2.0-SFT + DiffRO-MTR
    testset: seed-tts-eval zh
code_available: null
demo_available: null
url: https://www.isca-archive.org/interspeech_2025/gao25d_interspeech.html
related_concepts: [rlhf-speech, autoregressive-codec-tts, evaluation-metrics, emotion-synthesis]
related_papers: [2406.02430, 2412.10117, 2410.06885]
---

# Differentiable Reward Optimization for LLM based TTS system

**Paper:** [https://www.isca-archive.org/interspeech_2025/gao25d_interspeech.html](https://www.isca-archive.org/interspeech_2025/gao25d_interspeech.html)

**One-sentence contribution:** DiffRO applies RLHF to neural codec LM-based TTS by predicting rewards directly from codec tokens (bypassing the FM + vocoder), using Gumbel-Softmax to make the reward differentiable for direct backpropagation, achieving SOTA WER on seed-tts-eval and enabling zero-shot emotion/quality control via a multi-task reward model.

## Problem

Applying RLHF to neural codec LM-based TTS faces three specific challenges not present in text LLM RLHF: (1) The TTS pipeline requires a flow-matching model and vocoder to convert codec tokens to audio for reward computation, creating substantial computational overhead that makes large-scale RLHF data generation expensive. (2) Diverse token sequences from the same text tend to produce highly similar audio, making it difficult to construct clearly differentiated positive/negative pairs for reward model training. (3) TTS quality is multi-dimensional (pronunciation, naturalness, speaker similarity, emotion) and cannot be adequately captured by a single binary good/bad label.

## Method

**Token2Reward prediction.** Instead of decoding codec tokens to audio before computing rewards, DiffRO trains a reward model that operates directly on predicted codec token sequences. The language model generates a codec token sequence Ũ via autoregressive sampling, then the reward model predicts the ASR transcript from Ũ. The reward is the log-probability that the predicted transcript matches the input text — effectively using ASR performance on the codec token space as a proxy for pronunciation quality. To make this differentiable for end-to-end optimization, Gumbel-Softmax replaces the argmax token sampling operation, allowing gradients to flow from the reward signal back through the LM parameters without PPO's policy gradient loop.

**Multi-Task Reward (MTR) model.** A codec-based speech understanding model is trained jointly on four downstream tasks: ASR (WER), speech emotion recognition (SER, cross-entropy), speech quality assessment (SQA, MOS prediction via 5-class CE), and audio event detection. SenseVoice's CNN front-end is replaced with an embedding layer that accepts codec token inputs; attention pooling heads are added per task. Training uses 13,000+ hours of in-house ASR data with pseudo-labels for emotion (cross-entropy), MOS (5-class classification), age/gender (CE/MSE), and events (binary CE). The MTR model is used as a reward to guide the LM to generate tokens that carry desired attribute information.

**Training setup.** The baseline system is CosyVoice 2.0-0.5B. SFT fine-tuning uses 4,000 samples from 5 speakers (mostly Mandarin). RL training uses 10,000 Internet-sourced texts (90% Chinese/English, 10% Japanese/Korean). β=0.1 KL penalty, lr=1e-5, 4 × A800 GPUs.

**Instruction template for MTR control.** "Your emotion is E ⟨endofprompt⟩ text ⟨s⟩ tokens ⟨/s⟩" — emotion is controlled by maximizing the SER reward log P(Aemo = E | Ũ).

## Key Results

**ASR-based DiffRO (pronunciation accuracy):** On seed-tts-eval, CosyVoice2.0 + DiffRO-ASR achieves WER 0.78% (zh), 1.89% (en), 5.58% (hard) — compared to baseline 1.56%/2.75%/6.91%. This is SOTA on seed-tts-eval zh and hard. Applied to the SFT model, DiffRO-ASR (1.09% zh) outperforms DPO (1.27% zh). Multilingual generalization without explicit RL data: Japanese WER improves from 9.13% to 6.36%, Korean from 7.43% to 5.41%, because the ASR reward teaches the LM to produce phonetically accurate codec tokens in the codec's implicit multilingual space.

**MTR-based DiffRO (emotion control):** On EmoBox benchmark (100 samples per emotion, Chinese and English), DiffRO-MTR achieves emotion accuracy: Happy zh/en 1.00/0.92, Sad zh/en 0.76/0.96, Angry zh/en 0.84/0.92 — compared to CosyVoice2.0 baseline: 0.92/0.80, 0.52/0.84, 0.76/0.80. DiffRO-MTR also outperforms F5-TTS and GPT-SoVITS in most emotion categories. Notably, the LM learns to generate laughter, sobs, and breaths to convey emotion — without any emotion-labeled training data in the RL phase.

**MOS control:** DiffRO can shift the codec-level predicted MOS toward a target (MOS_t=2: codec MOS drops to 2.2 from 3.82 baseline; MOS_t=4: codec MOS rises to 3.96). However, the FM and vocoder largely denoise the output, so audio MOS change is modest (3.77–3.86), highlighting that quality is mostly determined by post-LM stages.

**Token2Reward model accuracy:** The MTR model outperforms domain-specific models on SQA (LCC 0.65 vs DNSMOS 0.61) and Age/Gender (65.8% vs Qwen-Audio 58.8% on Age), though ASR WER is slightly worse than SenseVoice (7.7% vs 7.59% zh).

## Novelty Assessment

The Gumbel-Softmax trick applied to codec token sampling to make RLHF fully differentiable is the most technically novel contribution — it eliminates the PPO/DPO loop for the LM optimization step. Computing rewards directly on codec tokens (bypassing FM+vocoder) is a practical speedup that also sidesteps the diversity problem. The MTR model as a multi-dimensional reward is a natural extension but requires substantial training infrastructure. The emergent multilingual generalization and learned paralinguistic vocalizations (laughter, breaths) from the emotion reward are interesting byproducts. The approach is built on and validated with CosyVoice 2.0 ([[2412.10117]]) specifically.

## Limitations and Open Questions

DiffRO only optimizes the LM component; the FM and vocoder are frozen, which limits the impact of rewards that primarily affect post-LM processing (e.g., speaker age/gender, which are partly determined by the FM). The MTR reward model requires a large in-house multi-task labeled corpus (13,000+ hours) that is not publicly available, limiting reproducibility. The approach is validated only on Mandarin-focused data; generalization to languages with distinct phonological properties (tonal, agglutinative) needs further study. Constructing diverse RL training data remains nontrivial.

## Wiki Connections

DiffRO is a direct contribution to [[rlhf-speech]], advancing the state of the art beyond DPO for codec LM-based TTS. It builds on [[autoregressive-codec-tts]] — specifically CosyVoice 2.0 ([[2412.10117]]) — as the base system. The emotion control experiment connects to [[emotion-synthesis]]. The MTR model draws on ideas from [[evaluation-metrics]] by operationalizing multiple evaluation dimensions as differentiable rewards. SeedTTS ([[2406.02430]]) provides the seed-tts-eval benchmark used for evaluation. F5-TTS ([[2410.06885]]) is included as an emotion control baseline.
