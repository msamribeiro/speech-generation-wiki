---
id: "interspeech-2025-0166"
title: "Frozen Large Language Models Can Perceive Paralinguistic Aspects of Speech"
authors: ["Wonjune Kang", "Junteng Jia", "Chunyang Wu", "Wei Zhou", "Egor Lakomkin", "Yashesh Gaur", "Leda Sari", "Suyoun Kim", "Ke Li", "Jay Mahadeokar", "Ozlem Kalinli"]
organization: null
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-07-02
task: ["SCA"]
architecture: ["transformer-enc-dec"]
conditioning: ["emotion-conditioned", "prompt-conditioned"]
training: ["supervised", "self-supervised", "fine-tuning"]
model_size: "1B params (Conformer encoder) + 100M adapter + Llama 3 8B (frozen)"
codec_used: "none"
datasets_train: ["proprietary (15M hours multilingual ASR pre-training)", "proprietary (230K hrs ASR + 90K hrs AST + 25K hrs spoken dialogue)", "proprietary (expressive speech ~2492 hrs, 15 emotion/style classes)"]
datasets_eval: ["proprietary (2157 samples, 2 held-out voice actor speakers)"]
metrics:
  - name: ROUGE-1
    value: 0.446
    system: SpeechEmotionLlama v3
    testset: proprietary test set
  - name: ROUGE-L
    value: 0.318
    system: SpeechEmotionLlama v3
    testset: proprietary test set
code_available: false
demo_available: false
url: "https://www.isca-archive.org/interspeech_2025/kang25_interspeech.html"
related_concepts: [spoken-language-model, self-supervised-speech, emotion-synthesis, evaluation-metrics]
related_papers: []
field_significance:
  level: moderate
  type: ["architectural-novelty"]
generation:
  date: 2026-07-02
  agent: speech-generation-ingest-agent
  model: claude-sonnet-4-6
  commit: "d8ecfd4"
---

