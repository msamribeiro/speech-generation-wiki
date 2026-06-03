---
slug: autoregressive-codec-tts
title: Autoregressive Codec TTS
aliases: [VALL-E family, codec language model, audio LM, AR speech LM, token-by-token decoding]
related_concepts: [neural-codec, spoken-language-model, flow-matching, zero-shot-tts]
last_updated: 2026-06-03
status: dominant
---

## Executive Summary

> [!abstract]
> Autoregressive codec TTS treats speech synthesis as a language modeling problem: a large language model generates discrete neural audio codec tokens sequentially, enabling strong in-context speaker generalization and prosodic naturalness that scales reliably with data and model size. Established by VALL-E in 2023, it is co-dominant with flow-matching-based systems in zero-shot TTS as of 2026. The field is actively bifurcating between scaling AR LMs with better tokenizers and RL alignment versus challenging whether sequential AR decoding is necessary at all.

## Current Status

dominant — Autoregressive codec TTS is co-dominant with flow-matching-based systems for zero-shot TTS as of early 2026. Leading industrial systems (Fish Audio S2, Qwen3-TTS) use Dual-AR architectures at multi-billion-parameter scale, while academic systems are exploring pseudo-autoregressive and codec-free alternatives that match or exceed AR quality with dramatically lower latency.

## Why This Matters

Autoregressive codec TTS achieves strong zero-shot speaker generalization (any speaker can be synthesized from a short prompt), prosodic naturalness (the LM captures phrase-level rhythm patterns), and scalability (more data and larger models reliably improve quality). The AR+NAR framework (or AR+flow matching) is the dominant TTS paradigm as of 2025.

The central efficiency challenge is the codec frame rate: the AR LM must generate tokens sequentially, and standard codecs (50–75 Hz) produce very long sequences. This motivates low-frame-rate codecs. [[2510.00981]] (FlexiCodec) demonstrates that 6.25 Hz AR tokens achieve the same TTS quality as 12.5 Hz with a 2.1× AR speedup, and that going lower in AR frame rate does not necessarily degrade final speech quality.

The semantic coherence challenge from [[2412.17048]] is equally important: even when codec tokens have reasonable semantic content, the paralinguistic variability (Factor C) and sequence length (Factor B) of speech tokens make LM training substantially harder than for text LMs.

## Core Idea

Autoregressive codec TTS is a family of speech synthesis systems that generate speech by autoregressively predicting discrete neural audio codec tokens using a language model (LM). The paradigm was established by VALL-E (Wang et al., 2023): an AR LM generates the first RVQ layer tokens (RVQ-1, semantic/content tokens) from text and a speaker prompt; a non-autoregressive (NAR) model then predicts the remaining RVQ layers in parallel to produce the full acoustic representation; a codec decoder synthesizes the waveform.

This paradigm treats TTS as a sequence-to-sequence language modeling problem, analogous to how text LLMs predict next tokens, and benefits directly from advances in LLM architectures, training techniques, and scale.

## Methods and Variants

**Standard AR+NAR (VALL-E style).** AR LM predicts RVQ-1 tokens; NAR model predicts RVQ-rest in parallel. The AR bottleneck limits speed proportionally to codec frame rate. Established by [[2301.02111]].

**AR+flow matching.** Replace the NAR discrete token prediction with continuous flow matching over mel-spectrograms or high-rate codec features. CosyVoice [[2407.05407]], CosyVoice 2 [[2412.10117]], FlexiCodec-TTS [[2510.00981]], Vevo2 [[2508.16332]], and LLaMA-Omni 2 [[2025.acl-long.912]] use this approach. Flow matching NAR provides better naturalness than discrete NAR at comparable speed.

**Dual-AR (slow semantic + fast acoustic).** Fish Audio S2 [[2603.08823]] decouples a large slow AR transformer (4B Qwen3 for semantic tokens) from a lightweight fast AR decoder (4 layers for remaining acoustic codebooks). Qwen3-TTS [[2601.15621]] uses a related dual-tokenizer approach (25 Hz and 12 Hz variants). This separation allows the expensive LM to focus on semantic/prosodic coherence while the fast decoder handles acoustic detail in parallel.

**Dynamic frame rate AR.** FlexiCodec-TTS [[2510.00981]] uses variable-length tokens where each token carries an explicit frame length attribute. Achieves sub-10 Hz effective AR rate.

