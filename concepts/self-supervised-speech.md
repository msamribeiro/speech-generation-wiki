---
slug: self-supervised-speech
title: Self-Supervised Speech Representations and Foundation Models
aliases: [SSL speech, HuBERT, WavLM, wav2vec 2.0, speech foundation model, self-supervised pre-training, SenseVoice, Whisper, large speech model]
related_concepts: [neural-codec, disentanglement, voice-conversion, speaker-adaptation, spoken-language-model, speech-to-speech]
last_updated: 2026-06-10
status: mature-infrastructure
---

## Executive Summary

> [!abstract]
> Self-supervised speech representations are learned feature extractors trained on large unlabeled speech corpora without human annotation; the dominant models (HuBERT, WavLM, wav2vec 2.0) are now stable infrastructure components rather than active research frontiers in the TTS/VC space. As of 2026, the field's attention has shifted from developing better SSL models toward comparing SSL against large-scale supervised foundation models (Whisper, SenseVoice) as codec semantic targets and content extractors, with emerging evidence that ASR-supervised features substantially outperform SSL for ultra-low-frame-rate codec design. The unresolved question is whether the SSL vs. supervised distinction will collapse as multi-task foundation models subsume both paradigms.

## Current Status

mature-infrastructure — HuBERT-Large and WavLM-Large are ubiquitous, stable components in TTS evaluation pipelines (WavLM-Large for SPK-SIM), codec semantic distillation (WavLM in Mimi), and VC content extraction (HuBERT in disentanglement systems); they are not actively being improved within the speech synthesis research community, which has shifted attention to supervised foundation models (SenseVoice, Whisper) as superior alternatives for downstream codec and SLM tasks. New SSL model development is largely happening outside the TTS corpus.

## Why This Matters

SSL representations provide rich phonetic and speaker information without requiring labeled data. In the TTS/VC/SCA context, they serve multiple roles:
- **Content extraction for VC:** [[2507.14534]] (Conan) distills HuBERT-Large to create a streaming content extractor; HuBERT is widely used as the phonetic "content" signal in disentanglement-based VC systems.
- **Semantic distillation for codecs:** SSL features (WavLM in Mimi, w2v-bert-2 in DualCodec) are distilled into RVQ-1 codec tokens to make them semantically meaningful. However, [[2510.00981]] shows that ASR features outperform SSL features for guiding dynamic frame merging at ultra-low frame rates.
- **Speaker similarity measurement:** WavLM-Large is the standard speaker verification model for SPK-SIM computation in zero-shot TTS evaluation ([[2509.19668]]).
- **Speech token baseline for SLMs:** [[2412.17048]] uses HuBERT-Large discrete clusters (2048 units at 50 Hz) as the speech modality in its modality-evolving study.
- **Speech understanding/diarization:** [[2025.findings-emnlp.424]] uses pyannote.audio (which builds on SSL embeddings) for speaker diarization and vocal consistency checking in the InteractSpeech pipeline.

## Core Idea

Self-supervised speech representations are learned feature extractors trained on large corpora of unlabeled speech using objectives that do not require human annotation. The dominant approaches use masked prediction (HuBERT, WavLM) or contrastive learning (wav2vec 2.0) to extract representations that capture phonetic, semantic, and speaker-level information in learned embeddings.

**Speech foundation models** are a broader, overlapping category: large pre-trained speech models (100k+ hours, often multi-task) that serve as general-purpose backbones for downstream tasks including TTS, VC, codec design, and evaluation. They may be trained with purely self-supervised objectives (HuBERT, WavLM) or with supervised objectives at scale (Whisper, SenseVoice), or both. The defining characteristic is scale and task-agnostic pre-training — the model is trained once and then used as a frozen feature extractor or fine-tuning base across many downstream tasks.

Key SSL models: wav2vec 2.0 (contrastive + masked prediction, 95M/317M params), HuBERT (masked prediction over K-means cluster targets, up to 300M params), WavLM (masked prediction + denoising, up to 600M params). Key supervised foundation models: Whisper (ASR-supervised, multilingual, up to 1.5B params), SenseVoice-Small (ASR + emotion + language ID, 230M params). HuBERT and WavLM are the most widely used in speech synthesis research; SenseVoice is emerging as a preferred backbone for codec semantic extraction.

## Methods and Variants

