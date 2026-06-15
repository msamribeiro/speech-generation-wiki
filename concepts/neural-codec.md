---
slug: neural-codec
title: Neural Audio Codec
aliases: [EnCodec, SoundStream, audio tokenizer, discrete speech representations, RVQ, residual vector quantization, low-frame-rate codec, dynamic codec]
related_concepts: [autoregressive-codec-tts, self-supervised-speech, spoken-language-model, gan-vocoder]
last_updated: 2026-06-15
status: mature-infrastructure
---

## Executive Summary

> [!abstract]
> Neural audio codecs are learned compression systems that convert raw audio waveforms into sequences of discrete tokens (or continuous latents) and reconstruct audio from those representations. As of 2026, they are the foundational building block of autoregressive TTS, speech language models, and multimodal audio LLMs. The field is converging on two competing futures: sub-10 Hz discrete codecs that match text-rate sequence lengths, and continuous waveform latent representations that bypass discretization entirely.

## Current Status

mature-infrastructure — Neural codecs underpin virtually every modern AR TTS and SLM system; EnCodec/DAC are stable baselines while the frontier pushes toward lower frame rates (6.25 Hz), FSQ quantization, and continuous VAE alternatives. The core design question has shifted from codec quality to downstream LM efficiency and codec-LM co-design.

## Why This Matters

Neural audio codecs are foundational to modern autoregressive TTS (VALL-E family), speech language models (Moshi, Mimi), and multimodal audio LLMs. The codec frame rate creates a direct trade-off: standard codecs (EnCodec 75 Hz, DAC 75 Hz) generate sequences 10–15× longer than text for the same content, creating a severe computational burden and modality mismatch with text LLMs. Reducing frame rate while preserving semantic content is the central challenge. Research in [[2412.17048]] identifies that the gap between text and speech LMs is caused partly by token sequence length and paralinguistic variability in speech tokens, motivating low-frame-rate and semantically-decoupled codecs.

Beyond frame rate, [[2025.acl-long.1498]] identifies a distinct structural problem in existing RVQ codecs: Discrete Representation Inconsistency (DRI). Because the RVQ encoder uses wide convolutional receptive fields to capture context, identical audio segments are tokenized differently depending on surrounding context. This many-to-one mapping from audio to token sequences degrades neural codec language model training, raising WER and reducing speaker similarity. DRI worsens with deeper RVQ layers and can be quantified via consistency accuracy — the fraction of token positions that agree between context-free and context-inclusive encoding of the same segment.

## Core Idea

A neural audio codec is a learned compression system that converts raw audio waveforms into sequences of discrete tokens and reconstructs audio from those tokens. The standard architecture is encoder–quantizer–decoder. The encoder downsamples the waveform to a sequence of latent vectors at a fixed frame rate; a Residual Vector Quantization (RVQ) module converts these to discrete indices across multiple codebook layers (codebook levels); and a convolutional decoder synthesizes the waveform from the quantized representations.

In the context of speech language models and TTS, the first RVQ layer (RVQ-1, "semantic" stream) captures phonetic/semantic content and drives autoregressive language models. The remaining RVQ layers (RVQ-rest, "acoustic" stream) capture fine spectral and prosodic detail and are typically predicted by a faster non-autoregressive model. The frame rate of the codec determines the sequence length fed to language models, directly affecting computational cost (quadratic in attention) and training/inference time.

## Methods and Variants

**Fixed-rate high-quality codecs (EnCodec, DAC).** Operate at 50–75 Hz, prioritizing reconstruction quality over downstream LM efficiency. DAC uses periodic activations and codebook utilization improvements. EnCodec (75 Hz, 8 RVQ layers) is the original codec used in VALL-E [[2301.02111]] and remains a widely cited baseline.

**SSL-distillation semantic codecs (SpeechTokenizer, Mimi, DualCodec).** Distill self-supervised representations (HuBERT, WavLM, w2v-bert-2) into the RVQ-1 stream to ensure it carries semantic content. Mimi and DualCodec operate at 12.5 Hz by using higher encoder downsampling rates. Adequate for 12.5 Hz but struggle below that.

**ASR-feature-guided dynamic codecs (FlexiCodec [[2510.00981]]).** Use a frozen ASR encoder (SenseVoice-Small) to extract semantic features and guide adaptive frame merging. Adjacent frames with cosine similarity above a threshold are merged, adaptively allocating more frames to phonetically dense regions. Achieves 3–12.5 Hz controllable rate with superior semantic preservation compared to fixed-rate approaches. Key insight: ASR features are more semantically concentrated than SSL features, enabling more effective frame merging.

**Text-assisted codecs (TaDiCodec, TASTE).** Use text transcription to assist ultra-low-rate coding. Achieve ≤ 6.25 Hz but require ground-truth text, limiting applicability in non-TTS settings. TaDiCodec [[2508.16790]] achieves 6.25 Hz / 0.0875 kbps via a single-stage end-to-end flow-matching training with Binary Spherical Quantization (BSQ), text-aware de-tokenization, and a prompt mechanism — without adversarial training or SSL distillation. The reconstruction-generation gap framing (degradation when tokens are predicted by a LM rather than encoding reference speech) is a practical lens for evaluating tokenizer LM-friendliness.

**Single-codebook codecs (WavTokenizer, SemantiCodec, X-Codec2/XCodec2-S).** Use a single FSQ/VQ codebook for simplicity, operating at 40–75 Hz. Llasa+ [[2508.06262]] uses XCodec2-S, a causal streaming adaptation of XCodec2 trained by freezing encoder+VQ and fine-tuning only the causal decoder.

**Continuous tokenizers.** [[2025.findings-naacl.184]] (Cont-SPT) challenges whether discrete quantization is necessary: continuous AR mel prediction (MELLE [[2025.acl-long.65]]) achieves human-parity quality without any codec. DiTAR [[2502.03930]] uses a continuous VAE at 40 Hz/64-dim as the generation target. LongCat-AudioDiT [[2603.29339]] uses a Wav-VAE at 11.72 Hz/64-dim, achieving PESQ 3.237 and STOI 0.967 while outperforming most discrete codecs at similar bitrate. VibeVoice [[2508.19205]] uses a causal VAE at 7.5 Hz (3200× compression) with a σ-VAE variant and a separate ASR-trained semantic tokenizer — removing the VQ bottleneck entirely while preserving the semantic-acoustic split useful for long-form generation.

**Entropy-guided adaptive compression.** Rather than changing the codec itself, [[2509.00503]] compresses HuBERT semantic tokens post-hoc using a lightweight autoregressive LM's predictive entropy to identify segment boundaries, achieving 7–24 Hz with comparable or better ASR/ST task performance, though VC quality degrades at the coarser end.

**Single-stage 2D patch quantization.** [[2509.02244]] treats the mel spectrogram as a 2D image and applies non-overlapping patch quantization with a single shared codebook (K=4096), bypassing RVQ stacks for a simpler streaming-compatible architecture — achieving competitive PESQ at 7.5 kbps though trailing DAC on most metrics.