> [!abstract] Interspeech · 2025 · Conference
> **Kang et al.** (MIT, Meta) · [→ Paper](https://www.isca-archive.org/interspeech_2025/kang25_interspeech.html) · Demo: ✗ · Code: ✗
>
> A training framework called behavioral alignment against emotion-conditioned responses enables a speech encoder to convey paralinguistic information to a completely frozen LLM, producing more empathetic spoken dialogue responses without fine-tuning the LLM's weights.

## Problem

Prior work on augmenting LLMs with speech understanding either (a) specialized models for narrow tasks and lost the base LLM's zero-shot capabilities, or (b) used modality adapters that convey only linguistic content, ignoring emotion and speaking style. Attempts to add paralinguistic understanding to the second category universally required fine-tuning the LLM via LoRA or similar methods, which complicates deployment and risks degrading the base model's general capabilities. The question this paper addresses is whether a frozen LLM can be made to understand paralinguistic cues through an appropriately trained speech encoder alone, without any modification to the LLM's weights.

## Method

The system combines a large frozen LLM (Llama 3 8B Instruct) with a trainable speech encoder. The encoder is a 24-layer Conformer (1B parameters) that takes 80-dim mel-spectrograms as input. Every four frames are stacked for 4x temporal compression, and the output passes through a 100M-parameter adapter (convolution + rotary Transformer + linear projection) that further compresses to one speech token every 80ms and maps to the LLM's 4096-dim embedding space. The Conformer is first pre-trained on approximately 15M hours of multilingual speech using BEST-RQ (a self-supervised random-projection quantization objective). It is then fine-tuned on 230K hours of ASR, 90K hours of speech translation, and 25K hours of spoken dialogue data to align speech tokens with their paired text representations, using causal language modeling loss computed only over the LLM's response tokens.

The core novelty is an additional fine-tuning stage using proprietary expressive speech data (approximately 2,492 hours across 15 emotion/style classes). Here, ground-truth LLM responses are generated from text prompts that include an emotion or style tag, so the LLM already responds empathetically to the text. The speech encoder is then trained to make the LLM produce those same empathetic responses when given only the raw speech input (no emotion tag at inference). This forces the encoder's token embeddings to capture and convey paralinguistic information implicitly. Four alignment tasks are combined: emotion-conditioned response generation, ASR, speech captioning, and speech emotion recognition (SER). The resulting system is called SpeechEmotionLlama.

![Behavioral alignment against emotion-conditioned responses. The speech encoder is trained to produce tokens such that the LLM's response to an expressive speech prompt matches its response to a semantically paired text prompt where the speaker's emotion or speaking style has also been specified.](assets/interspeech-2025-0166/figure-1.png)

## Key Results

On a proprietary test set of 2,157 expressive speech samples from two held-out voice actor speakers, SpeechEmotionLlama v3 achieves ROUGE-L 0.318 and BERTScore 94.79, compared to 0.250 and 94.78 for the fine-tuned non-emotion-conditioned baseline (Baseline 2). The most notable improvement is in LLM-annotated emotion understanding score: SpeechEmotionLlama v3 scores 7.54 vs. 5.83 for the best cascaded system (SER model prepended to Baseline 2), a gain of over 1.7 points on a 1-10 scale.

Cascading a standalone SER model (80.81% 15-way accuracy) on top of either baseline yields only marginal gains in emotion understanding (Baseline 2: 5.59 to 5.83), underscoring that cascaded emotion tags do not transfer paralinguistic context as effectively as encoder-level alignment.

SpeechEmotionLlama v3 achieves 81.51% on the SER task when prompted directly, slightly exceeding the standalone cascade SER model (80.81%), demonstrating that the proposed training makes the encoder's token embeddings genuinely discriminative for emotion. Linear probes on mean-pooled tokens confirm this: 78.40% for SpeechEmotionLlama v3 vs. 31.20% for the fine-tuned baseline.

All training and evaluation data are proprietary, which limits independent reproducibility and external comparability.

## Novelty Assessment

The behavioral alignment approach itself adapts a paradigm from prior frozen-LLM speech work (Fathullah et al., BLSP) and extends it to the paralinguistic domain. The genuine novelty is showing that this extension can be done without any LLM weight modification, which prior paralinguistic speech-LLM work did not achieve. The technique of generating emotion-conditioned ground-truth responses and aligning speech encoder outputs against them is a simple but effective idea. The speech encoder architecture is a large industrial-scale Conformer (1B parameters + 100M adapter), pre-trained on 15M hours; this scale is a non-trivial ingredient that is not available to academic reproducers. The evaluation is conducted entirely on proprietary data, so the generalization boundary is difficult to assess. The gains are meaningful in absolute terms (1.7+ points in LLM-judged empathy), but comparisons are limited to in-house baselines.

## Field Significance

Moderate — This paper establishes that paralinguistic perception (emotion, speaking style) can be induced in a frozen LLM through encoder-side behavioral alignment, providing a cleaner deployment story than LoRA-based alternatives. The contribution is primarily methodological within the spoken conversational agent space, with the caveat that scale (1B encoder, 15M hrs pre-training, proprietary data) plays a significant role in the outcome that is not fully separable from the training framework.

## Claims

- **supports:** Training a speech encoder to match LLM responses conditioned on emotion labels can transfer paralinguistic understanding to a frozen LLM without any LLM weight modification.
  > *Evidence:* SpeechEmotionLlama achieves an LLM-annotated emotion understanding score of 7.54 versus 5.83 for the best cascaded baseline, and 81.51% SER accuracy versus 80.81% for the standalone SER model, while keeping Llama 3 8B entirely frozen throughout training. *(§2.3, §4.1, §4.2, Table 1)*

- **complicates:** Cascading a high-accuracy standalone emotion classifier onto a speech-LLM system provides only marginal improvement in emotional response quality.
  > *Evidence:* Prepending SER model predictions (80.81% accuracy) to Baseline 2 raises the LLM-annotated emotion score from 5.59 to only 5.83, while encoder-level alignment raises it to 7.54, suggesting that discretized emotion tags do not capture the richness of paralinguistic information as effectively as continuous encoder embeddings. *(§3.2, §4.1, Table 1)*

- **complicates:** Evaluating paralinguistic speech-LLM systems is inherently difficult when training and evaluation data are fully proprietary.
  > *Evidence:* All datasets (pre-training, fine-tuning, and test sets) are proprietary, precluding external replication; the LLM-based evaluation metric (Llama 3 70B as judge) also introduces a dependency on the evaluator model's own behavior, which is not systematically validated against human listeners. *(§3.1, §3.3)*

- **supports:** Self-supervised speech pre-training on large-scale multilingual data provides a strong initialization for downstream paralinguistic understanding in speech encoders.
  > *Evidence:* The Conformer encoder is pre-trained with BEST-RQ on approximately 15M hours before fine-tuning on emotion tasks; the system substantially outperforms baselines that use a speech encoder that is not further fine-tuned on emotion-related tasks, indicating the pre-training provides useful representations that emotion-specific fine-tuning can build upon. *(§2.2, §3.2, §4.2)*

## Limitations and Open Questions

> [!warning]
> All training and evaluation data are proprietary; results cannot be replicated externally and comparisons are restricted to in-house baselines. The role of data scale (1B-parameter encoder, 15M hours pre-training) versus the training paradigm itself is not ablated.

The system is evaluated only on English expressive speech from voice actors and a large but single-language corpus, leaving generalization to other languages, accents, and naturalistic (non-acted) expressive speech open. The test set is intentionally constructed to control for linguistic content by using utterances where the same sentence is spoken in multiple styles, which may not reflect realistic distribution. LLM-judged metrics (Llama 3 70B as scorer) have not been validated against human listener judgments for empathy or response quality in this setup.

## Wiki Connections

- [[spoken-language-model|Spoken Language Model]] — this paper extends the spoken LM paradigm by training speech encoders to convey paralinguistic (not just linguistic) content to a frozen LLM, demonstrating that SCA interaction quality depends on more than transcription fidelity.
- [[self-supervised-speech|Self-Supervised Speech]] — the Conformer encoder is pre-trained with BEST-RQ, a random-projection quantization SSL objective, on 15M hours before task-specific fine-tuning; SSL pre-training is a core component of the system's ability to produce general speech representations.
- [[emotion-synthesis|Emotion Synthesis]] — while the paper targets emotion perception rather than synthesis, the behavioral alignment framework is directly dependent on emotion/style labels and emotion-conditioned response generation, informing how emotion is represented and transferred across modalities.
- [[evaluation-metrics|Evaluation Metrics]] — the paper uses LLM-as-judge scoring (Llama 3 70B) alongside ROUGE and BERTScore for evaluating empathetic response quality, raising questions about evaluator model bias and the absence of human listening tests.