**Masked prediction (HuBERT).** Offline cluster assignment creates pseudo-labels; model predicts cluster IDs for masked frames. Multiple iterations refine the cluster quality. Widely used as content extractor in VC [[2507.14534]] and as speech tokenizer for SLMs [[2412.17048]].

**Masked prediction + denoising (WavLM).** Extends HuBERT with a masked speech denoising objective to improve robustness to noise. WavLM-Large is the standard speaker verification backbone for SPK-SIM in TTS evaluation [[2509.19668]].

**Contrastive learning (wav2vec 2.0).** Contrastive loss over masked speech quantized representations. The w2v-bert-2 model (used in DualCodec) is a variant; [[2510.00981]] shows this performs poorly for dynamic frame merging due to leaked acoustic information.

**Large-scale supervised foundation models (Whisper, SenseVoice).** Trained with text supervision (ASR labels, emotion labels, language ID) at scale rather than self-supervised objectives. Whisper (OpenAI, up to 1.5B params, 680k hours multilingual) is the dominant speech recognition backbone. SenseVoice-Small (230M params, 300k hours, multi-task: ASR + emotion + language ID) is shown in [[2510.00981]] to produce more semantically concentrated features than SSL models for codec design. The key property is that supervised training aligns representations tightly with text semantics, suppressing the paralinguistic variability that makes SSL features problematic as low-frame-rate codec targets.

**Multi-task / universal speech encoders.** A newer generation that combines SSL and supervised objectives (e.g. data2vec 2.0, SONAR). Aim for universal representations that generalise across speech tasks. Underrepresented in current corpus; expected to grow as the field consolidates around foundation model paradigm.

## Major Claims

Claims are generalised propositions aggregated from paper evidence. The full claim registry with supporting paper lists is in `wiki/concepts/_evidence/self-supervised-speech.yaml`.

### Strongly Supported

- WavLM-Large is the de facto standard backbone for automatic speaker similarity (SPK-SIM) evaluation in zero-shot TTS, and conclusions about speaker similarity in the literature are implicitly conditioned on this choice.
  Supporting: [[2509.19668]], [[2025.acl-long.682]], [[2507.14534]]

- Self-supervised speech representations trained with masked language modeling objectives encode sufficient lexical and syntactic information to outperform earlier causal spoken language models on zero-resource linguistic benchmarks.
  Supporting: [[2209.03143]]

- HuBERT discrete cluster representations in speech language models retain paralinguistic variability that causes a combinatorial lexical explosion, substantially degrading semantic coherence relative to text LMs operating at equivalent sequence lengths.
  Supporting: [[2412.17048]], [[2025.acl-long.682]]

### Emerging

- ASR-supervised representations (SenseVoice, Whisper) produce more semantically concentrated features than SSL representations for low-frame-rate codec design, enabling dramatically better WER preservation at ultra-low rates where SSL-guided approaches collapse.
  Supporting: [[2510.00981]], [[2412.17048]]

- Supervised CTC training objectives directly on codec RVQ-1 achieve better phonetic tokenization metrics than SSL pseudo-label distillation, without requiring an external SSL teacher model.
  Supporting: [[interspeech-2025-0669]]

- Convolutional SSL-like encoders in RVQ codecs have wide receptive fields that cause context-dependent tokenization of identical audio segments (DRI), degrading downstream language model training.
  Supporting: [[2025.acl-long.1498]]

- WavLM-based SSL features improve paralinguistic expressiveness in voice conversion but introduce timbre leakage and noise sensitivity that require explicit mitigation; random feature erasure during training is an effective lightweight regulariser.
  Supporting: [[2508.04996]]

- HuBERT discrete tokens provide strong content-speaker disentanglement for voice conversion because K-means clustering naturally suppresses speaker-specific variation; this remains valuable even as supervised foundation models outperform SSL for codec semantic targets.
  Supporting: [[2508.06890]], [[2508.08399]]

- Speaker-invariant speech tokenization via SSL encoder fine-tuning (Spin, DC-Spin) produces tokens that improve SLM downstream tasks and resynthesis intelligibility simultaneously, with n-gram predictability and PNMI as more informative evaluation proxies than ABX.
  Supporting: [[interspeech-2025-0246]]

- Training a codec content encoder using WavLM cosine similarity loss routed to the decoder branch rather than the encoder achieves cleaner content-speaker disentanglement for voice conversion than routing the same loss to the encoder.
  Supporting: [[interspeech-2025-1440]]