**Disentangled/factorized codecs (DisCodec, FACodec, SecoustiCodec).** DisCodec [[2512.13251]] uses two-stage FSQ training with graduated soft orthogonality constraints to factorize content, prosody, and timbre while maintaining reconstruction fidelity. SecoustiCodec [[2508.02849]] proposes a single-codebook streaming codec combining VAE+FSQ quantisation with frame-level cross-modal contrastive learning between phoneme and speech representations, achieving explicit semantic-paralinguistic separation without SSL distillation or phoneme labels at inference. The core claim is that frame-level contrastive learning outperforms HuBERT/WavLM distillation for removing residual paralinguistic content from the semantic stream. SPCODEC [[interspeech-2025-0196]] introduces spectral-supervised disentanglement within the quantization stage: the codec embedding is explicitly split into low- and high-frequency groups, and an attention-based prediction module uses quantized low-frequency features to remove predictable high-frequency redundancy, improving codec efficiency at higher bitrates.

**Streaming causal codecs.** Qwen3-TTS [[2601.15621]] (12Hz, fully causal, 16-layer RVQ), Fish Audio S2 [[2603.08823]] (21 Hz, causal sliding-window transformer, DAC-based), and XCodec2-S [[2508.06262]] (single VQ, partial lookahead) all demonstrate that high-quality streaming codecs are achievable while maintaining semantic preservation and reconstruction quality. SpectroStream [[2508.05207]] demonstrates that operating in the 2D time-frequency domain rather than the waveform domain substantially improves perceptual quality for full-band stereo audio at low bitrates (2.7 kbps), with delayed-fusion cross-channel processing as the key enabling design for stereo phase coherence. Dragon-FM [[2507.22746]] introduces chunk-level FSQ quantisation at 12.5 Hz where chunk-wise AR generation across blocks is combined with within-block flow-matching denoising over continuous FSQ embeddings — a design that achieves KV-cache efficiency and bidirectional intra-chunk context simultaneously.

**Dual-stream direct SSL encoding (DualCodec).** [[interspeech-2025-0468]] introduces a two-stream encoder architecture that directly encodes SSL features (w2v-BERT-2.0 16th layer) into the first RVQ-layer tokens, rather than distilling them as a secondary training loss. The waveform stream then encodes only the residual, with the SSL features serving as the actual RVQ-1 codebook target. This reduces English WER in downstream TTS from 28.4% (distillation) to 2.98% at 25 Hz, and Chinese WER from similar levels to 2.91% — a large improvement over SSL distillation, especially for tonal languages.

**Multi-domain evaluation benchmark (CodecBench).** [[2508.20660]] introduces a 19-dataset evaluation framework revealing that acoustic reconstruction quality and semantic information retention are orthogonal objectives in codec design. Codecs trained without semantic objectives achieve best signal reconstruction but exhibit substantially higher WER in ASR probing tasks; the inverse holds for semantically-trained codecs. The finding that codec performance on clean speech does not generalise to noisier or musically rich domains is a practical design warning for SLM-era use cases.

**Speaker-invariant dual-codebook tokenizer (DC-Spin).** [[interspeech-2025-0246]] contributes a speech tokenizer for SLMs that uses a dual-codebook Spin training objective alongside SpinHuBERT pre-training to improve both SLM downstream performance and speech resynthesis intelligibility simultaneously. The proxy analysis finding — that ABX error rate negatively correlates with SLM task performance while n-gram predictability and PNMI are strongly positive predictors — has direct implications for how the field selects and evaluates tokenizers.

**Interpretability probing across codec families.** [[interspeech-2025-0115]] provides systematic analysis of how linguistic content, speaker identity, and pitch distribute across RVQ layers in four major codecs (DAC, SpeechTokenizer, Mimi, BigCodec). The finding that pitch remains the most diffusely encoded attribute even in explicitly disentangled codecs has implications for controllable generation systems that rely on codec representations for pitch manipulation.

**Watermark-aware codec encoder (Watermark-Aware Codecs).** [[interspeech-2025-1993]] demonstrates that codec encoder training can embed access-control logic by training the encoder to output silence when it detects watermarked audio, without any explicit hard-coded gate. The codec-level intervention naturally resists bypass attacks because the TTS model trained on this codec's distribution cannot easily switch to an unprotected codec without retraining.

**Disentangled low-bitrate codec (FreeCodec).** [[interspeech-2025-1440]] achieves competitive codec quality at 0.45 kbps / 57 tokens/s through three independent encoding streams (content at 50 Hz guided by WavLM, prosody at 7 Hz from low-mel-bins, speaker continuous) — surpassing supervised codecs at 2-3× the bitrate on MUSHRA (87.44 vs. 85.5 for DAC at 1 kbps). The result that self-supervised factorisation at ultra-low bitrate matches supervised approaches at higher bitrates is an important compression efficiency benchmark.

## Major Claims

Claims are generalised propositions aggregated from paper evidence. The full claim registry with supporting paper lists is in `wiki/concepts/_evidence/neural-codec.yaml`.

### Strongly Supported

- Reducing codec frame rate below 75 Hz substantially lowers the computational cost of AR TTS without degrading synthesis quality, provided semantic content is preserved in the low-rate tokens.
  Supporting: [[2510.00981]], [[2412.10117]], [[2601.15621]], [[2603.08823]]

- FSQ (Finite Scalar Quantization) achieves near-100% codebook utilization compared to ~23% for VQ, improving downstream ASR error rates for supervised semantic tokenizers.
  Supporting: [[2412.10117]], [[2512.13251]]

- Discrete Representation Inconsistency (DRI) in RVQ codecs, caused by wide convolutional receptive fields, degrades downstream language model training by producing context-dependent tokenizations of identical audio.
  Supporting: [[2025.acl-long.1498]]

- ASR-supervised features produce more semantically concentrated codec tokens than SSL features at ultra-low frame rates, enabling dramatically better WER preservation.
  Supporting: [[2510.00981]], [[2412.17048]]

### Emerging

- Continuous waveform VAE representations at low frame rates (≤12 Hz) can match or outperform discrete codecs for diffusion-based TTS quality, while higher VAE reconstruction fidelity does not necessarily improve downstream generation quality.
  Supporting: [[2603.29339]], [[2502.03930]]

- Jointly masking all codebook layers per position (full-codebook masking) provides denser training signal for NAR models than per-layer masking strategies.
  Supporting: [[2604.00688]]

- Frame-level cross-modal contrastive learning between phoneme and speech representations achieves cleaner semantic-paralinguistic separation in single-codebook codecs than distillation from HuBERT or WavLM, which retain residual paralinguistic content.
  Supporting: [[2508.02849]]

- VAE-augmented finite scalar quantisation (VAE+FSQ) achieves substantially higher codebook utilisation than VQ-VAE in single-codebook speech codecs, reducing long-tail token distributions that impair downstream LM training.
  Supporting: [[2508.02849]]

- Text conditioning in the codec decoder is a viable lever for achieving extreme compression rates in speech tokenization without adversarial training or SSL distillation.
  Supporting: [[2508.16790]]

- The reconstruction-generation gap — intelligibility degradation when tokens are predicted by a language model rather than encoding reference speech — varies substantially across tokenizer architectures and is not captured by reconstruction metrics alone.
  Supporting: [[2508.16790]]

- Single-stage end-to-end training of speech codec quantization and reconstruction via a flow-matching objective is sufficient to jointly optimise both stages without multi-stage pipelines.
  Supporting: [[2508.16790]]

- Neural speech codecs operating below approximately 1,100 bps degrade in intelligibility relative to uncoded speech even when overall quality metrics suggest acceptable performance.
  Supporting: [[interspeech-2025-0984]]

- WER from large-vocabulary ASR is not a valid proxy for subjective speech intelligibility in codec benchmarking on isolated word stimuli.
  Supporting: [[interspeech-2025-0984]]

