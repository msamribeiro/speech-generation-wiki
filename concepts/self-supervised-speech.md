---
slug: self-supervised-speech
title: Self-Supervised Speech Representations and Foundation Models
aliases: [SSL speech, HuBERT, WavLM, wav2vec 2.0, speech foundation model, self-supervised pre-training, SenseVoice, Whisper, large speech model]
related_concepts: [neural-codec, disentanglement, voice-conversion, speaker-adaptation, spoken-language-model, speech-to-speech]
last_updated: 2026-05-30
---
## What it is

Self-supervised speech representations are learned feature extractors trained on large corpora of unlabeled speech using objectives that do not require human annotation. The dominant approaches use masked prediction (HuBERT, WavLM) or contrastive learning (wav2vec 2.0) to extract representations that capture phonetic, semantic, and speaker-level information in learned embeddings.

**Speech foundation models** are a broader, overlapping category: large pre-trained speech models (100k+ hours, often multi-task) that serve as general-purpose backbones for downstream tasks including TTS, VC, codec design, and evaluation. They may be trained with purely self-supervised objectives (HuBERT, WavLM) or with supervised objectives at scale (Whisper, SenseVoice), or both. The defining characteristic is scale and task-agnostic pre-training — the model is trained once and then used as a frozen feature extractor or fine-tuning base across many downstream tasks.

Key SSL models: wav2vec 2.0 (contrastive + masked prediction, 95M/317M params), HuBERT (masked prediction over K-means cluster targets, up to 300M params), WavLM (masked prediction + denoising, up to 600M params). Key supervised foundation models: Whisper (ASR-supervised, multilingual, up to 1.5B params), SenseVoice-Small (ASR + emotion + language ID, 230M params). HuBERT and WavLM are the most widely used in speech synthesis research; SenseVoice is emerging as a preferred backbone for codec semantic extraction.

## Why it matters

SSL representations provide rich phonetic and speaker information without requiring labeled data. In the TTS/VC/SCA context, they serve multiple roles:
- **Content extraction for VC:** [[2507.14534]] (Conan) distills HuBERT-Large to create a streaming content extractor; HuBERT is widely used as the phonetic "content" signal in disentanglement-based VC systems.
- **Semantic distillation for codecs:** SSL features (WavLM in Mimi, w2v-bert-2 in DualCodec) are distilled into RVQ-1 codec tokens to make them semantically meaningful. However, [[2510.00981]] shows that ASR features outperform SSL features for guiding dynamic frame merging at ultra-low frame rates.
- **Speaker similarity measurement:** WavLM-Large is the standard speaker verification model for SPK-SIM computation in zero-shot TTS evaluation ([[2509.19668]]).
- **Speech token baseline for SLMs:** [[2412.17048]] uses HuBERT-Large discrete clusters (2048 units at 50 Hz) as the speech modality in its modality-evolving study.
- **Speech understanding/diarization:** [[2025.findings-emnlp.424]] uses pyannote.audio (which builds on SSL embeddings) for speaker diarization and vocal consistency checking in the InteractSpeech pipeline.

## Current state of the art

For speech synthesis research, HuBERT-Large and WavLM-Large remain the standard SSL models used for content extraction (VC), phonetic analysis, and speaker similarity measurement. SenseVoice-Small (an ASR model rather than SSL in the traditional sense) is shown to be superior to SSL for codec dynamic frame merging in [[2510.00981]].

The field has begun distinguishing SSL representations (trained with reconstruction/self-prediction objectives, retaining acoustic/paralinguistic information) from ASR-supervised representations (trained with text targets, more semantically concentrated). [[2510.00981]] provides compelling ablation evidence that for low-frame-rate codec design, ASR-supervised features produce dramatically better semantic preservation (6% WER vs. 27% WER for DualCodec's SSL approach at 6.25 Hz).

## Key variants and sub-approaches

**Masked prediction (HuBERT).** Offline cluster assignment creates pseudo-labels; model predicts cluster IDs for masked frames. Multiple iterations refine the cluster quality. Widely used as content extractor in VC [[2507.14534]] and as speech tokenizer for SLMs [[2412.17048]].

**Masked prediction + denoising (WavLM).** Extends HuBERT with a masked speech denoising objective to improve robustness to noise. WavLM-Large is the standard speaker verification backbone for SPK-SIM in TTS evaluation [[2509.19668]].

**Contrastive learning (wav2vec 2.0).** Contrastive loss over masked speech quantized representations. The w2v-bert-2 model (used in DualCodec) is a variant; [[2510.00981]] shows this performs poorly for dynamic frame merging due to leaked acoustic information.

**Large-scale supervised foundation models (Whisper, SenseVoice).** Trained with text supervision (ASR labels, emotion labels, language ID) at scale rather than self-supervised objectives. Whisper (OpenAI, up to 1.5B params, 680k hours multilingual) is the dominant speech recognition backbone. SenseVoice-Small (230M params, 300k hours, multi-task: ASR + emotion + language ID) is shown in [[2510.00981]] to produce more semantically concentrated features than SSL models for codec design. The key property is that supervised training aligns representations tightly with text semantics, suppressing the paralinguistic variability that makes SSL features problematic as low-frame-rate codec targets.

**Multi-task / universal speech encoders.** A newer generation that combines SSL and supervised objectives (e.g. data2vec 2.0, SONAR). Aim for universal representations that generalise across speech tasks. Underrepresented in current corpus; expected to grow as the field consolidates around foundation model paradigm.

## Comparison to alternatives

Text-supervised ASR features encode more concentrated semantic information but require labeled data. SSL features are more universal (can encode prosody, speaker, emotion) but this breadth is a disadvantage when only semantic content is needed (as in low-frame-rate codecs). For VC content extraction, HuBERT is preferred because it captures phonetic (not speaker) content when its discrete clusters are used. For speaker verification, WavLM-Large embeddings are the standard.

## Year-on-year trajectory

2020–2022: wav2vec 2.0, HuBERT, WavLM established SSL as the standard for downstream speech tasks. 2023–2024: SSL features became the semantic distillation target for low-frame-rate codecs (SpeechTokenizer, Mimi, DualCodec); supervised Whisper became the ASR backbone for most evaluation pipelines. 2025: Corpus papers reveal the limits of SSL for codec design — ASR features substantially outperform SSL features for dynamic frame merging at ultra-low rates ([[2510.00981]]), and SSL-based HuBERT tokens in SLMs exhibit a combinatorial lexical explosion problem ([[2412.17048]]) driven by retained paralinguistic variability. The field is shifting from "which SSL model?" toward "SSL vs. supervised foundation model?" as the core design question for codec and SLM architectures. SenseVoice-Small's strong performance in [[2510.00981]] is an early signal of this shift.

## Open questions

- Can SSL models be explicitly trained to suppress paralinguistic variability (Factor C from [[2412.17048]]) while retaining phonetic content, making them better codec targets?
- Is there a sweet spot between ASR-supervised (semantically concentrated) and SSL features (acoustically rich) for low-frame-rate codec design?
- WavLM-Large is used for SPK-SIM evaluation; how sensitive are conclusions about speaker similarity to the choice of SSL backbone?
- As supervised foundation models (SenseVoice, Whisper) increasingly replace SSL in codec design, does the field still need SSL-specific pre-training, or will the SSL vs. supervised distinction collapse?
- Do multi-lingual supervised foundation models (Whisper-large-v3, MMS) generalise better as codec semantic targets across languages than English-centric SSL models?

## Papers

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
