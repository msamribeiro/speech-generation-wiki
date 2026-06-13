---
slug: emotion-synthesis
title: Emotion Synthesis
aliases: [expressive TTS, affective speech synthesis, emotional TTS, style transfer]
related_concepts: [prosody-control, instruction-conditioned-tts, disentanglement, subjective-evaluation, spoken-language-model]
last_updated: 2026-06-13
status: emerging
---
## Executive Summary

> [!abstract]
> Emotion synthesis refers to the generation of speech that conveys a target emotional state through paralinguistic cues — pitch, rate, energy, voice quality, and temporal patterning. As of 2025, the frontier spans explicit training-based approaches (GRL disentanglement, reference audio, RL reward) and training-free post-hoc activation steering for pre-trained flow-matching models. End-to-end empathetic speech LMs (OpenS2S) add a joint perception-generation dimension, but simultaneous multi-attribute control without trade-offs remains unsolved.

## Current Status

emerging — Multiple viable approaches to emotion synthesis have emerged: label conditioning, reference audio transfer, natural language routing, RL reward optimization, and training-free activation steering. No single approach dominates; each trades off control precision, generalization, and training data requirements. End-to-end empathetic SLMs represent the newest direction, moving emotion synthesis from TTS post-processing into the dialogue loop.

## Why This Matters

Natural human speech is inherently emotional. Flat, neutral TTS is appropriate for information delivery but fails in conversational, customer service, entertainment, and accessibility contexts where emotional resonance matters. For spoken dialogue agents, empathetic responses require the system to detect user emotion from audio and generate a contextually appropriate emotional response — without losing the emotional signal that an ASR/TTS cascade would discard.

## Core Idea

Emotion synthesis in speech refers to the generation of speech that conveys a target emotional state — such as happiness, sadness, anger, surprise, or empathy — through appropriate paralinguistic cues: pitch contour, speaking rate, energy, voice quality, and temporal patterning. It is a sub-problem of expressive TTS and a core capability for empathetic spoken dialogue systems.

Emotion synthesis can be conditioned on discrete emotion labels (a fixed taxonomy), continuous valence-arousal dimensions, natural language style descriptions (see [[instruction-conditioned-tts]]), or reference audio clips. In speech-to-speech (SCA) systems, emotion synthesis additionally requires detecting and responding to the user's emotional state — a joint perception and generation task.

As of 2025, multiple approaches to emotion synthesis have been integrated into the corpus:

**Empathetic SCA (OpenS2S [[2025.emnlp-demos.70]])**: end-to-end empathetic speech LM with automated data construction (50K EN + 50K ZH pairs). URO-Bench UnderEmotion 46.9 EN / 67.68 ZH.

**AR duration-controlled emotion TTS (IndexTTS2 [[2506.21619]])**: GRL-based emotion–speaker disentanglement with a Conformer-based emotion perceiver, three-stage training curriculum, and T2E module (DeepSeek-R1 distilled to Qwen3 LoRA for natural language emotion routing). EMOS 4.22 vs. 3.37 for MaskGCT on a 12-speaker × 7-emotion evaluation. Strongest emotional expressiveness for a zero-shot AR system in the corpus.

**Training-free emotion steering (EmoSteer-TTS [[2508.03543]])**: activation steering in pre-trained DiT flow-matching models (F5-TTS, E2-TTS, CosyVoice2) at inference time without any retraining. Enables emotion conversion (E-SIM 0.280 F5+EmoSteer vs. 0.240 unsteered CosyVoice2), interpolation, erasure, and multi-emotion blending. The key finding is that pre-trained flow-matching models implicitly encode emotional information in their DiT activations, accessible without specialized training.

**Difference-in-means emotion embeddings with disentanglement (Marco-Voice [[2508.02038]])**: rotational emotion embeddings extracted as difference-in-means between emotional and neutral speech from same speakers, combined with cross-orthogonal speaker-emotion constraint and in-batch contrastive learning. Achieves emotional expression MOS 4.225 vs. CosyVoice2 3.240; SPK-SIM 0.8275 vs. 0.605 for CosyVoice2 in voice cloning. Both the REE technique and EmoSteer's difference-in-means approach demonstrate that direction arithmetic in representation space is an effective tool for emotion control.

