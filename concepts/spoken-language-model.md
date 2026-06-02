---
slug: spoken-language-model
title: Spoken Language Model
aliases: [speech LM, SpeechGPT, spoken dialogue system, SCA, speech foundation model, audio LLM, SLM, end-to-end spoken dialogue]
related_concepts: [autoregressive-codec-tts, neural-codec, streaming-tts, rlhf-speech, instruction-conditioned-tts, self-supervised-speech]
last_updated: 2026-06-02
status: emerging
---
## Executive Summary

> [!abstract]
> Spoken language models (SLMs) operate directly on discrete speech tokens, enabling speech-to-speech generation without text as an intermediate representation. As of 2025, pure SLMs still lag behind cascade systems (ASR → text LLM → TTS) on semantic coherence, but hybrid architectures like Moshi, LLaMA-Omni 2, and VocalNet are closing the gap rapidly. The field is converging on aligned multimodal LLMs that couple a speech encoder and decoder to a powerful text LLM backbone, rather than pursuing purely speech-native models.

## Current Status

emerging — SLMs are advancing rapidly across open-source and commercial fronts, with systems like Moshi and GPT-4o voice mode demonstrating real-time duplex capability, but fundamental challenges in semantic coherence and paralinguistic variability remain unsolved. Hybrid architectures dominate the frontier while pure speech-native models remain research targets.

## Why This Matters

SLMs are the pathway to natural, latency-efficient voice interaction that preserves the full richness of human speech — including prosody, emotion, non-verbal cues, and turn-taking dynamics. Cascade systems (ASR → text LLM → TTS) lose paralinguistic information and add latency at each stage. A true end-to-end SLM can model these phenomena natively. However, as [[2412.17048]] demonstrates, the gap between text LLMs and SLMs in semantic coherence is large and stems from fundamental properties of speech tokenization.

For interactive applications, SLMs must also handle fine-grained turn-taking dynamics: interruptions, backchannels, pauses, and gaps. The InteractSpeech dataset ([[2025.findings-emnlp.424]]) provides training and evaluation data specifically for these capabilities.

## Core Idea

A spoken language model (SLM) is a language model that operates directly on discrete speech tokens rather than text tokens, enabling speech-to-speech generation (or speech understanding) without text as an intermediate representation. SLMs process audio input and produce speech output without a TTS bridge, allowing them to model paralinguistic information (prosody, emotion, speaking style, non-verbal cues) that is lost when passing through text.

The input is discretized speech (typically from a neural audio codec or self-supervised model like HuBERT), and the model is trained to predict the next discrete speech token autoregressively. The model may also receive or produce text tokens, but in the purest form (GSLM, AudioLM), operates without text guidance.

In this wiki, SCA (spoken conversational agent) encompasses SLMs used in dialogue settings, including both purely speech-to-speech systems and hybrid architectures that use an intermediate text representation.

## Methods and Variants

**Pure speech-to-speech LMs.** Models trained without text supervision: GSLM uses de-duplicated HuBERT tokens; AudioLM uses hierarchical semantic (SoundStream RVQ-1) and acoustic (RVQ-rest) tokens. These achieve naturalness but limited semantic coherence.

**Interleaved speech-text LMs.** SpiritLM interleaves speech and text tokens at word level; SpeechGPT uses speech-text-paired fine-tuning; TWIST initializes from text LLM weights. These leverage text LLM capabilities while partially retaining speech-level modeling. [[2025.acl-long.682]] surveys this full spectrum.

**Duplex systems.** Moshi uses a multi-stream transformer to simultaneously process and generate speech in real time. This requires low-latency codec tokens and streaming architecture.

**Aligned multimodal LLMs (speech encoder + text LLM + speech decoder).** Systems like VocalNet [[2025.emnlp-main.989]], LLaMA-Omni, Mini-Omni, and Qwen2.5-Omni use a frozen speech encoder, an adaptor, and a large text LLM backbone with a dedicated speech decoder. The LLM's text capabilities are preserved while adding speech I/O. MTP [[2025.emnlp-main.989]] improves the speech decoder's output quality without changing the LLM.