- Imposing spectral structure on codec latent embeddings via supervised low/high frequency disentanglement improves codec efficiency at higher bitrates where predictable cross-band redundancy is largest.
  Supporting: [[interspeech-2025-0196]]

- Operating neural codecs in the 2D time-frequency domain instead of the waveform domain yields substantially better perceptual quality for full-band audio at low bitrates where waveform-domain architectures suffer.
  Supporting: [[2508.05207]]

- Matching codec token rates across modalities (speech and visual) enables frame-level synchronisation in joint audiovisual generation without post-hoc alignment.
  Supporting: [[2508.04585]]

- Directly encoding SSL features as the first RVQ-layer tokens (dual-stream architecture) substantially outperforms distillation from the same SSL model, especially for tonal languages where pitch fidelity is critical.
  Supporting: [[interspeech-2025-0468]]

- Acoustic reconstruction quality and semantic information retention are orthogonal objectives in codec design; optimising for one does not reliably improve the other.
  Supporting: [[2508.20660]]

- Self-supervised factorisation of speech into content, speaker, and prosody streams can achieve competitive reconstruction quality at ultra-low bitrate (below 0.5 kbps), matching supervised codecs at 2–3× the bitrate.
  Supporting: [[interspeech-2025-1440]]

- Pitch is the most diffusely encoded speech attribute in neural audio codecs regardless of whether explicit disentanglement objectives were applied, complicating any pitch-manipulation capability that relies on codec-level representations.
  Supporting: [[interspeech-2025-0115]]

### Contested

> [!warning]
> Whether discrete codec tokens or continuous representations are the superior target for AR speech language models remains unresolved, with papers arguing both directions.
> Supporting: [[2025.findings-naacl.184]], [[2508.19098]], [[2502.03930]] / Contradicting: [[2301.02111]], [[2407.05407]], [[2601.15621]]

## Relationship to Other Concepts

### Replaces or Supersedes
- [[gan-vocoder]] — in autoregressive TTS pipelines, neural codecs with learned decoders replace standalone GAN vocoders as the waveform synthesis stage, though GAN training objectives remain common inside codec decoders

### Extends or Builds On
- [[self-supervised-speech]] — SSL models (HuBERT, WavLM, w2v-bert-2) are distilled into RVQ-1 to ensure semantic content; ASR-supervised models (SenseVoice) are increasingly used instead for better semantic concentration

### Competes With
- [[autoregressive-codec-tts]] — neural codecs enable AR TTS but the relationship is symbiotic rather than competitive; continuous mel-spectrograms (no codec) remain competitive for non-AR systems: continuous mel-spectrograms remain the dominant representation for non-autoregressive TTS (flow-matching, diffusion); they do not require discrete tokenization. Discrete codecs are specifically required for autoregressive language model training. The FlexiCodec-TTS results ([[2510.00981]]) show that for the AR stage, 6.25 Hz codec tokens provide no degradation vs. 12.5 Hz while providing 1.4–2.1× AR speedup, but the NAR stage still benefits from high frame rate (50 Hz mel > 12.5 Hz codec features for naturalness)

### Commonly Paired With
- [[spoken-language-model]] — neural codecs are the standard speech tokenization layer for speech LMs; the codec frame rate and RVQ design directly determine the SLM's vocabulary and sequence length properties
- [[autoregressive-codec-tts]] — codec tokens are the generation target for AR TTS LMs (VALL-E family, CosyVoice); every AR TTS system in the corpus uses a neural codec as its representation layer

## Representative Papers

### Foundational
- [[2210.13438]] — EnCodec introduces the streaming RVQ-based convolutional codec with gradient balancer and MS-STFT discriminator that outperforms Opus and SoundStream across all tested bandwidths; open-source release establishes it as the reference codec for downstream speech and audio research
- [[2301.02111]] — first TTS system demonstrating that neural codec tokens (EnCodec 75 Hz) enable large-scale language model pre-training for zero-shot TTS, establishing the codec-LM paradigm

### Influential
- [[2412.10117]] — demonstrates FSQ achieves 100% codebook utilization vs. 23% for VQ, establishing FSQ as the preferred quantization method for supervised semantic tokenizers
- [[2025.acl-long.1498]] — introduces the DRI (Discrete Representation Inconsistency) phenomenon and consistency accuracy metric, providing a principled diagnosis of a previously overlooked codec failure mode
- [[2407.05407]] — supervised semantic tokenizer inserting VQ into ASR encoder for strong text-semantic alignment; established the supervised S3 tokenizer design pattern
- [[2510.00981]] — ASR-feature-guided dynamic frame merging achieving 6.25 Hz controllable rate, breaking the 12.5 Hz floor and providing 7.3× AR speedup over CosyVoice

### Recent Highlights
- [[2601.15621]] — fully causal 12.5 Hz 16-layer RVQ with w2v-BERT semantic distillation achieving PESQ-WB 3.21, outperforming Mimi and FireRedTTS2 tokenizer while enabling streaming
- [[2603.29339]] — waveform VAE at 11.72 Hz outperforming most discrete codecs for diffusion TTS while revealing that higher VAE fidelity counterintuitively does not improve downstream generation quality
- [[2604.00688]] — demonstrates full-codebook random masking provides 3× denser training signal than per-layer masking, resolving NAR intelligibility gaps

## Open Questions

- SecoustiCodec [[2508.02849]] requires duration-aligned phoneme labels for contrastive training; can frame-level cross-modal contrastive learning be extended to an unsupervised setting without text-speech alignment?
- Can dynamic frame merging be extended to sub-3 Hz without catastrophic quality loss?
- Is there a natural lower bound on frame rate below which semantic content cannot be preserved in discrete tokens?
- How does FlexiCodec's dynamic merging interact with prosodic information? [[2412.17048]] identifies paralinguistic variability in speech tokens as the main bottleneck — do dynamic tokens help with this?
- Streaming/causal encoding is not addressed by FlexiCodec; is dynamic frame merging compatible with online TTS pipelines?
- Multilingual semantic tokens require language-specific fine-tuning; can the ASR-guided merging generalize cross-lingually?
- DRI ([[2025.acl-long.1498]]) is shown to worsen with deeper RVQ layers; does this suggest the deeper-layer representation design needs fundamentally different treatment (e.g. non-contextual sub-quantizers)?
- [[2025.findings-naacl.184]] shows continuous tokens outperform RVQ on a single English benchmark; will continuous-token AR LMs scale to larger data and multilingual settings, or does the MSE regression objective become unstable?
- VocalNet [[2025.emnlp-main.989]] and other speech LLMs now use CosyVoice2 semantic tokens (25 Hz) rather than traditional RVQ codecs; does this represent a shift toward hybrid supervised-semantic tokenization as the default?
- LongCat-AudioDiT [[2603.29339]] shows higher VAE reconstruction fidelity does not improve TTS quality; what is the optimal VAE bottleneck dimension and frame rate for diffusion-based TTS?
- DisCodec [[2512.13251]] shows that content-prosody FSQ can be fused back into a single stream for LM training; does this generalize to timbre as well, enabling a single-stream codec with full factorization?
- Fish Audio S2 [[2603.08823]] uses a 21 Hz causal codec; Qwen3-TTS [[2601.15621]] uses 12.5 Hz — which frame rate offers the optimal latency/quality trade-off for production streaming?
- DualCodec [[interspeech-2025-0468]] reduces Chinese TTS WER to 2.49% at 12.5 Hz; does the dual-stream SSL encoding approach generalize to languages beyond English and Mandarin without re-validating the SSL model choice?
- CodecBench [[2508.20660]] identifies a clean-speech vs. expressive/noisy performance gap in all codecs; which architectural choice would most efficiently close this gap for SLM-era use cases?
- FreeCodec [[interspeech-2025-1440]] achieves strong results at 0.45 kbps; is a further reduction to 0.2–0.3 kbps achievable with factorised self-supervised designs, or is there a floor determined by phonetic coverage requirements?
- The ABX metric for tokenizer evaluation is shown by DC-Spin [[interspeech-2025-0246]] to negatively correlate with SLM downstream performance; which proxy metric should replace ABX as the primary evaluation signal for speech tokenizers?

