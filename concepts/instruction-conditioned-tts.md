---
slug: instruction-conditioned-tts
title: Instruction-Conditioned TTS
aliases: [controllable TTS, natural language style control, prompt-based TTS, text-prompted synthesis, style-controllable TTS]
related_concepts: [prosody-control, emotion-synthesis, zero-shot-tts, rlhf-speech, spoken-language-model, disentanglement]
last_updated: 2026-05-30
---
## What it is

Instruction-conditioned TTS (also called natural language style control or text-prompted TTS) is the ability to shape the speaking style of synthesized speech using free-form natural language descriptions as conditioning, rather than explicit scalar parameters (pitch value, speed factor) or discrete style labels. A typical input might be: "The woman speaks in a joyful, fast-paced manner." The system must interpret the semantic content of the instruction and generate speech whose prosody, emotion, pace, energy, and overall expressive quality matches the description.

This is distinct from speaker conditioning (which controls who speaks, not how), from emotion labels (which require a fixed discrete taxonomy), and from prosody transfer (which copies style from a reference audio rather than interpreting language).

## Why it matters

Natural language instructions provide a user-friendly, flexible, and open-vocabulary control interface. Unlike scalar sliders or fixed emotion categories, free-form descriptions can express nuanced combinations of style attributes (e.g., "slightly hushed but energetic") and can generalize to descriptions not seen during training if the underlying model captures cross-modal semantic alignment. This makes instruction-conditioned TTS particularly important for applications such as audiobook production, interactive voice assistants, expressive dubbing, and content creation platforms.

The core technical challenge is the semantic gap between natural language (sparse, high-level, polysemous) and acoustic features (dense, continuous, physically grounded). This gap creates ambiguity in both directions, formalized by [[2025.acl-long.346]] as the many-to-many problem: different descriptions can correspond to the same audio (many-to-one), and a single description can correspond to varying acoustic realizations (one-to-many).

## Current state of the art

As of 2025, [[2025.acl-long.346]] (ControlSpeech) represents the most capable instance of this paradigm in the corpus: it is the first system to simultaneously combine zero-shot voice cloning with zero-shot language-style control. Using FACodec disentanglement and a Style Mixture Semantic Density (SMSD) module based on Gaussian mixture density networks, ControlSpeech achieves MOS-SA 3.84 and MOS-SD 4.05 on its many-to-many test set, substantially outperforming prior style-only baselines (PromptStyle: 3.45/3.53, InstructTTS: 3.57/3.48). Its out-of-domain style generalization (test set C) shows significantly smaller degradation than all baselines, indicating better robustness to novel description phrasings.

Earlier systems in this paradigm include PromptTTS (attribute-based discrete control), InstructTTS (three-stage training with semantic tokens), and PromptTTS 2 (diffusion model with LLM-generated descriptions). None of these support zero-shot speaker cloning.

## Key variants and sub-approaches

**Attribute-based natural language control (PromptTTS family).** Instructions are decomposed into discrete attributes (gender, pitch, speed, volume, emotion) that map to control signals. Limited expressivity but easy to evaluate.

**Semantic embedding conditioning (InstructTTS, TextrolSpeech/Salle).** A pre-trained language encoder (BERT, Sentence-BERT) encodes the style description into a continuous embedding, which conditions the TTS backbone. More expressive than attribute-based but suffers from one-to-many ambiguity.

**Mixture density modeling for style (ControlSpeech [[2025.acl-long.346]]).** Explicitly models the stochastic mapping from style description to audio style representation via a Gaussian mixture density network (SMSD). Sampling from the mixture produces diverse acoustic realizations matching the description. Integrated with zero-shot speaker cloning for the first time.

**Diffusion-based one-to-many capture (PromptTTS 2).** Uses a diffusion model conditioned on the style description to generate a distribution over compatible acoustic styles, addressing one-to-many but without zero-shot speaker generalization.

## Comparison to alternatives

Prosody-conditioned TTS (reference audio style transfer) copies style from a reference recording — more accurate for known speakers but requires an example utterance. Instruction-conditioned TTS is more flexible for novel or imagined styles. Emotion-label TTS is a special case with a closed label set; instruction conditioning generalizes this to open vocabulary at the cost of more ambiguous supervision. Zero-shot TTS without style control (VALL-E family) handles speaker identity but does not allow speaking style adjustment — the combination of both is the frontier addressed by [[2025.acl-long.346]].

## Year-on-year trajectory

2022–2023: PromptTTS (ICASSP 2023) established the attribute annotation → text conditioning paradigm. InstructTTS and TextrolSpeech extended this with more natural descriptions. 2023–2024: PromptTTS 2 and AudioBox introduced LLM-generated descriptions and flow-matching generators. 2025: [[2025.acl-long.346]] (ControlSpeech) introduces the first joint zero-shot timbre cloning + zero-shot style control system. CosyVoice 2 [[2412.10117]] integrates instruction fine-tuning with fine-grained paralinguistic tags ([laughter], [breath], emphasis) into base training rather than as a separate model. Fish Audio S2 [[2603.08823]] demonstrates inline vocal tag injection (position-specific prosodic control via inline tokens like [angry], [whisper], [laugh]), learning instruction-following entirely from data via a rich-transcription ASR annotation pipeline — the most fine-grained and scalable instruction-conditioned TTS in the corpus, achieving APS 85.2 / DSD 81.1 on Fish Audio Instruction Benchmark. Qwen3-TTS [[2601.15621]] adds a "thinking pattern" for voice design via natural language instruction, using Qwen3 backbone reasoning to interpret complex style descriptions. The [[2025.emnlp-main.40]] survey provides a comprehensive taxonomy of instruction conditioning strategies across ~80 systems. 2026: Fish Audio S2 achieves 81.88% win rate on EmergentTTS-Eval (evaluating paralinguistic instruction following) against GPT-4o-audio and Gemini 2.5 Flash TTS — demonstrating that open-source instruction-conditioned TTS now surpasses frontier closed-source systems on expressive paralinguistic tasks.

## Open questions

- Can instruction-conditioned TTS generalize to instructions in languages other than the synthesis language?
- The many-to-many problem is formally characterized but not fully solved: SMSD improves diversity but still shows lower MOS-SA than style-only baselines on pitch accuracy ([[2025.acl-long.346]]).
- How many hours of style-annotated data are needed for robust generalization? ControlSpeech notes that tens of thousands of hours may be necessary.
- Can RLHF or preference optimization be used to better align generated style with human intent, complementing the MDN training objective?

## Papers

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
