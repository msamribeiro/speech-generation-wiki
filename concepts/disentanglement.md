---
slug: disentanglement
title: Disentanglement
aliases: [content-speaker disentanglement, style disentanglement, speech factorization, representation disentanglement]
status: established
related_concepts: [voice-conversion, self-supervised-speech, speaker-adaptation, prosody-control, emotion-synthesis]
last_updated: 2026-06-05
---

## Executive Summary

> [!abstract]
> Disentanglement in speech refers to the separation of the speech signal into independently controllable factors — primarily content (what is said), speaker identity (who is speaking), speaking style (how it is said), and optionally emotion and language. A well-disentangled representation allows each factor to be manipulated independently: changing the speaker while preserving content, or transferring speaking style without altering timbre. Disentanglement is the core enabling technology for voice conversion and speaker-adaptive TTS, and is increasingly achieved through factorized codecs, SSL-based content extraction, activation-space direction arithmetic, and clustering vector quantization.

## Current Status

established — Disentanglement is a well-recognized design requirement in modern TTS and VC systems, not a novel research question. The open problems have shifted from whether disentanglement is possible to how to achieve fine-grained, multi-factor separation (content + timbre + prosody + emotion) without information leakage or reconstruction loss. Multiple complementary approaches now coexist: codec-level factorization (DisCodec [[2512.13251]], FACodec), activation-space direction arithmetic (Marco-Voice [[2508.02038]], EmoSteer-TTS [[2508.03543]]), and latent-space compression (StyleTTS-ZS [[2025.naacl-long.242]], M3-TTS [[2512.04720]]).

## Why This Matters

Disentanglement is the core enabling technology for voice conversion and speaker-adaptive TTS. Without clean separation of content from speaker characteristics, transferring a voice to a new speaker inevitably introduces content distortion (changed words or phonemes) or fails to fully transfer the speaker identity. The degree of disentanglement also determines how well a system generalizes to unseen speakers (zero-shot capability).

More broadly, multi-factor disentanglement enables applications that require simultaneous independent control: voice cloning while changing emotion, or prosody transfer without speaker identity leakage.

## Core Idea

A disentangled speech model learns separate representations for each speech factor such that manipulating one factor does not affect others. This is typically achieved through one or more of: (1) information bottlenecks that force speaker information out of content embeddings; (2) SSL-derived discrete tokens (HuBERT, WavLM) that cluster by phoneme rather than speaker; (3) vector quantization with orthogonality or mutual information constraints; (4) factorized codec codebooks where separate RVQ groups encode separate factors; or (5) adversarial training with gradient reversal layers (GRL) to prevent factor leakage.

## Methods and Variants

**Information bottleneck methods.** AutoVC uses a bottleneck to force out speaker information from content embeddings. VQMIVC adds mutual information minimization between content and speaker to explicitly decorrelate them.

**SSL-based content extraction.** Using discrete HuBERT tokens as content removes most speaker information since K-means clusters phonemes, not speakers. This is the approach in Conan [[2507.14534]] and LM-VC.

**Clustering VQ style encoding.** Conan's [[2507.14534]] Adaptive Style Encoder uses CVQ (clustering vector quantization with contrastive loss) to capture detailed speaking style (prosody, emotion) at chunk level, separate from global timbre and phonetic content. This is a notable advance over systems that only capture global timbre.

**Factorized codec.** NaturalSpeech 3 uses factorized VQ to disentangle prosody from content and acoustic detail at the codec level. ControlSpeech [[2025.acl-long.346]] builds on this for multi-factor control. DisCodec [[2512.13251]] introduces a two-stage FSQ-based factorization that explicitly separates content, prosody, and timbre with graduated soft orthogonality constraints: strict orthogonality between timbre and prosody (β_t = 1e-4) but partial overlap between prosody and content (β_c = 0.01, since they share temporal dynamics). The key insight is that hard orthogonality causes information loss while no constraint causes leakage — soft targets are superior.

**Chromagram-based prosody tokenization.** Vevo2 [[2508.16332]] introduces a VQ-VAE prosody tokenizer based on chromagram features (octave-free, notation-free) rather than F0 or mel-subbands. This bridges the prosody distribution gap between speech and singing, enabling unified prosody disentanglement across domains without MIDI annotation.