- Entropy-based adaptive segmentation of SSL speech tokens preserves more task-relevant linguistic information than fixed-length downsampling at equivalent compression ratios, with 15 Hz achieving the best trade-off between compression and ASR/ST performance.
  Supporting: [[2509.00503]]

- Optimal compression granularity for SSL tokens differs systematically between understanding and generation tasks: ASR/ST benefit from moderate compression near phoneme rate, while voice conversion requires finer density to maintain acoustic fidelity.
  Supporting: [[2509.00503]]

- SSL-derived discrete tokens at standard 50 Hz rates contain substantial redundancy removable without degrading recognition performance; 70% sequence length reduction at 15 Hz improves ASR WER vs. the original 50 Hz baseline.
  Supporting: [[2509.00503]]

- Training k-means clustering on emotionally expressive speech increases prosodic sensitivity in the resulting SSL discrete tokens for most SSL model and layer combinations.
  Supporting: [[2508.11224]]

- SSL models using frame-wise masked prediction capture relative prosodic contours within utterances rather than absolute acoustic magnitudes, making them insensitive to global intensity rescaling.
  Supporting: [[2508.11224]]

### Contested

> [!warning]
> Whether SSL representations remain necessary for any codec or TTS component once supervised foundation models (SenseVoice, Whisper) are available is unresolved; the corpus contains evidence that supervised models outperform SSL for semantic codec targets but that SSL retains advantages for prosodic and acoustic richness.
> Supporting: [[2510.00981]] (supervised better for semantic) / Contradicting: [[2507.14534]], [[2025.coling-main.518]] (SSL preferred for content/prosody extraction)

## Relationship to Other Concepts

### Extends or Builds On
- [[neural-codec]] — SSL models are distilled into RVQ-1 codec tokens to ensure semantic content; the shift from SSL to supervised foundation models (SenseVoice) as distillation targets is actively redefining this relationship

### Competes With
- [[neural-codec]] — text-supervised ASR features encode more concentrated semantic information but require labeled data. SSL features are more universal (can encode prosody, speaker, emotion) but this breadth is a disadvantage when only semantic content is needed (as in low-frame-rate codecs). For VC content extraction, HuBERT is preferred because it captures phonetic (not speaker) content when its discrete clusters are used. For speaker verification, WavLM-Large embeddings are the standard.

### Commonly Paired With
- [[disentanglement]] — HuBERT discrete clusters are the standard content representation in disentanglement-based VC, providing phonetic content without speaker identity; WavLM-Large provides speaker embeddings; SSL models supply both poles of the content-speaker disentanglement
- [[voice-conversion]] — SSL content extractors (HuBERT) and speaker encoders (WavLM) are the two dominant feature sources for VC systems that operate without parallel training data
- [[spoken-language-model]] — SSL discrete tokens (HuBERT K-means) served as the first-generation speech modality for SLMs before neural codecs became dominant; SSL encoder outputs are still used as speaker verification and quality metrics throughout SLM evaluation pipelines

## Representative Papers

### Foundational
- [[2025.acl-long.682]] — comprehensive survey situating SSL models (Wav2vec 2.0, HuBERT, WavLM) within the SpeechLM training pipeline; taxonomizes SSL by objective type and downstream role

### Influential
- [[2209.03143]] — AudioLM demonstrates the complementary role of SSL semantic tokens and codec acoustic tokens; w2v-BERT XL 7th-layer features serve as the semantic tier that enables long-range coherence in the first hierarchical codec LM
- [[2412.17048]] — controlled study identifying paralinguistic variability in HuBERT tokens (Factor C) as the dominant cause of SLM coherence failure, providing the strongest evidence for why SSL tokens are a problematic SLM modality
- [[2510.00981]] — FlexiCodec ablations demonstrating that ASR-supervised features (SenseVoice) substantially outperform SSL (w2v-bert-2) for dynamic frame merging at ultra-low codec rates; key inflection point for the SSL vs. supervised question
- [[2507.14534]] — Conan: distills HuBERT-Large into a streaming content extractor for VC, demonstrating SSL's continued value for content-only feature extraction despite its limitations as a semantic codec target