**Multi-reward RL for emotion (GLM-TTS [[2512.14291]])**: GRPO RL with an emotion reward (and a separate laughter reward) alongside CER and SIM rewards. Demonstrates that including emotion in the RL reward trades off against pronunciation accuracy — revealing a fundamental tension in multi-attribute RL alignment for speech.

## Methods and Variants

**Emotion label conditioning.** A fixed discrete taxonomy (happy, sad, angry, neutral, etc.) is used as a one-hot or learned embedding conditioning signal. Widely used but limited by taxonomy size and cross-domain generalization.

**Reference audio style transfer.** The speaking style (including emotion) of a reference clip is transferred to synthesized speech via a style encoder. Conan [[2507.14534]] captures local speaking style at chunk level using clustering VQ. Marco-Voice [[2508.02038]] extracts a speaker-independent emotion direction from paired emotional/neutral reference clips of the same speaker.

**Training-free activation steering.** EmoSteer-TTS [[2508.03543]] steers emotion by adding difference-in-means vectors to intermediate DiT layer activations at inference time, requiring no labeled emotional training data for the target model.

**Natural language instruction conditioning.** Free-form text descriptions of emotional style are used as conditioning (see [[instruction-conditioned-tts]]). ControlSpeech [[2025.acl-long.346]] and IndexTTS2 [[2506.21619]] (T2E module) both support natural language emotion routing.

**End-to-end empathetic SLM.** [[2025.emnlp-demos.70]] (OpenS2S) trains a full speech-in speech-out model to detect and respond to user emotion through multi-stage training and automated empathetic data synthesis.

**Freestyle natural language emotion prompting.** EmoVoice [[2504.12867]] extends a Qwen2.5-based LLM-TTS backbone with freestyle text emotion prompting, demonstrating that open-vocabulary descriptions can control emotional expressiveness without a fixed taxonomy. The phoneme-boost parallel prediction head (EmoVoice-PP) additionally reduces intelligibility errors by outputting phoneme tokens as secondary supervision alongside audio tokens. The key negative finding is that automatic emotion similarity metrics correlate only ~40% with human MOS at the sentence level, and that multimodal LLMs are not yet reliable emotion judges.

**Cross-lingual emotion transfer via retrieval.** XEmoRAG [[2508.07302]] uses language-agnostic Emo2Vec embeddings to retrieve emotionally matched target-language prompts from a pool, bypassing the need for parallel bilingual emotional data and avoiding foreign-accent artefacts from direct prosody transfer between typologically distant languages.

**Reference and prompt dual-mode EVC.** ClapFM-EVC [[interspeech-2025-0203]] trains a CLAP-style model (EVC-CLAP) to align speech and text emotional representations, enabling emotional voice conversion from either a reference clip or a free-form natural language prompt. Soft-label contrastive training with both categorical and prompt labels improves emotion embedding quality over single-source training.

**Temporal emotion alignment for EVC.** Maestro-EVC [[2508.06890]] uses a cross-attention mechanism in which content representations query frame-level emotion embeddings from a speech diarisation model, producing temporally aligned, content-aware emotion features. The adaptive intensity gate in ClapFM-EVC provides scalar control analogous to Maestro-EVC's explicit prosody conditioning.

**ProsodyLM as a dialogue prosody-emotion system.** [[2507.20091]] demonstrates that replacing codec tokens with word-level prosody tokens enables emotion recognition as an emergent LLM capability, with the model correctly identifying sadness and excitement categories through probability shifts without any emotion-specific supervision.

## Major Claims

Claims are generalised propositions aggregated from paper evidence. The full claim registry with supporting paper lists is in `wiki/concepts/_evidence/emotion-synthesis.yaml`.

