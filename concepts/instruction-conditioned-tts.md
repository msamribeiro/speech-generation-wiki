---
slug: instruction-conditioned-tts
title: Instruction-Conditioned TTS
aliases: [controllable TTS, natural language style control, prompt-based TTS, text-prompted synthesis, style-controllable TTS]
related_concepts: [prosody-control, emotion-synthesis, zero-shot-tts, rlhf-speech, spoken-language-model, disentanglement]
last_updated: 2026-06-01
status: emerging
---
## Executive Summary

> [!abstract]
> Instruction-conditioned TTS enables speech style control via free-form natural language descriptions rather than scalar parameters or discrete labels. As of 2026, systems like Fish Audio S2 and Qwen3-TTS demonstrate that open-source instruction-conditioned TTS now surpasses frontier closed-source systems (GPT-4o-audio, Gemini 2.5 Flash TTS) on expressive paralinguistic tasks. The core challenge — the many-to-many semantic gap between language descriptions and acoustic realizations — is partially addressed by mixture density modeling and RLHF, but precise pitch control under multi-factor conditioning remains the weakest link.

## Current Status

emerging — Instruction-conditioned TTS has advanced rapidly from attribute-based slot-filling (PromptTTS, 2022) to open-vocabulary instruction following at scale (Fish Audio S2, Qwen3-TTS, 2026). The paradigm is now integrated into production TTS systems and spoken dialogue backends. Key open challenges: standardized evaluation (no single benchmark), generalization to multi-language instructions, and sufficient style-annotated data.

## Why This Matters

Natural language instructions provide a user-friendly, flexible, and open-vocabulary control interface. Unlike scalar sliders or fixed emotion categories, free-form descriptions can express nuanced combinations of style attributes (e.g., "slightly hushed but energetic") and can generalize to descriptions not seen during training if the underlying model captures cross-modal semantic alignment. This makes instruction-conditioned TTS particularly important for applications such as audiobook production, interactive voice assistants, expressive dubbing, and content creation platforms.

The core technical challenge is the semantic gap between natural language (sparse, high-level, polysemous) and acoustic features (dense, continuous, physically grounded). This gap creates ambiguity in both directions, formalized by [[2025.acl-long.346]] as the many-to-many problem: different descriptions can correspond to the same audio (many-to-one), and a single description can correspond to varying acoustic realizations (one-to-many).

## Core Idea

Instruction-conditioned TTS (also called natural language style control or text-prompted TTS) is the ability to shape the speaking style of synthesized speech using free-form natural language descriptions as conditioning, rather than explicit scalar parameters (pitch value, speed factor) or discrete style labels. A typical input might be: "The woman speaks in a joyful, fast-paced manner." The system must interpret the semantic content of the instruction and generate speech whose prosody, emotion, pace, energy, and overall expressive quality matches the description.

This is distinct from speaker conditioning (which controls who speaks, not how), from emotion labels (which require a fixed discrete taxonomy), and from prosody transfer (which copies style from a reference audio rather than interpreting language).

As of 2025, [[2025.acl-long.346]] (ControlSpeech) represents the most capable instance of this paradigm in the corpus: it is the first system to simultaneously combine zero-shot voice cloning with zero-shot language-style control. Using FACodec disentanglement and a Style Mixture Semantic Density (SMSD) module based on Gaussian mixture density networks, ControlSpeech achieves MOS-SA 3.84 and MOS-SD 4.05 on its many-to-many test set, substantially outperforming prior style-only baselines (PromptStyle: 3.45/3.53, InstructTTS: 3.57/3.48). Its out-of-domain style generalization (test set C) shows significantly smaller degradation than all baselines, indicating better robustness to novel description phrasings.

Earlier systems in this paradigm include PromptTTS (attribute-based discrete control), InstructTTS (three-stage training with semantic tokens), and PromptTTS 2 (diffusion model with LLM-generated descriptions). None of these support zero-shot speaker cloning.