### Recent Highlights
- [[interspeech-2025-0669]] — PAST: supervised CTC training on RVQ-1 outperforms SSL pseudo-label distillation on phonetic metrics without any external SSL teacher, challenging the assumption that SSL is necessary for hybrid tokenization

### Cautionary / Negative Evidence
- [[2025.acl-long.1498]] — shows that the convolutional encoder architecture underlying SSL-inspired codec encoders introduces DRI (Discrete Representation Inconsistency), a structural failure mode that degrades downstream LM training; not fixable by SSL pre-training alone

## Open Questions

- Can SSL models be explicitly trained to suppress paralinguistic variability (Factor C from [[2412.17048]]) while retaining phonetic content, making them better codec targets?
- Is there a sweet spot between ASR-supervised (semantically concentrated) and SSL features (acoustically rich) for low-frame-rate codec design?
- WavLM-Large is used for SPK-SIM evaluation; how sensitive are conclusions about speaker similarity to the choice of SSL backbone?
- As supervised foundation models (SenseVoice, Whisper) increasingly replace SSL in codec design, does the field still need SSL-specific pre-training, or will the SSL vs. supervised distinction collapse?
- Do multi-lingual supervised foundation models (Whisper-large-v3, MMS) generalise better as codec semantic targets across languages than English-centric SSL models?

## Trend Summary

2020–2022: wav2vec 2.0, HuBERT, WavLM established SSL as the standard for downstream speech tasks. 2023–2024: SSL features became the semantic distillation target for low-frame-rate codecs (SpeechTokenizer, Mimi, DualCodec); supervised Whisper became the ASR backbone for most evaluation pipelines. 2025: Corpus papers reveal the limits of SSL for codec design — ASR features substantially outperform SSL features for dynamic frame merging at ultra-low rates ([[2510.00981]]), and SSL-based HuBERT tokens in SLMs exhibit a combinatorial lexical explosion problem ([[2412.17048]]) driven by retained paralinguistic variability. The field is shifting from "which SSL model?" toward "SSL vs. supervised foundation model?" as the core design question for codec and SLM architectures. SenseVoice-Small's strong performance in [[2510.00981]] is an early signal of this shift. Wave-level cross-modal contrastive learning (SecoustiCodec [[2508.02849]]) proposes eliminating SSL distillation entirely in favour of frame-level phoneme-speech contrastive alignment, arguing that SSL representations inherently retain paralinguistic content that prevents clean disentanglement. REF-VC [[2508.04996]] demonstrates that SSL (WavLM) features combined with ASR bottleneck features are complementary for voice conversion, with SSL providing paralinguistic expressiveness and ASR providing noise-robust phonetic content. DualSpeechLM [[2508.08961]] trains a new speech tokenizer optimised against text LLM objectives rather than reconstruction, representing a departure from both SSL distillation and ASR supervision paradigms toward a directly LLM-aligned tokenisation approach. MahaTTS-v2 [[2508.14049]] uses XLS-R (multilingual wav2vec 2.0, 1B params) k-means tokens as its semantic layer, demonstrating SSL's continued practical value for multilingual low-resource TTS in a production setting. ProsodyLM [[2507.20091]] replaces codec tokens entirely with explicit word-level prosody annotations derived from Whisper transcription, showing that prosody emergence requires token design rather than data scale. Integration pass 6 (Interspeech 2025) adds further SSL usage patterns: DC-Spin [[interspeech-2025-0246]] improves HuBERT tokenization via speaker-invariant clustering and dual-codebook training; FreeCodec [[interspeech-2025-1440]] demonstrates that WavLM supervision routed to the codec decoder provides stronger disentanglement than encoder-side supervision; mHuBERT-147 is used for multilingual discrete content in VC [[interspeech-2025-0815]]; DualCodec [[interspeech-2025-0468]] uses w2v-BERT-2.0 as a direct encoding path for RVQ-1 rather than distillation; and the interpretability analysis [[interspeech-2025-0115]] uses HuBERT 100-class clusters as a reference for probing codec token alignment.

## All Papers