**Continuous mel AR (codec-free).** MELLE [[2025.acl-long.65]] replaces codec tokens with directly predicted continuous mel-spectrogram frames via regression loss + spectrogram flux loss + variational latent sampling. Achieves CMOS -0.032 vs. ground truth on LibriSpeech — effectively human parity — without any RVQ quantization step.

**Multi-token prediction (MTP).** [[2025.emnlp-main.989]] (VocalNet) and Qwen3-TTS [[2601.15621]] (12Hz variant) predict multiple future tokens per AR step, reducing WER by ~47% relative vs. NTP (VocalNet) and halving first-packet latency.

**Pseudo-autoregressive (PAR).** PALLE [[2504.10352]] implements span-level causal commitment via a bidirectional masked generative transformer: at each inference step, all token positions are predicted in parallel and only the leftmost span is committed. Achieves O(1) inference steps regardless of output length, yielding 10x RTF speedup over AR systems while achieving best WER among LibriTTS-trained models (WER-H 2.83 on LibriSpeech test-clean).

**Discrete AR+masked diffusion (DiSTAR paradigm).** DiSTAR [[2510.12210]] combines a causal AR LM for inter-patch (cross-frame) prediction with a discrete masked diffusion transformer (MDM) for intra-patch (within-frame) parallel refinement — operating entirely in RVQ code space. Provides test-time bitrate control via RVQ layer pruning without retraining.

**Continuous token AR.** [[2025.findings-naacl.184]] (Cont-SPT) replaces RVQ with a continuous speech tokenizer; [[2502.03930]] (DiTAR) uses a patch-based continuous AR+bidirectional-diffusion architecture. These challenge the necessity of discrete tokenization.

## Major Claims

Claims are generalised propositions aggregated from paper evidence. The full claim registry with supporting paper lists is in `wiki/concepts/_evidence/autoregressive-codec-tts.yaml`.

### Strongly Supported

- Autoregressive codec TTS scales reliably with data and model size, and benefits directly from advances in LLM architectures and training techniques developed for text LMs.
  Supporting: [[2301.02111]], [[2406.02430]], [[2603.08823]], [[2601.15621]]

- Lowering the AR codec frame rate does not degrade final speech quality as long as the NAR or vocoder stage can compensate, and substantially reduces AR inference latency.
  Supporting: [[2510.00981]], [[2601.15621]], [[2603.08823]]

- Post-training alignment (RLHF, DPO, GRPO) reliably improves intelligibility, speaker similarity, and expressiveness in AR codec TTS systems without sacrificing naturalness.
  Supporting: [[2406.02430]], [[2512.14291]], [[2603.08823]], [[2601.15621]]

### Emerging

- Pseudo-autoregressive inference (PAR) — span-level causal commitment with bidirectional intra-span generation — can match or exceed AR TTS quality at dramatically lower latency, challenging the necessity of token-by-token decoding.
  Supporting: [[2504.10352]]

- Explicit phoneme position supervision during AR codec training eliminates alignment failures (skipping, repetition, one-to-many mapping) more effectively than phoneme identity co-prediction or monotonic decoding constraints.
  Supporting: [[interspeech-2025-1641]]

- Speculative decoding adapted for speech — with a tolerance-factor acceptance criterion exploiting the many-to-one mapping between token sequences and perceptual quality — can accelerate AR TTS without modifying the target model.
  Supporting: [[interspeech-2025-2447]]

- Codec-free continuous mel AR achieves human-parity speech quality, demonstrating that discrete tokenization is not necessary for competitive zero-shot TTS.
  Supporting: [[2025.acl-long.65]], [[2502.03930]]

- The Dual-AR decoupling (separate large slow semantic model and lightweight fast acoustic model) is emerging as the dominant industrial architecture for high-quality low-latency AR TTS.
  Supporting: [[2603.08823]], [[2601.15621]]

### Contested

> [!warning]
> Paralinguistic variability (Factor C) in speech codec tokens makes LM training substantially harder than for text LMs, and it is unclear whether this can be addressed within the AR+NAR framework or requires fundamentally different architectures.
> Supporting: [[2412.17048]] / Partial mitigation: [[2025.acl-long.1498]]

> [!warning]
> Whether separating LLM input tokens from output tokens (the DualSpeechLM [[2508.08961]] dual-token approach) is a viable solution to the understanding-generation tension in unified AR speech LMs, or whether it introduces new problems (tokenizer training cost, modality gap), is unresolved.
> Supporting as useful: [[2508.08961]] / No contradicting evidence yet

