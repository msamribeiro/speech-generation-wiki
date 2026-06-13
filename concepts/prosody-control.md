---
slug: prosody-control
title: Prosody Control
aliases: [pitch control, rhythm control, intonation modelling, duration modelling, prosody prediction]
related_concepts: [emotion-synthesis, instruction-conditioned-tts, transformer-enc-dec-tts, disentanglement, voice-conversion]
last_updated: 2026-06-13
status: established
---
## Executive Summary

> [!abstract]
> Prosody control refers to the ability of a TTS or voice synthesis system to shape suprasegmental speech features — pitch, duration, and energy — independently of phoneme content. As of 2025, natural language instruction conditioning (ControlSpeech, CosyVoice 2) and factorized disentanglement (FACodec, DisCodec) are the dominant frontier approaches, extending beyond parametric FastSpeech-era methods to open-vocabulary style control. Precise duration control in autoregressive systems and the entanglement of prosody with speaker identity remain key open challenges.

## Current Status

established — Parametric prosody control (FastSpeech 2) is mature infrastructure; reference audio style transfer and natural language conditioning are well-explored. The 2025 frontier is post-hoc activation steering for pre-trained models, RL-based prosody alignment, and principled duration control in AR systems. No single approach dominates for all control modalities simultaneously.

## Why This Matters

Even syntactically identical sentences can convey different meanings, emotions, or emphases depending on prosody. A TTS system that cannot control prosody independently of content is limited to a single "averaged" speaking style, unsuitable for applications requiring expressivity such as audiobooks, virtual agents, dubbing, or accessibility tools for users who need specific speaking rates or pitches.

Prosody control is also the dominant axis of variation captured by style-controllable TTS systems: most natural language style descriptions (fast, slow, high-pitched, calm, energetic) map to prosodic attributes rather than timbre or phonetic content.

## Core Idea

Prosody control refers to the ability of a TTS or voice synthesis system to shape suprasegmental features of speech — primarily pitch (F0 contour), duration (speaking rate, rhythm), and energy (loudness/volume) — independently of the phoneme content. Prosody is the primary carrier of speaker affect, emphasis, sentence modality (statement vs. question), and naturalness. Controlling prosody is distinct from speaker identity control (timbre), though the two are often entangled in practice.

Control modalities include: (1) explicit parametric control (directly specifying pitch or duration targets), (2) reference audio style transfer (copying prosody from a reference recording), (3) natural language instructions specifying desired style, and (4) latent prosody embeddings predicted from context.

[[2025.acl-long.346]] (ControlSpeech) provides the most comprehensive controllable prosody system in the corpus, achieving speed accuracy 0.829, volume accuracy 0.894, and emotion accuracy 0.557 on VccmDataset test set A — simultaneously with zero-shot timbre cloning. The SMSD module explicitly addresses the ambiguity of natural language prosody descriptions via Gaussian mixture density modeling, enabling both accuracy (MOS-SA 3.84) and diversity (MOS-SD 4.05) in style generation.

Pitch accuracy remains the weakest controlled attribute (0.833), attributed to interference between simultaneous timbre and style conditioning. This represents an open challenge in fine-grained prosody control under multi-factor conditioning.

Complementary theoretical grounding for prosody modeling is provided by [[2025.acl-long.1471]], which establishes that pitch, loudness, and prominence are redundant with 3–8 past words but only 1–2 future words of linguistic context (measured via mutual information). This asymmetry has direct implications for prosody predictor design in TTS: adequate past-context window is ~8 words; future context adds little beyond the immediately following word. Duration and pause behave differently — they are better predicted from future context, due to sentence-boundary effects.

[[2025.acl-industry.42]] demonstrates tonal prosody control for Thai via a Phoneme-Tone BERT trained on 1M Thai sentences. The five-tone system requires explicit tone marker representation in the phoneme sequence, and contextual BERT modeling of tone-consonant interactions is the key to accurate Thai prosody generation.

## Methods and Variants

**Parametric prosody control.** FastSpeech 2 (Ren et al., 2020) introduces explicit variance adaptors for pitch, duration, and energy prediction from text, enabling direct parameter-level manipulation. Simple, interpretable, but stylistically limited. [[2025.acl-industry.42]] extends this for tonal languages by training a Phoneme-Tone BERT to provide contextual prosody representations that capture tone-consonant interactions in Thai.