## Trend Summary

2021–2022: SoundStream, EnCodec established neural audio codecs as the standard compression tool at 50–75 Hz. 2023: SpeechTokenizer introduced semantic distillation for RVQ-1; first purpose-built TTS codecs. 2024: Mimi and DualCodec reached 12.5 Hz while maintaining semantic quality; CosyVoice 2 [[2412.10117]] demonstrated FSQ achieves 100% codebook utilization vs. 23% for VQ — establishing FSQ as the preferred quantization method for supervised semantic tokenizers. 2025: FlexiCodec ([[2510.00981]]) breaks the 12.5 Hz floor to 6.25 Hz; DisCodec [[2512.13251]] introduces two-stage disentangled FSQ for independent prosody/timbre control; XCodec2-S [[2508.06262]] demonstrates streaming single-codebook causal adaptation; MELLE [[2025.acl-long.65]] and DiTAR [[2502.03930]] challenge whether discrete codecs are necessary at all. SecoustiCodec [[2508.02849]] proposes that frame-level cross-modal contrastive learning outperforms SSL distillation for semantic disentanglement in single-codebook streaming codecs, and that VAE+FSQ nearly eliminates codebook underutilisation. SPCODEC [[interspeech-2025-0196]] demonstrates that spectral supervision of codec embeddings and inter-group cross-frequency prediction reduce bitstream redundancy and improve POLQA at standard bitrates. SpectroStream [[2508.05207]] establishes a time-frequency domain baseline for general audio (music, 48 kHz stereo) showing large ViSQOL gains over waveform-domain DAC at low bitrates. Dragon-FM [[2507.22746]] introduces a hybrid chunk-AR+flow-matching design using FSQ at 12.5 Hz, demonstrating that continuous denoising of discrete token embeddings is a viable target representation. 2026: Qwen3-TTS [[2601.15621]] demonstrates a 12.5 Hz 16-layer RVQ achieving PESQ-WB 3.21 — outperforming Mimi and FireRedTTS2 tokenizer while enabling fully causal streaming; Fish Audio S2 [[2603.08823]] demonstrates a 21 Hz streaming DAC-based codec with w2v-BERT semantic distillation and ConvNeXt V2 extensions; LongCat-AudioDiT [[2603.29339]] establishes waveform VAE at 11.72 Hz as competitive with discrete codecs for diffusion TTS, revealing that higher VAE fidelity counterintuitively does not improve downstream generation quality. The field is now tracking two competing codec futures: lower-frame-rate discrete (toward text-rate ~4.5 Hz) and continuous waveform latent (bypassing codec discretization entirely). Integration pass 7 (Aug–Sep 2025) adds four papers: TaDiCodec [[2508.16790]] establishes 6.25 Hz single-codebook text-assisted diffusion tokenization as a viable extreme-compression option and introduces the reconstruction-generation gap as a practical LM-friendliness metric; VibeVoice [[2508.19205]] demonstrates a 7.5 Hz continuous causal VAE achieving PESQ 3.068 and UTMOS 4.181 — suggesting continuous tokenizers are competitive even at very low frame rates; the entropy-based compression approach [[2509.00503]] demonstrates 15 Hz adaptive compression of SSL tokens that outperforms fixed-length downsampling for ASR while degrading for VC; and SITool [[interspeech-2025-0984]] provides the first systematic correlation study between DRT intelligibility and objective metrics for 13 codecs, with the key negative finding that WER is uncorrelated with subjective intelligibility on isolated-word stimuli. Additionally, the variable frame rate (TFC) approach [[interspeech-2025-1289]] introduces the first entropy-based temporal resolution allocation within a codec, achieving better WER and UTMOS than fixed-rate DAC at equivalent bitrate. A 2D patch quantization alternative to RVQ [[2509.02244]] extends the design space of single-stage codecs, though results currently trail DAC at comparable bitrates. Interspeech 2025 adds four noteworthy contributions to this landscape: DualCodec [[interspeech-2025-0468]] demonstrates that dual-stream SSL encoding substantially outperforms distillation for semantic preservation, especially in tonal languages; CodecBench [[2508.20660]] establishes that acoustic fidelity and semantic content preservation are orthogonal evaluation axes; FreeCodec [[interspeech-2025-1440]] demonstrates self-supervised 3-way factorisation at 0.45 kbps beating supervised codecs at 2-3× the bitrate; and the interpretability analysis [[interspeech-2025-0115]] reveals that pitch remains the most opaque attribute across all four major codec families, including explicitly disentangled ones. A security dimension has also emerged: watermark-aware codec training [[interspeech-2025-1993]] establishes codec-level intervention as a practical access-control mechanism for TTS pipelines. DC-Spin [[interspeech-2025-0246]] challenges the standard ABX proxy metric, showing that n-gram predictability and phoneme mutual information are stronger predictors of SLM downstream performance.

## All Papers