**Empathetic SLMs.** [[2025.emnlp-demos.70]] (OpenS2S) extends the aligned multimodal paradigm with empathy modeling via multi-stage training (speech understanding, speech generation, empathetic instruction tuning) and an automated pipeline for constructing empathetic speech dialogue data. Streaming interleaved decoding (M=4 LLM states → N=8 speech tokens per step) enables low-latency generation.

**Interaction-capable systems.** [[2025.findings-emnlp.424]] fine-tunes Qwen-2.5-Omni with GRPO on InteractSpeech to classify turn-taking events (backchannels, interruptions, gaps, pauses). LLaMA 3-8B fine-tuned on InteractSpeech text achieves 86% human agreement on interactive dialogue generation.

## Major Claims

Claims are generalised propositions aggregated from paper evidence. The full claim registry with supporting paper lists is in `wiki/concepts/_evidence/spoken-language-model.yaml`.

### Strongly Supported

- Paralinguistic variability in speech tokens (not phonetic-vs-semantic type) is the dominant bottleneck preventing SLMs from achieving text-LLM-level semantic coherence.
  Supporting: [[2412.17048]], [[2025.acl-long.682]], [[interspeech-2025-0310]]

- Cross-modal distillation from a frozen text LLM using only ASR-paired data can match or outperform SFT-trained speech LLMs on instruction-following tasks, at substantially lower compute cost.
  Supporting: [[2025.acl-long.388]], [[2025.acl-long.912]]

- Multi-token prediction (MTP) for speech decoding in aligned multimodal LLMs improves quality and reduces data requirements compared to next-token prediction, without requiring ASR/TTS pretraining stages.
  Supporting: [[2025.emnlp-main.989]]

### Emerging

- Targeted fine-tuning on interaction-annotated dialogue data substantially improves turn-taking event classification accuracy in SLMs, suggesting that interaction capability is data-limited rather than architecture-limited.
  Supporting: [[2025.findings-emnlp.424]]

- Codec design directly impacts SLM performance: lower per-channel information entropy (via MCRVQ redistribution) improves downstream AR language model speaker similarity.
  Supporting: [[2025.acl-long.654]]

- Non-autoregressive flow-matching architectures can match AR SLMs on dialogue generation quality while being 15× faster, suggesting AR is not required for the spoken dialogue use case.
  Supporting: [[2507.09318]]

- Full-duplex spoken dialogue models that rely on end-to-end architectures interrupt user pauses at high rates; explicit turn-state prediction modules substantially reduce unwanted interruptions.
  Supporting: [[2503.04721]]

- Backchannel generation remains a largely unsolved capability in current full-duplex systems; most open-source models produce near-zero backchannel frequency.
  Supporting: [[2503.04721]]

- LLM joint prediction of interleaved speech and visual tokens from a multimodal dialogue context is feasible and outperforms cascaded speech-then-video generation on emotion consistency and lip synchronisation.
  Supporting: [[2508.04585]]

- The quality of spoken role-playing in both cascaded and end-to-end systems is more strongly determined by the underlying language model's text reasoning ability than by the speech synthesis architecture.
  Supporting: [[2508.02013]]

- Separating the token used for LLM input from the token used for generation output resolves the understanding-generation tension in unified speech LLMs while reducing data requirements substantially.
  Supporting: [[2508.08961]]

### Contested

> [!warning]
> Whether pure end-to-end SLMs are a necessary development path or whether hybrid architectures (text LLM + speech I/O) will permanently dominate commercial deployment remains unresolved. Evidence increasingly favors hybrids on efficiency and semantic coherence.
> Supporting hybrids: [[2025.acl-long.912]], [[2025.emnlp-main.989]], [[2025.acl-long.388]] / Supporting pure SLMs as research target: [[2412.17048]], [[interspeech-2025-0310]]

## Relationship to Other Concepts

### Extends or Builds On
- [[neural-codec]] — SLMs depend on discrete speech tokens produced by neural codecs; codec design directly affects SLM sequence length, paralinguistic variability, and semantic coherence
- [[autoregressive-codec-tts]] — the autoregressive token prediction paradigm is shared; SLMs extend AR codec TTS from text-conditioned to speech-conditioned generation
- [[self-supervised-speech]] — early SLMs (GSLM, AudioLM) used SSL representations (HuBERT) for speech tokenization; SSL and codec tokens remain competing approaches