| ID | Title | Venue | Year | Key use of this concept |
|----|-------|-------|------|------------------------|
| [[2507.14534]] | Conan | arXiv (ASRU 2025) | 2025 | HuBERT-Large used as offline teacher for distilling streaming content extractor (Emformer SCE) |
| [[2510.00981]] | FlexiCodec | arXiv (ICLR 2026) | 2025 | Compares ASR vs. SSL features for codec dynamic frame merging; finds ASR features substantially outperform SSL at ultra-low frame rates |
| [[2412.17048]] | Why Do SLMs Fail? | arXiv (ICASSP 2026) | 2026 | HuBERT-Large discrete clusters used as the speech modality in modality-evolving analysis; shows retained paralinguistic variability in HuBERT tokens drives lexical modeling failure |
| [[2509.19668]] | Selective CFG for Zero-shot TTS | arXiv | 2025 | WavLM-Large used for SPK-SIM speaker similarity evaluation |
| [[2025.findings-emnlp.424]] | InteractSpeech | EMNLP 2025 | 2025 | SSL-based pyannote.audio for speaker diarization and vocal consistency in dataset construction pipeline |
| [[2025.acl-long.388]] | DiVA: Distilling an End-to-End Voice Assistant Without Instruction Training Data | ACL | 2025 | Whisper-Large-v3 encoder used as frozen audio feature extractor; Whisper decoder weights repurposed to initialize Q-Former connector, dramatically improving training efficiency vs. random init |
| [[2025.findings-naacl.184]] | Continuous Speech Tokenizer in Text To Speech | NAACL | 2025 | CTC loss on ASR task used during tokenizer pre-training to ensure continuous speech tokens carry phonetic content; draws on self-supervised learning ideas for representation quality |
| [[2025.emnlp-main.989]] | VocalNet | EMNLP | 2025 | Whisper-large-v3 used as frozen speech encoder; its representations are downsampled 5× by an adaptor and fed to LLaMA backbone; demonstrates that SSL/supervised encoder choice interacts with the MTP training objective |
| [[2025.acl-long.682]] | Recent Advances in Speech Language Models: A Survey | ACL | 2025 | Comprehensive taxonomy of SSL models (Wav2vec 2.0, HuBERT, WavLM, USM) and their roles as speech tokenizers; surveys how SSL objectives shape the semantic vs. acoustic content of speech representations |
| [[2025.acl-long.1471]] | The time scale of redundancy between prosody and linguistic context | ACL | 2025 | Uses BERT-large/RoBERTa-large (fine-tuned linear heads) to predict prosodic features from context windows; shows that 3–8 past words and 1–2 future words provide most prosodic MI — empirical characterization of how contextual SSL models relate linguistic and prosodic information |
| [[2025.acl-long.1498]] | Analyzing and Mitigating Inconsistency in Discrete Speech Tokens for Neural Codec Language Models | ACL | 2025 | Motivates DRI: convolutional SSL-like encoders in RVQ codecs have wide receptive fields causing context-dependent tokenization; proposes consistency constraints to reduce this variability |
| [[interspeech-2025-0143]] | Multimodal Prosody Modeling: A Use Case for Multilingual Sentence Mode Prediction | Interspeech | 2025 | WavLM Large embeddings outperform knowledge-based acoustic features and phoneme-level XPhoneBERT for sentence mode prediction; early fusion with XLM-RoBERTa word-level embeddings achieves best UAR 74.22% German |
| [[interspeech-2025-0669]] | PAST: Phonetic-Acoustic Speech Tokenizer | Interspeech | 2025 | Challenges the assumption that SSL pseudo-label distillation is needed for hybrid tokenization; supervised CTC + phoneme classification directly on RVQ-1 achieves better phonetic metrics than SpeechTokenizer without any external SSL teacher |
| [[interspeech-2025-0973]] | A Dataset for Automatic Assessment of TTS Quality in Spanish | Interspeech | 2025 | DenseMOS uses wav2vec 2.0 base representations; finds CNN encoder output (local features) more predictive of MOS than deep transformer layers (contextual features); PCC 0.62 for MOS prediction on Spanish TTS |
| [[2025.coling-main.518]] | ProsodyFlow: High-fidelity TTS through Conditional Flow Matching and Prosody Modeling | workshop | 2025 | Frozen WavLM-Base-plus (12 transformer layers, averaged) extracts prosody vectors as FM training targets; component ablation shows WavLM removal causes -0.18 CMOS loss |
| [[2507.20091]] | ProsodyLM | arXiv | 2025 | Uses Whisper for ASR and StyleTTS2 aligner for extracting word-level prosody features; argues that prosody emergence in LLMs requires explicit prosody token design rather than data scale |
| [[2508.02849]] | SecoustiCodec | arXiv | 2025 | Proposes frame-level cross-modal contrastive learning between phoneme and speech representations as alternative to SSL distillation; argues SSL retains residual paralinguistic content that contrastive training removes |
| [[2508.04996]] | REF-VC | arXiv | 2025 | Combines WavLM SSL features (paralinguistic expressiveness) with Wenet ASR bottleneck features (phonetic content) in a flow-matching DiT; random erasing regularisation balances the two-stream trade-off |
| [[2508.05385]] | Non-Verbal Speech Generation Pipeline | arXiv | 2025 | wav2vec-bert 2.0 with learnable weighted layer fusion as the backbone for frame-level NV event detection; shows cross-lingual generalisation to Chinese without retraining |
| [[2508.06890]] | Maestro-EVC | ASRU | 2025 | HuBERT-base K-means (500 clusters) for phonetic content extraction; gradient reversal layer on content suppresses residual emotional leakage into content tokens |
| [[2508.08399]] | Exploring Disentangled Neural Speech Codecs | arXiv | 2025 | WavLM-Large (6th layer) as frozen encoder; k-means VQ on raw hidden vectors provides speaker-independent phonetic content; instance normalization separates time-invariant speaker statistics from time-variant prosody |
| [[2508.08961]] | DualSpeechLM | arXiv | 2025 | Whisper-medium encoder as basis for USTokenizer; understanding-driven tokenizer trained against LLM next-token prediction objective rather than SSL reconstruction |
| [[2508.11326]] | MoE-TTS | arXiv | 2025 | CosyVoice2 supervised semantic tokenizer (25 Hz) as speech representation; relies on SSL-adjacent architecture for token extraction in description-based TTS |
| [[2508.14049]] | MahaTTS | arXiv | 2025 | XLS-R wav2vec2.0 1B-param multilingual model with k-means clustering as semantic token front-end for 22-language Indic TTS; demonstrates SSL cross-lingual generalisation for low-resource language coverage |
| [[interspeech-2025-0203]] | ClapFM-EVC | Interspeech | 2025 | HuBERT as audio encoder for EVC-CLAP contrastive pretraining; aligns speech and text emotional representations in a shared latent space |
| [[interspeech-2025-0246]] | DC-Spin | Interspeech | 2025 | SpinHuBERT replaces K-means pseudo-labels with Spin codeword IDs during HuBERT pre-training; dual-codebook Spin achieves speaker-invariant tokens with strong SLM downstream correlation |
| [[interspeech-2025-1440]] | FreeCodec | Interspeech | 2025 | WavLM-Large as content encoder training target (cosine similarity loss); codec routing WavLM supervision to decoder branch rather than encoder improves speaker-content disentanglement for VC |
| [[interspeech-2025-0115]] | Bringing Interpretability to Neural Audio Codecs | Interspeech | 2025 | HuBERT 100-class semantic tokens used as reference for co-occurrence analysis; t-SNE and CLUB MI estimation probe SSL alignment across RVQ scales in four codecs |
| [[interspeech-2025-1779]] | ReFlow-VC | Interspeech | 2025 | HuBERT-Soft (Soft-VC) as content extractor for continuous content features in zero-shot VC; combined with VQ-VAE pitch and ECAPA-TDNN speaker embedding |
| [[interspeech-2025-0815]] | Non-AR Zero-Shot Expressive VC | Interspeech | 2025 | mHuBERT-147 discrete units as multilingual content representation; speaker-invariance achieved through discrete quantisation of multilingual phonetic clusters |
| [[interspeech-2025-0816]] | SSANSVC | Interspeech | 2025 | ContentVec (HuBERT-derived singing content model) as content feature extractor for the SVC backbone in the singing VC pipeline |
| [[interspeech-2025-2043]] | MKL-VC | Interspeech | 2025 | WavLM-Large embeddings as the feature space for factorized optimal transport; method exploits WavLM-specific Gaussian-in-blocks property |
| [[2508.08095]] | Dual Information SLMs for Emotional Conversations | arXiv | 2025 | Whisper-large-v3 as frozen speech encoder; dual adapters interface its representations with the LLM without modifying the encoder |
| [[interspeech-2025-1595]] | Scheduled Interleaved Speech-Text Training for S2ST | Interspeech | 2025 | w2v-BERT fine-tuned for CTC ASR as semantic unit extractor; k-means clustering on 21-language data for 2048-category units; word-level alignments from CTC decoder used for scheduled interleaving |
| [[interspeech-2025-0468]] | DualCodec | Interspeech | 2025 | w2v-BERT-2.0 (600M, frozen) 16th-layer features as primary SSL input for dual-stream encoding; direct SSL feature encoding into RVQ-1 rather than distillation |
| [[interspeech-2025-0948]] | PromptEVC | Interspeech | 2025 | Emotion2Vec (large-scale SSL model for speech emotion recognition) for extracting emotion embeddings used as reference targets in the prompt mapper training |
| [[2508.07273]] | Incorporating Contextual Paralinguistic Understanding in Large Speech-Language Models | arXiv | 2025 | Whisper large-v3 encoder as frozen speech representation backbone in Speech-LLM; MLP adapter compresses 1500-length sequence to 100 embeddings |
| [[2508.07375]] | TurnGuide | arXiv | 2025 | VAD from pyannote (SSL-based) for turn segmentation; Whisper medium for word-level alignment in full-duplex dialogue |
| [[2508.09600]] | OSUM-EChat | arXiv | 2025 | Whisper-Medium encoder via 1D convolutional adapter for speech understanding in empathetic dialogue system |
| [[2508.11224]] | Benchmarking Prosody Encoding in Discrete Speech Tokens | ASRU | 2025 | Systematic comparison of HuBERT, ContentVec, data2vec, emotion2vec on prosody sensitivity; controlled TER-based analysis of SSL tokenization design choices |
| [[2508.15565]] | Any-to-any Speaker Attribute Perturbation for Voice Anonymization | arXiv | 2025 | ECAPA-TDNN (trained on VoxCeleb) as frozen speaker encoder for adversarial anonymization training |
| [[2508.16188]] | AVLM (Seeing is Believing) | EMNLP | 2025 | SpiritLM (interleaved semantic/style/pitch speech tokens from SSL-adjacent model) as backbone for audio-visual speech LM |
| [[2508.16790]] | TaDiCodec | arXiv | 2025 | No SSL distillation — end-to-end flow-matching training without SSL teacher represents departure from SSL-guided codec design |
| [[2509.00503]] | Entropy-based Coarse and Compressed Semantic Speech Representation Learning | arXiv | 2025 | Entropy-based adaptive compression of HuBERT discrete tokens; CALE cross-attention aggregation; 15 Hz outperforms 50 Hz for ASR/ST; evaluation compares SSL token compression strategies |
| [[2509.00675]] | Speaker-Conditioned Phrase Break Prediction | arXiv | 2025 | MP BERT (phoneme-level SSL-adjacent pre-trained LM) outperforms all subword BERT variants on phrase break prediction |
| [[2509.01391]] | MixedG2P-T5 | arXiv | 2025 | HuBERT-family ContentVec as SSL encoder; k-means (500 clusters) produces pseudo-language labels for G2P-free Japanese TTS |
| [[2509.03292]] | Improving Perceptual Audio Aesthetic Assessment via Triplet Loss | arXiv | 2025 | BEATs self-supervised audio model as feature backbone; weighted layer combination over all BEATs transformer layers for multi-axis aesthetic prediction |
| [[2509.04072]] | Computational Narrative Understanding for Expressive TTS | arXiv | 2025 | WavLM-large cosine similarity (SPK-SIM) for speaker similarity in LibriQuote benchmark evaluation |
| [[2209.03143]] | AudioLM: a Language Modeling Approach to Audio Generation | arXiv | 2022 | Uses w2v-BERT XL 7th-layer representations as semantic tokens; demonstrates that SSL features encode linguistic content and prosody while acoustic codec tokens encode speaker identity and recording conditions |
| [[2212.04356]] | Robust Speech Recognition via Large-Scale Weak Supervision (Whisper) | arXiv | 2022 | Supervised at-scale speech recognition model used as ASR backbone in TTS evaluation pipelines; demonstrates that weakly supervised scaling surpasses SSL-based ASR on out-of-distribution robustness |
| [[2312.15185]] | emotion2vec: Self-Supervised Pre-Training for Speech Emotion Representation | arXiv | 2023 | SSL pre-trained emotion representation model via online distillation on 262 hours unlabeled emotion data; used as frozen feature extractor in emotion-conditioned TTS evaluation |