### Strongly Supported

- Direction arithmetic in representation space (difference-in-means between emotional and neutral speech) is an effective and generalizable technique for emotion control, applicable both during training (Marco-Voice) and at inference time without retraining (EmoSteer-TTS).
  Supporting: [[2508.02038]], [[2508.03543]], [[2506.21619]]

- RL-based emotion reward optimization can substantially improve emotion accuracy in codec LM TTS, but introduces trade-offs with pronunciation accuracy under multi-reward alignment.
  Supporting: [[2512.14291]], [[2406.02430]], [[2510.05758]]

- Automated LLM+TTS pipelines can construct large-scale emotionally annotated speech datasets at high inter-annotator consistency (κ>0.90), substantially reducing the need for costly human annotation.
  Supporting: [[interspeech-2025-0648]], [[2025.emnlp-demos.70]]

### Emerging

- GRL-based emotion–speaker disentanglement with a three-stage training curriculum enables the strongest zero-shot emotional expressiveness in AR TTS systems currently in the corpus.
  Supporting: [[2506.21619]]

- Pre-trained flow-matching DiT models implicitly encode emotional information in their intermediate activations, making training-free post-hoc emotion control possible for any such model.
  Supporting: [[2508.03543]]

- Fine-grained natural language emotion descriptions provide richer control over expressive speech synthesis than coarse categorical labels, at the cost of requiring emotion-specific training data and introducing a training-evaluation circularity risk.
  Supporting: [[2504.12867]]

- Multimodal LLMs are not yet reliable judges of emotional speech quality, exhibiting both low correlation with human ratings and inter-run instability.
  Supporting: [[2504.12867]]

- Natural language prompts can control emotional voice conversion at parity with reference speech for the majority of listeners when contrastive alignment bridges speech and text emotional representations.
  Supporting: [[interspeech-2025-0203]]

- Combining categorical emotion labels with free-form prompt labels through soft-label contrastive training (symKL loss) improves emotion embedding quality over single-source training.
  Supporting: [[interspeech-2025-0203]]

- LLM-based TTS systems fine-tuned with appropriate speaker conditioning substantially outperform conventional non-autoregressive models on emotion naturalness for low-resource languages.
  Supporting: [[2508.08715]]

- Full-face visual features encode complementary emotional information not captured by audio alone; combining visual and acoustic modalities yields meaningfully higher emotion recognition and more emotionally aligned generation.
  Supporting: [[2508.16188]]

- Explicit chain-of-thought reasoning over paralinguistic cues before generating a response improves empathetic response quality in end-to-end spoken dialogue systems.
  Supporting: [[2508.09600]]

- Synthetic emotional speech derived from TTS systems exhibits reduced expressiveness compared to real human speech, creating a systematic domain gap that degrades empathetic dialogue evaluation.
  Supporting: [[2508.09600]]

- All current spoken dialogue systems fail to sustain emotionally coherent responses across multi-turn conversations; degradation compounds across turns and is not captured by single-turn evaluation.
  Supporting: [[2508.17623]]

- Flow-matching TTS models show larger expressivity gains from fine-tuning on targeted expressive speech data (narrative-segmented audiobooks) than autoregressive models.
  Supporting: [[2509.04072]]

### Contested

> [!warning]
> Whether the fundamental trade-off between emotion accuracy and pronunciation accuracy in multi-reward RL is resolvable through better reward weighting or curriculum design, or whether it reflects a deeper representational tension, is unresolved.
> Supporting as fundamental: [[2512.14291]] / No direct contradicting evidence yet

## Relationship to Other Concepts

# TODO: expand

### Extends or Builds On
- [[prosody-control]] — emotion synthesis is realized primarily through prosodic attributes (pitch, rate, energy, voice quality); emotion synthesis systems build on and extend prosody control methods
- [[disentanglement]] — separating emotion from speaker identity is a core prerequisite for zero-shot emotion synthesis; GRL, contrastive learning, and codec-level disentanglement are all applied here

