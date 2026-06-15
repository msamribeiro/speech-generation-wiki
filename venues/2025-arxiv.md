---
title: "arXiv 2025"
venue: arXiv
year: 2025
papers_ingested: 89
last_updated: 2026-06-15
---

# arXiv 2025

## Overview

This page tracks arXiv preprints ingested into the corpus. Eighty-four papers are currently represented, spanning early 2025 through mid-2025. Four papers from this corpus represent major early-2025 industrial SCA systems: [[2502.11946]] (Step-Audio) demonstrates that scaling an AR LLM backbone from 3B to 130B produces substantial TTS quality gains and introduces RLHF counter-example training to prevent evasive non-answers; [[2501.06282]] (MinMo) integrates TTS, ASR, and full-duplex dialogue in a single framework with 268ms latency; [[2502.17239]] (Baichuan-Audio) introduces staged pretraining and flow-matching post-refinement to preserve LLM intelligence in end-to-end speech systems; and [[2507.16632]] (Step-Audio 2) extends to retrieval-augmented generation, tool calling, and paralinguistic comprehension at production scale. Two are confirmed accepted to top venues: [[2510.00981]] (FlexiCodec → ICLR 2026) and [[2507.14534]] (Conan → ASRU 2025). [[2412.17048]] is accepted to ICASSP 2026. [[2502.03930]] (DiTAR) is accepted to ICML 2025. [[2504.10352]] (PALLE) is accepted to ACM MM 2025. The remaining papers are preprints ranging from 2023 to 2025.

The arXiv 2025 corpus spans the full range of tasks: TTS (majority), VC ([[2507.14534]], [[2508.16332]]), codec ([[2510.00981]]), singing ([[2508.16332]]), and SCA ([[2412.17048]], [[2509.02020]]). Dominant themes in the Aug–Sep 2025 integration window: (1) extreme codec compression — TaDiCodec [[2508.16790]] achieves 6.25 Hz single-codebook tokenization via text-conditioned flow-matching; VibeVoice [[2508.19205]] demonstrates 7.5 Hz continuous VAE for hour-long multi-speaker TTS; (2) empathetic spoken dialogue — OSUM-EChat [[2508.09600]] and TurnGuide [[2508.07375]] advance emotional awareness and semantic coherence in full-duplex systems; (3) dialogue evaluation — EMO-Reasoning [[2508.17623]], VoxRole [[2509.03940]], and LibriQuote [[2509.04072]] introduce new benchmarks for emotional coherence, role-playing, and narrative expressivity; (4) automatic evaluation methodology — QAMRO [[2508.08957]] and AESA-Net [[2509.03292]] improve MOS predictor training via ranking objectives; SITool [[interspeech-2025-0984]] provides negative result against WER as codec intelligibility proxy; (5) speaker anonymization and attribute disentanglement — [[2508.15565]] (any-to-any adversarial perturbation), [[2508.15931]] (timbre attribute ranking), [[2508.17031]] (cross-modal speech insertion); (6) prosody front-end — [[2509.00675]] (speaker-conditioned phrase break), [[2508.17494]] (French SSML control); (7) SSL token compression — [[2509.00503]] achieves adaptive 15 Hz compression of HuBERT tokens with better ASR than 50 Hz original. Earlier 2025 themes also present: unified speech + singing ([[2508.16332]] Vevo2); emotion control both training-based ([[2508.02038]] Marco-Voice, [[2508.11273]] EmoSSLSphere) and training-free ([[2508.03543]] EmoSteer-TTS); hybrid AR+diffusion ([[2502.03930]] DiTAR); attention-free flow matching ([[2510.02848]] Flamed-TTS); production-level GRPO RL ([[2512.14291]] GLM-TTS); codec-free AR TTS ([[2502.11128]] FELLE, [[2508.19098]] CLEAR); RL preference optimization ([[2509.00685]] MPO); streaming latency ([[2509.15969]] VoXtream); flow-matching distillation ([[2510.07979]] IntMeanFlow).

## Papers

