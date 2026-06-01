---
slug: spoken-language-model
title: Spoken Language Model
aliases: [speech LM, SpeechGPT, spoken dialogue system, SCA, speech foundation model, audio LLM, SLM, end-to-end spoken dialogue]
related_concepts: [autoregressive-codec-tts, neural-codec, streaming-tts, rlhf-speech, instruction-conditioned-tts, self-supervised-speech]
last_updated: 2026-05-30
---
## What it is

A spoken language model (SLM) is a language model that operates directly on discrete speech tokens rather than text tokens, enabling speech-to-speech generation (or speech understanding) without text as an intermediate representation. SLMs process audio input and produce speech output without a TTS bridge, allowing them to model paralinguistic information (prosody, emotion, speaking style, non-verbal cues) that is lost when passing through text.

The input is discretized speech (typically from a neural audio codec or self-supervised model like HuBERT), and the model is trained to predict the next discrete speech token autoregressively. The model may also receive or produce text tokens, but in the purest form (GSLM, AudioLM), operates without text guidance.

In this wiki, SCA (spoken conversational agent) encompasses SLMs used in dialogue settings, including both purely speech-to-speech systems and hybrid architectures that use an intermediate text representation.

## Why it matters

SLMs are the pathway to natural, latency-efficient voice interaction that preserves the full richness of human speech — including prosody, emotion, non-verbal cues, and turn-taking dynamics. Cascade systems (ASR → text LLM → TTS) lose paralinguistic information and add latency at each stage. A true end-to-end SLM can model these phenomena natively. However, as [[2412.17048]] demonstrates, the gap between text LLMs and SLMs in semantic coherence is large and stems from fundamental properties of speech tokenization.

For interactive applications, SLMs must also handle fine-grained turn-taking dynamics: interruptions, backchannels, pauses, and gaps. The InteractSpeech dataset ([[2025.findings-emnlp.424]]) provides training and evaluation data specifically for these capabilities.

## Current state of the art

Real-time dialogue SLMs as of 2025: GPT-4o voice mode, Moshi (first fully duplex open-source system), Mini-Omni, and Qwen2.5-Omni represent the frontier. These systems largely use hybrid architectures with text LLM components rather than pure SLMs.

Pure SLMs (no text guidance): GSLM, AudioLM, TWIST, SpiritLM, and Align-SLM. These systems still underperform cascade systems on semantic coherence by a large margin. [[2412.17048]] provides a systematic analysis showing that paralinguistic variability in speech tokens (Factor C, primarily) and sequence length (Factor B) are the main causes, not the phonetic vs. semantic nature of speech tokens (Factor A). [[2025.acl-long.682]] surveys the full landscape of SpeechLM architectures, training recipes (pretraining, instruction tuning, RLHF), and evaluation benchmarks as of mid-2025 — the most comprehensive taxonomic reference in the corpus.

For training paradigm, [[2025.acl-long.388]] (DiVA) demonstrates that cross-modal context distillation from a frozen text LLM — using only ASR-paired data and without annotated instruction responses — can match or outperform SFT-trained speech LLMs. DiVA achieves a 72% user preference win rate against Qwen 2 Audio (trained with >100× more compute), establishing that instruction following can be transferred from text to speech via distillation rather than SFT annotation.

For data efficiency, [[2025.emnlp-main.989]] (VocalNet) demonstrates that multi-token prediction (MTP) for speech generation, combined with a simplified two-stage training (no ASR/TTS pre-training), matches or beats SLMs trained on 100–200× more data. VocalNet-8B achieves WER 3.56% and UTMOS 4.49 with ~6K hours training vs. competitors using 887K–1.4M hours.

For empathetic speech interaction, [[2025.emnlp-demos.70]] (OpenS2S) provides a fully open-source SLM with empathy modeling via an automated data construction pipeline and streaming interleaved decoding. It demonstrates competitive performance on URO-Bench emotion understanding tasks without requiring the large-scale pre-training data of closed-source competitors.

For interaction modeling, [[2025.findings-emnlp.424]] shows that with targeted fine-tuning on InteractSpeech, a model can achieve 73.4% accuracy on interactional event classification (vs. 54.2% for zero-shot GPT-4o), demonstrating that data quality matters significantly for interaction capabilities.

For codec efficiency, [[2510.00981]] (FlexiCodec) shows that 6.25 Hz AR tokens can drive TTS with competitive quality, suggesting the same approach could reduce SLM sequence lengths by 2–8× vs. current codecs.

## Key variants and sub-approaches

**Pure speech-to-speech LMs.** Models trained without text supervision: GSLM uses de-duplicated HuBERT tokens; AudioLM uses hierarchical semantic (SoundStream RVQ-1) and acoustic (RVQ-rest) tokens. These achieve naturalness but limited semantic coherence.

**Interleaved speech-text LMs.** SpiritLM interleaves speech and text tokens at word level; SpeechGPT uses speech-text-paired fine-tuning; TWIST initializes from text LLM weights. These leverage text LLM capabilities while partially retaining speech-level modeling. [[2025.acl-long.682]] surveys this full spectrum.