**Reference audio prosody transfer.** GST (Global Style Tokens) and GMVAE models extract a style embedding from a reference audio and condition the TTS backbone. More expressive than parametric but requires a reference recording.

**Natural language prosody conditioning.** PromptTTS, InstructTTS, ControlSpeech ([[2025.acl-long.346]]) all use BERT-encoded text descriptions to condition prosody generation. Avoids the need for reference audio but introduces text-acoustic ambiguity.

**Factorized prosody disentanglement.** NaturalSpeech 3 / FACodec represent prosody as a separate codebook stream (distinct from content and acoustic detail), enabling independent codec-level prosody manipulation. Used as the backbone by [[2025.acl-long.346]].

## Major Claims

Claims are generalised propositions aggregated from paper evidence. The full claim registry with supporting paper lists is in `wiki/concepts/_evidence/prosody-control.yaml`.

### Strongly Supported

- Pitch, loudness, and prominence are redundant with 3–8 past words and only 1–2 future words of linguistic context; duration and pause are better predicted from future context, providing principled design constraints for TTS prosody predictors.
  Supporting: [[2025.acl-long.1471]]

- Natural language instruction conditioning for prosody (natural language → pitch/speed/volume/emotion) achieves competitive accuracy across multiple attributes simultaneously when combined with mixture density modeling to handle one-to-many ambiguity.
  Supporting: [[2025.acl-long.346]], [[2025.emnlp-main.180]]

- Factorized prosody disentanglement (separate prosody codebook stream in neural codecs) enables independent prosody control without reference audio, and is compatible with simultaneous zero-shot speaker cloning.
  Supporting: [[2025.acl-long.346]], [[2512.13251]], [[2025.naacl-long.242]]

### Emerging

- Training-free post-hoc prosody steering via activation difference-in-means in pre-trained DiT flow-matching models is effective for emotion/prosody control without labeled training data for the target model.
  Supporting: [[2508.03543]]

- RL-based prosody alignment (via F0-RMSE or emotion accuracy rewards) can improve prosody accuracy simultaneously with intelligibility and speaker similarity, but multi-reward trade-offs exist.
  Supporting: [[2509.00685]], [[2510.05758]]

- Probabilistic duration generation (via flow matching) produces substantially more natural temporal pauses than deterministic NAR baselines, partially closing the AR/NAR naturalness gap in pacing.
  Supporting: [[2510.02848]]

- Explicit word-level prosody token design in speech LMs enables prosody understanding and generation as emergent capabilities through pre-training, without task-specific fine-tuning.
  Supporting: [[2507.20091]]

- Conventional codec-token speech LMs do not develop prosodic understanding or generation as emergent capabilities through pre-training; the tokenisation design, not data scale, is the binding constraint.
  Supporting: [[2507.20091]]

- Training-time prosody augmentation (temporal shifting and piecewise time warping of reference prosody) improves robustness to prosody-mismatched reference conditions in emotional voice conversion without sacrificing naturalness.
  Supporting: [[2508.06890]]

- Explicit F0 and energy conditioning from a reference utterance transfers temporal prosody dynamics more faithfully than implicit prediction from latent codes in emotional VC.
  Supporting: [[2508.06890]], [[interspeech-2025-0203]]

- Natural language prosody control via a diffusion-based prompt mapper bridging text LM and speech emotion embeddings can match reference-audio-level emotion accuracy in voice conversion without reference audio at inference.
  Supporting: [[interspeech-2025-0948]]

- Enriching global style embeddings with explicit pitch and energy features substantially improves emotion transfer fidelity in expressive voice conversion beyond mel-spectrogram-only style encoding.
  Supporting: [[interspeech-2025-0815]]

- Speaker-specific phrasing behavior is a substantive source of variance in respiratory pause placement that generic multi-speaker front-end models fail to capture; modeling speaker identity explicitly in phrase-break prediction improves both objective and perceptual phrasing quality.
  Supporting: [[2509.00675]]

- Phoneme-level language models outperform subword-level models on phrase break prediction, even at smaller model sizes, because phoneme representations carry acoustic information more directly relevant to pause insertion.
  Supporting: [[2509.00675]]