## Relationship to Other Concepts

### Extends or Builds On
- [[neural-codec]] — AR codec TTS depends entirely on neural audio codecs (EnCodec, DAC, FACodec, etc.) for tokenization; codec design choices (frame rate, codebook size, RVQ layers) directly affect AR LM training difficulty and inference speed.
- [[spoken-language-model]] — AR codec TTS shares architectural foundations with spoken language models; the AR codec LM is the speech-domain analog of a text LM, and advances in LLM architecture (Transformer scaling, MTP, RL alignment) transfer directly.

### Competes With
- [[flow-matching]] — Flow-matching non-autoregressive TTS is faster per utterance (no sequential decoding) but relies on CFG-based conditioning rather than an AR LM's natural in-context learning. AR codec TTS provides stronger in-context learning and speaker consistency but has higher latency. The trend (CosyVoice 2, FlexiCodec-TTS) is hybrid: an AR stage for semantic/speaker coherence + flow matching for acoustic quality.

### Commonly Paired With
- [[zero-shot-tts]] — AR codec TTS is one of the two dominant paradigms for zero-shot TTS (alongside flow matching); the AR LM's in-context learning from a short reference clip is the primary mechanism for speaker generalization.
- [[neural-codec]] — Every AR codec TTS system depends on a neural audio codec for both tokenization (AR input/output) and waveform reconstruction (codec decoder).

## Representative Papers

### Foundational
- [[2301.02111]] — VALL-E established the codec language modeling paradigm for TTS, demonstrating that an AR LM conditioned on a 3-second speaker prompt could achieve strong zero-shot speaker generalization without per-speaker fine-tuning.

### Influential
- [[2406.02430]] — Seed-TTS achieved human-parity synthesis (CMOS -0.07) at foundation-model scale with RL post-training, and introduced Seed-TTS-Eval as the standard zero-shot TTS benchmark for subsequent work.
- [[2407.05407]] — CosyVoice established the LLM+flow-matching hybrid as a dominant paradigm, combining supervised semantic tokens with OT-CFM acoustic synthesis.
- [[2510.00981]] — FlexiCodec demonstrated that 6.25 Hz AR tokens are viable, providing an important design principle: lower AR frame rate does not degrade quality while higher NAR frame rate improves naturalness.
- [[2412.17048]] — Identified the root causes of AR LM failures on speech tokens (sequence length and paralinguistic variability), providing a theoretical framework for understanding the codec LM bottleneck.

### Recent Highlights
- [[2603.08823]] — Fish Audio S2 achieved the best open-source WER on Seed-TTS-Eval with a Dual-AR architecture and GRPO multi-dimensional RL, trained on 10M hours across 80+ languages.
- [[2601.15621]] — Qwen3-TTS demonstrated sub-100 ms streaming first-packet latency with DPO+GRPO post-training at scale across 10 languages.
- [[2504.10352]] — PALLE introduced the PAR paradigm achieving 10x RTF speedup over AR with best WER among LibriTTS-trained models, demonstrating that sequential token generation is not necessary.
- [[2604.00688]] — OmniVoice covered 600+ languages with a single discrete NAR model, demonstrating that LLM initialization resolves the prior NAR intelligibility gap.

### Cautionary / Negative Evidence
- [[2412.17048]] — Documents that speech codec tokens are substantially harder to model than text tokens due to paralinguistic variability and sequence length, raising fundamental questions about whether AR LMs are the right inductive bias for speech.

## Open Questions