**Duplex systems.** Moshi uses a multi-stream transformer to simultaneously process and generate speech in real time. This requires low-latency codec tokens and streaming architecture.

**Aligned multimodal LLMs (speech encoder + text LLM + speech decoder).** Systems like VocalNet [[2025.emnlp-main.989]], LLaMA-Omni, Mini-Omni, and Qwen2.5-Omni use a frozen speech encoder, an adaptor, and a large text LLM backbone with a dedicated speech decoder. The LLM's text capabilities are preserved while adding speech I/O. MTP [[2025.emnlp-main.989]] improves the speech decoder's output quality without changing the LLM.

**Empathetic SLMs.** [[2025.emnlp-demos.70]] (OpenS2S) extends the aligned multimodal paradigm with empathy modeling via multi-stage training (speech understanding, speech generation, empathetic instruction tuning) and an automated pipeline for constructing empathetic speech dialogue data. Streaming interleaved decoding (M=4 LLM states → N=8 speech tokens per step) enables low-latency generation.

**Interaction-capable systems.** [[2025.findings-emnlp.424]] fine-tunes Qwen-2.5-Omni with GRPO on InteractSpeech to classify turn-taking events (backchannels, interruptions, gaps, pauses). LLaMA 3-8B fine-tuned on InteractSpeech text achieves 86% human agreement on interactive dialogue generation.

## Comparison to alternatives

**Cascade systems (ASR + text LLM + TTS).** Mature, high semantic coherence, but lose paralinguistic information and have additive latency (~500 ms+ for pipeline). All major commercial voice assistants currently use this approach. [[2412.17048]] essentially measures the semantic coherence gap that a pure SLM must close to match cascade systems.

**Pure TTS with LLM backend.** Not a true SLM — produces speech from text responses. Cannot model the full speech signal (no prosody preservation of input, no backchannel-based responses). Systems like Qwen2.5-Omni and Mini-Omni fall into this category.

## Year-on-year trajectory

2021: GSLM establishes that de-duplicated HuBERT tokens allow rudimentary speech language modeling. 2022: AudioLM adds acoustic conditioning for naturalness. 2023: TWIST, SpeechGPT bring text LLM initialization and fine-tuning. 2024: Moshi achieves real-time duplex operation; GPT-4o voice mode demonstrates commercial-scale capability. 2025: Systematic analysis ([[2412.17048]]) reveals Factor C (paralinguistic variability) as the dominant bottleneck. InteractSpeech ([[2025.findings-emnlp.424]]) provides targeted training data for interaction dynamics. DiVA ([[2025.acl-long.388]]) establishes cross-modal distillation as a data-efficient alternative to SFT. VocalNet ([[2025.emnlp-main.989]]) shows that MTP halves WER vs. NTP and ASR/TTS pretraining stages add cost without benefit. OpenS2S ([[2025.emnlp-demos.70]]) demonstrates fully open-source empathetic SLMs with automated data construction. LLaMA-Omni 2 [[2025.acl-long.912]] demonstrates that modular SpeechLMs can match native SpeechLMs at 200K synthetic samples vs. millions of hours — establishing a new data efficiency benchmark for real-time spoken chatbots, with the key insight that integrating a high-quality autoregressive streaming TTS module (CosyVoice 2 chunk-aware FM) closes the naturalness gap of prior NAR-based modular systems. [[2025.acl-long.682]] provides the first comprehensive survey of the field as of mid-2025.

## Open questions

- [[2412.17048]]: What is the most effective way to address Factor C (paralinguistic variability)? Is codec design the answer, or are explicit supervision/data augmentation strategies required?
- Can variable-length speech tokenization (as in FlexiCodec [[2510.00981]]) reduce the Factor B (sequence length) problem while maintaining resynthesis quality?
- What level of interaction modeling is needed for practical applications? The InteractSpeech ([[2025.findings-emnlp.424]]) classification benchmark provides a starting point but end-to-end duplex evaluation remains standardization-free.
- How much of the semantic coherence gap can be closed by scaling the SLM vs. improving the tokenization?
- Are pure end-to-end SLMs a necessary intermediate step, or will cascade architectures continue to dominate commercial systems?
- VocalNet [[2025.emnlp-main.989]] shows MTP improves speech decoding quality significantly; can MTP also improve paralinguistic modeling (prosody, emotion), or does it only help intelligibility?
- OpenS2S [[2025.emnlp-demos.70]] uses automated LLM+TTS data construction for empathy; what is the quality ceiling for this approach compared to human-annotated empathetic data?

## Papers

