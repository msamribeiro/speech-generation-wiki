---
slug: fine-tuning
title: Fine-Tuning Foundation Models for Speech Generation
aliases: [parameter-efficient fine-tuning, PEFT, LoRA for TTS, adapter fine-tuning, model adaptation]
status: established
related_concepts: [speaker-adaptation, multilingual-tts, zero-shot-tts, self-supervised-speech, autoregressive-codec-tts, flow-matching]
last_updated: 2026-06-05
---

## Executive Summary

> [!abstract]
> Fine-tuning adapts a pre-trained speech generation model to a new task, language, domain, or control capability by updating a subset of its parameters on a small targeted dataset. In 2025 the dominant pattern is parameter-efficient fine-tuning (PEFT) — LoRA, adapters, and prefix methods — applied to large TTS and VC foundation models, which reduces resource cost and, crucially, preserves the pre-trained model's zero-shot and multi-speaker capabilities that full fine-tuning destroys.

## Current Status

established — Fine-tuning is the default training strategy whenever a task-specific behaviour must be added to an existing foundation model without retraining from scratch. With models like CosyVoice 2, F5-TTS, and VALL-E growing to hundreds of millions of parameters, full fine-tuning is computationally expensive and risks catastrophic forgetting of pre-trained capabilities. The field has converged on PEFT as the practical standard for cross-lingual adaptation, pronunciation correction, and style/domain specialization in 2025.

## Why This Matters

Pre-trained TTS and VC models encode large amounts of general acoustic and linguistic knowledge. Retraining this knowledge from scratch for every new language, speaker type, or control dimension is wasteful and often impractical in low-resource settings. Fine-tuning — especially PEFT — allows practitioners to add targeted capabilities (e.g., Japanese pitch accent correction, Korean language support) with a few hours of data and a single GPU while keeping all other capabilities intact. This is particularly important for zero-shot multi-speaker models, where catastrophic forgetting would eliminate the ability to generalise to unseen speakers.

## Core Idea

Fine-tuning updates a subset of a pre-trained model's parameters on a small task-specific dataset. The spectrum runs from:

- **Full fine-tuning**: all parameters updated; highest plasticity, highest forgetting risk, highest compute cost.
- **Adapter-based PEFT**: small bottleneck modules inserted between frozen layers; only adapter parameters updated.
- **LoRA (Low-Rank Adaptation)**: weight updates are constrained to a low-rank decomposition of the update matrix; inserted at specific projection layers; controllable rank trades off adaptation quality against generalisation preservation.
- **Prefix/prompt tuning**: learnable token prefixes prepended to inputs; no weight updates to the base model.

In speech generation, PEFT targets are typically the text encoder (for language/phoneme adaptation), attention projection layers in the acoustic model (for speaker or style adaptation), and joint embedding layers (for prompt-based conditioning). The choice of which modules to adapt — and at what compression level — is empirically tuned and depends on the linguistic and acoustic distance between source and target domains.

## Methods and Variants

**LoRA for task-specific pronunciation control.** [[2508.09767]] (UtterTune) applies rank-16 LoRA to the AR LM of CosyVoice 2 alongside two phoneme-tag special tokens that flag pitch accent patterns in Japanese. Only 0.5% of the model's 500M parameters are updated. Japanese accent correctness improves from 0.499 to 0.899 while MOS rises from 3.44 to 3.88, demonstrating that LoRA can insert narrow task-specific capabilities with minimal impact on general synthesis quality.

**Three-adapter PEFT for cross-lingual continual adaptation.** [[interspeech-2025-1344]] adapts F5-TTS (335.8M params) to Korean using three targeted adapter modules — a Conditioning Adapter on the text encoder's depth-wise convolution (compression 0.25), a Prompt Adapter with LoRA + DropPath on the text-audio joint embedding, and a DiT LoRA (rank 16) on the acoustic transformer — totalling 1.72% of parameters (5.81M). Trained on 12.65 hours of single-speaker Korean data on a single GPU. The critical finding is that full fine-tuning destroys zero-shot capability for previously learned languages entirely, while PEFT preserves it. This makes PEFT not merely computationally convenient but functionally necessary for continual cross-lingual adaptation.

# TODO: expand with domain adaptation (child speech, expressive TTS), PEFT for SCA/LLM-speech, prefix tuning, instruction fine-tuning

## Major Claims

Claims are generalised propositions aggregated from paper evidence. The full claim registry with supporting paper lists is in `wiki/concepts/_evidence/fine-tuning.yaml`.

### Strongly Supported

- Full fine-tuning of multilingual TTS models on single-language, single-speaker data destroys zero-shot multi-speaker capability for previously learned languages; PEFT (LoRA, adapters) preserves it.
  Supporting: [[interspeech-2025-1344]]

- LoRA fine-tuning with task-specific special tokens can insert narrow pronunciation control capabilities into pre-trained multilingual TTS systems at less than 0.5% of model parameters.
  Supporting: [[2508.09767]]

### Emerging