### Commonly Paired With
- [[instruction-conditioned-tts]] — natural language emotion routing (T2E in IndexTTS2, ControlSpeech) is a primary instantiation of instruction conditioning; the two concepts are deeply intertwined
- [[spoken-language-model]] — end-to-end empathetic SLMs (OpenS2S) integrate emotion synthesis into the dialogue generation loop, requiring joint emotion perception and generation

## Representative Papers

### Foundational
- [[2406.02430]] — first use of RL with emotion recognition reward in TTS; establishes that RL can substantially improve emotion category accuracy in a zero-shot system

### Influential
- [[2506.21619]] — strongest zero-shot emotion expressiveness in the corpus; introduces GRL disentanglement + T2E LLM routing as a scalable training paradigm
- [[2508.02038]] — demonstrates difference-in-means rotational emotion embeddings with cross-orthogonal disentanglement; large gap over CosyVoice2 baseline on both emotion and speaker similarity
- [[2508.03543]] — establishes training-free activation steering as viable for emotion control in pre-trained flow-matching models; broadest applicability of any method in this concept

### Recent Highlights
- [[2025.emnlp-demos.70]] — first fully open-source empathetic SLM; moves emotion synthesis into the dialogue loop with automated data construction
- [[interspeech-2025-0648]] — largest open-source emotionally labeled dataset (131.2h, 26 categories); demonstrates automated labeling at κ=0.93

## Open Questions

- How well does automated LLM+TTS empathetic data (as in OpenS2S [[2025.emnlp-demos.70]]) substitute for human-annotated empathetic dialogues?
- Can streaming voice conversion systems (Conan [[2507.14534]]) reliably capture and transfer emotion at chunk level, or does emotion modeling require full-utterance context?
- EmoSteer-TTS [[2508.03543]] assumes linearity of emotional representations in DiT activations — is this assumption valid, and does it hold for models not trained on any emotional data?
- GLM-TTS [[2512.14291]] shows emotion reward trades off against pronunciation accuracy in RL; is this fundamental, or can a better reward weighting or curriculum resolve it?
- IndexTTS2 [[2506.21619]] uses 7 basic emotion categories; how can continuous or compositional emotion control be achieved in AR systems without an exponential growth in reference data?
- Vevo2 [[2508.16332]] extends emotion expressiveness to singing via chromagram prosody; does this prosody space generalize to spoken emotion as well?
- EmoVoice [[2504.12867]] is trained and evaluated on synthetic emotional data from GPT-4o-audio; how does this circularity affect claims about emotional naturalness compared to systems trained on human emotional speech?
- ClapFM-EVC [[interspeech-2025-0203]] evaluates in an any-to-one setting; does the emotion control quality degrade in any-to-any EVC, and how does it interact with unseen target speakers?
- Cross-lingual emotion transfer (XEmoRAG [[2508.07302]]) relies on language-agnostic emotion embeddings; how well do these embeddings capture culture-specific emotional expression conventions?
- PromptEVC [[interspeech-2025-0948]] achieves 61.3% mixed-emotion accuracy versus 77.6–91.5% for single-attribute control; is compound affective state synthesis fundamentally harder, or is it a training data coverage problem?
- ZSDEVC [[interspeech-2025-1101]] operates in zero-shot settings but trains on in-the-wild podcast data (MSP-Podcast); how much of the zero-shot generalisation depends on the data diversity rather than the MI disentanglement mechanism?
- [[2508.08095]] demonstrates adapter-level paralinguistic disentanglement; can this approach extend to multi-turn emotional tracking where the user's emotional state evolves throughout a dialogue?
- SpeechRole [[2508.02013]] shows that emotion appropriateness in spoken role-playing lags naturalness and coherence, with cascaded systems outperforming end-to-end models; is this a training data or architecture limitation?
- The non-verbal speech synthesis pipeline [[2508.05385]] targets non-verbal vocalizations as a prosodic expressiveness mechanism; how does NV synthesis interact with emotional TTS quality in a unified framework?