### Competes With
- [[streaming-tts]] — cascade systems using a streaming TTS component (e.g., LLMVoX [[2025.findings-acl.1051]], LLaMA-Omni 2 [[2025.acl-long.912]]) compete directly with pure SLMs for the real-time voice assistant use case; the key trade-off is semantic coherence (cascade wins) vs. paralinguistic preservation (SLM wins)

### Commonly Paired With
- [[instruction-conditioned-tts]] — aligned multimodal SLMs that use a TTS decoder (e.g., CosyVoice 2, OpenS2S) inherit instruction-conditioning capabilities from the TTS component
- [[rlhf-speech]] — RLHF/GRPO is increasingly applied to SLMs for interaction modeling ([[2025.findings-emnlp.424]]) and alignment

## Representative Papers

### Foundational
- [[2412.17048]] — systematic factor analysis isolating the three root causes of SLM coherence failure; foundational diagnostic for the field
- [[2025.acl-long.682]] — first comprehensive survey of the SpeechLM landscape, covering tokenizers, architectures, training paradigms, and evaluation benchmarks

### Influential
- [[2025.acl-long.388]] — establishes cross-modal distillation as a viable alternative to SFT for speech instruction-following
- [[2025.emnlp-main.989]] — demonstrates MTP as a key efficiency lever for aligned multimodal SLMs, eliminating costly pretraining stages
- [[2025.acl-long.912]] — shows modular SpeechLMs can match native SpeechLMs at 200K synthetic samples vs. millions of hours; closes naturalness gap of prior NAR-based modular systems
- [[2025.acl-long.654]] — demonstrates codec design's direct impact on SLM performance via MCRVQ redistribution

### Recent Highlights
- [[2507.09318]] — NAR flow-matching dialogue generation 15× faster than AR baselines, challenging the assumption that AR is necessary for spoken dialogue
- [[2025.emnlp-demos.70]] — first fully open-source empathetic SLM with automated data construction pipeline
- [[interspeech-2025-0310]] — systematic grid search showing that optimized HuBERT tokenization (N=80ms, K=16384) achieves better SLU performance with 50% less data than GSLM-style defaults

## Open Questions

- [[2412.17048]]: What is the most effective way to address Factor C (paralinguistic variability)? Is codec design the answer, or are explicit supervision/data augmentation strategies required?
- Can variable-length speech tokenization (as in FlexiCodec [[2510.00981]]) reduce the Factor B (sequence length) problem while maintaining resynthesis quality?
- What level of interaction modeling is needed for practical applications? The InteractSpeech ([[2025.findings-emnlp.424]]) classification benchmark provides a starting point but end-to-end duplex evaluation remains standardization-free.
- How much of the semantic coherence gap can be closed by scaling the SLM vs. improving the tokenization?
- Are pure end-to-end SLMs a necessary intermediate step, or will cascade architectures continue to dominate commercial systems?
- VocalNet [[2025.emnlp-main.989]] shows MTP improves speech decoding quality significantly; can MTP also improve paralinguistic modeling (prosody, emotion), or does it only help intelligibility?
- OpenS2S [[2025.emnlp-demos.70]] uses automated LLM+TTS data construction for empathy; what is the quality ceiling for this approach compared to human-annotated empathetic data?
- Full-Duplex-Bench [[2503.04721]] reveals a consistent tension: models that take the turn aggressively miss backchannels, while cautious models miss turn-end cues; is this a fundamental trade-off or addressable by training?
- ProsodyLM [[2507.20091]] achieves prosody emergence without codec tokens by using explicit prosody annotations; can this approach scale to spontaneous conversational speech where prosody structure is less predictable?
- DualSpeechLM [[2508.08961]] demonstrates competitive quality at 4.5K hours; does the dual-token advantage persist at larger data scales where the understanding-generation tension may naturally resolve?
- SpeechRole [[2508.02013]] shows interaction quality is text-LM bottlenecked; how much of the gap between cascaded and E2E systems is paralinguistic modelling vs. text understanding quality?
- SecoustiCodec [[2508.02849]] is designed explicitly for voice-dialogue streaming; can its explicit three-way semantic/acoustic/paralinguistic decomposition help address Factor C from [[2412.17048]]?