## Methods and Variants

**Attribute-based natural language control (PromptTTS family).** Instructions are decomposed into discrete attributes (gender, pitch, speed, volume, emotion) that map to control signals. Limited expressivity but easy to evaluate.

**Semantic embedding conditioning (InstructTTS, TextrolSpeech/Salle).** A pre-trained language encoder (BERT, Sentence-BERT) encodes the style description into a continuous embedding, which conditions the TTS backbone. More expressive than attribute-based but suffers from one-to-many ambiguity.

**Mixture density modeling for style (ControlSpeech [[2025.acl-long.346]]).** Explicitly models the stochastic mapping from style description to audio style representation via a Gaussian mixture density network (SMSD). Sampling from the mixture produces diverse acoustic realizations matching the description. Integrated with zero-shot speaker cloning for the first time.

**Diffusion-based one-to-many capture (PromptTTS 2).** Uses a diffusion model conditioned on the style description to generate a distribution over compatible acoustic styles, addressing one-to-many but without zero-shot speaker generalization.

## Major Claims

Claims are generalised propositions aggregated from paper evidence. The full claim registry with supporting paper lists is in `wiki/concepts/_evidence/instruction-conditioned-tts.yaml`.

### Strongly Supported

- Mixture density modeling (SMSD) for the style description → acoustic style mapping substantially improves both style accuracy and acoustic diversity compared to single-embedding conditioning, addressing the many-to-many ambiguity problem.
  Supporting: [[2025.acl-long.346]]

- Inline positional vocal tag injection (learning instruction-following entirely from rich-transcription ASR annotations without manual style labeling) is a scalable and effective approach for fine-grained, position-specific paralinguistic control.
  Supporting: [[2603.08823]], [[2412.10117]]

- Large-scale style-annotated datasets with rich paralinguistic tags substantially improve style-prompted TTS control compared to small-scale or weakly annotated baselines.
  Supporting: [[2025.emnlp-main.180]], [[2603.08823]]

### Emerging

- Cross-modal distillation from a text LLM can transfer instruction-following capability to the speech modality without any speech instruction annotations, enabling instruction-conditioned behavior in speech models trained on unannotated ASR data.
  Supporting: [[2025.acl-long.388]]

- Adding a "thinking pattern" (LLM reasoning about the style description before generating) substantially improves instruction interpretation for complex style descriptions compared to direct conditioning.
  Supporting: [[2601.15621]]

### Contested

> [!warning]
> Whether standardized benchmarks (EmergentTTS-Eval, Fish Audio Instruction Benchmark) capture meaningfully different capabilities than standard MOS and WER, and how they relate to human perceptual judgments of instruction adherence, remains contested.
> Introducing new benchmarks: [[2603.08823]], [[2601.15621]] / No cross-benchmark correlation study available yet

## Relationship to Other Concepts

### Extends or Builds On
- [[prosody-control]] — instruction conditioning is primarily implemented as natural language prosody control; most style descriptions target prosodic attributes; the two paradigms share the same conditioning interface
- [[disentanglement]] — zero-shot speaker cloning combined with instruction conditioning (ControlSpeech) requires disentangling timbre from style at the codec level

### Competes With
- [[zero-shot-tts]] — zero-shot TTS without style control (VALL-E family) handles speaker identity but not speaking style; instruction conditioning extends zero-shot TTS to open-vocabulary style without requiring a reference recording

### Commonly Paired With
- [[emotion-synthesis]] — emotion is a primary target of natural language style instructions; instruction-conditioned TTS and emotion synthesis are implemented jointly in most recent systems (ControlSpeech, CosyVoice 2, IndexTTS2)
- [[rlhf-speech]] — preference optimization is a natural complement to instruction conditioning for aligning generated style with human intent
- [[spoken-language-model]] — aligned multimodal SLMs (CosyVoice 2, OpenS2S) use instruction-conditioned TTS as the speech generation backend for empathetic and stylistically controlled dialogue responses