## Trend Summary

2024: FillerSpeech [[2025.emnlp-main.1730]] (in corpus) addresses filler word insertion as a paralinguistic naturalness feature adjacent to emotion synthesis. Interspeech 2025 adds three contributions: MIKU-PAL ([[interspeech-2025-0648]]) provides the largest open-source emotionally labeled speech dataset (131.2h, 26 emotion categories using Gemini 2.0 Flash for automated labeling at Fleiss κ=0.93 vs. human κ=0.43); EME-TTS ([[interspeech-2025-0754]]) jointly models emphasis and emotion via an Emphasis Perception Enhancement (EPE) block preventing emotional prosody from distorting word-level emphasis; DiffRO ([[interspeech-2025-0704]]) demonstrates that RL-based emotion control via a differentiable multi-task reward model can teach a codec LM to generate laughter, sobs, and breaths without any emotion-labeled RL training data. 2025: OpenS2S [[2025.emnlp-demos.70]] is the first fully open-source empathetic SLM in corpus. FireRedTTS-2 ([[2509.02020]]) demonstrates implicit emotion inference from chat context without explicit emotion labels. IndexTTS2 [[2506.21619]] introduces a three-stage training curriculum for robust emotion conditioning with GRL-based disentanglement, showing the strongest emotional expressiveness for a zero-shot system; its T2E module (LLM distillation for soft emotion routing) provides a scalable path to natural language emotion control. Marco-Voice [[2508.02038]] demonstrates that difference-in-means emotion embeddings combined with cross-orthogonal disentanglement substantially improve both speaker similarity and emotional naturalness simultaneously. EmoSteer-TTS [[2508.03543]] reveals that training-free activation steering can provide emotion control for any DiT-based flow-matching model, suggesting that emotional information is implicitly encoded in pre-trained models. GLM-TTS [[2512.14291]] shows that multi-reward RL can include emotion as a trainable objective, revealing a pronunciation/emotion trade-off. The 2025 trajectory reveals two parallel directions: (1) better supervision for explicit emotion conditioning during training (IndexTTS2, Marco-Voice, GLM-TTS); (2) post-hoc control for pre-trained models without labeled emotional data (EmoSteer-TTS). Integration pass 5 adds four new approaches: freestyle natural language emotion prompting with parallel phoneme guidance (EmoVoice [[2504.12867]]), cross-lingual retrieval-based emotion transfer without parallel data (XEmoRAG [[2508.07302]]), dual-mode reference+prompt EVC with contrastive soft-label training and scalar intensity control (ClapFM-EVC [[interspeech-2025-0203]]), and frame-level temporal emotion alignment for EVC (Maestro-EVC [[2508.06890]]). The SpeechRole benchmark [[2508.02013]] reveals that emotion appropriateness is among the hardest evaluation dimensions for current spoken role-playing agents, with cascaded systems outperforming E2E models. EmoVoice [[2504.12867]] provides the clearest negative finding in this pass: sentence-level automatic emotion similarity metrics correlate only ~40% with human MOS, and multimodal LLMs fail as emotion judges — with practical implications for the evaluation infrastructure the field relies on. Integration pass 6 (Interspeech 2025) adds four contributions: PromptEVC [[interspeech-2025-0948]] extends the natural-language control interface from TTS into emotional VC, using a diffusion-based prompt mapper; ZSDEVC [[interspeech-2025-1101]] establishes zero-shot generalisation as viable in emotional VC through mutual information disentanglement and inference-time expressive guidance; the non-AR expressive VC system [[interspeech-2025-0815]] documents the intelligibility cost of discrete content representations for emotion transfer; and the dual-adapter SLM [[2508.08095]] demonstrates that heterogeneous adapter designs with ERR training enable emotionally aware spoken dialogue without modifying frozen LLM or encoder components. These contributions confirm that emotional VC and emotionally aware dialogue are now active practical concerns, not just TTS-side research topics.