- SSML-based prosody control yields substantial perceptual gains over neutral commercial TTS voices even when the underlying synthesiser is not retrained, with cascaded task decomposition (structure prediction before parameter regression) substantially outperforming joint LLM generation.
  Supporting: [[2508.17494]]

- Cross-modal attention using phoneme queries over audio-context keys is more effective than global speaker embeddings for preserving localised prosodic context in speech insertion tasks.
  Supporting: [[2508.17031]]

### Contested

> [!warning]
> Whether pitch is fundamentally harder to control than speed/volume under multi-factor conditioning — as observed in [[2025.acl-long.346]] — or whether this reflects a data or architecture limitation is unresolved. No follow-up paper has directly addressed this bottleneck.
> Supporting as fundamental: [[2025.acl-long.346]] / No contradicting evidence yet

## Relationship to Other Concepts

### Extends or Builds On
- [[transformer-enc-dec-tts]] — parametric prosody control originates in the FastSpeech family of transformer encoder-decoder TTS systems; variance adaptors for pitch, duration, and energy are the standard interface
- [[disentanglement]] — factorized prosody disentanglement (FACodec, DisCodec) is the codec-era extension of earlier GST-based disentanglement approaches

### Competes With
- [[instruction-conditioned-tts]] — natural language prosody conditioning (ControlSpeech, CosyVoice 2) competes with explicit parametric control; the key trade-off is flexibility (natural language wins) vs. precision (parametric control wins)

### Commonly Paired With
- [[emotion-synthesis]] — emotion is primarily carried through prosodic attributes (pitch, energy, rate); most emotion synthesis systems operate on prosody representations, and the two concepts are deeply intertwined
- [[instruction-conditioned-tts]] — natural language style descriptions overwhelmingly target prosodic attributes; instruction conditioning and prosody control are implemented jointly in most recent systems

## Representative Papers

### Foundational
- [[2025.acl-long.1471]] — establishes empirical MI-based constraints on prosody predictor design; the only theoretical grounding for context window requirements in this corpus

### Influential
- [[2025.acl-long.346]] — most capable joint prosody + speaker control system in the corpus; introduces SMSD for many-to-many ambiguity
- [[2025.naacl-long.242]] — fixed-length RVQ prosody latent with distilled single-step diffusion; separates timbre from time-varying prosody cleanly
- [[2510.02848]] — probabilistic flow matching for duration generation; demonstrates concrete naturalness gains over deterministic NAR duration predictors

### Recent Highlights
- [[2506.21619]] — positional embedding tying for AR duration control; first principled solution to duration control in autoregressive TTS without sequence-level supervision
- [[2508.03543]] — training-free post-hoc prosody/emotion steering via activation steering; demonstrates implicit prosody encoding in pre-trained flow-matching models
- [[2025.emnlp-main.180]] — large-scale style-tagged dataset (ParaSpeechCaps) showing that data scaling substantially improves style-prompted prosody control

## Open Questions

- Can natural language prosody control generalize to fine-grained instructions beyond the 5 attribute dimensions (pitch, speed, volume, emotion, gender) in datasets like VccmDataset?
- Is pitch the most difficult attribute to independently control under multi-factor conditioning, as suggested by [[2025.acl-long.346]]? What architectural changes would resolve this?
- Can prosody control from natural language descriptions be evaluated objectively (beyond classifier accuracy) to capture human-perceived nuance?
- [[2025.acl-long.1471]] establishes 3–8 past words as the prosody MI saturation point for English audiobooks; does this generalize to other speech styles (conversational, spontaneous) and languages?
- Duration and pause require more future context than past context per [[2025.acl-long.1471]]; how should TTS prosody predictors handle these features differently from pitch and energy?
- ProsodyLM [[2507.20091]] is trained on audiobooks only; does explicit word-level prosody tokenisation generalise to conversational or spontaneous speech where prosody patterns differ fundamentally?
- Maestro-EVC [[2508.06890]] conditions on smoothed prosody from a reference utterance; how should prosody conditioning adapt when the target content requires fundamentally different duration patterns than the reference?
- REF-VC [[2508.04996]] preserves source prosody via ABX evaluation but users may prefer target speaker style transfer; how can the prosody preservation vs. style transfer trade-off be made explicit and controllable?
- LoRA-based pitch accent correction (UtterTune [[2508.09767]]) requires user-supplied phonemic transcription; what is the minimal phonological expertise burden that makes this practical for end users?

