---
slug: voice-conversion
title: Voice Conversion
aliases: [VC, speaker style transfer, speaker transformation, cross-speaker conversion]
related_concepts: [zero-shot-tts, disentanglement, self-supervised-speech, speaker-adaptation, streaming-tts, gan-vocoder]
last_updated: 2026-05-27
---

# Voice Conversion

## What it is

Voice conversion (VC) is the task of transforming speech produced by one speaker so that it sounds as though it was produced by a different target speaker, while preserving the linguistic content. The source utterance's phonetic content is extracted and then rendered using the target speaker's vocal characteristics — timbre, prosody, speaking style, and potentially emotion. VC is distinct from TTS in that the content is derived from existing speech rather than from text, and the output preserves the timing, rhythm, and paralinguistic attributes of the original utterance (unless deliberately altered).

Modern VC systems decompose speech into disentangled representations: a content stream (phonetic/semantic information) and a style/speaker stream (timbre, prosody, emotion), then recombine the source content with the target speaker's style. This separation is achieved through information bottlenecks, self-supervised speech representations, vector quantization, or adversarial training.

## Why it matters

Voice conversion has direct applications in real-time communication (voice privacy, voice disguise, accessibility), entertainment (dubbing, gaming avatars, virtual humans), and multimedia production. Online (real-time) VC systems that operate with low latency are particularly valuable for interactive applications. Zero-shot VC — which generalizes to unseen speakers at inference from a brief reference clip without speaker-specific fine-tuning — is the dominant frontier because it removes the registration requirement.

The VC task also serves as a testbed for disentanglement methods: success requires cleanly separating content from speaker characteristics, which is a general challenge across TTS, SCA, and codec design.

## Current state of the art

As of 2025, the leading streaming zero-shot VC systems in the corpus are [[2507.14534]] (Conan) and [[2025.acl-demo.37]] (RT-VC). Conan achieves 85.71% speaker similarity (Resemblyzer cosine similarity) and MOS-Q of 4.06 on LibriTTS→VCTK with 37–140 ms latency, outperforming prior streaming baseline StreamVC (77.81% SIM) and even offline systems in speaker similarity. RT-VC achieves 61.4 ms CPU latency — 13.3% faster than StreamVC — using an articulatory coding (SPARC-based) pipeline with a causal DDSP vocoder, achieving SPK-SIM 76.65% and MOS 3.87 on the same benchmark; it prioritizes interpretability and CPU efficiency at a slight cost in speaker similarity vs. Conan.

For offline zero-shot VC, diffusion-based systems (Diff-VCTK) and language-model-based approaches (LM-VC, using HuBERT semantic + SoundStream acoustic tokens) have set strong baselines, but online operation requires architectural changes that typically trade some quality for latency.

## Key variants and sub-approaches

**Offline zero-shot VC.** Systems like VQMIVC, AutoVC, and NaturalSpeech 3 operate non-causally over full utterances. They can achieve high quality but cannot stream. VQMIVC uses vector quantization with mutual information minimization for content-speaker disentanglement. AutoVC uses an information bottleneck architecture with an autoencoder loss.

**Online/streaming VC.** Systems that process source speech chunk-by-chunk. StreamVC uses a lightweight causal CNN to extract soft speech unit representations. [[2507.14534]] (Conan) advances this with Emformer-based content extraction (distilled from HuBERT), clustering-VQ style encoding, and a Causal Shuffle Vocoder. The key design challenge is maintaining content accuracy while enforcing causality. Conan's fast setting achieves 37 ms latency with an RTF of 0.74 on A100 GPU. [[2025.acl-demo.37]] (RT-VC) takes an orthogonal approach: it replaces SSL-based content extraction with articulatory coding (SPARC EMA features via a causal dilated convolution EMA inverter) and uses a DDSP vocoder instead of HiFi-GAN, achieving 61.4 ms CPU latency with better pitch tracking (PCC 0.865 vs. 0.842 for StreamVC) at the cost of slightly lower speaker similarity.

**Style transfer scope.** Earlier systems transfer only global timbre (using x-vectors or d-vectors). More recent systems including Conan also capture fine-grained speaking styles (prosody, emotion) through chunk-level clustering VQ and align-attention mechanisms.

**ASR-PPG-based VC.** Some systems extract phonetic posteriorgrams from an ASR model as content features, avoiding dependence on large SSL encoders but incurring ASR latency.

## Comparison to alternatives

TTS with voice cloning (zero-shot TTS) shares the speaker generalization goal but starts from text rather than speech. TTS systems can optimize for naturalness independently of content extraction. VC must preserve source content exactly, making content accuracy (WER) a hard constraint. This distinction makes VC architecturally different even when they share vocoder and speaker encoder components.

Speech anonymization is a related task that uses VC-style pipelines but deliberately targets a different (anonymized) speaker rather than a specific reference.

## Year-on-year trajectory

Through 2022–2024, the field progressed from parallel/non-parallel offline methods to online streaming approaches. The gap between online and offline quality has narrowed substantially. [[2507.14534]] (2025) demonstrates that online systems can now exceed offline baselines in speaker similarity (Conan Full 85.71% vs. Diff-VCTK offline 81.47%), marking a qualitative shift where the streaming constraint no longer entails quality degradation. In the same year, [[2025.acl-demo.37]] (RT-VC) demonstrates that interpretable articulatory-space disentanglement (via SPARC) can be made real-time on CPU, offering an alternative design axis (interpretability and CPU efficiency) to the SSL-heavy approaches like Conan and StreamVC. Two distinct streaming VC paradigms now exist: SSL-distillation content extraction (Conan) and articulatory-space content extraction (RT-VC), each with different trade-offs in speaker similarity, latency, interpretability, and hardware requirements.

## Open questions

- Can streaming VC achieve sub-20 ms latency without quality loss on commodity hardware?
- Cross-lingual VC (source and target speakers speak different languages) remains largely unaddressed.
- Emotional style transfer in zero-shot online settings is demonstrated by [[2507.14534]] but not independently evaluated — how well does the style encoder actually capture emotion vs. timbre?
- Evaluation standards are fragmented: WER, SPK-SIM, and MOS are standard but measured on different test sets and with different ASR/speaker-verification models across papers.

## Papers

| ID | Title | Venue | Year | Key use of this concept |
|----|-------|-------|------|------------------------|
| [[2507.14534]] | Conan: A Chunkwise Online Network for Zero-Shot Adaptive Voice Conversion | arXiv (ASRU 2025) | 2025 | Introduces a streaming zero-shot VC system achieving state-of-the-art speaker similarity at 37–140 ms latency via Emformer content extraction, CVQ style encoding, and Causal Shuffle Vocoder |
| [[2025.acl-demo.37]] | RT-VC: Real-Time Zero-Shot Voice Conversion with Speech Articulatory Coding | ACL | 2025 | Achieves 61.4 ms CPU latency for zero-shot VC using articulatory coding (SPARC EMA inverter + WavLM speaker encoder + DDSP vocoder); 13.3% faster than StreamVC; best pitch tracking (PCC 0.865) in corpus |