| ID | Title |
|----|-------|
| 2301.02111 | Neural Codec Language Models are Zero-Shot Text to Speech Synthesizers (VALL-E) |
| 2412.17048 | Why Do Speech Language Models Fail to Generate Semantically Coherent Outputs? |
| 2502.03930 | DiTAR: Diffusion Transformer Autoregressive Modeling for Speech Generation |
| 2504.10352 | Pseudo-Autoregressive Neural Codec Language Models for Efficient Zero-Shot Text-to-Speech Synthesis |
| 2507.14534 | Conan: A Chunkwise Online Network for Zero-Shot Adaptive Voice Conversion |
| 2508.02038 | Marco-Voice Technical Report |
| 2508.03543 | EmoSteer-TTS: Fine-Grained and Training-Free Emotion-Controllable Text-to-Speech via Activation Steering |
| 2508.06262 | Llasa+: Free Lunch for Accelerated and Streaming Llama-Based Speech Synthesis |
| 2508.16332 | Vevo2: A Unified and Controllable Framework for Speech and Singing Voice Generation |
| 2509.02020 | FireRedTTS-2: Towards Long Conversational Speech Generation for Podcast and Chatbot |
| 2509.19668 | Selective Classifier-free Guidance for Zero-shot Text-to-speech |
| 2510.00981 | FlexiCodec: A Dynamic Neural Audio Codec for Low Frame Rates |
| 2510.02848 | Flamed-TTS: Flow Matching Attention-Free Models for Efficient Generating and Dynamic Pacing Zero-shot Text-to-Speech |
| 2512.14291 | GLM-TTS Technical Report |
| 2506.21619 | IndexTTS2: A Breakthrough in Emotionally Expressive and Duration-Controlled Auto-Regressive Zero-Shot TTS |
| 2510.12210 | DiSTAR: Diffusion over a Scalable Token Autoregressive Representation for Speech Generation |
| 2509.00685 | MPO: Multidimensional Preference Optimization for Language Model-based Text-to-Speech |
| 2512.13251 | DisCo-Speech: Controllable Zero-Shot Speech Generation with A Disentangled Speech Codec |
| 2512.04720 | M3-TTS: Multi-modal DiT Alignment & Mel-latent for Zero-shot High-fidelity Speech Synthesis |
| 2509.09631 | DiFlow-TTS: Compact and Low-Latency Zero-Shot Text-to-Speech with Factorized Discrete Flow Matching |
| 2508.11273 | EmoSSLSphere: Multilingual Emotional Speech Synthesis with Spherical Vectors and Discrete Speech Tokens |
| 2508.04141 | Parallel GPT: Harmonizing the Independence and Interdependence of Acoustic and Semantic Information for Zero-Shot Text-to-Speech |
| 2502.11128 | FELLE: Autoregressive Speech Synthesis with Token-Wise Coarse-to-Fine Flow Matching |
| 2508.19098 | CLEAR: Continuous Latent Autoregressive Modeling for High-quality and Low-latency Speech Synthesis |
| 2508.12001 | FNH-TTS: A Fast, Natural, and Human-Like Speech Synthesis System with advanced prosodic modeling based on Mixture of Experts |
| 2509.15969 | VoXtream: Full-Stream Text-to-Speech with Extremely Low Latency |
| 2510.07979 | IntMeanFlow: Few-step Speech Generation with Integral Velocity Distillation |
| 2508.00317 | Advancing Speech Quality Assessment Through Scientific Challenges and Open-source Activities |
| 2507.22746 | Next Tokens Denoising for Speech Synthesis |
| 2508.01796 | Enhancing Spectrogram Realism in Singing Voice Synthesis via Explicit Bandwidth Extension Prior to Vocoder |
| 2508.02013 | SpeechRole: A Large-Scale Dataset and Benchmark for Evaluating Speech Role-Playing Agents |
| 2508.02849 | SecoustiCodec: Cross-Modal Aligned Streaming Single-Codebook Speech Codec |
| 2508.14049 | MahaTTS: A Unified Framework for Multilingual Text-to-Speech Synthesis |
| 2508.04585 | UniTalker: Conversational Speech-Visual Synthesis |
| 2508.04996 | REF-VC: Robust, Expressive and Fast Zero-Shot Voice Conversion with Diffusion Transformers |
| 2508.05207 | SpectroStream: A Versatile Neural Codec for General Audio |
| 2507.20091 | ProsodyLM: Uncovering the Emerging Prosody Processing Capabilities in Speech Language Models |
| 2508.05385 | A Scalable Pipeline for Enabling Non-Verbal Speech Generation and Understanding |
| 2508.06870 | Text to Speech System for Meitei Mayek Script |
| 2508.06890 | Maestro-EVC: Controllable Emotional Voice Conversion Guided by References and Explicit Prosody |
| 2508.07302 | XEmoRAG: Cross-Lingual Emotion Transfer with Controllable Intensity Using Retrieval-Augmented Generation |
| 2508.07711 | Is GAN Necessary for Mel-Spectrogram-based Neural Vocoder? |
| 2508.08399 | Exploring Disentangled Neural Speech Codecs from Self-Supervised Representations |
| 2508.08961 | DualSpeechLM: Towards Unified Speech Understanding and Generation via Dual Speech Token Modeling |
| 2504.12867 | EmoVoice: LLM-based Emotional Text-To-Speech Model with Freestyle Text Prompting |
| 2508.11326 | MoE-TTS: Enhancing Out-of-Domain Text Understanding for Description-based TTS via Mixture-of-Experts |
| 2508.09767 | UtterTune: LoRA-Based Target-Language Pronunciation Edit and Control in Multilingual TTS |
| 2508.08715 | MultiGen: Child-Friendly Multilingual Speech Generator with LLMs |
| 2503.04721 | Full-Duplex-Bench: A Benchmark to Evaluate Full-duplex Spoken Dialogue Models on Turn-taking Capabilities |
| 2508.08095 | Dual Information Speech Language Models for Emotional Conversations |
| 2508.15827 | Mini-Omni-Reasoner: Token-Level Thinking-in-Speaking in Large Speech M |
| 2508.20660 | CodecBench: A Comprehensive Benchmark for Acoustic and Semantic Evalua |
| 2508.07375 | TurnGuide: Enhancing Meaningful Full Duplex Spoken Interactions via Dy |
| 2508.16790 | TaDiCodec: Text-aware Diffusion Speech Tokenizer for Speech Language M |
| 2508.07273 | Incorporating Contextual Paralinguistic Understanding in Large Speech- |
| 2508.08957 | QAMRO: Quality-aware Adaptive Margin Ranking Optimization for Human-al |
| 2508.09600 | OSUM-EChat: Enhancing End-to-End Empathetic Spoken Chatbot via Underst |
| 2508.09702 | $\text{M}^3\text{PDB}$: A Multimodal, Multi-Label, Multilingual Prompt |
| 2508.13028 | Integrating Feedback Loss from Bi-modal Sarcasm Detector for Sarcastic |
| 2508.15565 | Any-to-any Speaker Attribute Perturbation for Asynchronous Voice Anony |
| 2508.15931 | QvTAD: Differential Relative Attribute Learning for Voice Timbre Attri |
| 2508.17031 | RephraseTTS: Dynamic Length Text based Speech Insertion with Speaker S |
| 2508.17623 | EMO-Reasoning: Benchmarking Emotional Reasoning Capabilities in Spoken |
| 2508.18006 | Unseen Speaker and Language Adaptation for Lightweight Text-To-Speech  |
| 2508.19205 | VibeVoice Technical Report |
| 2509.00503 | Entropy-based Coarse and Compressed Semantic Speech Representation Lea |
| 2509.00675 | Speaker-Conditioned Phrase Break Prediction for Text-to-Speech with Ph |
| 2509.01391 | MixedG2P-T5: G2P-free Speech Synthesis for Mixed-script texts using Sp |
| 2509.02244 | Spectrogram Patch Codec: A 2D Block-Quantized VQ-VAE and HiFi-GAN for  |
| 2509.03292 | Improving Perceptual Audio Aesthetic Assessment via Triplet Loss and S |
| 2509.03940 | VoxRole: A Comprehensive Benchmark for Evaluating Speech-Based Role-Pl |
| 2509.04072 | Computational Narrative Understanding for Expressive Text-to-Speech |
| 2503.20215 | Qwen2.5-Omni Technical Report |
| 2503.01710 | Spark-TTS: An Efficient LLM-Based Text-to-Speech Model with Single-Stream Decoupled Speech Tokens |
| 2505.17589 | CosyVoice 3: Towards In-the-wild Speech Generation via Scaling-up and Post-training |
| 2504.18425 | Kimi-Audio Technical Report |
| 2502.04128 | Llasa: Scaling Train-Time and Inference-Time Compute for Llama-based S |
| 2505.09388 | Qwen3 Technical Report |
| 2507.06261 | Gemini 2.5: Pushing the Frontier with Advanced Reasoning, Multimodalit |
| 2501.12948 | DeepSeek-R1: Incentivizing Reasoning Capability in LLMs via Reinforcem |
| 2502.11946 | Step-Audio: Unified Understanding and Generation in Intelligent Speech |
| 2501.06282 | MinMo: A Multimodal Large Language Model for Seamless Voice Interactio |
| 2502.17239 | Baichuan-Audio: A Unified Framework for End-to-End Speech Interaction |
| 2507.16632 | Step-Audio 2 Technical Report |
| 2505.07916 | MiniMax-Speech: Intrinsic Zero-Shot Text-to-Speech with a Learnable Sp |
| 2502.06490 | Recent Advances in Discrete Speech Tokens: A Review |
| 2502.05512 | IndexTTS: An Industrial-Level Controllable and Efficient Zero-Shot Tex |
| 2504.08528 | On The Landscape of Spoken Language Models: A Comprehensive Survey |
| 2502.05139 | Meta Audiobox Aesthetics: Unified Automatic Quality Assessment for Spe |
