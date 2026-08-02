---
id: "neurips-2025-RTjr4DnS79"
title: "Metis: A Foundation Speech Generation Model with Masked Generative Pre-training"
authors: ["Yuancheng Wang", "Jiachen Zheng", "Junan Zhang", "Xueyao Zhang", "Huan Liao", "Zhizheng Wu"]
organization: "The Chinese University of Hong Kong, Shenzhen"
venue: NeurIPS
venue_type: conference
year: 2025
month: 12
published_date: "2025-12-10"
ingested_date: "2026-08-02"
task: [TTS, VC]
architecture: [diffusion]
conditioning: [text-conditioned, speaker-conditioned, zero-shot, prompt-conditioned, multilingual]
training: [self-supervised, supervised, fine-tuning]
model_size: "pre-trained backbone size not reported; fine-tuned adaptations use 2-32M trainable LoRA parameters (task-dependent) or full fine-tuning"
codec_used: "own two-stage discrete representation: VQ-quantized SSL tokens (semantic) + VQ acoustic tokens (waveform reconstruction), not an existing off-the-shelf codec"
datasets_train: ["Emilia (100K hrs) + self-collected via Emilia pipeline (200K hrs), 300K hrs total pre-training", "LibriTTS", "VCTK", "LibriMix", "DNS-Challenge 2020", "LRW/LRS2/LRS3"]
datasets_eval: ["SeedTTS test-en", "SeedTTS test-zh", "LibriSpeech test-clean", "VCTK", "LibriMix test", "EmiliaMix test", "DNS-Challenge 2020 test sets", "LRS2", "LRS3"]
metrics:
  - name: WER
    value: 2.28
    system: "Metis-TTS fine-tune (10K hrs)"
    testset: "SeedTTS test-en"
  - name: SPK-SIM
    value: 0.72
    system: "Metis-TTS LoRA 32 (1K hrs)"
    testset: "SeedTTS test-en"
  - name: DNSMOS
    value: 3.47
    system: "Metis-TTS LoRA 32 (1K hrs)"
    testset: "SeedTTS test-en"
code_available: true
demo_available: true
url: "https://openreview.net/forum?id=RTjr4DnS79"
related_concepts: [zero-shot-tts, voice-conversion, self-supervised-speech, multilingual-tts]
related_papers: ["2301.02111", "2406.02430", "2407.05407"]
field_significance:
  level: "foundational"
  type: [architectural-novelty]
generation:
  schema_version: 2
  date: "2026-08-02"
  runtime: "claude-code"
  provider: "anthropic"
  agent: speech-generation-ingest-agent
  model: "claude-sonnet-5"
  commit: "b6e27f4"
---

> [!abstract] NeurIPS · 2025 · Conference
> **Wang et al.** (The Chinese University of Hong Kong, Shenzhen) · [→ Paper](https://openreview.net/forum?id=RTjr4DnS79) · Demo: ✓ · Code: ✓
>
> Pre-trains a single masked generative model unconditionally on 300K hours of discrete SSL speech tokens, then fine-tunes it with lightweight, task-specific conditions to reach state-of-the-art results across five distinct speech generation tasks, including a cross-modal lip-to-speech task, using far less data and far fewer trainable parameters than task-specific baselines.

## Problem

Most speech generation systems are either narrow experts built for a single task, or multi-task models (UniAudio, SpeechX) that use autoregressive language modeling over paired condition-target data for every task they support. The latter approach requires large amounts of task-specific paired data, generalizes poorly to new tasks without retraining, and inherits the inefficiency and occasionally suboptimal quality of purely autoregressive generation. Meanwhile, self-supervised pre-training on unlabeled data has driven large gains in NLP and vision by decoupling general-purpose representation learning from task-specific adaptation, but this recipe remained underexplored for speech generation specifically (as opposed to speech understanding).

## Method

Metis reframes speech generation as a common two-stage process shared across tasks: first generating discrete SSL tokens (which carry semantic and prosodic information) from task-specific conditions, then generating acoustic tokens (for high-fidelity waveform reconstruction) from those SSL tokens. The paper's core contribution targets the first stage: a masked generative model (a bidirectional-attention, Llama-architecture Transformer, following MaskGCT's backbone but without a text embedding) is pre-trained unconditionally, with no task-specific condition at all, on SSL tokens from 300K hours of diverse, multilingual, in-the-wild speech (100K hours from Emilia plus 200K hours self-collected via the same pipeline, spanning Chinese, English, German, French, Japanese, and Korean). Training uses standard masked-token-prediction: a Bernoulli mask schedule replaces a subset of tokens with a [MASK] token, and the model is trained to predict the full sequence conditioned on unmasked tokens; a randomly-sampled unmasked prefix is retained as an in-context prompt with probability 0.8 to encourage prompt-conditioned generalization useful for zero-shot tasks.

