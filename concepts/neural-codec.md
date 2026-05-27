---
slug: neural-codec
title: Neural Audio Codec
aliases: [EnCodec, SoundStream, audio tokenizer, discrete speech representations, RVQ, residual vector quantization, low-frame-rate codec, dynamic codec]
related_concepts: [autoregressive-codec-tts, self-supervised-speech, spoken-language-model, gan-vocoder]
last_updated: 2026-05-27
---

# Neural Audio Codec

## What it is

A neural audio codec is a learned compression system that converts raw audio waveforms into sequences of discrete tokens and reconstructs audio from those tokens. The standard architecture is encoder–quantizer–decoder. The encoder downsamples the waveform to a sequence of latent vectors at a fixed frame rate; a Residual Vector Quantization (RVQ) module converts these to discrete indices across multiple codebook layers (codebook levels); and a convolutional decoder synthesizes the waveform from the quantized representations.

In the context of speech language models and TTS, the first RVQ layer (RVQ-1, "semantic" stream) captures phonetic/semantic content and drives autoregressive language models. The remaining RVQ layers (RVQ-rest, "acoustic" stream) capture fine spectral and prosodic detail and are typically predicted by a faster non-autoregressive model. The frame rate of the codec determines the sequence length fed to language models, directly affecting computational cost (quadratic in attention) and training/inference time.

## Why it matters

Neural audio codecs are foundational to modern autoregressive TTS (VALL-E family), speech language models (Moshi, Mimi), and multimodal audio LLMs. The codec frame rate creates a direct trade-off: standard codecs (EnCodec 75 Hz, DAC 75 Hz) generate sequences 10–15× longer than text for the same content, creating a severe computational burden and modality mismatch with text LLMs. Reducing frame rate while preserving semantic content is the central challenge. Research in [[2412.17048]] identifies that the gap between text and speech LMs is caused partly by sequence length (Factor B) and paralinguistic variability in speech tokens (Factor C), motivating low-frame-rate and semantically-decoupled codecs.

Beyond frame rate, [[2025.acl-long.1498]] identifies a distinct structural problem in existing RVQ codecs: Discrete Representation Inconsistency (DRI). Because the RVQ encoder uses wide convolutional receptive fields to capture context, identical audio segments are tokenized differently depending on surrounding context. This many-to-one mapping from audio to token sequences degrades neural codec language model training, raising WER and reducing speaker similarity. DRI worsens with deeper RVQ layers and can be quantified via consistency accuracy — the fraction of token positions that agree between context-free and context-inclusive encoding of the same segment.

## Current state of the art

As of late 2025, the frontier for low-frame-rate codecs is [[2510.00981]] (FlexiCodec, ICLR 2026), which achieves:
- 4.15% WER (RVQ-1 reconstruction) at 6.25 Hz average frame rate, vs. 31.5% for retrained DualCodec at the same rate.
- Controllable frame rates from 3 to 12.5 Hz from a single trained model via inference-time threshold adjustment.
- UTMOS 4.18 and PESQ 2.76 at 6.25 Hz (competitive with higher-rate systems at similar bitrate).

Previous state-of-the-art at 12.5 Hz was DualCodec (RVQ-1 WER 5.93% at 12.5 Hz). FlexiCodec matches this at 12.5 Hz (2.76% WER, benefiting from ASR features) while extending meaningfully below it.

## Key variants and sub-approaches

**Fixed-rate high-quality codecs (EnCodec, DAC).** Operate at 50–75 Hz, prioritizing reconstruction quality over downstream LM efficiency. DAC uses periodic activations and codebook utilization improvements. EnCodec (75 Hz, 8 RVQ layers) is the original codec used in VALL-E [[2301.02111]] and remains a widely cited baseline.

**SSL-distillation semantic codecs (SpeechTokenizer, Mimi, DualCodec).** Distill self-supervised representations (HuBERT, WavLM, w2v-bert-2) into the RVQ-1 stream to ensure it carries semantic content. Mimi and DualCodec operate at 12.5 Hz by using higher encoder downsampling rates. Adequate for 12.5 Hz but struggle below that.

**ASR-feature-guided dynamic codecs (FlexiCodec [[2510.00981]]).** Use a frozen ASR encoder (SenseVoice-Small) to extract semantic features and guide adaptive frame merging. Adjacent frames with cosine similarity above a threshold are merged, adaptively allocating more frames to phonetically dense regions. Achieves 3–12.5 Hz controllable rate with superior semantic preservation compared to fixed-rate approaches. Key insight: ASR features are more semantically concentrated than SSL features, enabling more effective frame merging.

**Text-assisted codecs (TaDiCodec, TASTE).** Use text transcription to assist ultra-low-rate coding. Achieve ≤ 6.25 Hz but require ground-truth text, limiting applicability in non-TTS settings.

**Single-codebook codecs (WavTokenizer, SemantiCodec).** Use a single FSQ/VQ codebook for simplicity, operating at 40–75 Hz. More suitable for non-LM applications.

**Continuous tokenizers.** [[2025.findings-naacl.184]] (Cont-SPT) challenges whether discrete quantization is necessary at all: the encoder output is retained as continuous speech tokens (no RVQ), and the AR LM predicts these via MSE regression. This eliminates quantization-induced information loss (especially at high frequencies: 0.55 vs. 0.34 retention at 8 kHz), improving WER and speaker similarity in downstream TTS. The tradeoff is that the training objective changes from classification to regression, and generation requires a flow-matching acoustic decoder rather than simple codebook lookup.