**Difference-in-means emotion direction extraction.** Marco-Voice [[2508.02038]] and EmoSteer-TTS [[2508.03543]] both demonstrate that subtracting neutral speech representations from emotional speech representations (difference-in-means) produces a direction vector that captures emotion while suppressing speaker-specific variation. Cross-orthogonal constraint (Marco-Voice) forces speaker and emotion embeddings to be perpendicular, enabling independent control. This vector-arithmetic approach connects speech disentanglement to the activation engineering literature from LLMs.

**Waveform-latent disentanglement trade-off.** LongCat-AudioDiT [[2603.29339]] demonstrates a counterintuitive finding: higher VAE reconstruction fidelity (larger latent dim, higher frame rate) does not improve downstream TTS quality — the diffusion model must absorb excess complexity, limiting generation quality. This reveals a fundamental tension between reconstruction fidelity and generative tractability in continuous latent spaces.

**Articulatory coding.** RT-VC [[2025.acl-demo.37]] uses SPARC articulatory coding for physically grounded content-speaker disentanglement: content is extracted as pseudo-EMA articulatory features (causal EMA inverter), speaker as WavLM embeddings, F0/periodicity separately. This interpretable disentanglement operates in a real-time pipeline on CPU.

**Instance normalization for speaker-prosody separation.** [[2508.08399]] uses instance normalization on WavLM hidden residuals to separate time-invariant speaker statistics (µ, σ) from time-variant prosodic content, providing a label-free mechanism that does not require external supervision. Group-residual VQ (GRVQ) quantises the speaker component; a trade-off between LLM compatibility (discrete speaker codes) and speaker fidelity is empirically quantified.

**Frame-level cross-modal contrastive learning.** SecoustiCodec [[2508.02849]] trains a speech codec with a frame-level contrastive objective that aligns phoneme embeddings and speech frame embeddings in a shared space, forcing speech tokens to cluster by phoneme rather than speaker. This approach achieves better semantic-paralinguistic separation than SSL distillation without requiring external phoneme labels at inference.

**Temporal emotion alignment with content-aware attention.** Maestro-EVC [[2508.06890]] uses a cross-attention mechanism in which content representations query frame-level emotion embeddings, producing temporally aligned emotion features that vary with phoneme context rather than remaining utterance-uniform. A gradient reversal layer on a content classifier then suppresses residual phonetic leakage from the emotion representation.

**Spectral disentanglement in codec embeddings.** SPCODEC [[interspeech-2025-0196]] explicitly separates low-frequency and high-frequency content within the codec's quantization stage through supervised reconstruction objectives on each frequency partition, then uses the quantized low-frequency representation to predict and remove redundancy in the high-frequency stream. This is a form of spectral-semantic disentanglement that reduces cross-band information leakage.

**Self-supervised three-way codec factorisation (FreeCodec).** [[interspeech-2025-1440]] disentangles speech into content (WavLM-guided 50 Hz stream), prosody (low-mel-bin 7 Hz stream), and speaker (continuous ECAPA-TDNN embedding) using only self-supervised objectives — no phoneme labels or F0 supervision. Moving WavLM supervision to the decoder path rather than the encoder improves speaker-content disentanglement for voice conversion.

**Dual-adapter paralinguistic-linguistic separation (2508.08095).** [[2508.08095]] implements heterogeneous adapters — a pooled-transformer adapter for utterance-level paralinguistic attributes (emotion, pitch, energy) and a compressed-MLP adapter for frame-level linguistic content — and enforces their independence through Equivalence Replacement Regularization (ERR), which randomises the source of paralinguistic embeddings during linguistic task training. This prevents adapter collapse into task-specific vectors.

**Interpretability probing as a disentanglement diagnostic.** [[interspeech-2025-0115]] applies token co-occurrence analysis, t-SNE visualisation, and CLUB mutual information estimation to four codecs, confirming that even explicitly disentangled codecs fail to fully separate pitch, and that pitch remains the most diffusely encoded attribute regardless of whether semantic objectives were used. This provides a principled diagnostic framework for evaluating disentanglement in codec-based representations.

**Mix-style layer normalisation for VC content-speaker mismatch.** [[interspeech-2025-0815]] introduces mix-style layer normalisation (Mix-LN) — computing affine parameters as a convex combination of sample style and batch-shuffled style — to reduce content representations' dependence on source speaker. Ablation shows removing Mix-LN causes the most severe WER degradation on LibriTTS (19.83%), confirming it is the primary mechanism for reducing train-inference mismatch in the content encoder.