Once pre-trained, the model adapts to a specific task by fine-tuning with that task's condition injected as additional input: non-frame-level conditions (e.g., text/phonemes for TTS) are concatenated along the time dimension, while frame-level conditions (e.g., source speech for voice conversion, noisy speech for enhancement, visual features for lip-to-speech) are time-aligned via interpolation and an MLP adapter before being added to the input. Fine-tuning can be full-parameter or LoRA-based (as few as 2-32M trainable parameters depending on rank and task). A second, separately trained masked generative model handles the task-independent second stage, predicting acoustic tokens layer-by-layer conditioned on the SSL tokens and a prompt, serving as a shared acoustic decoder across all downstream tasks.

![An illustration of Metis. (a) provides an overview of the two-stage speech generation framework, which consists of task-specific (yellow block) and task-independent (light blue block) processes. In this work, we focus on developing a pre-training model for the first stage, as illustrated in (b). (c) demonstrates the fine-tuning process, where the pre-trained model is adapted to specific tasks.](assets/neurips-2025-RTjr4DnS79/figure-1.png)

## Key Results

Across five fine-tuned tasks, Metis matches or exceeds task-specific and multi-task state-of-the-art baselines while using dramatically less data and far fewer trainable parameters. On zero-shot TTS, Metis-TTS fine-tuned on just 10K hours (versus MaskGCT's 100K hours) achieves a better WER on SeedTTS test-en (2.28 vs. 2.47) with comparable similarity and quality, and even a 32M-parameter LoRA variant fine-tuned on only 1K hours is competitive with baselines trained on 10-100x more data. On voice conversion (VCTK), Metis-VC achieves the highest speaker similarity of any compared system (0.55 vs. the best baseline's 0.38) using only 400 hours of fine-tuning data. On target speaker extraction, Metis-TSE achieves state-of-the-art audio-quality (NISQA) scores that exceed even the ground-truth recordings' scores on both test sets. On speech enhancement, Metis-SE achieves the best scores across all DNS-Challenge 2020 test conditions (with reverb, without reverb, and real recordings), including with a lightweight 2M-parameter LoRA variant. On the cross-modal lip-to-speech task, Metis-L2S nearly doubles speaker similarity over a dedicated expert baseline (29.34 to 59.73 on LRS2) while also improving WER and audio quality. A jointly fine-tuned multi-task variant, Metis-Omni, further generalizes to a task combination never explicitly trained, text-guided target speaker extraction, reducing WER to 2.70.

## Novelty Assessment

