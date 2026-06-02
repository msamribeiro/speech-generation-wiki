---
slug: voice-conversion
title: Voice Conversion
aliases: [VC, speaker style transfer, speaker transformation, cross-speaker conversion]
status: established
related_concepts: [zero-shot-tts, disentanglement, self-supervised-speech, speaker-adaptation, streaming-tts, gan-vocoder]
last_updated: 2026-06-02
---

## Executive Summary

> [!abstract]
> Voice conversion (VC) transforms speech from one speaker so that it sounds as though produced by a different target speaker, while preserving the linguistic content. Modern VC systems decompose speech into disentangled representations — content (phonetic/semantic) and style/speaker (timbre, prosody, emotion) — then recombine source content with target speaker style. Zero-shot VC, which generalizes to unseen speakers from a brief reference clip without fine-tuning, is the dominant frontier. As of 2025, streaming zero-shot VC systems now match or exceed offline quality in speaker similarity, marking a qualitative shift where the real-time constraint no longer entails quality degradation.

## Current Status

established — Voice conversion is a mature task with a clear evaluation protocol (WER, SPK-SIM, MOS on LibriTTS→VCTK or similar). The primary open frontier is streaming (online) VC, where two complementary paradigms now coexist: SSL-distillation content extraction (Conan [[2507.14534]]) and articulatory-space content extraction (RT-VC [[2025.acl-demo.37]]). Offline zero-shot VC quality is strong; the challenge has shifted to latency, hardware requirements, and emotional style transfer.

## Why This Matters

Voice conversion has direct applications in real-time communication (voice privacy, accessibility), entertainment (dubbing, gaming avatars, virtual humans), and multimedia production. Online VC systems that operate with low latency are particularly valuable for interactive applications. Zero-shot VC removes the registration requirement, making any speaker's voice available at inference time without enrollment data.

The VC task also serves as a testbed for disentanglement methods: success requires cleanly separating content from speaker characteristics, which is a general challenge across TTS, SCA, and codec design.

## Core Idea

VC systems extract a content representation from source speech (removing speaker information) and a speaker/style representation from a target reference, then generate speech that has the source content rendered with the target speaker's characteristics. The quality of disentanglement — how cleanly content is separated from speaker — directly determines whether the output preserves the source words and adopts the target voice.

Modern zero-shot VC achieves this without any speaker-specific training by using reference audio at inference time to provide the speaker embedding or style conditioning.

## Methods and Variants

**Offline zero-shot VC.** Systems like VQMIVC, AutoVC, and NaturalSpeech 3 operate non-causally over full utterances. They achieve high quality but cannot stream. VQMIVC uses vector quantization with mutual information minimization for content-speaker disentanglement. AutoVC uses an information bottleneck architecture.

**Online/streaming VC.** Systems that process source speech chunk-by-chunk. StreamVC uses a lightweight causal CNN to extract soft speech unit representations. Conan [[2507.14534]] advances this with Emformer-based content extraction (distilled from HuBERT), clustering-VQ style encoding, and a Causal Shuffle Vocoder. The Causal Shuffle Vocoder replaces transposed convolutions with pixel shuffle to avoid spectral artifacts from zero-padding. Conan's fast setting achieves 37 ms latency with RTF 0.74 on A100 GPU. RT-VC [[2025.acl-demo.37]] takes an orthogonal approach: it replaces SSL-based content extraction with articulatory coding (SPARC EMA features via a causal dilated convolution EMA inverter) and uses a DDSP vocoder, achieving 61.4 ms CPU latency with better pitch tracking (PCC 0.865 vs. 0.842 for StreamVC) at the cost of slightly lower speaker similarity.

**Style transfer scope.** Earlier systems transfer only global timbre (using x-vectors or d-vectors). More recent systems including Conan also capture fine-grained speaking styles (prosody, emotion) through chunk-level clustering VQ and alignment-attention mechanisms.

**ASR-PPG-based VC.** Some systems extract phonetic posteriorgrams from an ASR model as content features, avoiding dependence on large SSL encoders but incurring ASR latency.

**Watermark-aware VC.** VoiceMark [[interspeech-2025-0575]] builds watermarks specifically designed to survive zero-shot VC pipelines, exploiting the fact that VC must transfer speaker-specific latents (VQ-2-8 in SpeechTokenizer RVQ). This demonstrates the security implications of VC-pipeline design choices.

## Major Claims

Claims are generalised propositions aggregated from paper evidence.

### Strongly Supported

- Streaming zero-shot VC can now match or exceed offline VC quality in speaker similarity, eliminating the quality penalty previously associated with the real-time constraint.
  Supporting: [[2507.14534]]