## Representative Papers

### Foundational
- [[2025.acl-long.346]] — first joint zero-shot speaker cloning + zero-shot style control; introduces SMSD and the many-to-many evaluation framework

### Influential
- [[2412.10117]] — establishes instruction fine-tuning with inline paralinguistic tags as a scalable integration paradigm; widely used as a backend in subsequent SLM papers
- [[2025.emnlp-main.40]] — provides the most comprehensive taxonomy of instruction conditioning strategies (~80 systems); identifies instruction-guided control as the frontier with no standardized evaluation
- [[2025.emnlp-main.180]] — demonstrates that large-scale style-tagged datasets substantially improve style-prompted prosody control; provides an open-source 2709h dataset

### Recent Highlights
- [[2603.08823]] — inline vocal tag injection at scale; 81.88% win rate on EmergentTTS-Eval vs. GPT-4o-audio; establishes open-source instruction-conditioned TTS as surpassing frontier closed-source on expressive tasks
- [[2601.15621]] — "thinking pattern" via Qwen3 backbone reasoning for complex style interpretation; outperforms GPT-4o-mini-tts by 28+ APS points on Chinese instruction benchmark

## Open Questions

- Can instruction-conditioned TTS generalize to instructions in languages other than the synthesis language?
- The many-to-many problem is formally characterized but not fully solved: SMSD improves diversity but still shows lower MOS-SA than style-only baselines on pitch accuracy ([[2025.acl-long.346]]).
- How many hours of style-annotated data are needed for robust generalization? ControlSpeech notes that tens of thousands of hours may be necessary.
- Can RLHF or preference optimization be used to better align generated style with human intent, complementing the MDN training objective?

## Trend Summary

2022–2023: PromptTTS (ICASSP 2023) established the attribute annotation → text conditioning paradigm. InstructTTS and TextrolSpeech extended this with more natural descriptions. 2023–2024: PromptTTS 2 and AudioBox introduced LLM-generated descriptions and flow-matching generators. 2025: [[2025.acl-long.346]] (ControlSpeech) introduces the first joint zero-shot timbre cloning + zero-shot style control system. CosyVoice 2 [[2412.10117]] integrates instruction fine-tuning with fine-grained paralinguistic tags ([laughter], [breath], emphasis) into base training rather than as a separate model. Fish Audio S2 [[2603.08823]] demonstrates inline vocal tag injection (position-specific prosodic control via inline tokens like [angry], [whisper], [laugh]), learning instruction-following entirely from data via a rich-transcription ASR annotation pipeline — the most fine-grained and scalable instruction-conditioned TTS in the corpus, achieving APS 85.2 / DSD 81.1 on Fish Audio Instruction Benchmark. Qwen3-TTS [[2601.15621]] adds a "thinking pattern" for voice design via natural language instruction, using Qwen3 backbone reasoning to interpret complex style descriptions. The [[2025.emnlp-main.40]] survey provides a comprehensive taxonomy of instruction conditioning strategies across ~80 systems. 2026: Fish Audio S2 achieves 81.88% win rate on EmergentTTS-Eval (evaluating paralinguistic instruction following) against GPT-4o-audio and Gemini 2.5 Flash TTS — demonstrating that open-source instruction-conditioned TTS now surpasses frontier closed-source systems on expressive paralinguistic tasks.

## All Papers