The central idea, decoupling unconditional generative pre-training (learning to recover masked SSL tokens from context and prompts alone) from task-specific fine-tuning, is a direct and effective adaptation of the pre-train/fine-tune paradigm from NLP and vision to speech generation specifically, distinguishing Metis from prior speech foundation-style models (UniAudio, SpeechX, VoiceBox) that either require paired data per task or remain limited to frame-level conditions during pre-training (as in SpeechFlow, the paper's closest prior comparison). The empirical demonstration that a single unconditional pre-training run can be efficiently specialized, including to a genuinely cross-modal task (lip-to-speech) never seen during pre-training, is the paper's strongest evidence that the pre-training captures generically useful structure rather than task-specific shortcuts.

## Field Significance

> [!important] foundational — Demonstrates, with extensive quantitative validation across five distinct tasks (including a cross-modal one) and against strong dedicated baselines, that a single task-agnostic pre-training objective can serve as a genuine foundation for the broader speech generation problem, not just for TTS variants.

The combination of state-of-the-art or near-state-of-the-art results with an order-of-magnitude-or-more reduction in both fine-tuning data and trainable parameters, plus emergent generalization to untrained task combinations in the multi-task variant, indicates the pre-training is capturing reusable structure rather than being a narrow architectural tweak.

## Claims

- **supports:** Unconditional masked generative pre-training on discrete SSL tokens, with no task-specific condition at all, can serve as a general-purpose foundation for diverse downstream speech generation tasks, requiring only lightweight fine-tuning to specialize.
  > *Evidence:* A single pre-trained backbone, fine-tuned separately with task-specific conditions, achieves competitive or state-of-the-art results across five distinct tasks (zero-shot TTS, voice conversion, target speaker extraction, speech enhancement, lip-to-speech) using as few as 9-32M trainable LoRA parameters per task. *(§4.2, Tables 1-5)*
- **supports:** Pre-training a speech generation backbone on large-scale unlabeled data before task-specific fine-tuning yields both faster convergence and better final performance than training the same architecture from scratch on the same fine-tuning data.
  > *Evidence:* On zero-shot TTS with 10K hours of fine-tuning data, the pre-trained model achieves WER 2.28 versus 4.91 for the same architecture trained from scratch for the same number of steps without pre-training. *(§4.2.1, Table 1)*
- **supports:** A speech generation foundation model can be efficiently fine-tuned into a genuinely cross-modal task, visual speech input to audio output, using the same masked generative mechanism designed for audio-only conditions, without architectural changes.
  > *Evidence:* Fine-tuning the pre-trained model on lip-to-speech nearly doubles speaker similarity over a dedicated expert lip-to-speech baseline (29.34 to 59.73 on LRS2, 32.05 to 56.74 on LRS3) while also improving WER and audio quality. *(§4.2.5, Table 5)*
- **supports:** Jointly fine-tuning a pre-trained speech generation backbone on multiple tasks can produce emergent generalization to task combinations the model was never explicitly trained on.
  > *Evidence:* Metis-Omni, fine-tuned jointly on four tasks, generalizes to text-guided target speaker extraction, a combination absent from its training data, reducing WER to 2.70, a substantial improvement over all other compared models. *(§4.3, Table 6)*
- **complicates:** Masked generative pre-training without any semantic condition captures prosodic style and speaker identity from a prompt but does not by itself guarantee intelligible content, so task-specific conditioning remains necessary even after large-scale pre-training.
  > *Evidence:* The unconditional pre-trained model mimics a prompt's prosodic style and timbre but "often lacks intelligibility, producing random word concatenations due to the absence of semantic guidance," motivating fine-tuning with task-specific conditions. *(§3.4)*

## Limitations and Open Questions

The paper does not report the total parameter count of the pre-trained backbone itself (only fine-tuning-stage trainable-parameter counts), making it harder to assess the model's overall scale relative to baselines beyond the reported data and trainable-parameter reductions. Several task-specific fine-tuning setups rely on auxiliary tools not evaluated as part of Metis itself, for instance, voice conversion timbre perturbation depends on a separate lightweight VC model, and speech enhancement noise simulation depends on external RIR and noise datasets, so some of Metis's downstream performance is coupled to these auxiliary components. Subjective evaluation (CMOS/SMOS) results are deferred to the appendix rather than presented as part of the core results.

## Wiki Connections

- [[zero-shot-tts|Zero-Shot TTS]] — fine-tunes the pre-trained backbone for zero-shot text-to-speech, matching or exceeding baselines trained on 10-100x more data.
- [[voice-conversion|Voice Conversion]] — fine-tunes the same backbone for zero-shot voice conversion using a lightweight timbre-perturbation strategy, achieving the highest speaker similarity among compared systems.
- [[self-supervised-speech|Self-Supervised Speech]] — pre-trains unconditionally via masked token prediction on discrete tokens derived from self-supervised speech features, with no task-specific supervision at pre-training time.
- [[multilingual-tts|Multilingual TTS]] — pre-trains on speech spanning six languages and evaluates zero-shot TTS on both English and Chinese test sets with per-language metrics.
- [[2301.02111|VALL-E]] — used as a zero-shot TTS baseline in the SeedTTS test-en comparison.
- [[2406.02430|Seed-TTS]] — its SeedTTS test-en and test-zh test sets are used as the primary zero-shot TTS evaluation benchmarks.
- [[2407.05407|CosyVoice]] — used as a zero-shot TTS baseline in both SeedTTS test-en and test-zh comparisons.