- SSL-derived discrete content tokens (HuBERT, WavLM) are the most effective content representation for zero-shot VC because K-means clustering naturally removes speaker information while preserving phonemic identity.
  Supporting: [[2507.14534]], [[2025.acl-demo.37]]

### Emerging

- Articulatory-space content extraction (SPARC EMA features) provides an interpretable, CPU-deployable alternative to SSL-heavy VC pipelines with better pitch tracking, at a slight cost in speaker similarity.
  Supporting: [[2025.acl-demo.37]]

- Chunk-level style encoding (capturing local prosody and emotion) is necessary for full speaking style transfer; global timbre encoding alone is insufficient for voice identity.
  Supporting: [[2507.14534]]

- SSL features improve paralinguistic expressiveness in VC but introduce timbre leakage and noise sensitivity; random feature erasure during training can mitigate this without information bottleneck machinery.
  Supporting: [[2508.04996]]

- Three independent reference inputs (content, speaker, emotion) with dedicated encoders provide finer-grained emotional voice conversion than single-reference methods, particularly for temporal prosody transfer.
  Supporting: [[2508.06890]]

- Natural language prompts can control emotional voice conversion at parity with reference speech for the majority of listeners when a contrastive alignment model bridges speech and text emotional representations.
  Supporting: [[interspeech-2025-0203]]

- Fully discrete disentanglement of phonetic, prosodic, and speaker information in a neural codec enables voice conversion via codebook-level operations without an explicit VC training objective.
  Supporting: [[2508.08399]]

- Unsupervised frame-level cross-modal contrastive learning between phoneme and speech representations removes paralinguistic content from the semantic codec stream more effectively than SSL distillation, improving voice conversion quality.
  Supporting: [[2508.02849]]

### Contested

> [!warning]
> The trade-off between speaker similarity, latency, and hardware requirements differs substantially between the SSL-distillation approach (Conan: high SIM, GPU required) and the articulatory approach (RT-VC: lower SIM, CPU-deployable). Neither dominates on all axes. The evaluation standards are also fragmented — WER, SPK-SIM, and MOS are measured on different test sets and with different ASR/speaker-verification models across papers, making direct comparison unreliable.
> SSL-distillation: [[2507.14534]] / Articulatory: [[2025.acl-demo.37]]

## Relationship to Other Concepts

### Extends or Builds On
- [[disentanglement]] — VC quality is directly determined by disentanglement quality; VC is the primary application driving disentanglement research
- [[self-supervised-speech]] — SSL representations (HuBERT, WavLM) are the standard source of speaker-disentangled content features

### Competes With
- [[zero-shot-tts]] — shares the speaker generalization goal but starts from text rather than speech; TTS can optimize for naturalness independently of content extraction, while VC must preserve source content exactly

### Commonly Paired With
- [[streaming-tts]] — online VC requires the same causal design principles as streaming TTS
- [[gan-vocoder]] — HiFi-GAN and DDSP vocoders are the primary waveform synthesizers in VC pipelines; Causal Shuffle Vocoder is a streaming-specific adaptation

## Representative Papers

### Foundational
- [[2507.14534]] — Conan: first streaming zero-shot VC to match offline quality in speaker similarity; introduces Emformer distillation, CVQ style encoding, and Causal Shuffle Vocoder

### Influential
- [[2025.acl-demo.37]] — RT-VC: first CPU-deployable real-time zero-shot VC via articulatory coding; demonstrates an alternative to SSL-heavy approaches with better pitch tracking

### Recent Highlights
- [[interspeech-2025-0575]] — VoiceMark: security-oriented analysis of VC-pipeline design; embeds VC-resistant watermarks in speaker-specific codec latents

### Cautionary / Negative Evidence
- [[2025.acl-demo.37]] — RT-VC achieves lower speaker similarity (76.65%) than Conan Full (85.71%), confirming that the articulatory approach trades some voice identity fidelity for CPU deployability and interpretability

## Open Questions

- Can streaming VC achieve sub-20 ms latency without quality loss on commodity hardware?
- Cross-lingual VC (source and target speakers speak different languages) remains largely unaddressed in the corpus.
- Emotional style transfer in zero-shot online settings is demonstrated by Conan [[2507.14534]] but not independently evaluated — how well does the CVQ style encoder actually capture emotion vs. timbre?
- Evaluation standards are fragmented: WER, SPK-SIM, and MOS are measured with different models and test sets across papers; a standard benchmark is needed.
- Can articulatory-space VC (RT-VC approach) reach the speaker similarity levels of SSL-distillation approaches with a better speaker encoder?
- REF-VC [[2508.04996]] shows better noise robustness than Seed-VC on a noisy test set; can this advantage be reproduced in diverse noise conditions and on standardized benchmarks?
- Maestro-EVC [[2508.06890]] is evaluated only on the ESD corpus (350 utterances, 10 speakers); do the three-reference framework and temporal emotion alignment generalise to spontaneous or cross-domain speech?
- The VoiceMOS/AudioMOS Challenge [[2508.00317]] identifies VC quality assessment as an active track with no generalised predictor; what evaluation framework would serve the growing VC field adequately?