- In flow-matching TTS, the text conditioning module requires more adapter capacity (lower compression, more parameters) to adapt to a typologically distant language than the acoustic generation module.
  Supporting: [[interspeech-2025-1344]]

- Low-rank adaptation in AR TTS models exhibits a rank-dependent trade-off between task-specific accuracy and generalisation to pre-trained capabilities; optimal rank decreases as dataset scale decreases.
  Supporting: [[interspeech-2025-1344]], [[2508.09767]]

## Relationship to Other Concepts

### Extends or Builds On
- [[self-supervised-speech]] — SSL representations (HuBERT, WavLM, CosyVoice internal VQ) serve as the frozen pre-trained backbone that fine-tuning builds upon; the quality of the SSL representation is the ceiling for fine-tuned performance.
- [[autoregressive-codec-tts]] — many fine-tuned models in 2025 are AR codec LMs (VALL-E family, CosyVoice 2); PEFT for AR LMs is an established pattern borrowed from NLP (LoRA for LLaMA, GPT).
- [[flow-matching]] — flow-matching TTS systems (F5-TTS, Voicebox) are increasingly the fine-tuning target for cross-lingual and speaker adaptation.

### Competes With
- [[zero-shot-tts]] — zero-shot inference eliminates the need for fine-tuning when a reference clip provides sufficient conditioning; fine-tuning remains necessary only when zero-shot quality falls short for unusual speakers, languages, or prosodic targets.

### Commonly Paired With
- [[speaker-adaptation]] — speaker adaptation is a common *purpose* of fine-tuning; many speaker adaptation papers use LoRA or adapter fine-tuning as the *method*. The two concepts are closely related but distinct: speaker-adaptation focuses on the target (speaker identity), fine-tuning focuses on the training procedure.
- [[multilingual-tts]] — cross-lingual language extension is the dominant use case for TTS fine-tuning in 2025; nearly all multilingual TTS papers that add a new low-resource language do so via fine-tuning.

## Representative Papers

### Influential
- [[interspeech-2025-1344]] — demonstrates that three-adapter PEFT (1.72% of parameters) adapts F5-TTS to Korean with 12.65 hours of data while fully preserving zero-shot capability; establishes PEFT as functionally necessary, not just efficient, for continual cross-lingual TTS adaptation.
- [[2508.09767]] — UtterTune: LoRA at <0.5% parameters with phoneme-tag tokens corrects Japanese pitch accent in CosyVoice 2 without degrading general synthesis quality; exemplifies narrow task-specific PEFT.

## Open Questions

- At what data scale does full fine-tuning outperform PEFT for TTS, if ever? Current evidence suggests PEFT is preferable even at moderate data scales when zero-shot capability preservation matters.
- Does the optimal PEFT module selection (which layers to adapt, which to freeze) transfer across model architectures (AR LM vs. flow-matching vs. diffusion)?
- Can instruction fine-tuning (natural language style prompts) be applied to TTS models the same way it has transformed LLMs, enabling new capabilities without task-specific data?
- How does PEFT interact with post-training (RLHF/GRPO)? Can PEFT-adapted models be further improved via preference optimization without undoing the adaptation?
- What is the minimum dataset size for reliable LoRA-based cross-lingual TTS adaptation across language families with varying degrees of phonological distance from the pre-training distribution?

## Trend Summary

2025: Fine-tuning has shifted from full parameter updates to targeted PEFT as the default approach. The key driver is preservation of pre-trained zero-shot capability — a concern unique to speech generation models (unlike pure classification fine-tuning), where generalization to unseen speakers is a primary product feature. Two representative papers from August 2025 establish the practical pattern: [[2508.09767]] (UtterTune) shows LoRA at <0.5% of parameters for pronunciation control, and [[interspeech-2025-1344]] (PEFT-TTS) shows three-adapter PEFT for cross-lingual adaptation at 1.72% of parameters with a single GPU. Both share the finding that the parameter budget is not the limiting factor; the module selection (which layers to adapt) and regularization (DropPath, rank selection) are the design choices that matter most.

## All Papers

| ID | Title | Venue | Year | Key use of this concept |
|----|-------|-------|------|------------------------|
| [[2508.09767]] | UtterTune: LoRA-Based Target-Language Pronunciation Edit and Control in Multilingual Text-to-Speech | arXiv | 2025 | Rank-16 LoRA with phoneme-tag tokens on CosyVoice 2 for Japanese pitch accent; <0.5% trainable params; accent correctness 0.499 → 0.899 |
| [[interspeech-2025-1344]] | Parameter-Efficient Fine-Tuning for Low-Resource TTS via Cross-Lingual Continual Learning | Interspeech | 2025 | Three-adapter PEFT (1.72%) adapts F5-TTS to Korean on 12.65h; full fine-tuning destroys zero-shot capability while PEFT preserves it |
| [[2508.18006]] | Unseen Speaker and Language Adaptation for Lightweight TTS with Adapters | arXiv | 2025 | Bottleneck acoustic + convolutional vocoder adapters (10% params) for language and speaker adaptation; outperforms full fine-tuning on naturalness and speaker similarity |