- At what AR frame rate does performance degrade irreversibly for a given codec design?
- Can semantic speech tokens approach text-rate (~4.5 Hz) without quality loss?
- Is the Factor C problem (paralinguistic variability) addressable within the AR+NAR framework, or does it require fundamentally different architectures?
- [[2025.acl-long.1498]] shows DRI can be mitigated at codec training time; does reducing DRI also address part of Factor C?
- [[2025.findings-naacl.184]] proposes continuous tokens and [[2025.acl-long.65]] (MELLE) demonstrates codec-free AR; what are their stability and scaling behaviors compared to discrete-token systems?
- [[2025.emnlp-main.989]] (VocalNet) shows MTP reduces WER substantially; can MTP also improve speaker similarity?
- PALLE [[2504.10352]] achieves 10x speedup over AR via PAR at 580h; does this advantage persist at 100k-hour scale?
- The Dual-AR decoupling in Fish Audio S2 [[2603.08823]] and Qwen3-TTS [[2601.15621]] seems to be a new industrial standard; what is the optimal split between slow (semantic) and fast (acoustic) model capacity?
- Does the full-codebook masking in OmniVoice [[2604.00688]] generalize to multilingual settings beyond Chinese/English, and can it eliminate the need for a separate AR stage in production systems?
- MoE-TTS [[2508.11326]] shows that instruction-conditioned AR TTS systems fail to generalise to figurative or metaphorical descriptions; can LLM-aligned AR models overcome this with better tokenizer design or training data?
- Modality-based MoE routing (frozen text experts, trained speech experts) preserves LLM capabilities during speech fine-tuning in [[2508.11326]]; is this approach necessary for all LLM-backbone AR TTS, or only for instruction-conditioned variants?
- DualSpeechLM [[2508.08961]] demonstrates competitive understanding and generation at 4.5K hours; does the dual-token advantage persist at 100K+ hour scale where the modality gap may narrow?
- interspeech-2025-1641 demonstrates that position supervision generalises alignment across all three failure mode categories; can this approach be extended to languages lacking forced-alignment annotations without quality loss?
- SSD [[interspeech-2025-2447]] achieves only 1.4× speedup with the current draft model; could a larger-data draft training narrow the WER gap (3.67% → 5.70%) and enable higher acceptance rates for >2× speedup?

## Trend Summary

2023: VALL-E [[2301.02111]] established the paradigm at 75 Hz with a hierarchical AR+NAR architecture on 60K hours of LibriLight data. 2024: Systems pushed to 25–50 Hz (CosyVoice [[2407.05407]], SoundStorm). Seed-TTS [[2406.02430]] achieved human-parity synthesis (CMOS -0.07) at scale with RL post-training. CosyVoice 2 [[2412.10117]] introduced FSQ tokenization, LLM backbone, and chunk-aware causal flow matching for streaming. 2025: [[2510.00981]] demonstrates 6.25 Hz AR is viable and competitive. [[2025.acl-long.65]] (MELLE) achieves human-parity quality without any codec. DiTAR [[2502.03930]] demonstrates patch-based continuous AR+diffusion at 3-43x lower compute than competing NAR diffusion systems. GLM-TTS [[2512.14291]] introduces 4-reward GRPO for TTS. PALLE [[2504.10352]] introduces the PAR paradigm for O(1)-step inference. IndexTTS2 [[2506.21619]] solves the AR duration control problem via positional embedding tying. 2026: Fish Audio S2 [[2603.08823]] achieves best open-source WER on Seed-TTS-Eval with a Dual-AR architecture trained on 10M hours across 80+ languages. Qwen3-TTS [[2601.15621]] demonstrates sub-100 ms streaming with DPO+GRPO post-training at scale. DiSTAR [[2510.12210]] unifies AR temporal modeling with discrete masked diffusion intra-patch refinement. OmniVoice [[2604.00688]] covers 600+ languages with a single NAR model via full-codebook masking and LLM initialization. The field has bifurcated: one track scales AR LMs with better tokenizers and RL alignment; the other challenges whether AR decoding is necessary at all (MELLE, PALLE, OmniVoice).

## All Papers