**Dual-stream SSL encoding for disentangled codec tokens.** [[interspeech-2025-0468]] introduces an architectural separation where SSL-derived tokens serve as RVQ-1 and waveform features encode only the residual. This structural approach to disentanglement substantially outperforms distillation for RVQ-1 semantic content purity.

**Speaker-invariant speech tokenizer (DC-Spin).** [[interspeech-2025-0246]] achieves speaker-content disentanglement at the tokenizer level by training a dual-codebook encoder to produce tokens that are simultaneously phonetically rich and speaker-invariant, using an auxiliary large codebook to improve primary codebook quality without increasing output token rate.

## Major Claims

Claims are generalised propositions aggregated from paper evidence.

### Strongly Supported

- Hard orthogonality constraints between speech factors cause information loss; soft orthogonality targets (allowing partial overlap where factors share structure) achieve better disentanglement-reconstruction balance.
  Supporting: [[2512.13251]], [[2508.02038]]

- SSL-derived discrete tokens (HuBERT, WavLM) provide strong content-speaker disentanglement without explicit supervision because K-means clustering operates at the phoneme level rather than the speaker level.
  Supporting: [[2507.14534]], [[2025.acl-demo.37]], [[2025.acl-long.346]]

- Factorized codec codebooks (separate RVQ groups for content, prosody, timbre) enable independent downstream control of each speech factor without modifying the generation architecture.
  Supporting: [[2512.13251]], [[2025.acl-long.346]], [[2025.acl-long.1043]]

- Fully discrete disentanglement of phonetic, prosodic, and speaker information in a speech codec is achievable without phoneme labels or F0 supervision, at the cost of a small reconstruction quality degradation.
  Supporting: [[2508.08399]]

- Quantizing speaker vectors into discrete codes introduces a measurable trade-off between LLM compatibility and speaker identity fidelity; continuous speaker representations preserve speaker identity better.
  Supporting: [[2508.08399]]

### Emerging

- Difference-in-means direction vectors in pre-trained model activations capture semantic emotion factors while suppressing speaker-specific variation, enabling training-free emotion-speaker disentanglement.
  Supporting: [[2508.02038]], [[2508.03543]]

- Chunk-level style encoding (prosody, emotion) rather than global timbre encoding enables fine-grained speaking style transfer in streaming contexts, not just voice identity.
  Supporting: [[2507.14534]]

- Frame-level emotion representations derived from speech emotion diarisation models improve prosody transfer fidelity and intelligibility in emotional voice conversion over utterance-level representations.
  Supporting: [[2508.06890]]

- Random feature erasure during training reduces a model's over-reliance on noise-sensitive SSL representations without requiring information bottleneck machinery, providing a lightweight disentanglement regulariser.
  Supporting: [[2508.04996]]

- Chromagram-based prosody tokenization, being octave-free and notation-free, is a more universal prosody representation across speech and singing domains than F0-based alternatives.
  Supporting: [[2508.16332]]

- Self-supervised three-way factorisation of speech into content, prosody, and speaker streams can achieve disentanglement quality competitive with supervised methods at substantially lower bitrate when WavLM supervision is applied to the decoder rather than the encoder.
  Supporting: [[interspeech-2025-1440]]

- Training-time randomisation of paralinguistic embedding sources (speech, text caption, or absent) prevents adapter collapse into task-specific representations and preserves multi-turn contextual generalisation in SLM adapters.
  Supporting: [[2508.08095]]

- Pitch is the most diffusely encoded speech attribute in neural audio codecs regardless of explicit disentanglement objectives, indicating that RVQ-based architectures cannot fully separate pitch from other speech attributes.
  Supporting: [[interspeech-2025-0115]]

### Contested

> [!warning]
> Whether disentanglement quality (independence of factors) and reconstruction quality (fidelity of the synthesized signal) are fundamentally in tension, or whether this is a training objective design problem, is unresolved. LongCat-AudioDiT [[2603.29339]] suggests the tension is real in continuous latent spaces; DisCodec [[2512.13251]] argues it can be managed with soft constraints in discrete spaces.
> Tension is real: [[2603.29339]] / Manageable: [[2512.13251]]

## Relationship to Other Concepts

### Replaces or Supersedes
- Speaker x-vector / d-vector conditioning — provides disentanglement only at the global speaker level, not prosody or emotion; replaced by factorized approaches in frontier systems