## Trend Summary

2020–2022: FastSpeech 2 established parametric duration/pitch control as standard. GST-based approaches extended this to reference audio style transfer. 2023: PromptTTS and InstructTTS introduced natural language style conditioning. 2025: [[2025.acl-long.346]] (ControlSpeech) integrates natural language prosody control with zero-shot speaker cloning, introduces mixture density modeling for probabilistic style distributions. [[2025.acl-long.1471]] provides empirical grounding for context requirements. [[2025.acl-industry.42]] addresses tonal language prosody. StyleTTS-ZS [[2025.naacl-long.242]] introduces fixed-length RVQ prosody latent as an efficient compact representation for the time-varying style dimension. Flamed-TTS [[2510.02848]] introduces probabilistic duration and silence generation via flow matching, enabling 4-5x more natural temporal pauses than deterministic NAR baselines — addressing the NAR/AR naturalness gap in pacing. IndexTTS2 [[2506.21619]] introduces positional embedding tying (W_sem = W_num) for precise AR duration control without sequence-level supervision — the first principled solution to duration control in autoregressive TTS. DisCodec [[2512.13251]] enables independent prosody control via disentangled FSQ codec factorization. Vevo2 [[2508.16332]] introduces chromagram-based prosody tokenization bridging speech and singing prosody spaces. EmoSteer-TTS [[2508.03543]] demonstrates that prosody (as captured in emotion dimensions) can be steered post-hoc in pre-trained models without any retraining. ProsodyLM [[2507.20091]] provides a conceptually distinct approach: rather than manipulating prosody in existing models, it argues that replacing codec tokens with explicit word-level five-dimensional prosody annotations enables prosody as an emergent pre-training capability in a text LLM, achieving contrastive focus, emotion recognition, and style continuity without task-specific fine-tuning. The tradeoff is audio quality fidelity for prosodic expressiveness. UtterTune [[2508.09767]] demonstrates the practical utility of parameter-efficient LoRA for pitch accent correction in BPE-based multilingual TTS — a lightweight corrective layer that compensates for the absence of explicit G2P front-ends. Maestro-EVC [[2508.06890]] advances temporal prosody transfer in emotional VC through frame-level emotion alignment and prosody augmentation, with explicit energy alongside F0 conditioning as a differentiator from prior work.

## All Papers