## Trend Summary

2021: GSLM establishes that de-duplicated HuBERT tokens allow rudimentary speech language modeling. 2022: AudioLM adds acoustic conditioning for naturalness. 2023: TWIST, SpeechGPT bring text LLM initialization and fine-tuning. 2024: Moshi achieves real-time duplex operation; GPT-4o voice mode demonstrates commercial-scale capability. 2025: Systematic analysis ([[2412.17048]]) reveals Factor C (paralinguistic variability) as the dominant bottleneck. InteractSpeech ([[2025.findings-emnlp.424]]) provides targeted training data for interaction dynamics. DiVA ([[2025.acl-long.388]]) establishes cross-modal distillation as a data-efficient alternative to SFT. VocalNet ([[2025.emnlp-main.989]]) shows that MTP halves WER vs. NTP and ASR/TTS pretraining stages add cost without benefit. OpenS2S ([[2025.emnlp-demos.70]]) demonstrates fully open-source empathetic SLMs with automated data construction. LLaMA-Omni 2 [[2025.acl-long.912]] demonstrates that modular SpeechLMs can match native SpeechLMs at 200K synthetic samples vs. millions of hours — establishing a new data efficiency benchmark for real-time spoken chatbots, with the key insight that integrating a high-quality autoregressive streaming TTS module (CosyVoice 2 chunk-aware FM) closes the naturalness gap of prior NAR-based modular systems. [[2025.acl-long.682]] provides the first comprehensive survey of the field as of mid-2025. Integration pass 5 adds: ProsodyLM [[2507.20091]] demonstrating prosody emergence via explicit token design; Full-Duplex-Bench [[2503.04721]] providing the first automated benchmark for turn-taking in full-duplex systems; SpeechRole [[2508.02013]] providing the first large-scale speech-to-speech role-playing benchmark (111K dialogues, 98 roles), finding that text-LM quality bottlenecks interaction; SecoustiCodec [[2508.02849]] providing a streaming single-codebook codec explicitly targeting voice-dialogue downstream tasks; UniTalker [[2508.04585]] extending multimodal SLM generation to joint speech-visual outputs; DualSpeechLM [[2508.08961]] proposing dual-token understanding-generation decoupling as a solution to the shared-token bottleneck in unified speech LLMs.

## All Papers

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
| [[2503.04721]] | Full-Duplex-Bench | arXiv | 2025 | First automated benchmark for full-duplex SLM turn-taking across four scenarios (pause handling, backchanneling, smooth turn-taking, interruption management); reveals consistent aggressiveness-caution tension across all tested models |
| [[2507.20091]] | ProsodyLM | arXiv | 2025 | Demonstrates prosody emergence in LLMs via word-level prosody token design; contrastive focus, emotion recognition, and style continuity without task-specific fine-tuning |
| [[2508.02013]] | SpeechRole | arXiv | 2025 | First large-scale S2S speech role-playing benchmark (111K dialogues, 98 roles); finds text-LM quality is the primary determinant of role-playing fidelity and interaction quality |
| [[2508.02849]] | SecoustiCodec | arXiv | 2025 | Single-codebook streaming codec with explicit semantic-acoustic-paralinguistic decomposition; designed for voice-dialogue downstream tasks; 12.08 ms initial latency |
| [[2508.04585]] | UniTalker | arXiv | 2025 | Joint prediction of interleaved speech and facial landmark tokens from multimodal dialogue context in a Qwen2.5-based SLM; frame-level bimodal alignment for synchronised audiovisual output |
| [[2508.08961]] | DualSpeechLM | arXiv | 2025 | Dual-token architecture: USTokenizer (understanding-driven LLM-aligned input tokens) + WavTokenizer (acoustic output tokens); competitive at 4.5K hours on ASR, S2TT, SER, TTS, VC |