### Extends or Builds On
- [[self-supervised-speech]] — SSL representations are the primary source of speaker-disentangled content features
- [[neural-codec]] — factorized codec design (FACodec, DisCodec) is the primary vehicle for codec-level disentanglement

### Competes With
- End-to-end conditioning (direct speaker embedding without explicit disentanglement) — simpler to train but provides less controllability and typically lower zero-shot generalization

### Commonly Paired With
- [[voice-conversion]] — VC success depends directly on content-speaker disentanglement quality
- [[prosody-control]] — prosody is the third major factor after content and speaker; disentangling it enables independent prosody transfer
- [[emotion-synthesis]] — emotion-speaker disentanglement is increasingly required for emotionally controllable TTS

## Representative Papers

### Foundational
- [[2507.14534]] — Conan: three-way factorization of content, global timbre, and local style/emotion with CVQ; first streaming system to exceed offline VC in speaker similarity

### Influential
- [[2025.acl-long.346]] — ControlSpeech: uses frozen FACodec for simultaneous multi-factor control; ablation shows decoupling is essential
- [[2512.13251]] — DisCodec: principled graduated soft orthogonality for FSQ-based three-way factorization; superior VC disentanglement vs. Vevo and CosyVoice2

### Recent Highlights
- [[2508.02038]] — Marco-Voice: cross-orthogonal constraint forces speaker-emotion perpendicularity; activation engineering approach to disentanglement
- [[2508.03543]] — EmoSteer-TTS: training-free emotion steering via difference-in-means vectors in DiT activations
- [[2508.16332]] — Vevo2: chromagram-based prosody tokenizer bridges speech and singing disentanglement

### Cautionary / Negative Evidence
- [[2603.29339]] — LongCat-AudioDiT: higher VAE reconstruction fidelity does not improve downstream TTS quality; reveals fundamental tension between disentanglement fidelity and generative tractability in continuous latent spaces

## Open Questions