## Trend Summary

Through 2022–2024, the field progressed from parallel/non-parallel offline methods to online streaming approaches. The gap between online and offline quality has narrowed substantially. Conan [[2507.14534]] (2025) demonstrates that streaming VC can now exceed offline baselines in speaker similarity (85.71% vs. Diff-VCTK offline 81.47%), marking a qualitative shift. In the same year, RT-VC [[2025.acl-demo.37]] demonstrated that interpretable articulatory-space disentanglement can be made real-time on CPU. Two distinct streaming VC paradigms now exist: SSL-distillation content extraction (Conan) and articulatory-space content extraction (RT-VC), each with different trade-offs in speaker similarity, latency, interpretability, and hardware requirements. Integration pass 5 adds four distinct new directions for offline zero-shot VC: REF-VC [[2508.04996]] combining ASR+SSL features with random erasing regularisation for noise robustness; Maestro-EVC [[2508.06890]] with three-reference inputs and frame-level temporal emotion alignment; ClapFM-EVC [[interspeech-2025-0203]] with dual natural-language+reference conditioning and scalar intensity control; and [[2508.08399]] with fully-discrete three-factor codec disentanglement from WavLM without phoneme labels. VoiceMOS/AudioMOS [[2508.00317]] documents VC quality assessment as an active evaluation track. SecoustiCodec [[2508.02849]] proposes frame-level contrastive learning as a more effective alternative to SSL distillation for VC-oriented codec disentanglement.

## All Papers

| ID | Title | Venue | Year | Role in this concept |
|----|-------|-------|------|---------------------|
| [[2507.14534]] | Conan: A Chunkwise Online Network for Zero-Shot Adaptive Voice Conversion | arXiv (ASRU 2025) | 2025 | Streaming zero-shot VC at 37–140 ms latency; Emformer content extraction + CVQ style encoding + Causal Shuffle Vocoder |
| [[2025.acl-demo.37]] | RT-VC: Real-Time Zero-Shot Voice Conversion with Speech Articulatory Coding | ACL | 2025 | 61.4 ms CPU latency; articulatory coding (SPARC EMA inverter) + DDSP vocoder; best pitch tracking in corpus |
| [[interspeech-2025-0575]] | VoiceMark: Zero-Shot Voice Cloning-Resistant Watermarking Approach Leveraging Speaker-Specific Latents | Interspeech | 2025 | VC-resistant watermarking exploiting SpeechTokenizer RVQ disentanglement |
| [[2508.00317]] | Advancing Speech Quality Assessment through Challenges | arXiv | 2025 | VC quality assessment as an active VoiceMOS challenge track; no general-purpose VC quality predictor exists |
| [[2508.02849]] | SecoustiCodec | arXiv | 2025 | Streaming codec with explicit semantic-paralinguistic separation; VC via reference speaker embedding at decoder; frame-level contrastive learning for cleaner disentanglement than SSL distillation |
| [[2508.04996]] | REF-VC | arXiv | 2025 | DiT flow-matching VC; random erasing of SSL features for noise robustness; ASR+SSL complementary fusion; Shortcut Models for 4-step inference; CER 8.03% vs. 12.45% for Seed-VC on noisy set |
| [[2508.06890]] | Maestro-EVC | ASRU | 2025 | Three-reference EVC (content/speaker/emotion); TCEM frame-level temporal emotion alignment; GRL on content classifier; EEPT explicit F0+energy transfer; prosody augmentation for mismatch |
| [[2508.08399]] | Exploring Disentangled Neural Speech Codecs | arXiv | 2025 | Fully-discrete three-factor codec disentanglement from WavLM without phoneme labels; VC via speaker code swap; quantifies fidelity cost of speaker code discretization |
| [[2508.08961]] | DualSpeechLM | arXiv | 2025 | VC as a downstream task for the dual-token architecture; speaker embedding from 3D-Speaker conditions WavTokenizer acoustic generation |
| [[interspeech-2025-0203]] | ClapFM-EVC | Interspeech | 2025 | EVC-CLAP contrastive alignment for dual-mode (reference speech or NL prompt) EVC; AIG intensity gate; any-to-one with flow-matching decoder; MOS 4.09 vs. MixEmo 2.98 |