## Comparison to alternatives

Continuous mel-spectrograms remain the dominant representation for non-autoregressive TTS (flow-matching, diffusion); they do not require discrete tokenization. Discrete codecs are specifically required for autoregressive language model training. The FlexiCodec-TTS results ([[2510.00981]]) show that for the AR stage, 6.25 Hz codec tokens provide no degradation vs. 12.5 Hz while providing 1.4–2.1× AR speedup, but the NAR stage still benefits from high frame rate (50 Hz mel > 12.5 Hz codec features for naturalness).

## Year-on-year trajectory

2021–2022: SoundStream, EnCodec established neural audio codecs as the standard compression tool at 50–75 Hz. 2023: SpeechTokenizer introduced semantic distillation for RVQ-1; first purpose-built TTS codecs. 2024: Mimi and DualCodec reached 12.5 Hz while maintaining semantic quality. 2025: FlexiCodec ([[2510.00981]]) breaks the 12.5 Hz floor, achieving 6.25 Hz (and 3 Hz dynamically) with near-ground-truth semantic preservation — a major step toward text-rate (~4.5 Hz) speech tokens.

## Open questions

- Can dynamic frame merging be extended to sub-3 Hz without catastrophic quality loss?
- Is there a natural lower bound on frame rate below which semantic content cannot be preserved in discrete tokens?
- How does FlexiCodec's dynamic merging interact with prosodic information? [[2412.17048]] identifies paralinguistic variability (Factor C) as the main bottleneck — do dynamic tokens help with this?
- Streaming/causal encoding is not addressed by FlexiCodec; is dynamic frame merging compatible with online TTS pipelines?
- Multilingual semantic tokens require language-specific fine-tuning; can the ASR-guided merging generalize cross-lingually?
- DRI ([[2025.acl-long.1498]]) is shown to worsen with deeper RVQ layers; does this suggest the deeper-layer representation design needs fundamentally different treatment (e.g. non-contextual sub-quantizers)?
- [[2025.findings-naacl.184]] shows continuous tokens outperform RVQ on a single English benchmark; will continuous-token AR LMs scale to larger data and multilingual settings, or does the MSE regression objective become unstable?
- VocalNet [[2025.emnlp-main.989]] and other speech LLMs now use CosyVoice2 semantic tokens (25 Hz) rather than traditional RVQ codecs; does this represent a shift toward hybrid supervised-semantic tokenization as the default?

## Papers

| ID | Title | Venue | Year | Key use of this concept |
|----|-------|-------|------|------------------------|
| [[2301.02111]] | Neural Codec Language Models are Zero-Shot Text to Speech Synthesizers (VALL-E) | arXiv | 2023 | First TTS system built on top of a neural audio codec (EnCodec 75 Hz, 8 RVQ layers); demonstrates that codec tokens enable large-scale language model pre-training for zero-shot TTS |
| [[2510.00981]] | FlexiCodec: A Dynamic Neural Audio Codec for Low Frame Rates | arXiv (ICLR 2026) | 2025 | Proposes ASR-feature-guided dynamic frame merging to achieve 3–12.5 Hz controllable codec with near-GT semantic preservation at 6.25 Hz; enables TTS AR speedup of 7.3× over CosyVoice |
| [[2025.acl-long.1498]] | Analyzing and Mitigating Inconsistency in Discrete Speech Tokens for Neural Codec Language Models | ACL | 2025 | Introduces consistency accuracy metric and DRI (Discrete Representation Inconsistency) phenomenon; proposes slice-consistency and perturbation-consistency losses that increase RVQ consistency by up to 36% and reduce downstream VALL-E WER by 1.98% |
| [[2025.findings-naacl.184]] | Continuous Speech Tokenizer in Text To Speech | NAACL | 2025 | Replaces RVQ with a continuous speech tokenizer; retains encoder output directly, avoiding high-frequency quantization loss; shows information retention advantage at 8 kHz (0.55 vs. 0.34 for discrete) and improved downstream TTS quality |
| [[2025.acl-long.346]] | ControlSpeech: Towards Simultaneous and Independent Zero-shot Speaker Cloning and Zero-shot Language Style Control | ACL | 2025 | Uses frozen FACodec (NaturalSpeech 3 factorized codec) as disentanglement backbone; content, prosody, acoustic detail, and timbre codes are separately controlled — demonstrating that a pre-trained factorized codec can serve as the control interface for simultaneous multi-factor TTS |
| [[2025.emnlp-main.989]] | VocalNet: Speech LLMs with Multi-Token Prediction for Faster and High-Quality Generation | EMNLP | 2025 | Uses CosyVoice2 semantic speech tokens (25 Hz) as the codec for speech generation; the codec choice enables compatibility with flow-matching acoustic decoder and HiFi-GAN vocoder in a full speech LLM pipeline |
| [[2025.acl-long.682]] | Recent Advances in Speech Language Models: A Survey | ACL | 2025 | Surveys all major neural codec architectures (SoundStream, EnCodec, DAC, SpeechTokenizer, Mimi) and their roles as speech tokenizers; classifies by objective (semantic understanding, acoustic generation, mixed) and discusses discrete vs. continuous representations |