| ID | Title | Venue | Year | Key use of this concept |
|----|-------|-------|------|------------------------|
| [[2412.17048]] | Why Do Speech Language Models Fail to Generate Semantically Coherent Outputs? A Modality Evolving Perspective | arXiv (ICASSP 2026) | 2026 | Systematically isolates three factors causing SLM coherence failure; identifies paralinguistic variability as the dominant bottleneck |
| [[2025.findings-emnlp.424]] | InteractSpeech: A Speech Dialogue Interaction Corpus for Spoken Dialogue Model | EMNLP 2025 | 2025 | Provides 150h interaction-annotated dialogue corpus; demonstrates fine-grained turn-taking training and evaluation for duplex SDMs |
| [[2510.00981]] | FlexiCodec: A Dynamic Neural Audio Codec for Low Frame Rates | arXiv (ICLR 2026) | 2025 | Shows 6.25 Hz codec tokens with near-GT semantic preservation, enabling substantially shorter SLM input sequences while maintaining downstream TTS quality |
| [[2025.acl-long.388]] | Distilling an End-to-End Voice Assistant Without Instruction Training Data | ACL | 2025 | Proposes cross-modal context distillation from text LLM to speech using only ASR data; 72% user preference win rate vs. Qwen 2 Audio at >100× less training compute |
| [[2025.emnlp-main.989]] | VocalNet: Speech LLMs with Multi-Token Prediction for Faster and High-Quality Generation | EMNLP | 2025 | Introduces MTP for speech LLMs; eliminates ASR/TTS pretraining stages; VocalNet-8B matches Qwen2.5-Omni-class systems using ~6K hours vs. 887K–1.4M hours in competitors |
| [[2025.emnlp-demos.70]] | OpenS2S: Advancing Fully Open-Source End-to-End Empathetic Large Speech Language Model | EMNLP | 2025 | Fully open-source empathetic SLM; automated LLM+TTS data synthesis pipeline; streaming interleaved decoding for low-latency response; competitive on URO-Bench emotion understanding |
| [[2025.acl-long.682]] | Recent Advances in Speech Language Models: A Survey | ACL | 2025 | Comprehensive survey of SpeechLM taxonomy: tokenizer families, LM architectures, vocoders, training paradigms (pretraining, instruction tuning, RLHF), and evaluation benchmarks |
| [[2509.02020]] | FireRedTTS-2: Towards Long Conversational Speech Generation for Podcast and Chatbot | arXiv | 2025 | Autoregressive SLM for multi-speaker dialogue; interleaved text-speech input format with context-aware prosody; implicit emotion inference from chat history — positions AR TTS as a conversational SCA component |
| [[2025.acl-long.1252]] | Finding A Voice: Exploring the Potential of African American Dialect and Voice Generation for Chatbots | ACL | 2025 | Evaluates spoken chatbots using TTS synthesis in a specific demographic context; finds that prosodic persona (AA voice accent) outperforms lexical-syntactic style matching (AAE dialect text) for user acceptance — an evaluation-oriented SCA contribution |
| [[2025.acl-long.912]] | LLaMA-Omni 2: LLM-based Real-time Spoken Chatbot with Autoregressive Streaming Speech Synthesis | ACL | 2025 | Modular SpeechLM (0.5B–14B) coupling a Qwen2.5 LLM with a streaming AR TTS LM; gate-fused LLM hidden states + text embeddings; 582 ms latency; surpasses GLM-4-Voice and LLaMA-Omni on spoken QA, instruction following, and naturalness while training on only 200K synthetic samples |
| [[2025.acl-long.654]] | Language-Codec: Bridging Discrete Codec Representations and Speech Language Models | ACL | 2025 | Designs a codec specifically to improve speech language model training: MCRVQ redistribution reduces per-channel information entropy, improving downstream AR LM SPK-SIM by 14%; demonstrates the direct impact of codec design on SLM performance |
| [[2511.12347]] | VoiceCraft-X: Unifying Multilingual Voice-Cloning Speech Synthesis and Editing | EMNLP | 2025 | AR neural codec language model extended to 11 languages via Qwen3 LLM backbone; demonstrates that pretrained text LLM initialization enables multilingual in-context learning for both speech generation and editing |
| [[interspeech-2025-0310]] | Exploring the Effect of Segmentation and Vocabulary Size on Speech Tokenization for Speech LMs | Interspeech | 2025 | Systematic 64-configuration grid search over HuBERT K-means segmentation × vocabulary; N=80ms + K=16384 beats original GSLM-style tokenization on 5 SLU benchmarks with 50% data reduction |
| [[2025.findings-acl.1051]] | LLMVoX: Autoregressive Streaming Text-to-Speech Model for Any LLM | ACL | 2025 | 30M AR transformer decoupled from any base LLM; multi-queue streaming scheduler processes LLM text tokens concurrently with speech token decoding; achieves 475ms end-to-end latency and WER 3.70% |
| [[2507.09318]] | ZipVoice-Dialog: Non-Autoregressive Spoken Dialogue Generation with Flow Matching | arXiv | 2026 | Extends FM TTS to spoken dialogue with learnable speaker-turn embeddings; 15× faster than AR baselines (Dia 1.61B), demonstrating NAR architecture's advantage for the dialogue SCA use case |