| ID | Title | Venue | Year | Key use of this concept |
|----|-------|-------|------|------------------------|
| [[2025.acl-long.346]] | ControlSpeech: Towards Simultaneous and Independent Zero-shot Speaker Cloning and Zero-shot Language Style Control | ACL | 2025 | Simultaneously controls pitch, speed, volume, and emotion from natural language descriptions via SMSD module; uses FACodec prosody codes as factorized control target; introduces MOS-SA and MOS-SD evaluation metrics |
| [[2025.acl-long.1471]] | The time scale of redundancy between prosody and linguistic context | ACL | 2025 | Establishes via mutual information estimation that pitch, loudness, and prominence are redundant with 3–8 past words and 1–2 future words; provides empirical design constraints for TTS prosody predictors |
| [[2025.acl-industry.42]] | Scaling Under-Resourced TTS: A Data-Optimized Framework with Advanced Acoustic Modeling for Thai | ACL | 2025 | Trains a Phoneme-Tone BERT on 1M Thai sentences to capture tonal prosody context for five-tone language TTS; demonstrates domain-specific prosody modeling for tonal languages |
| [[2025.naacl-long.242]] | StyleTTS-ZS: Efficient High-Quality Zero-Shot TTS with Distilled Time-Varying Style Diffusion | NAACL | 2025 | Fixed-length K=50 RVQ prosody latent (time-varying style) with cross-attention compression; separates timbre (global) from prosody (time-varying), enabling distilled single-step prosody diffusion |
| [[2506.21619]] | IndexTTS2: Emotionally Expressive and Duration-Controlled AR Zero-Shot TTS | arXiv | 2025 | W_sem=W_num positional embedding tying enables precise AR duration control without sequence-level supervision; token number error rate below 0.02% at 1x scaling |
| [[2510.02848]] | Flamed-TTS: Flow Matching Attention-Free Models for Efficient Zero-shot TTS | arXiv | 2025 | Probabilistic duration and silence generators via OT-CFM; 4-5x more natural temporal pauses (4.47 per utterance vs. 1.18 for deterministic NAR); prosody variability closer to AR systems |
| [[2512.13251]] | DisCo-Speech: Controllable Zero-Shot Speech Generation with A Disentangled Speech Codec | arXiv | 2025 | DisCodec FSQ factorizes prosody as a separate subspace; AR LM continuation from prosody prompt enables independent prosody control distinct from timbre cloning |
| [[2508.16332]] | Vevo2: A Unified and Controllable Framework for Speech and Singing Voice Generation | arXiv | 2025 | Chromagram-based VQ-VAE prosody tokenizer (6.25 Hz, octave-free) bridges speech and singing prosody; EPL/IPL mixed training for explicit/implicit prosody control |
| [[2508.03543]] | EmoSteer-TTS: Training-Free Emotion-Controllable TTS via Activation Steering | arXiv | 2025 | Post-hoc emotion/prosody steering via difference-in-means vectors in DiT activations; demonstrates implicit prosody control is accessible in pre-trained models without labeled emotional training data |
| [[2025.acl-long.911]] | DNASpeech: A Contextualized and Situated TTS Dataset with Dialogue, Narrative, and Action Prompts | ACL | 2025 | New benchmark evaluating prosody appropriateness for situated contexts (Dialogue, Narrative, Action); reveals that codec LMs score higher on objective metrics while TacotronGST-style models score better on situated naturalness MOS-S |
| [[2025.coling-main.352]] | DiffStyleTTS: Diffusion-based Hierarchical Prosody Modeling for TTS | workshop | 2025 | Two-level prosody diffusion: coarse-grained implicit style (utterance-level noise conditioning) + fine-grained explicit attributes (variance adaptor with F0/energy/duration); MOS 4.18 on Mandarin, best among all compared systems |
| [[2508.11273]] | EmoSSLSphere: Multilingual Emotional Speech Synthesis with Spherical VAE | arXiv | 2025 | Spherical AVD (Arousal-Valence-Dominance) emotion embeddings combined with HuBERT k-means prosody tokens for fine-grained multilingual prosody control; F0 correlation 0.69 and energy correlation 0.83 on English ESD |
| [[2509.00685]] | MPO: Multidimensional Preference Optimization for LM-based TTS | arXiv | 2025 | RL-based prosody alignment via multidimensional preference optimization; improves log F0 RMSE from 0.337 to 0.279 simultaneously with CER and SPK-SIM improvements |
| [[2510.05758]] | EMORL-TTS: Reinforcement Learning for Fine-Grained Emotion Control in TTS | ICASSP | 2026 | Intensity-conditional emotion tokens in a codec LM, fine-tuned with GRPO using dual reward (recognition accuracy + naturalness); enables continuous intensity control through token representation |
| [[2603.18090]] | MOSS-TTS Technical Report | arXiv | 2026 | Large-scale AR TTS with rich prosody modeling; instruct fine-tuning controls speaking rate, emotion, and timbre via natural language; WER 2.04% (EN) on Seed-TTS-Eval |
| [[interspeech-2025-0723]] | Counterfactual Activation Editing for Post-hoc Prosody and Mispronunciation Correction | Interspeech | 2025 | CAE enables retraining-free prosody and pronunciation control in a frozen Tacotron 2 encoder via gradient-ascent in a manifold-preserving beta-VAE latent space; WER drops 0.151→0.056, CMOS +0.764 |
| [[interspeech-2025-0754]] | EME-TTS: Unlocking the Emphasis and Emotion Link in Speech Synthesis | Interspeech | 2025 | Variance-based emphasis features (pitch/duration deviation) with EPE block attention modulation prevent emotional prosody from distorting word-level emphasis; emphasis recognition 0.78 vs. 0.73 without EPE |
| [[interspeech-2025-0143]] | Multimodal Prosody Modeling: A Use Case for Multilingual Sentence Mode Prediction | Interspeech | 2025 | Early fusion of WavLM acoustic and XLM-RoBERTa linguistic features for multilingual sentence mode (declarative/interrogative/exclamatory) prediction; UAR 74.22% German; cross-lingual transfer up to 63.62% |
| [[interspeech-2025-0762]] | Intrasentential English in Swedish TTS: perceived English-accentedness | Interspeech | 2025 | Psychometric calibration maps a per-phoneme accentedness conditioning parameter to perceptually distinguishable accentedness levels (≥9 significant distinctions); listener preference varies by type of English insertion |
| [[2025.coling-main.518]] | ProsodyFlow: High-fidelity TTS through Conditional Flow Matching and Prosody Modeling | workshop | 2025 | CFM over WavLM-extracted prosody latent enables inference-time prosody diversity without reference audio; MOS 4.23 on LJSpeech approaching ground truth (4.25); RTF faster than VITS |
| [[2025.emnlp-main.180]] | Scaling Rich Style-Prompted Text-to-Speech Datasets | EMNLP | 2025 | ParaSpeechCaps taxonomy: 33 intrinsic (speaker-level) + 26 situational (utterance-level) style tags; scaling via acoustic matching with Gemini 2.0 Flash audio LLM substantially improves style-prompted prosody control |
| [[2507.20091]] | ProsodyLM | arXiv | 2025 | Replaces codec tokens with word-level prosody tokens (F0 range/median/slope, duration, energy); enables prosody understanding (contrastive focus, emotion recognition) as emergent pre-training capabilities |
| [[2508.02013]] | SpeechRole | arXiv | 2025 | Prosodic Consistency (PC) and Emotion Appropriateness (EA) as dedicated evaluation dimensions for speech role-playing; cascaded systems outperform E2E on prosody consistency |
| [[2508.04585]] | UniTalker | arXiv | 2025 | Emotion-guided CFM renderer with predicted emotion labels as explicit conditioning improves emotional expressiveness in conversational speech synthesis |
| [[2508.04996]] | REF-VC | arXiv | 2025 | Multi-scale pitch encoder (PBTC modules) provides F0 conditioning; ABX prosody preference tests confirm source prosody and non-verbal element preservation |
| [[2508.05385]] | Non-Verbal Speech Generation | arXiv | 2025 | Temporal-semantic alignment (TSA) for accurate positioning of non-verbal vocalizations; tag positional accuracy more important than dataset scale for NV controllability |
| [[2508.06890]] | Maestro-EVC | ASRU | 2025 | Temporal content-aware emotion module (TCEM) with cross-attention; explicit energy+F0 from Savitzky-Golay smoothed reference; prosody augmentation (shift + piecewise warp) for mismatch robustness |
| [[2508.07426]] | Scalable Controllable Accented TTS | ASRU | 2025 | Accent conditioning via accent token replacement for language token; geolocation-based data filtering improves accent-prosody conditioning |
| [[2508.08399]] | Exploring Disentangled Neural Speech Codecs | arXiv | 2025 | Prosody as the time-variant residual after instance normalization; UMAP shows quantized prosody vectors align with speaker-normalized F0 deviation |
| [[2508.09767]] | UtterTune | arXiv | 2025 | LoRA with phoneme-tag tokens for pitch accent control in Japanese; accent correctness 0.498 → 0.899; demonstrates parameter-efficient G2P-free prosody correction |
| [[interspeech-2025-0203]] | ClapFM-EVC | Interspeech | 2025 | Adaptive intensity gate (AIG) for scalar emotion strength control; flow-matching decoder produces richer prosodic variation than GAN baselines |
| [[interspeech-2025-0815]] | Towards Better Disentanglement in Non-Autoregressive Zero-Shot Expressive Voice Conversion | Interspeech | 2025 | Enriched ECAPA-TDNN style encoder with explicit F0/energy features; cross-attention F0 injection for finer prosodic alignment to target; Mix-LN reduces style-content mismatch |
| [[interspeech-2025-0948]] | PromptEVC: Controllable Emotional Voice Conversion with Natural Language Prompts | Interspeech | 2025 | Diffusion-based prompt mapper bridging RoBERTa and Emotion2Vec embeddings; multi-attribute prosody control (pitch, speed, volume, intensity, mixed emotions) from natural language prompts in EVC |
| [[2508.09702]] | M3PDB | arXiv | 2025 | Multi-modal prompt database (400K h, 18 langs) for zero-shot TTS; latency-aware cascaded similarity retrieval for prompt selection including prosody matching dimensions |
| [[2508.11224]] | Benchmarking Prosody Encoding in Discrete Speech Tokens | ASRU | 2025 | Systematic TER-based benchmark of prosody sensitivity in SSL discrete tokens; guidance for tokenisation design targeting prosody control |
| [[2508.13028]] | Integrating Feedback Loss from Bi-modal Sarcasm Detector for Sarcastic Speech Synthesis | arXiv | 2025 | Feedback loss from sarcasm detector guides prosody toward sarcasm-recognizable patterns; demonstrates classifier-guided prosody training for a specific communicative style |
| [[2508.17031]] | RephraseTTS | arXiv | 2025 | Cross-modal attention between phoneme and audio representations transfers localised prosodic context for speech insertion; outperforms global speaker embedding conditioning |
| [[2508.17494]] | Improving French Synthetic Speech Quality via SSML Prosody Control | workshop | 2025 | Cascaded LLM pipeline (QwenA for structure, QwenB for values) for automated SSML prosody markup; MOS 3.20→3.87 over commercial TTS baseline |
| [[2509.00675]] | Speaker-Conditioned Phrase Break Prediction | arXiv | 2025 | Speaker identity injection into phrasing front-end; phoneme-level PLMs (MP BERT) outperform subword models for respiratory pause placement |
| [[2509.01391]] | MixedG2P-T5 | arXiv | 2025 | SSL-derived pseudo-language labels carry implicit accent and intonation patterns for G2P-free Japanese TTS prosody modeling |
| [[2509.03940]] | VoxRole | arXiv | 2025 | Paralinguistic appropriateness as the weakest capability in role-playing spoken dialogue; acoustic feature bins as LLM judge prosody proxies |
| [[2509.04072]] | Computational Narrative Understanding for Expressive TTS | arXiv | 2025 | LibriQuote narrative-aware corpus; flow-matching models benefit more from expressive fine-tuning than autoregressive models; contextual narrative conditioning improves expressivity |
| [[2503.01710]] | Spark-TTS: An Efficient LLM-Based Text-to-Speech Model with Single-Stream Decoupled Speech Tokens | arXiv | 2025 | Global speaker tokens conditioning via BiCodec; demonstrates that decoupling speaker timbre (global) from content (local) within a single token stream improves prosodic naturalness |
| [[2502.04128]] | Llasa: Scaling Train-Time and Inference-Time Compute for Llama-based Speech Synthesis | arXiv | 2025 | Uses speech understanding verifiers including prosody metrics at inference time; demonstrates that prosody quality benefits from inference-time compute scaling via best-of-N selection |
| [[2304.09116]] | NaturalSpeech 2: Latent Diffusion Models are Natural and Zero-Shot Speech and Singing Synthesizers | arXiv | 2023 | Prosody adherence improves monotonically with reference prompt length (up to 10 seconds); in-context prosody learning without explicit prosody supervision |
| [[1712.05884]] | Natural TTS Synthesis by Conditioning WaveNet on Mel Spectrogram Predictions (Tacotron 2) | arXiv | 2017 | Pre-net information bottleneck ensures stable attention alignment; established mel spectrogram prediction as the standard prosody-implicit representation for seq2seq TTS |
| [[1609.03499]] | WaveNet: A Generative Model for Raw Audio | arXiv | 2016 | Receptive field size as binding constraint for prosodic naturalness; insufficient receptive field degrades phrase-level F0 contours even when segmental quality is high |
| [[1703.10135]] | Tacotron: Towards End-to-End Speech Synthesis | arXiv | 2017 | Multiple-frames-per-step attention decoder improves training stability and implicit prosody modeling; established end-to-end prosody modeling from characters |
| [[2402.01912]] | Natural language guidance of high-fidelity text-to-speech with synthetic annotations | arXiv | 2024 | Natural language prosody conditioning via automatic acoustic labeling; accent control achievable in a single model covering dozens of accents |
| [[2402.08093]] | BASE TTS: Lessons from building a billion-parameter TTS model on 100K hours | arXiv | 2024 | Scale-dependent emergence of prosodic naturalness on linguistically complex inputs; BPE reduces sequence length improving long-context prosody |