| ID | Title | Venue | Year | Key use of this concept |
|----|-------|-------|------|------------------------|
| [[2210.13438]] | High Fidelity Neural Audio Compression (EnCodec) | arXiv | 2022 | Introduces the streaming RVQ codec with gradient balancer and MS-STFT discriminator that becomes the reference codec for downstream AR TTS and SLM research |
| [[2301.02111]] | Neural Codec Language Models are Zero-Shot Text to Speech Synthesizers (VALL-E) | arXiv | 2023 | First TTS system built on top of a neural audio codec (EnCodec 75 Hz, 8 RVQ layers); demonstrates that codec tokens enable large-scale language model pre-training for zero-shot TTS |
| [[2510.00981]] | FlexiCodec: A Dynamic Neural Audio Codec for Low Frame Rates | arXiv (ICLR 2026) | 2025 | Proposes ASR-feature-guided dynamic frame merging to achieve 3–12.5 Hz controllable codec with near-GT semantic preservation at 6.25 Hz; enables TTS AR speedup of 7.3× over CosyVoice |
| [[2025.acl-long.1498]] | Analyzing and Mitigating Inconsistency in Discrete Speech Tokens for Neural Codec Language Models | ACL | 2025 | Introduces consistency accuracy metric and DRI (Discrete Representation Inconsistency) phenomenon; proposes slice-consistency and perturbation-consistency losses that increase RVQ consistency by up to 36% and reduce downstream VALL-E WER by 1.98% |
| [[2025.findings-naacl.184]] | Continuous Speech Tokenizer in Text To Speech | NAACL | 2025 | Replaces RVQ with a continuous speech tokenizer; retains encoder output directly, avoiding high-frequency quantization loss; shows information retention advantage at 8 kHz (0.55 vs. 0.34 for discrete) and improved downstream TTS quality |
| [[2025.acl-long.346]] | ControlSpeech: Towards Simultaneous and Independent Zero-shot Speaker Cloning and Zero-shot Language Style Control | ACL | 2025 | Uses frozen FACodec (NaturalSpeech 3 factorized codec) as disentanglement backbone; content, prosody, acoustic detail, and timbre codes are separately controlled — demonstrating that a pre-trained factorized codec can serve as the control interface for simultaneous multi-factor TTS |
| [[2025.emnlp-main.989]] | VocalNet: Speech LLMs with Multi-Token Prediction for Faster and High-Quality Generation | EMNLP | 2025 | Uses CosyVoice2 semantic speech tokens (25 Hz) as the codec for speech generation; the codec choice enables compatibility with flow-matching acoustic decoder and HiFi-GAN vocoder in a full speech LLM pipeline |
| [[2025.acl-long.682]] | Recent Advances in Speech Language Models: A Survey | ACL | 2025 | Surveys all major neural codec architectures (SoundStream, EnCodec, DAC, SpeechTokenizer, Mimi) and their roles as speech tokenizers; classifies by objective (semantic understanding, acoustic generation, mixed) and discusses discrete vs. continuous representations |
| [[2412.17048]] | Why Do Speech Language Models Fail to Generate Semantically Coherent Outputs? | arXiv | 2026 | Uses HuBERT-Large discrete clusters as codec tokens in a controlled study; identifies paralinguistic variability in speech tokens as the dominant cause of SLM coherence failure — motivating codec designs that suppress paralinguistic variation |
| [[2025.acl-long.1043]] | OZSpeech: One-step Zero-shot Speech Synthesis with Learned-Prior-Conditioned Flow Matching | ACL | 2025 | Uses FACodec (NaturalSpeech 3 factorized codec) as the speech representation for learned-prior OT-CFM; demonstrates that factorized codec tokens enable a prior codes generator to separately predict prosody, content, acoustic detail, and timbre for single-step inference |
| [[2412.10117]] | CosyVoice 2: Scalable Streaming Speech Synthesis with Large Language Models | arXiv | 2024 | Introduces FSQ for speech tokenization: 100% codebook utilization vs. 23% for VQ; ASR error 10.67% vs. 18.26% — establishes FSQ as the preferred quantization method for semantic tokenizers |
| [[2406.02430]] | Seed-TTS: A Family of High-Quality Versatile Speech Generation Models | arXiv | 2024 | Investigates both continuous and discrete tokenizers for TTS; argues tokenizer design is critical to system quality; custom speech tokenizer with consistency distillation for streaming deployment |
| [[2407.05407]] | CosyVoice: A Scalable Multilingual Zero-shot TTS based on Supervised Semantic Tokens | arXiv | 2024 | Supervised S3 tokenizer: inserts VQ into ASR encoder (SenseVoice-Large) to produce tokens with strong text-semantic alignment; single codebook (4096 codes), 25 Hz |
| [[2510.12210]] | DiSTAR: Diffusion over a Scalable Token Autoregressive Representation | arXiv | 2025 | Custom MAGICODEC-based 9-layer RVQ at 64 Hz; test-time bitrate control via stochastic layer truncation; top RVQ layers encode acoustic detail (WER stable when removed, SIM drops) |
| [[2502.03930]] | DiTAR: Diffusion Transformer Autoregressive Modeling for Speech Generation | arXiv | 2025 | Custom VAE-based continuous tokenizer (40 Hz, 64-dim latent) replacing discrete codec; adversarial training via BigVGAN decoder; achieves high reconstruction quality without RVQ |
| [[2512.14291]] | GLM-TTS Technical Report | arXiv | 2025 | Whisper-VQ tokenizer at 25 Hz with 32k codebook, pitch estimator module, and non-causal architecture; outperforms Mimi on PESQ/UTMOS/MOS metrics |
| [[2508.06262]] | Llasa+: Free Lunch for Accelerated and Streaming Llama-Based Speech Synthesis | arXiv | 2025 | XCodec2-S: streaming causal adaptation of XCodec2 (freeze encoder+VQ, fine-tune decoder only); achieves 95% of XCodec2 performance while enabling streaming waveform reconstruction |
| [[2512.13251]] | DisCo-Speech: Controllable Zero-Shot Speech Generation with A Disentangled Speech Codec | arXiv | 2025 | DisCodec: two-stage FSQ disentanglement with per-attribute quantization (content, prosody, timbre); fuses content+prosody before re-quantization for LM training; superior VC disentanglement vs. MSR-Codec and FACodec |
| [[2604.00688]] | OmniVoice: Towards Omnilingual Zero-Shot TTS with Diffusion Language Models | arXiv | 2026 | Higgs-audio 8-codebook tokenizer; full-codebook random masking provides 3x denser training signal than per-layer masking; resolves NAR intelligibility gap with LLM initialization |
| [[2508.16332]] | Vevo2: A Unified and Controllable Framework for Speech and Singing Voice Generation | arXiv | 2025 | VQ-VAE prosody tokenizer (6.25 Hz, chromagram-based) + content-style tokenizer (12.5 Hz) bridging speech and singing prosody distributions without MIDI annotation |
| [[2510.02848]] | Flamed-TTS: Flow Matching Attention-Free Models for Efficient Zero-shot TTS | arXiv | 2025 | FACodec (NaturalSpeech 3) used as the prior distribution for flow matching; 6 disentangled code streams (prosody, content, acoustic) enable attention-free denoiser |
| [[2512.04720]] | M3-TTS: Multi-modal DiT Alignment and Mel-latent for Zero-shot TTS | arXiv | 2025 | Mel-VAE (43 Hz, 40-dim) vs. standard 100-dim mel-spectrogram: 3x training speedup with similar WER; reveals SIM cost of latent bottleneck for voice cloning |
| [[2603.29339]] | LongCat-AudioDiT: High-Fidelity Diffusion TTS in the Waveform Latent Space | arXiv | 2026 | Wav-VAE (11.72 Hz, 64-dim): PESQ 3.237, STOI 0.967 on LibriTTS test-clean; higher fidelity VAE counterintuitively does not improve downstream TTS quality |
| [[2601.15621]] | Qwen3-TTS Technical Report | arXiv | 2026 | Qwen-TTS-Tokenizer-12Hz: fully causal 12.5 Hz 16-layer RVQ with w2v-BERT semantic distillation and GAN training; PESQ-WB 3.21, UTMOS 4.16, SPK-SIM 0.95 — outperforms Mimi and FireRedTTS2 tokenizer |
| [[2603.08823]] | Fish Audio S2 Technical Report | arXiv | 2026 | Streaming DAC-based codec: 21 Hz, 10-layer RVQ, causal sliding-window transformer, w2v-BERT semantic distillation, ConvNeXt V2 extensions, EVA-GAN decoder |
| [[2025.ccl-1.80]] | Lao-English Code-Switched Speech Synthesis Via Neural Codec Language Modeling | workshop | 2025 | Uses EnCodec (inherited from VALL-E X) for low-resource Lao-English code-switching; demonstrates that phoneme alignment extensions compensate for semantic limitations of VALL-E X's EnCodec tokens |
| [[2509.09631]] | DiFlow-TTS: Compact and Low-Latency Zero-Shot TTS with Factorized Discrete Flow Matching | arXiv | 2025 | FACodec (NaturalSpeech 3) used as the factorized discrete target space for discrete flow matching; prosody and acoustic tokens as separate DFM targets |
| [[2025.acl-long.654]] | Language-Codec: Bridging Discrete Codec Representations and Speech Language Models | ACL | 2025 | Introduces MCRVQ (Masked Channel RVQ) that redistributes information load across early codebook channels, lowering prediction entropy for downstream AR LMs; downstream VALL-E SPK-SIM improves from 0.612 to 0.700 (+14%) and MobileSpeech SPK-SIM to 0.771 |
| [[2502.11128]] | FELLE: Autoregressive Speech Synthesis with Token-Wise Coarse-to-Fine Flow Matching | arXiv | 2025 | Uses mel-spectrogram as generation target (codec-free AR), with a per-token flow-matching head using the previous mel frame as informed prior rather than Gaussian noise; achieves higher WER/MOS on LibriSpeech than MELLE |
| [[2508.19098]] | CLEAR: Continuous Latent Autoregressive Modeling for High-quality and Efficient TTS | arXiv | 2025 | Replaces discrete RVQ tokens with continuous VAE latents decoded per-token via MLP rectified-flow head; WER 1.88%, UTMOS 4.22 on LibriSpeech-PC test-clean, outperforming DiTAR with lower latency |
| [[2603.18090]] | MOSS-TTS Technical Report | arXiv | 2026 | MOSS-Audio-Tokenizer: causal Transformer-based tokenizer at 1000–4000 bps achieving PESQ-WB 3.69 (EN) and SIM 0.88 on LibriSpeech, outperforming Mimi and XY-Tokenizer at the same bitrate |
| [[2604.12438]] | An Ultra-Low Latency End-to-End Streaming Speech Synthesis Architecture | arXiv | 2026 | Uses depth-wise sequential decoding over 32 Mimi codec RVQ layers in a non-autoregressive transformer, achieving RTF ~0.0033 (303× real-time) and 48.99 ms first-byte latency |
| [[interspeech-2025-0669]] | PAST: Phonetic-Acoustic Speech Tokenizer | Interspeech | 2025 | Hybrid tokenizer replacing SSL pseudo-label distillation with supervised CTC + phoneme classification losses directly on RVQ-1; state-of-the-art phonetic metrics (PNMI 0.75, WER 15.7%) while maintaining reconstruction (PESQ 3.55) without external SSL teacher |
| [[interspeech-2025-0319]] | Improving Noise Robustness of LLM-based Zero-shot TTS via Discrete Acoustic Token Denoising | Interspeech | 2025 | Operates speech enhancement in the discrete acoustic token domain; two-stage Conformer denoiser (token predictor + embedding refiner) removes noise from RVQ-encoded prompts at 1.10G FLOPs vs. 38.93G for waveform-domain SE |
| [[interspeech-2025-0310]] | Exploring the Effect of Segmentation and Vocabulary Size on Speech Tokenization for Speech LMs | Interspeech | 2025 | Systematic 64-configuration grid search over HuBERT K-means segmentation width (20–280ms) and vocabulary size (128–16384); finds N=80ms + K=16384 beats original-resolution tokenization with 50% data and 70% runtime reduction |
| [[interspeech-2025-0989]] | HiFiTTS-2: A Large-Scale High Bandwidth Speech Dataset | Interspeech | 2025 | Bandwidth estimation pipeline for 22kHz/44.1kHz dataset construction; empirical finding that speaker diversity rather than audio quality alone drives zero-shot codec LM performance |
| [[2025.findings-acl.1051]] | LLMVoX: Autoregressive Streaming Text-to-Speech Model for Any LLM | ACL | 2025 | Uses WavTokenizer (single-layer RVQ, 4096-vocab, 40-75 tokens/sec) for streaming speech synthesis; demonstrates that a single-codebook discrete tokenizer is sufficient for high-quality streaming TTS when the sequence is manageable |
| [[2507.22746]] | Next Tokens Denoising for Speech Synthesis (Dragon-FM) | arXiv | 2025 | Custom FSQ codec at 12.5 Hz, 48 kHz; chunk-wise AR generation with within-chunk flow-matching denoising over continuous FSQ embeddings; causal decoder enables streaming |
| [[2508.02849]] | SecoustiCodec | arXiv | 2025 | Single-codebook streaming codec with VAE+FSQ and frame-level cross-modal contrastive learning; 98% codebook utilisation; 12.08 ms initial latency; proposes contrastive learning outperforms SSL distillation for semantic disentanglement |
| [[2508.04585]] | UniTalker | arXiv | 2025 | Custom FSQ-based speech codec (SenseVoice-Large + FSQ, 25 Hz) and LmkCodec for facial landmarks at matching rate; demonstrates matching token rates enable joint audiovisual synchronisation |
| [[2508.05207]] | SpectroStream | arXiv | 2025 | Time-frequency domain neural codec for 48 kHz stereo; 2D convolutional encoder with delayed-fusion stereo; ViSQOL 3.21 vs. DAC 1.47 at 2.7 kbps; establishes TF domain as superior for full-band low-bitrate audio |
| [[2508.07302]] | XEmoRAG | arXiv | 2025 | Uses X-Codec2 as the backbone codec for cross-lingual emotion TTS; flow-matching alignment bridges discrete codec tokens and mel-spectrograms for speaker identity preservation |
| [[2508.08961]] | DualSpeechLM | arXiv | 2025 | Proposes Understanding-driven Speech Tokenizer (USTokenizer) trained with LLM next-token prediction loss; 250 bps understanding-oriented codec separate from WavTokenizer output codec |
| [[2508.11326]] | MoE-TTS | arXiv | 2025 | Uses CosyVoice2 speech tokenizer at 25 Hz for description-based TTS; 6,561-token vocabulary appended to LLM for speech generation |
| [[interspeech-2025-0196]] | SPCODEC | Interspeech | 2025 | Split-and-prediction codec: spectral supervision of low/high frequency embeddings with inter-group prediction; POLQA 4.5 at 10.66 kbps; demonstrates performance advantage scales with bitrate |
| [[interspeech-2025-0468]] | DualCodec: A Low-Frame-Rate, Semantically-Enhanced Neural Audio Codec | Interspeech | 2025 | Dual-stream encoder directly integrates SSL features as RVQ-1 tokens rather than distilling them; achieves WER 2.98% EN / 2.91% ZH at 25 Hz, substantially outperforming SSL-distilled codecs |
| [[2508.20660]] | CodecBench: A Comprehensive Benchmark for Acoustic and Semantic Evaluation | arXiv | 2025 | Multi-domain evaluation framework exposing acoustic-semantic orthogonality across 14 codec configurations; ASR probing and classification tasks complement signal-level metrics |
| [[interspeech-2025-0246]] | DC-Spin: A Speaker-invariant Speech Tokenizer for Spoken Language Models | Interspeech | 2025 | Dual-codebook Spin tokenizer with SpinHuBERT pre-training; n-gram predictability and phoneme NMI are stronger SLM proxies than ABX; competitive with high-resource SLMs at 1/100th the compute |
| [[interspeech-2025-1440]] | FreeCodec: A Disentangled Neural Speech Codec with Fewer Tokens | Interspeech | 2025 | Self-supervised 3-way factorisation (content 50Hz / prosody 7Hz / speaker continuous) achieves MUSHRA 87.44 at 0.45 kbps, beating DAC at 1 kbps; WavLM-guided content encoder for disentanglement |
| [[interspeech-2025-0115]] | Bringing Interpretability to Neural Audio Codecs | Interspeech | 2025 | Systematic analysis of content/speaker/pitch encoding across RVQ layers in DAC, SpeechTokenizer, Mimi, BigCodec; pitch most diffusely encoded even in disentanglement-oriented codecs |
| [[interspeech-2025-1993]] | Defending Unauthorized Voice Cloning with Watermark-Aware Codecs | Interspeech | 2025 | Codec encoder trained to output silence on watermarked audio; structural defense against voice cloning without hard-coded gates; robustness against 13 audio distortion attack types |
| [[2508.11224]] | Benchmarking Prosody Encoding in Discrete Speech Tokens | ASRU | 2025 | TER-based comparison of prosody sensitivity across SSL codec tokenizer configurations; contrasts semantic vs. acoustic token design for prosody representation |
| [[interspeech-2025-0984]] | Benchmarking Neural Speech Codec Intelligibility with SITool | Interspeech | 2025 | First subjective intelligibility benchmark for 13 codecs; WER uncorrelated with DRT scores; STOI adequate aggregate proxy but not for fine-grained variation |
| [[interspeech-2025-1289]] | Unlocking Temporal Flexibility: Neural Speech Codec with Variable Frame Rate | Interspeech | 2025 | First variable frame rate (TFC) strategy for neural codecs; entropy-guided temporal allocation reduces sequence length while improving WER and UTMOS at matched bitrate |
| [[2508.15931]] | QvTAD | arXiv | 2025 | Uses frozen FACodec (NaturalSpeech3) encoder as speaker embedding backbone for timbre attribute pairwise comparison |
| [[2508.16790]] | TaDiCodec | arXiv | 2025 | 6.25 Hz single-codebook text-conditioned diffusion tokenizer; BSQ quantization; end-to-end single-objective training; documents reconstruction-generation gap as a practical LM-friendliness metric |
| [[2508.19205]] | VibeVoice Technical Report | arXiv | 2025 | 7.5 Hz causal VAE tokenizer (3200× compression); continuous representation without VQ; PESQ 3.068, UTMOS 4.181 on LibriTTS test-clean |
| [[2509.00503]] | Entropy-based Coarse and Compressed Semantic Speech Representation Learning | arXiv | 2025 | Entropy-guided post-hoc compression of HuBERT tokens to 7–24 Hz; outperforms fixed-length pooling for ASR/ST; VC quality degrades at coarser compression |
| [[2509.02244]] | Spectrogram Patch Codec | arXiv | 2025 | Single-stage 2D patch VQ-VAE on mel spectrograms, bypassing RVQ stacks; competitive PESQ at 7.5 kbps with simpler streaming-friendly architecture |
| [[2308.16692]] | SpeechTokenizer: Unified Speech Tokenizer for Speech Large Language Models | arXiv | 2023 | Introduces RVQ-based codec with HuBERT distillation into RVQ-1 to disentangle semantic and acoustic content; foundational SSL-distillation semantic codec design |
| [[2408.16532]] | WavTokenizer: an Efficient Acoustic Discrete Codec Tokenizer for Audio Language Modeling | arXiv | 2024 | Single large-codebook codec achieving higher perceptual reconstruction than multi-codebook RVQ systems; demonstrates single-codebook discrete tokens can outperform multi-codebook in downstream AR generation |
| [[2304.09116]] | NaturalSpeech 2: Latent Diffusion Models are Natural and Zero-Shot Speech and Singing Synthesizers | arXiv | 2023 | Uses continuous codec latent vectors from EnCodec as diffusion target; demonstrates continuous codec representations as viable alternative to discrete tokens |
| [[2409.00750]] | MaskGCT: Zero-Shot Text-to-Speech with Masked Generative Codec Transformer | arXiv | 2024 | Uses VQ-VAE acoustic tokens from SemantiCodec; demonstrates VQ-VAE vector quantisation reduces information loss in tonal languages vs. k-means quantisation of SSL features |
| [[2406.05370]] | VALL-E 2: Neural Codec Language Models are Human Parity Zero-Shot Text to Speech Synthesizers | arXiv | 2024 | Uses EnCodec with grouped codec code prediction to reduce effective sequence length; demonstrates multi-token grouping as AR efficiency improvement |
| [[2503.01710]] | Spark-TTS: An Efficient LLM-Based Text-to-Speech Model with Single-Stream Decoupled Speech Tokens | arXiv | 2025 | BiCodec: single-stream codec combining global speaker tokens and semantic speech tokens for LLM-native TTS; uses FSQ for global quantisation |
| [[2502.04128]] | Llasa: Scaling Train-Time and Inference-Time Compute for Llama-based Speech Synthesis | arXiv | 2025 | Uses XCodec single-codebook codec for single-stage AR TTS; demonstrates scaling compute via inference-time verifiers over discrete codec tokens |
| [[2407.04051]] | FunAudioLLM: Voice Understanding and Generation Foundation Models for Natural Interaction Between Humans and LLMs | arXiv | 2024 | Uses CosyVoice S3 tokenizer (ASR-derived supervised semantic tokens); establishes ASR encoder-derived codec as standard in Chinese TTS systems |
| [[2305.11000]] | SpeechGPT: Empowering Large Language Models with Intrinsic Cross-Modal Conversational Abilities | arXiv | 2023 | Discretises speech using HuBERT-based k-means tokens; early demonstration of discrete speech tokenisation enabling unified speech LLM without separate codec |
| [[2409.06666]] | LLaMA-Omni: Seamless Speech Interaction with Large Language Models | arXiv | 2024 | Uses HuBERT discrete units for input speech tokenisation; streaming speech output via non-autoregressive CTC from LLM hidden states without a generative codec |
| [[2411.00774]] | Freeze-Omni: A Smart and Low Latency Speech-to-speech Dialogue Model with Frozen LLM | arXiv | 2024 | Uses Whisper encoder for speech input and streaming codec-based TTS for speech output; demonstrates codec integration in a frozen-LLM spoken dialogue architecture |
| [[2409.03283]] | FireRedTTS: A Foundation Text-To-Speech Framework for Industry-Level Generative Speech Applications | arXiv | 2024 | Custom two-stage waveform generation with mel decoder and super-resolution vocoder; demonstrates separation of low-rate codec generation from high-rate vocoder as design pattern |
| [[2408.16725]] | Mini-Omni: Language Models Can Hear, Talk While Thinking in Streaming | arXiv | 2024 | Uses multi-codebook codec with batch inference strategy for parallel speech token generation; demonstrates simultaneous text-audio generation in a streaming speech LM |
| [[1609.03499]] | WaveNet: A Generative Model for Raw Audio | arXiv | 2016 | Pre-codec autoregressive raw-waveform model; established the sequential synthesis baseline that codecs later replaced for LM-based TTS |
| [[2305.02765]] | HiFi-Codec: Group-residual Vector Quantization for High Fidelity Audio Codec | arXiv | 2023 | Introduces group-RVQ reducing downstream AR burden; demonstrates that codebook count is a practical constraint for generative models |
| [[2409.05377]] | BigCodec: Pushing the Limits of Low-Bitrate Neural Speech Codec | arXiv | 2024 | Shows model capacity rather than data is the binding constraint at ~1 kbps; LSTM encoder improves speaker similarity; scaling saturates at ~159M params |
| [[2411.19842]] | Scaling Transformers for Low-Bitrate High-Quality Speech Coding (TAAE) | arXiv | 2024 | Transformer-scaled FSQ codec with WavLM perceptual loss; demonstrates systematic quality gains from architecture scaling; FSQ achieves near-perfect codebook utilization |
| [[2305.09636]] | SoundStorm: Efficient Parallel Audio Generation | arXiv | 2023 | Uses AudioLM codec tokens (SoundStream) for parallel masked decoding; demonstrates that fine-level RVQ tokens are conditionally independent given coarse tokens |
| [[2306.12925]] | AudioPaLM: A Large Language Model That Can Speak and Listen | arXiv | 2023 | AudioPaLM tokenizer quality identified as primary bottleneck in LLM-based speech; stronger semantic tokenizers yield large downstream gains independent of LM scale |
| [[2303.03926]] | VALL-E X: Cross-lingual Neural Codec Language Modeling | arXiv | 2023 | Cross-lingual extension of EnCodec-based VALL-E; uses same 75 Hz EnCodec tokenizer for cross-lingual voice transfer |
| [[2305.07243]] | Better speech synthesis through scaling (Tortoise TTS) | arXiv | 2023 | Uses discrete tokens from VQVAE for AR stage; demonstrates cascaded discrete AR + continuous diffusion as alternative to pure codec-LM TTS |
| [[2402.01912]] | Natural language guidance of high-fidelity text-to-speech with synthetic annotations | arXiv | 2024 | Shows codec choice has measurable effect on MOS; higher-fidelity codecs directly improve perceptual quality in AR TTS |
| [[2402.08093]] | BASE TTS: Lessons from building a billion-parameter TTS model on 100K hours | arXiv | 2024 | Uses disentangled SSL-based speech tokens (not RVQ) for AR stage; BPE on speech tokens reduces sequence length 40%; shows SSL tokens outperform acoustic codecs for low-resource languages |
| [[2406.04904]] | XTTS: a Massively Multilingual Zero-Shot Text-to-Speech Model | arXiv | 2024 | Low-frequency codec codebook entry pruning improves expressiveness; Perceiver Resampler over codec tokens for speaker conditioning |
| [[2406.18009]] | E2 TTS: Embarrassingly Easy Fully Non-Autoregressive Zero-Shot TTS | arXiv | 2024 | Uses mel-spectrogram rather than codec tokens; unsupervised pretraining on unlabeled speech improves downstream zero-shot TTS |
| [[2403.16973]] | VoiceCraft: Zero-Shot Speech Editing and Text-to-Speech in the Wild | arXiv | 2024 | Autoregressive infilling over EnCodec tokens; demonstrates codec token rearrangement for bidirectional context conditioning |
| [[2407.08551]] | MELLE: Autoregressive Speech Synthesis without Vector Quantization | arXiv | 2024 | Codec-free AR TTS using continuous mel frames; shows continuous representations preserve more speaker-relevant information than VQ-quantized codec codes |
| [[2502.11946]] | Step-Audio: Unified Understanding and Generation in Intelligent Speech Interaction | arXiv | 2025 | Dual-codebook interleaved tokenizer combining linguistic and semantic representations; achieves lower ASR error than either codebook alone |
| [[2502.17239]] | Baichuan-Audio: A Unified Framework for End-to-End Speech Interaction | arXiv | 2025 | 8-layer multi-codebook RVQ with semantic alignment; each additional layer reduces ASR error substantially up to 8 layers; flow-matching decoder recovers quality lost in quantization |
| [[2310.00704]] | UniAudio: An Audio Foundation Model Toward Universal Audio Generation | arXiv | 2023 | Hierarchical RVQ factorization into inter-frame and intra-frame modeling reduces training cost; uses EnCodec and other codecs for multi-task audio generation |
| [[2501.06282]] | MinMo: A Multimodal Large Language Model for Seamless Voice Interaction | arXiv | 2025 | Streaming voice decoder over codec tokens; AR streaming decoder outperforms CTC for naturalness in aligned speech LMs |
| [[2507.16632]] | Step-Audio 2 Technical Report | arXiv | 2025 | Uses interleaved discrete audio tokens; RL-enhanced codec token prediction for audio language modeling |
| [[2104.00355]] | Speech Resynthesis from Discrete Disentangled Self-Supervised Representations | arXiv | 2021 | Foundational demonstration that SSL content units (HuBERT 50-unit) can serve as an ultra-low-bitrate codec (365 bps) outperforming Opus at 9 kbps in subjective quality |
| [[2406.07855]] | VALL-E R: Robust and Efficient Zero-Shot TTS via Monotonic Alignment | arXiv | 2024 | EnCodec 75 Hz with codec-merging: first-layer 2x downsampling halves AR steps with negligible PESQ/STOI impact; demonstrates codec can be modified at inference without retraining |
| [[2406.14294]] | DASB: Discrete Audio and Speech Benchmark | arXiv | 2024 | Evaluates acoustic tokenizers (EnCodec, DAC, Mimi, WavTokenizer) and semantic tokenizers on 11 downstream tasks without relying on waveform reconstruction metrics |
| [[2411.01156]] | Fish-Speech: LLMs for Advanced Multilingual TTS | arXiv | 2024 | GFSQ (Grouped Finite Scalar Vector Quantization): achieves 100% codebook utilisation by combining FSQ with group quantization; first practical GFSQ deployment in a production TTS codec |
| [[2502.05512]] | IndexTTS: Industrial-Level Controllable Zero-Shot TTS | arXiv | 2025 | FSQ-based VAE at 25 Hz, 8192 codes; demonstrates FSQ reaches 100% codebook utilisation with less training data than VQ, though VQ eventually converges at large scale |
| [[2502.06490]] | Recent Advances in Discrete Speech Tokens: A Review | arXiv | 2025 | Controlled cross-paradigm benchmarking; finds reconstruction-semantics trade-off is fundamental: no current tokenizer excels on both axes simultaneously |
| [[2410.11190]] | Mini-Omni2: Open-source GPT-4o with Vision, Speech and Duplex Capabilities | arXiv | 2024 | SNAC (seven-layer RVQ) codec for discrete audio output in a tri-modal spoken conversational agent |
| [[2410.17799]] | OmniFlatten: End-to-end GPT Model for Seamless Voice Conversation | arXiv | 2024 | CosyVoice single-codebook VQ semantic tokenizer (4096 codes) flattened into a single GPT sequence for full-duplex spoken dialogue |
| [[2412.15649]] | SLAM-Omni: Timbre-Controllable Voice Interaction with Single-Stage Training | arXiv | 2024 | CosyVoice supervised semantic tokens at 50 Hz, downsampled 5x; decouples timbre from content tokens for zero-shot speaker control |
| [[2410.03751]] | Recent Advances in Speech Language Models: A Survey | arXiv | 2024 | Surveys neural codecs along the semantic-to-acoustic axis; covers SpeechTokenizer, Mimi, and their roles as mixed-objective tokenizers for SpeechLMs |
| [[2504.08528]] | On The Landscape of Spoken Language Models: A Comprehensive Survey | arXiv | 2025 | Reviews audio codec tokens vs. phonetic tokens; covers VQ-RVQ hierarchy, RVQ layer merging, and BPE-style post-hoc compression as sequence-length reduction strategies |
| [[2505.07916]] | MiniMax-Speech: Intrinsic Zero-Shot TTS with a Learnable Speaker Encoder | arXiv | 2025 | Custom mel-spectrogram VQ tokenizer at 25 tokens/sec with CTC supervision; combined with Flow-VAE decoder for high-quality waveform reconstruction |