## All Papers

| ID | Title | Venue | Year | Key use of this concept |
|----|-------|-------|------|------------------------|
| [[2025.emnlp-demos.70]] | OpenS2S: Advancing Fully Open-Source End-to-End Empathetic Large Speech Language Model | EMNLP | 2025 | First fully open-source empathetic speech-to-speech LLM; automated pipeline synthesizes 100K empathetic bilingual dialogues; multi-stage training for emotion detection and empathetic response generation |
| [[2509.02020]] | FireRedTTS-2: Towards Long Conversational Speech Generation for Podcast and Chatbot | arXiv | 2025 | Demonstrates implicit emotion inference from chat context history (no explicit emotion labels); achieves 76.7–93.3% accuracy across 6 emotions (surprise, sadness, happiness, concern, apology, anger) via SFT on a 15-hour curated emotion corpus |
| [[2506.21619]] | IndexTTS2: Emotionally Expressive and Duration-Controlled AR Zero-Shot TTS | arXiv | 2025 | GRL-based emotion–speaker disentanglement with Conformer emotion perceiver; T2E module distills DeepSeek-R1 into Qwen3 LoRA for natural language emotion routing; EMOS 4.22 on 12-speaker × 7-emotion evaluation |
| [[2508.02038]] | Marco-Voice Technical Report | arXiv | 2025 | Rotational emotion embeddings via difference-in-means from paired emotional/neutral speech; cross-orthogonal constraint prevents speaker-emotion entanglement; emotional expression MOS 4.225 vs. 3.240 for CosyVoice2 |
| [[2508.03543]] | EmoSteer-TTS: Training-Free Emotion-Controllable TTS via Activation Steering | arXiv | 2025 | Training-free emotion control via difference-in-means steering vectors in DiT intermediate layers; supports conversion, interpolation, erasure, blending across F5-TTS, E2-TTS, CosyVoice2 |
| [[2412.10117]] | CosyVoice 2: Scalable Streaming Speech Synthesis with Large Language Models | arXiv | 2024 | Instruction fine-tuning with emotion, speaking rate, dialect, and paralinguistic tokens ([laughter], [breath]) integrated into base training |
| [[2512.14291]] | GLM-TTS Technical Report | arXiv | 2025 | Multi-reward GRPO RL with emotion reward and laughter reward alongside pronunciation and similarity; demonstrates pronunciation/emotion trade-off in multi-reward alignment |
| [[2406.02430]] | Seed-TTS: A Family of High-Quality Versatile Speech Generation Models | arXiv | 2024 | RL-SER variant uses emotion recognition accuracy as reward; improves emotion category accuracy from 0.13–0.53 to 0.78–0.91 for surprise/happy/sad/angry |
| [[2508.16332]] | Vevo2: A Unified and Controllable Framework for Speech and Singing Voice Generation | arXiv | 2025 | Chromagram-based prosody tokenizer captures melodic/emotional prosody across speech and singing; GRPO post-training with joint intelligibility and prosody similarity rewards |
| [[2025.emnlp-main.40]] | Towards Controllable Speech Synthesis in the Era of LLMs: A Systematic Survey | EMNLP | 2025 | Surveys emotion control strategies: style tagging, reference audio, natural language description, instruction-guided; identifies simultaneous multi-attribute control as an open problem |
| [[2508.11273]] | EmoSSLSphere: Multilingual Emotional Speech Synthesis with Spherical VAE | arXiv | 2025 | Combines spherical AVD emotion vectors with HuBERT k-means prosody tokens in a FastSpeech2-style model; emotion and language conditioned in a unified framework; nMOS 4.13 on English ESD |
| [[2510.05758]] | EMORL-TTS: Reinforcement Learning for Fine-Grained Emotion Control in TTS | ICASSP | 2026 | GRPO RL applied to frozen LLM-based TTS for fine-grained continuously controllable emotion intensity; mean emotion accuracy 0.88 vs. 0.85 for EmoSphere++ — RL improves over SFT-only (0.81) for emotion control |
| [[2601.03888]] | IndexTTS 2.5 Technical Report | arXiv | 2026 | Extends IndexTTS 2's emotion-controlled synthesis to 4 languages via semantic codec compression; RLAIF emotional alignment fine-tuning as one of three optimization components |
| [[interspeech-2025-0648]] | MIKU-PAL: Automated Multimodal Method for Speech Paralinguistic and Affect Labeling | Interspeech | 2025 | Automated pipeline using Gemini 2.0 Flash for zero-shot emotion labeling at Fleiss κ=0.93 (vs. human 0.43); releases 131.2h MIKU-EmoBench covering 26 emotion categories; fine-tuned Fish-Speech achieves MOS 4.12 and emotion similarity 0.92 |
| [[interspeech-2025-0754]] | EME-TTS: Unlocking the Emphasis and Emotion Link in Speech Synthesis | Interspeech | 2025 | First joint modeling of emphasis and emotion in TTS; EPE block prevents emotional prosody from distorting word-level emphasis; MOS 4.22 vs. EmoSpeech 4.14; emphasis recognition accuracy 0.78 vs. 0.73 without EPE |
| [[interspeech-2025-0704]] | Differentiable Reward Optimization for LLM based TTS system | Interspeech | 2025 | MTR-based DiffRO teaches a codec LM to generate laughter, sobs, and breaths for emotion via RL reward without emotion-labeled RL data; emotion accuracy Happy zh/en 1.00/0.92, outperforming F5-TTS |
| [[2504.12867]] | EmoVoice | arXiv | 2025 | Freestyle text emotion prompting on a Qwen2.5-based AR codec LM; parallel phoneme-boost decoding head; emotion recall 0.424 on EmoVoice-DB; negative finding that automatic emotion similarity correlates ~40% with human MOS |
| [[2507.20091]] | ProsodyLM | arXiv | 2025 | Explicit word-level prosody tokens enable emotion recognition as emergent LM capability; codec-token baselines show near-zero emotion sensitivity |
| [[2508.02013]] | SpeechRole | arXiv | 2025 | Emotion Appropriateness dimension of speech role-playing benchmark; cascaded systems outperform E2E models on EA; dataset covers 98 roles across 111K dialogues |
| [[2508.04585]] | UniTalker | arXiv | 2025 | Emotion-guided flow-matching renderer in conversational speech-visual synthesis; predicted emotion tokens explicitly condition speech acoustics |
| [[2508.05385]] | Non-Verbal Speech Generation Pipeline | arXiv | 2025 | NV vocalizations (laughter, sighs, coughs) as a complementary expressiveness layer; automated NVS pipeline with emotion category annotations across 10 non-verbal categories |
| [[2508.06890]] | Maestro-EVC | ASRU | 2025 | Frame-level TCEM with content-aware cross-attention; explicit F0+energy conditioning; prosody augmentation for mismatch robustness; ESD evaluation with 25 listeners |
| [[2508.07302]] | XEmoRAG | arXiv | 2025 | Language-agnostic Emo2Vec retrieval for cross-lingual emotion transfer; EMOS 4.65 vs. DelightfulTTS 3.89; no parallel bilingual emotional data required |
| [[2508.08715]] | MultiGen | arXiv | 2025 | Child-friendly speech with x-vector conditioning achieves MOS 4.05 on Tamil and 3.97 on Mandarin; emotional prosody primarily from foundation model fine-tuning |
| [[interspeech-2025-0948]] | PromptEVC: Controllable Emotional Voice Conversion | Interspeech | 2025 | Diffusion-based prompt mapper bridges RoBERTa text embeddings and Emotion2Vec speech emotion embeddings; multi-attribute control (category, intensity, pitch, speed, volume); MOS 4.22, emotion similarity 81.3% |
| [[interspeech-2025-1101]] | ZSDEVC: Zero-Shot Diffusion-based Emotional VC | Interspeech | 2025 | MI disentanglement + expressive guidance for zero-shot EVC; ECA 0.53 vs. 0.256 for EMOCONV-DIFF; inference-time naturalness/emotion trade-off via guidance scale |
| [[interspeech-2025-0815]] | Non-AR Zero-Shot Expressive VC | Interspeech | 2025 | Enriched global style embedding with pitch and energy features improves emotion transfer (eMOS 3.72 vs. 2.94 for Hierspeech++); cross-attention F0 fusion for temporal prosody alignment |
| [[2508.08095]] | Dual Information Speech LMs for Emotional Conversations | arXiv | 2025 | Dual adapters for paralinguistic/linguistic separation in a frozen LLM; 90% emotion classification accuracy; competitive with Qwen-Audio on MELD while training adapters only |
| [[interspeech-2025-0203]] | ClapFM-EVC | Interspeech | 2025 | EVC-CLAP contrastive model with symKL soft-label training; AIG for scalar intensity control; 57.4% of listeners show no preference between reference and prompt modes |
| [[2508.07273]] | Incorporating Contextual Paralinguistic Understanding in Large Speech-Language Models | arXiv | 2025 | Dimensional emotion (VAD) annotations in LLM-generated CPQA training data improve empathetic reasoning in Speech-LLMs; 46% gain over baseline on contextual emotion questions |
| [[2508.09600]] | OSUM-EChat | arXiv | 2025 | Three-stage curriculum with chain-of-thought emotional reasoning; EChat-200K corpus; strongest open-source result on multi-label empathetic dialogue |
| [[2508.13028]] | Integrating Feedback Loss from Bi-modal Sarcasm Detector for Sarcastic Speech Synthesis | arXiv | 2025 | First targeted sarcasm TTS system; feedback loss from bi-modal sarcasm detector guides prosody toward detector-recognizable sarcastic patterns |
| [[2508.16188]] | AVLM (Seeing is Believing) | EMNLP | 2025 | Q-Former prefix fusion of full-face visual features into SpiritLM; +4.9 F1 on IEMOCAP emotion recognition; +4.1 F1 on generated emotional speech |
| [[2508.17623]] | EMO-Reasoning | arXiv | 2025 | First unified 3-axis benchmark for emotional coherence in spoken dialogue; reveals that all SLMs fail multi-turn emotional consistency |
| [[2509.03940]] | VoxRole | arXiv | 2025 | Paralinguistic appropriateness is the weakest capability across all tested spoken dialogue models; movie-character benchmark with acoustically-aware LLM evaluation |
| [[2509.04072]] | Computational Narrative Understanding for Expressive TTS | arXiv | 2025 | Narrative-segmented LibriQuote corpus; flow-matching fine-tuning gains larger expressivity than autoregressive fine-tuning; ContextMOS metric for narrative appropriateness |
| [[2409.00750]] | MaskGCT: Zero-Shot Text-to-Speech with Masked Generative Codec Transformer | arXiv | 2024 | Non-autoregressive emotion-conditioned synthesis via semantic token control; multilingual emotion generation evaluated across English, Chinese, French, German, Japanese, Korean |
| [[2407.04051]] | FunAudioLLM: Voice Understanding and Generation Foundation Models for Natural Interaction Between Humans and LLMs | arXiv | 2024 | Instruction fine-tuning on natural language style descriptions substantially improves emotion controllability; near-ceiling accuracy on anger, sadness, and surprise categories after instruction fine-tuning |
| [[2501.06282]] | MinMo: A Multimodal Large Language Model for Seamless Voice Interaction | arXiv | 2025 | Instruction-controlled emotion synthesis across multiple speaking styles; demonstrates emotion conditioning in aligned multimodal architecture |
| [[2402.05755]] | Spirit LM: Interleaved Spoken and Written Language Model | arXiv | 2024 | Expressive speech tokens (sentiment, pitch, style) for cross-modal emotion preservation; discrete expressive token stream enables cross-modal sentiment consistency |