- How well does Conan's CVQ style encoder actually capture emotion independently of timbre? No isolated emotion evaluation is reported in [[2507.14534]].
- Can disentanglement be achieved without large SSL models, which are incompatible with low-resource deployment?
- Is there a fundamental trade-off between disentanglement quality and reconstruction quality in continuous latent spaces, or is this a training objective design problem?
- Does activation-space direction arithmetic (difference-in-means) scale to more complex multi-factor disentanglement (simultaneous emotion + prosody + speaker control)?
- Can factorized codecs disentangle language from speaker in multilingual settings, enabling true cross-lingual voice identity transfer?
- SecoustiCodec [[2508.02849]] requires duration-aligned phoneme labels for training; can frame-level contrastive learning be extended to a fully unsupervised setting?
- [[2508.08399]] shows a measurable trade-off between speaker code discretization and speaker identity fidelity; what codebook design mitigates this while preserving LLM compatibility?
- How does XEmoRAG's [[2508.07302]] retrieval-based emotion transfer compare to explicit disentanglement in terms of accent artefacts and naturalness in cross-lingual settings?
- FreeCodec [[interspeech-2025-1440]] t-SNE visualisation suggests partial speaker clustering in content embeddings despite the WavLM guidance; how much speaker information leaks into the content stream under adversarial analysis?
- [[interspeech-2025-0115]] finds pitch most opaque in all four codecs; is pitch fundamentally incompatible with RVQ-based disentanglement, or would a dedicated pitch stream (as in FreeCodec's prosody encoder) resolve this?
- DC-Spin [[interspeech-2025-0246]] achieves speaker-invariant tokens without explicit F0 supervision; how do its tokens compare on emotional VC tasks where prosodic transfer is critical?

## Trend Summary

2021–2023: VQMIVC and NANSY established unsupervised disentanglement using VQ and mutual information. 2024–2025: Streaming systems ([[2507.14534]]) demonstrated chunk-level disentanglement with CVQ. ControlSpeech [[2025.acl-long.346]] used frozen FACodec for simultaneous multi-factor control. 2025: DisCodec [[2512.13251]] introduced graduated soft orthogonality constraints — a principled resolution of the disentanglement-reconstruction trade-off in discrete codecs. Vevo2 [[2508.16332]] bridged speech and singing prosody through chromagram-based tokenization. Marco-Voice [[2508.02038]] and EmoSteer-TTS [[2508.03543]] applied difference-in-means direction arithmetic to emotion-speaker disentanglement, connecting speech research to LLM activation engineering. StyleTTS-ZS [[2025.naacl-long.242]] achieved efficient disentanglement via fixed-length RVQ latents for global timbre and time-varying prosody. REF-VC [[2508.04996]] demonstrated random feature erasure as a lightweight training-time regulariser that prevents timbre leakage from SSL streams without information bottleneck machinery. SecoustiCodec [[2508.02849]] proposed frame-level cross-modal contrastive learning as an alternative to SSL distillation, with explicit three-way modelling of acoustic, semantic, and paralinguistic representations. Maestro-EVC [[2508.06890]] demonstrated frame-level temporal emotion alignment with content-aware cross-attention and prosody augmentation for mismatched reference conditions in emotional VC. [[2508.08399]] provided the first systematic ablation of fully-discrete codec disentanglement from WavLM features, quantifying the trade-off between speaker code discretization and speaker identity fidelity. The field is converging on multiple complementary approaches: codec-level factorization (DisCodec, FACodec, SecoustiCodec, SPCODEC), activation-space direction arithmetic (Marco-Voice, EmoSteer-TTS), latent-space compression (StyleTTS-ZS, M3-TTS), and regularisation-based approaches (REF-VC). The primary open frontier has shifted to multi-factor simultaneous disentanglement (content + timbre + prosody + emotion) and cross-domain transfer (speech-singing, cross-lingual). Integration pass 6 adds further evidence on multiple fronts: FreeCodec [[interspeech-2025-1440]] demonstrates self-supervised three-way factorisation at ultra-low bitrate, providing a practical comparison point for supervised codec families; the interpretability analysis [[interspeech-2025-0115]] systematically confirms that pitch remains the hardest attribute to isolate across all codec architectures; DC-Spin [[interspeech-2025-0246]] contributes speaker-invariant tokenizer design grounded in SLM downstream evaluation; the dual-adapter SLM [[2508.08095]] introduces ERR as a regularisation strategy for preventing adapter-level information entanglement; and the cross-modal singing VC [[interspeech-2025-0816]] extends disentanglement to the underexplored speech-singing domain gap.

## All Papers

| ID | Title | Venue | Year | Role in this concept |
|----|-------|-------|------|---------------------|
| [[2507.14534]] | Conan: A Chunkwise Online Network for Zero-Shot Adaptive Voice Conversion | arXiv (ASRU 2025) | 2025 | Three-way disentanglement: content (Emformer HuBERT distillation), global timbre, local style/emotion (CVQ) |
| [[2025.acl-long.346]] | ControlSpeech: Towards Simultaneous and Independent Zero-shot Speaker Cloning and Language Style Control | ACL | 2025 | Frozen FACodec for simultaneous multi-factor control; ablation shows decoupling is essential |
| [[2025.acl-long.1043]] | OZSpeech: One-step Zero-shot Speech Synthesis with Learned-Prior-Conditioned Flow Matching | ACL | 2025 | Uses FACodec factorized tokens (prosody, content, acoustic detail, timbre) for disentangled prior code generation |
| [[2025.acl-demo.37]] | RT-VC: Real-Time Zero-Shot Voice Conversion with Speech Articulatory Coding | ACL | 2025 | Articulatory coding (SPARC EMA inverter) for physically grounded content-speaker disentanglement |
| [[2506.21619]] | IndexTTS2: Emotionally Expressive and Duration-Controlled AR Zero-Shot TTS | arXiv | 2025 | GRL-based emotion–speaker disentanglement; emotion perceiver forced orthogonal to speaker perceiver |
| [[2508.02038]] | Marco-Voice Technical Report | arXiv | 2025 | Cross-orthogonal constraint forces speaker-emotion perpendicularity; difference-in-means approach |
| [[2508.03543]] | EmoSteer-TTS: Training-Free Emotion-Controllable TTS via Activation Steering | arXiv | 2025 | Difference-in-means emotion direction vectors from DiT activations; implicit disentanglement in pre-trained FM models |
| [[2512.13251]] | DisCo-Speech: Controllable Zero-Shot Speech Generation with A Disentangled Speech Codec | arXiv | 2025 | DisCodec two-stage FSQ with graduated soft orthogonality; superior VC disentanglement vs. Vevo and CosyVoice2 |
| [[2508.16332]] | Vevo2: A Unified and Controllable Framework for Speech and Singing Voice Generation | arXiv | 2025 | Chromagram-based prosody tokenizer disentangles melodic prosody from timbre; bridges speech and singing |
| [[2025.naacl-long.242]] | StyleTTS-ZS: Efficient High-Quality Zero-Shot TTS with Distilled Time-Varying Style Diffusion | NAACL | 2025 | Fixed-length RVQ latents for timbre (global) vs. prosody (time-varying) separation |
| [[2512.04720]] | M3-TTS: Multi-modal DiT Alignment and Mel-latent for Zero-shot TTS | arXiv | 2025 | Mel-VAE compresses mel features into compact 40-dim latent, disentangling temporal and feature-dimension complexity |
| [[2604.00688]] | OmniVoice: Towards Omnilingual Zero-Shot TTS with Diffusion Language Models | arXiv | 2026 | Speaker-independent content tokens via full-codebook masking; LLM initialization for multilingual content-speaker disentanglement |
| [[2025.acl-long.654]] | Language-Codec: Bridging Discrete Codec Representations and Speech Language Models | ACL | 2025 | MCRVQ redistributes information load across early codebook channels to equalize entropy per channel |
| [[2508.04141]] | Parallel GPT: Harmonizing the Independence and Interdependence of Acoustic Tokens | arXiv | 2025 | Parallel Tokenizer extracts semantically independent tokens and acoustically dependent tokens simultaneously |
| [[2508.11273]] | EmoSSLSphere: Multilingual Emotional Speech Synthesis with Spherical VAE | arXiv | 2025 | Spherical AVD emotion space combined with HuBERT-derived prosody tokens; emotion and speaker kept orthogonal |
| [[interspeech-2025-0575]] | VoiceMark: Zero-Shot Voice Cloning-Resistant Watermarking Approach Leveraging Speaker-Specific Latents | Interspeech | 2025 | SpeechTokenizer RVQ disentanglement (VQ-1 content, VQ-2-8 speaker) for VC-resistant watermarking |
| [[interspeech-2025-0596]] | Facilitating Personalized TTS for Dysarthric Speakers Using Knowledge Anchoring and Curriculum Learning | Interspeech | 2025 | Teacher-student speaker encoder implicitly disentangles timbre from articulation quality |
| [[interspeech-2025-0723]] | Counterfactual Activation Editing for Post-hoc Prosody and Mispronunciation Correction | Interspeech | 2025 | Beta-VAE latent space separates prosody from pronunciation; VQ-VAE prototype anchoring prevents unintended shifts |
| [[2603.29339]] | LongCat-AudioDiT: High-Fidelity Diffusion TTS in the Waveform Latent Space | arXiv | 2026 | Higher VAE reconstruction fidelity does not improve TTS quality; fundamental disentanglement-tractability tension |
| [[2508.02849]] | SecoustiCodec | arXiv | 2025 | Frame-level cross-modal contrastive learning between phonemes and speech frames; explicit three-way separation of semantic, acoustic, and paralinguistic representations; VAE+FSQ quantisation |
| [[2508.04996]] | REF-VC | arXiv | 2025 | Random feature erasure regularises SSL/ASR fusion to prevent timbre leakage; implicit alignment suppresses noise-carrying frame reconstruction |
| [[2508.06890]] | Maestro-EVC | ASRU | 2025 | Frame-level emotion diarisation model for temporally aligned emotion embeddings; GRL on content classifier; prosody augmentation for mismatched reference conditions |
| [[2508.07302]] | XEmoRAG | arXiv | 2025 | Language-agnostic Emo2Vec embeddings as cross-lingual retrieval signal; retrieval-based emotion transfer avoids direct prosody disentanglement across typologically distant languages |
| [[2508.07426]] | Scalable Controllable Accented TTS | ASRU | 2025 | kNN-VC timbre augmentation for speaker-accent disentanglement in low-resource accent synthesis; geolocation-based accent label discovery |
| [[2508.08399]] | Exploring Disentangled Neural Speech Codecs | arXiv | 2025 | Systematic ablation of fully-discrete three-factor codec disentanglement from WavLM; instance normalization for time-invariant speaker separation; GRVQ for speaker discretization; quantifies fidelity cost of discrete speaker codes |
| [[2508.08961]] | DualSpeechLM | arXiv | 2025 | Decouples understanding-oriented (USToken) and acoustic-generation (WavTokenizer) speech token streams to resolve the semantic-acoustic tension in unified speech LLMs |
| [[2508.11326]] | MoE-TTS | arXiv | 2025 | Modality-based MoE routing separates text and speech parameter spaces; frozen text experts prevent catastrophic forgetting of language understanding during speech fine-tuning |
| [[interspeech-2025-0196]] | SPCODEC | Interspeech | 2025 | Spectral disentanglement in codec quantization stage; supervised low/high frequency partition with cross-band prediction; reduces high-frequency redundancy without separate subband codec |
| [[interspeech-2025-0203]] | ClapFM-EVC | Interspeech | 2025 | CLAP-style contrastive alignment separates emotional from non-emotional content; adaptive intensity gate (AIG) provides scalar control of emotion strength; any-to-one EVC with three inference modes |
| [[interspeech-2025-1440]] | FreeCodec: A Disentangled Neural Speech Codec with Fewer Tokens | Interspeech | 2025 | Self-supervised three-way factorisation (WavLM-guided content, low-mel-bin prosody, ECAPA-TDNN speaker) at 0.45 kbps; routing WavLM supervision to decoder improves VC disentanglement |
| [[interspeech-2025-0115]] | Bringing Interpretability to Neural Audio Codecs | Interspeech | 2025 | Systematic multi-codec analysis revealing that pitch remains diffusely encoded in all four codec families; confirms content concentrates in RVQ-1 but leaks at low frame rates |
| [[interspeech-2025-0246]] | DC-Spin: A Speaker-invariant Speech Tokenizer | Interspeech | 2025 | Dual-codebook speaker-invariant tokenizer; large auxiliary codebook improves small primary codebook quality; speaker-invariance verified through SLM downstream evaluation |
| [[2508.08095]] | Dual Information Speech Language Models for Emotional Conversations | arXiv | 2025 | Heterogeneous dual adapters for paralinguistic/linguistic separation; ERR training prevents adapter collapse; paralinguistic pooled-transformer vs. linguistic compressed-MLP |
| [[interspeech-2025-0468]] | DualCodec | Interspeech | 2025 | Dual-stream SSL encoding structurally separates semantic content (SSL path, RVQ-1) from acoustic residual (waveform path, RVQ-rest); substantially outperforms distillation for semantic disentanglement |
| [[interspeech-2025-0815]] | Non-AR Zero-Shot Expressive VC with Better Disentanglement | Interspeech | 2025 | Mix-LN reduces train-inference content-speaker mismatch; perturbation similarity loss suppresses source leakage; mHuBERT-147 discrete units for cross-lingual zero-shot capability |
| [[interspeech-2025-0816]] | Bridging Speech and Singing via SSAN | Interspeech | 2025 | Cross-modal speaker-singer embedding alignment via cosine loss; cycle training addresses absence of paired speech-singing data; extends disentanglement to speech-singing domain gap |
| [[2508.11224]] | Benchmarking Prosody Encoding in Discrete Speech Tokens | ASRU | 2025 | TER-based measurement of prosody/speaker/phonetic separation in SSL tokens; shows moving average on features adjusts prosody-speaker disentanglement trade-off |
| [[2508.15565]] | Any-to-any Speaker Attribute Perturbation for Voice Anonymization | arXiv | 2025 | Batch mean pseudo-speaker loss as adversarial disentanglement of speaker identity; STFT-domain perturbation preserving speech content while obscuring speaker identity |
| [[2508.15931]] | QvTAD | arXiv | 2025 | Differential attention to suppress shared timbral information between two speaker embeddings; DAG-based transitivity augmentation for pairwise timbre attribute labels |
| [[2508.16188]] | AVLM (Seeing is Believing) | EMNLP | 2025 | SpiritLM interleaved semantic/style/pitch token design as disentangled expressive speech representation; Q-Former fusion disentangles visual from acoustic conditioning |
| [[2508.17031]] | RephraseTTS | arXiv | 2025 | Cross-modal attention implicitly disentangles phoneme content from audio-context style; triplet style matching loss promotes intra-utterance style consistency |
| [[2508.19205]] | VibeVoice Technical Report | arXiv | 2025 | Separate acoustic and semantic tokenizers trained with task-specific objectives; semantic tokenizer discarded after pretraining; acoustic-semantic disentanglement at tokenization level |
| [[2509.00503]] | Entropy-based Coarse and Compressed Semantic Speech Representation Learning | arXiv | 2025 | Entropy-guided compression as implicit disentanglement: removes redundant from informative tokens in SSL representations |