| ID | Title | Venue | Year | Key use of this concept |
|----|-------|-------|------|------------------------|
| [[2025.acl-long.346]] | ControlSpeech: Towards Simultaneous and Independent Zero-shot Speaker Cloning and Zero-shot Language Style Control | ACL | 2025 | Proposes SMSD mixture density module for many-to-many style control; first joint zero-shot cloning + zero-shot style control system; defines and evaluates MOS-SA and MOS-SD metrics |
| [[2025.acl-long.388]] | DiVA: Distilling an End-to-End Voice Assistant Without Instruction Training Data | ACL | 2025 | Shows that instruction-following capability can be transferred from a text LLM to the speech modality via cross-modal distillation without any speech instruction annotations, demonstrating that instruction adherence in speech models does not require explicit speech instruction data |
| [[2025.emnlp-demos.70]] | OpenS2S: Advancing Fully Open-Source End-to-End Empathetic Large Speech Language Model | EMNLP | 2025 | Conditions speech response generation on empathetic instructions (emotion label + response style) synthesized by a large LLM; uses CosyVoice2 as the controllable TTS backend for empathetic dialogue data construction |
| [[2412.10117]] | CosyVoice 2: Scalable Streaming Speech Synthesis with Large Language Models | arXiv | 2024 | Unified instruction fine-tuning: 1,500h instruction data with inline tags ([laughter], [breath], emphasis) integrated into base training; MOS-I 4.06 vs. CosyVoice-Instruct 3.09 |
| [[2407.05407]] | CosyVoice: A Scalable Multilingual Zero-shot TTS based on Supervised Semantic Tokens | arXiv | 2024 | CosyVoice-instruct: fine-tuned on 556h of paralinguistic instructions covering emotion, gender, speaking rate, pitch, laughter, breath, emphasis; x-vector speaker embedding replaced by text instructions |
| [[2603.08823]] | Fish Audio S2 Technical Report | arXiv | 2026 | Inline vocal tag injection (position-specific [angry], [whisper], [laugh]) learned from rich-transcription ASR annotations; achieves 81.88% win rate on EmergentTTS-Eval vs. GPT-4o-audio and Gemini 2.5 Flash TTS; best APS 85.2 on Fish Audio Instruction Benchmark |
| [[2601.15621]] | Qwen3-TTS Technical Report | arXiv | 2026 | "Thinking pattern" for voice design via natural language instruction: Qwen3 backbone reasons about style descriptions before generating; APS 85.2 / DSD 81.1 on InstructTTSEval (zh), outperforming GPT-4o-mini-tts by 28+ APS points |
| [[2025.emnlp-main.40]] | Towards Controllable Speech Synthesis in the Era of LLMs: A Systematic Survey | EMNLP | 2025 | Four-strategy taxonomy (style tagging, reference speech, natural language description, instruction-guided); identifies instruction-guided control as the frontier with no standardized evaluation; surveys ~80 systems with multi-attribute controllability checkmarks |
| [[2025.acl-long.911]] | DNASpeech: A Contextualized and Situated TTS Dataset with Dialogue, Narrative, and Action Prompts | ACL | 2025 | Introduces DNA (Dialogue, Narrative, Action) situated prompts as a new instruction-conditioning paradigm for contextualized TTS; 12-hour dataset with human-annotated situational prompts from movies; defines a leaderboard for instruction-conditioned TTS in context-aware settings |
| [[2603.18090]] | MOSS-TTS Technical Report | arXiv | 2026 | Instruction fine-tuning with natural language style instructions (speaking rate, emotion, timbre) alongside zero-shot voice cloning; large-scale training demonstrates scalability of instruction conditioning |
| [[2025.emnlp-main.180]] | Scaling Rich Style-Prompted Text-to-Speech Datasets | EMNLP | 2025 | ParaSpeechCaps: first large-scale open-source dataset (2709h) with 59 rich paralinguistic style tags (intrinsic speaker-level + situational utterance-level); automatic scaling via perceptual speaker similarity and Gemini audio LLM; improves Parler-TTS intrinsic tag recall from 33% to 69.5% |
| [[interspeech-2025-0047]] | Revival with Voice: Multi-modal Controllable Text-to-Speech Synthesis | Interspeech | 2025 | RV-TTS generates speech from face images while allowing fine-grained speech characteristic control via descriptive text; demonstrates instruction-conditioned voice generation from visual face prompt |