| ID | Title | Venue | Year | Key use of this concept |
|----|-------|-------|------|------------------------|
| [[2301.02111]] | Neural Codec Language Models are Zero-Shot Text to Speech Synthesizers (VALL-E) | arXiv | 2023 | Foundational paper establishing the AR+NAR codec language modeling paradigm for TTS; 75 Hz EnCodec tokens, 60K hours training, achieves strong zero-shot speaker generalization via in-context learning |
| [[2510.00981]] | FlexiCodec: A Dynamic Neural Audio Codec for Low Frame Rates | arXiv (ICLR 2026) | 2025 | Demonstrates 6.25 Hz AR codec tokens achieve competitive TTS quality with 7.3× AR speedup; finds lower AR frame rate does not degrade quality while higher NAR frame rate improves naturalness |
| [[2412.17048]] | Why Do Speech Language Models Fail to Generate Semantically Coherent Outputs? | arXiv (ICASSP 2026) | 2026 | Identifies sequence length (Factor B) and paralinguistic variability (Factor C) as the primary bottlenecks for AR LMs over speech codec tokens |
| [[2025.acl-long.1498]] | Analyzing and Mitigating Inconsistency in Discrete Speech Tokens for Neural Codec Language Models | ACL | 2025 | Introduces DRI phenomenon; shows consistency-trained codec reduces VALL-E WER by 1.98% abs and improves SPK-SIM by 5.52% at no reconstruction quality cost |
| [[2025.findings-naacl.184]] | Continuous Speech Tokenizer in Text To Speech | NAACL | 2025 | Replaces RVQ discretization with continuous speech tokens in an AR TTS framework; improves WER from 12.73% (VALL-E) to 6.59% and SPK-SIM from 0.53 to 0.73 on LibriSpeech test-clean |
| [[2025.acl-long.1043]] | OZSpeech: One-step Zero-shot Speech Synthesis with Learned-Prior-Conditioned Flow Matching | ACL | 2025 | Combines a prior-codes generator (feed-forward transformer) with OT-CFM initialized from a learned prior rather than Gaussian noise; achieves single-step inference (NFE=1) with best WER of 0.05% on LibriSpeech test-clean |
| [[2025.emnlp-main.989]] | VocalNet: Speech LLMs with Multi-Token Prediction for Faster and High-Quality Generation | EMNLP | 2025 | Introduces MTP (multi-token prediction) for speech LLMs; predicting 3 tokens per step reduces WER by ~47% relative vs. NTP; VocalNet-8B achieves WER 3.56% and UTMOS 4.49 with ~6K hours training |
| [[2025.acl-long.682]] | Recent Advances in Speech Language Models: A Survey | ACL | 2025 | Comprehensive survey taxonomizing SpeechLM architectures including AR codec LM systems; covers tokenizer, LM, and vocoder design choices across the full ecosystem |
| [[2509.02020]] | FireRedTTS-2: Towards Long Conversational Speech Generation for Podcast and Chatbot | arXiv | 2025 | Extends the AR codec LM paradigm to multi-speaker dialogue: interleaved text-speech sequences, dual-transformer (backbone + residual decoder), 12.5 Hz custom tokenizer with semantic injection for stable long-sequence generation |
| [[2406.02430]] | Seed-TTS: A Family of High-Quality Versatile Speech Generation Models | arXiv | 2024 | Large-scale AR+diffusion TTS foundation model achieving human-parity synthesis (CMOS -0.07); introduces self-distillation for timbre disentanglement and REINFORCE-based post-training for robustness |
| [[2407.05407]] | CosyVoice: A Scalable Multilingual Zero-shot TTS based on Supervised Semantic Tokens | arXiv | 2024 | Introduces supervised S3 tokens (VQ inserted into ASR encoder) + OT-CFM hybrid architecture; establishes LLM+flow-matching as a dominant AR codec TTS paradigm |
| [[2412.10117]] | CosyVoice 2: Scalable Streaming Speech Synthesis with Large Language Models | arXiv | 2024 | Extends CosyVoice with FSQ tokenizer (100% codebook utilization), Qwen2.5-0.5B backbone, and chunk-aware causal flow matching for streaming; unified streaming/non-streaming model |
| [[2025.acl-long.65]] | Autoregressive Speech Synthesis without Vector Quantization (MELLE) | ACL | 2025 | Generates continuous mel-spectrogram frames directly via AR LM (regression loss + flux loss + LSM); demonstrates codec-free AR achieves CMOS -0.032 vs. ground truth |
| [[2506.21619]] | IndexTTS2: A Breakthrough in Emotionally Expressive and Duration-Controlled AR Zero-Shot TTS | arXiv | 2025 | Introduces W_sem=W_num positional embedding tying for precise AR duration control; GRL-based emotion–speaker disentanglement; achieves WER 1.521% on SeedTTS test-en |
| [[2510.12210]] | DiSTAR: Diffusion over a Scalable Token Autoregressive Representation | arXiv | 2025 | Couples causal AR LM with discrete masked diffusion (MDM) for intra-patch parallel refinement in RVQ code space; WER 1.32% on SeedTTS test-en at 0.3B |
| [[2502.03930]] | DiTAR: Diffusion Transformer Autoregressive Modeling for Speech Generation | arXiv | 2025 | Patch-based AR framework with continuous VAE tokens + bidirectional LocDiT for intra-patch generation; 3-43x lower FLOPs than competing NAR diffusion systems |
| [[2512.14291]] | GLM-TTS Technical Report | arXiv | 2025 | Production-level Chinese TTS with 4-reward GRPO RL (CER, SIM, emotion, laughter), Phoneme-In scheme, and Vocos2D vocoder; CER 0.89% on Seed-TTS-eval test-zh at 1.5B |
| [[2504.10352]] | Pseudo-Autoregressive Neural Codec Language Models for Efficient Zero-Shot TTS | arXiv | 2025 | Introduces PAR paradigm — bidirectional masked generative model with span-level causal commitment; 10x RTF speedup over AR; best WER among LibriTTS-trained models at only 580h training |
| [[2508.06262]] | Llasa+: Free Lunch for Accelerated and Streaming Llama-Based Speech Synthesis | arXiv | 2025 | Adds frozen MTP modules + token verification algorithm to Llasa for 1.48x speedup without quality loss; demonstrates plug-and-play AR acceleration |
| [[2603.08823]] | Fish Audio S2 Technical Report | arXiv | 2026 | Dual-AR (4B slow semantic + 4-layer fast acoustic), 21 Hz streaming codec, GRPO multi-dimensional RL; best open-source WER 0.54% zh / 0.99% en on Seed-TTS-Eval |
| [[2601.15621]] | Qwen3-TTS Technical Report | arXiv | 2026 | Dual-tokenizer (25 Hz single-codebook + 12.5 Hz 16-layer RVQ) with DPO+GRPO post-training; 97 ms first-packet latency, state-of-the-art on 10-language multilingual zero-shot evaluation |
| [[2604.00688]] | OmniVoice: Towards Omnilingual Zero-Shot TTS with Diffusion Language Models | arXiv | 2026 | Single-stage discrete NAR covering 600+ languages via full-codebook random masking and LLM initialization; WER 1.30%, SIM-o 0.729 on LibriSpeech-PC |
| [[2508.16332]] | Vevo2: A Unified and Controllable Framework for Speech and Singing Voice Generation | arXiv | 2025 | Unifies TTS and singing in AR+FM via chromagram-based prosody tokenizer; multi-objective GRPO post-training with joint intelligibility and prosody rewards |
| [[2512.13251]] | DisCo-Speech: Controllable Zero-Shot Speech Generation with A Disentangled Speech Codec | arXiv | 2025 | DisCodec factorizes speech into content, prosody, and timbre subspaces enabling independent prosody/timbre control via standard AR LM continuation |
| [[2025.ccl-1.80]] | Lao-English Code-Switched Speech Synthesis Via Neural Codec Language Modeling | workshop | 2025 | Extends VALL-E X to Lao-English code-switching via shared phoneme latent space and per-token language ID embeddings without bilingual training data |
| [[2509.09631]] | DiFlow-TTS: Compact and Low-Latency Zero-Shot TTS with Factorized Discrete Flow Matching | arXiv | 2025 | Applies discrete flow matching over factorized codec tokens (prosody + acoustic) with a 164M DiT; establishes DFM as a viable alternative to continuous-space AR and NAR approaches |
| [[2025.acl-long.654]] | Language-Codec: Bridging Discrete Codec Representations and Speech Language Models | ACL | 2025 | Codec designed for AR LM compatibility via MCRVQ; downstream VALL-E with Language-Codec achieves SPK-SIM 0.700 (vs. 0.612 with EnCodec), showing codec design directly affects AR TTS performance |
| [[2025.acl-short.81]] | Zero-Shot Text-to-Speech for Vietnamese | ACL | 2025 | Benchmarks VALL-E, VoiceCraft, and XTTS-v2 for zero-shot TTS on Vietnamese; XTTS-v2 fine-tuned on PhoAudiobook achieves best WER 8.32 and SMOS 3.39, demonstrating the codec LM paradigm's adaptability to low-resource languages |
| [[2502.11128]] | FELLE: Autoregressive Speech Synthesis with Token-Wise Coarse-to-Fine Flow Matching | arXiv | 2025 | Hybrid AR+flow-matching codec-free system combining LM backbone with per-token flow head; achieves higher WER and MOS than MELLE on LibriSpeech while maintaining zero-shot voice cloning capability |
| [[2508.04141]] | Parallel GPT: Harmonizing the Independence and Interdependence of Acoustic Tokens | arXiv | 2025 | Hybrid AR+NAR with Parallel Tokenizer that extracts independent semantic and acoustic tokens simultaneously; NAR refinement step corrects intra-token dependencies; MOS 4.11 on LibriTTS, WER 0.211 |
| [[2508.15442]] | Mitigating Hallucinations in LM-Based TTS Models via Distribution Alignment | EMNLP | 2025 | Applies GFlowNets to LM-based TTS to reduce autoregressive hallucinations; >50% CER reduction on Seed-TTS-Eval hard subset (13.72% → 6.28%); first application of trajectory flow optimization to AR TTS |
| [[2508.19098]] | CLEAR: Continuous Latent Autoregressive Modeling for High-quality and Efficient TTS | arXiv | 2025 | AR TTS with continuous VAE latents instead of discrete codec tokens; WER 1.88% on LibriSpeech-PC test-clean — competitive with leading discrete-token systems with lower latency |
| [[2509.00685]] | MPO: Multidimensional Preference Optimization for Language Model-based TTS | arXiv | 2025 | Multidimensional preference optimization for AR LM TTS simultaneously improving CER (3.9 vs. 4.72), SPK-SIM (0.577 vs. 0.548), and prosody (F0-RMSE 0.279 vs. 0.337) over SFT baseline |
| [[2509.15969]] | VoXtream: Full-Stream Text-to-Speech with Extremely Low Latency | arXiv | 2025 | Fully AR streaming zero-shot TTS achieving 102 ms GPU first-packet latency (lowest reported for AR TTS); demonstrates that full-streaming AR is viable without sacrificing naturalness |
| [[2511.12347]] | VoiceCraft-X: Unifying Multilingual Voice-Cloning Speech Synthesis and Editing | EMNLP | 2025 | Extends VoiceCraft to 11 languages via Qwen3 LLM backbone with text-speech token reordering; unifies TTS and in-context speech editing; CMOS 0.63 — highest among compared systems on Seed-TTS test-en |
| [[2601.03888]] | IndexTTS 2.5 Technical Report | arXiv | 2026 | Extends AR codec TTS to 4 languages with 2.28x inference speedup via semantic codec compression and Zipformer-based flow matching; token-level concatenation enables zero multilingual TTS without language-ID embeddings |
| [[2603.18090]] | MOSS-TTS Technical Report | arXiv | 2026 | Large-scale AR TTS with causal Transformer tokenizer; millions of hours training data; WER 2.04% (EN) and CER 2.07% (ZH) on Seed-TTS-Eval; demonstrates that open-source large-scale AR TTS can match Qwen3-TTS and CosyVoice 3 |
| [[2603.26364]] | LLaDA-TTS: Unifying Speech Synthesis and Zero-Shot Editing via Masked Discrete Diffusion | arXiv | 2026 | Drops AR LM stage of a CosyVoice-style pipeline and replaces with masked discrete diffusion; achieves CER 0.98% (zh) on Seed-TTS-Eval while enabling parallel generation and step-count-independent inference cost |
| [[2604.01760]] | T5Gemma-TTS Technical Report | arXiv | 2026 | Encoder-decoder codec LM with cross-attention text conditioning; resolves text dilution problem of decoder-only AR TTS; achieves best WER among compared systems on English, Italian, Portuguese, and Chinese |
| [[2604.12438]] | An Ultra-Low Latency End-to-End Streaming Speech Synthesis Architecture | arXiv | 2026 | Non-autoregressive streaming TTS using depth-wise sequential decoding over 32 Mimi RVQ layers; RTF ~0.0033 (303× real-time) with 48.99 ms first-byte latency — fastest in corpus |
| [[interspeech-2025-0551]] | Monotonic Attention for Robust TTS Synthesis in Large Language Model Frameworks | Interspeech | 2025 | Stepwise monotonic attention (SMA) fine-tuning of selected alignment heads in decoder-only LLM TTS reduces deletion/insertion errors without external aligners; CER 8.53% vs. 10.42% on Seed-ZH-Hard at 300M/150K-hour scale |
| [[interspeech-2025-1641]] | Robust Neural Codec Language Modeling with Phoneme Position Prediction | Interspeech | 2025 | Joint AR prediction of codec tokens and phoneme position indices eliminates all three alignment failure modes (skipping, repetition, one-to-many); 52.7% relative CER reduction on Seed-TTS test-zh; position supervision outperforms identity prediction |
| [[interspeech-2025-2447]] | Accelerating Autoregressive Speech Synthesis Inference with Speech Speculative Decoding | Interspeech | 2025 | Draft model from target's upper layers + tolerance-factor acceptance criterion; 1.4× LM-RTF speedup on CosyVoice 2 with no perceptible quality change |
| [[interspeech-2025-1993]] | Defending Unauthorized Voice Cloning with Watermark-Aware Codecs | Interspeech | 2025 | EnCodec-based VALL-E protected against voice cloning via watermark-aware codec encoder training; structural defense that degrades to silence on watermarked prompts |
| [[interspeech-2025-0468]] | DualCodec: A Low-Frame-Rate, Semantically-Enhanced Neural Audio Codec | Interspeech | 2025 | VALL-E downstream TTS with DualCodec 25Hz achieves WER 3.4% EN / 2.49% ZH on Seed-TTS-Eval, vs. 15.4%/21.5% for SpeechTokenizer — establishing DualCodec as a strong codec choice for AR TTS |
| [[2504.12867]] | EmoVoice | arXiv | 2025 | Extends Qwen2.5-based AR codec LM with parallel phoneme-boost decoding head; parallel phoneme output reduces WER; LLM pretraining initialisation essential for emotion-controllable TTS |
| [[2507.20091]] | ProsodyLM | arXiv | 2025 | Proposes replacing codec tokens with explicit prosody-text dual tokens in a Llama 3.1 backbone; demonstrates that prosody emergence requires tokenisation design, not data scale |
| [[2507.22746]] | Next Tokens Denoising for Speech Synthesis (Dragon-FM) | arXiv | 2025 | Hybrid chunk-wise AR + within-chunk flow-matching in a unified model; KV-cache compatible AR backbone with bidirectional intra-chunk denoising; FSQ codec at 12.5 Hz |
| [[2508.04585]] | UniTalker | arXiv | 2025 | Qwen2.5-0.5B backbone predicts interleaved speech and facial landmark tokens from multimodal dialogue context; joint AR prediction of two synchronised token streams at matching rate |
| [[2508.08715]] | MultiGen | arXiv | 2025 | Fine-tunes CosyVoice-300M AR codec LM for child-friendly speech in three low-resource Southeast Asian languages; x-vector speaker conditioning steers age-appropriate vocal characteristics |
| [[2508.08961]] | DualSpeechLM | arXiv | 2025 | Separates understanding-oriented (USTokenizer) and acoustic (WavTokenizer) token streams in a single unified architecture; understanding-driven tokenizer training against LLM objectives reduces modality gap |
| [[2508.09767]] | UtterTune | arXiv | 2025 | LoRA on CosyVoice2 AR codec LM with phoneme-tag tokens for Japanese pitch accent correction; <0.5% parameters; raises accent correctness from 0.498 to 0.899 |
| [[2508.11326]] | MoE-TTS | arXiv | 2025 | Modality-based MoE routing with frozen text experts and trained speech experts; preserves LLM text understanding during speech fine-tuning; out-of-domain description generalisation |
| [[2508.14049]] | MahaTTS | arXiv | 2025 | Gemma-based 0.5B AR LM for text-to-semantic generation; conditional flow matching for acoustics; 22-language Indic TTS from 20K hours of open-source data |
| [[interspeech-2025-0704]] | Differentiable Reward Optimization for LLM based TTS system | Interspeech | 2025 | DiffRO applies RLHF to codec LM TTS via Gumbel-Softmax differentiable token sampling and a multi-task reward model (ASR, emotion, MOS) that operates directly on codec tokens; WER 0.78% zh on seed-tts-eval |
| [[interspeech-2025-0787]] | Gradual modeling of the Lombard effect by modifying speaker embeddings from a Text-To-Speech model | Interspeech | 2025 | Applies embedding-space manipulation to Metavoice 1B (autoregressive codec LM) to generate speaker-style-modified (Lombard) speech via spherical interpolation |
| [[2025.emnlp-main.180]] | Scaling Rich Style-Prompted Text-to-Speech Datasets | EMNLP | 2025 | ParaSpeechCaps fine-tunes Parler-TTS (880M DAC-token autoregressive LM) on 2709 hours of rich style-annotated data, improving intrinsic tag recall from 33% to 69.5% |
| [[2025.findings-acl.1051]] | LLMVoX: Autoregressive Streaming Text-to-Speech Model for Any LLM | ACL | 2025 | 30M autoregressive transformer decoder predicts WavTokenizer speech tokens from phoneme embeddings for streaming LLM-to-speech conversion; WER 3.70% at 475ms latency, decoupled from base LLM |
