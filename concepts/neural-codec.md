---
slug: neural-codec
title: Neural Audio Codec
aliases: [EnCodec, SoundStream, audio tokenizer, discrete speech representations, RVQ, residual vector quantization, low-frame-rate codec, dynamic codec]
related_concepts: [autoregressive-codec-tts, self-supervised-speech, spoken-language-model, gan-vocoder]
last_updated: 2026-05-29
---

# Neural Audio Codec

## What it is

A neural audio codec is a learned compression system that converts raw audio waveforms into sequences of discrete tokens and reconstructs audio from those tokens. The standard architecture is encoder–quantizer–decoder. The encoder downsamples the waveform to a sequence of latent vectors at a fixed frame rate; a Residual Vector Quantization (RVQ) module converts these to discrete indices across multiple codebook layers (codebook levels); and a convolutional decoder synthesizes the waveform from the quantized representations.

In the context of speech language models and TTS, the first RVQ layer (RVQ-1, "semantic" stream) captures phonetic/semantic content and drives autoregressive language models. The remaining RVQ layers (RVQ-rest, "acoustic" stream) capture fine spectral and prosodic detail and are typically predicted by a faster non-autoregressive model. The frame rate of the codec determines the sequence length fed to language models, directly affecting computational cost (quadratic in attention) and training/inference time.

## Why it matters

Neural audio codecs are foundational to modern autoregressive TTS (VALL-E family), speech language models (Moshi, Mimi), and multimodal audio LLMs. The codec frame rate creates a direct trade-off: standard codecs (EnCodec 75 Hz, DAC 75 Hz) generate sequences 10–15× longer than text for the same content, creating a severe computational burden and modality mismatch with text LLMs. Reducing frame rate while preserving semantic content is the central challenge. Research in [[2412.17048]] identifies that the gap between text and speech LMs is caused partly by sequence length (Factor B) and paralinguistic variability in speech tokens (Factor C), motivating low-frame-rate and semantically-decoupled codecs.

Beyond frame rate, [[2025.acl-long.1498]] identifies a distinct structural problem in existing RVQ codecs: Discrete Representation Inconsistency (DRI). Because the RVQ encoder uses wide convolutional receptive fields to capture context, identical audio segments are tokenized differently depending on surrounding context. This many-to-one mapping from audio to token sequences degrades neural codec language model training, raising WER and reducing speaker similarity. DRI worsens with deeper RVQ layers and can be quantified via consistency accuracy — the fraction of token positions that agree between context-free and context-inclusive encoding of the same segment.

## Current state of the art

As of early 2026, the frontier spans both discrete codec design and the move toward waveform-latent continuous representations:

**Low-frame-rate discrete codecs:** [[2510.00981]] (FlexiCodec, ICLR 2026) achieves 4.15% RVQ-1 WER at 6.25 Hz. Qwen3-TTS [[2601.15621]] introduces the Qwen-TTS-Tokenizer-12Hz — a fully causal 12.5 Hz 16-layer RVQ tokenizer with w2v-BERT 2.0 semantic distillation and GAN-based training — achieving PESQ-WB 3.21, PESQ-NB 3.68, STOI 0.96, UTMOS 4.16, SPK-SIM 0.95 on LibriSpeech test-clean, outperforming Mimi and FireRedTTS2 tokenizer. Fish Audio S2 [[2603.08823]] introduces a streaming 21 Hz DAC-based codec with causal convolutions, SemantiCodec-style first-codebook semantic distillation from w2v-BERT 2.0, and ConvNeXt V2 extensions.

**FSQ (Finite Scalar Quantization):** CosyVoice 2 [[2412.10117]] demonstrated that replacing VQ with FSQ achieves 100% codebook utilization (vs. 23% for VQ) and halves ASR error rate on CommonVoice EN (10.67% vs. 18.26%), establishing FSQ as a preferred alternative to VQ for supervised semantic tokenizers. DisCodec [[2512.13251]] extends FSQ to disentangled factorized speech representation with per-attribute FSQ modules.

**Continuous waveform VAE:** LongCat-AudioDiT [[2603.29339]] demonstrates that a Wav-VAE at 11.72 Hz / 64-dim achieves PESQ 3.237 and STOI 0.967 on LibriTTS test-clean, outperforming most discrete codecs at similar bitrate. M3-TTS [[2512.04720]] uses a Mel-VAE at 43 Hz / 40-dim for a 3x training speedup at modest SIM cost.

**Full-codebook masking for NAR:** OmniVoice [[2604.00688]] demonstrates that jointly masking all codebook layers per position (vs. per-layer masking in SoundStorm/MaskGCT) provides 3x denser training signal and substantially better downstream WER from a frozen Higgs-audio 8-codebook tokenizer.

## Key variants and sub-approaches

**Fixed-rate high-quality codecs (EnCodec, DAC).** Operate at 50–75 Hz, prioritizing reconstruction quality over downstream LM efficiency. DAC uses periodic activations and codebook utilization improvements. EnCodec (75 Hz, 8 RVQ layers) is the original codec used in VALL-E [[2301.02111]] and remains a widely cited baseline.

**SSL-distillation semantic codecs (SpeechTokenizer, Mimi, DualCodec).** Distill self-supervised representations (HuBERT, WavLM, w2v-bert-2) into the RVQ-1 stream to ensure it carries semantic content. Mimi and DualCodec operate at 12.5 Hz by using higher encoder downsampling rates. Adequate for 12.5 Hz but struggle below that.

**ASR-feature-guided dynamic codecs (FlexiCodec [[2510.00981]]).** Use a frozen ASR encoder (SenseVoice-Small) to extract semantic features and guide adaptive frame merging. Adjacent frames with cosine similarity above a threshold are merged, adaptively allocating more frames to phonetically dense regions. Achieves 3–12.5 Hz controllable rate with superior semantic preservation compared to fixed-rate approaches. Key insight: ASR features are more semantically concentrated than SSL features, enabling more effective frame merging.

**Text-assisted codecs (TaDiCodec, TASTE).** Use text transcription to assist ultra-low-rate coding. Achieve ≤ 6.25 Hz but require ground-truth text, limiting applicability in non-TTS settings.

**Single-codebook codecs (WavTokenizer, SemantiCodec, X-Codec2/XCodec2-S).** Use a single FSQ/VQ codebook for simplicity, operating at 40–75 Hz. Llasa+ [[2508.06262]] uses XCodec2-S, a causal streaming adaptation of XCodec2 trained by freezing encoder+VQ and fine-tuning only the causal decoder.

**Continuous tokenizers.** [[2025.findings-naacl.184]] (Cont-SPT) challenges whether discrete quantization is necessary: continuous AR mel prediction (MELLE [[2025.acl-long.65]]) achieves human-parity quality without any codec. DiTAR [[2502.03930]] uses a continuous VAE at 40 Hz/64-dim as the generation target. LongCat-AudioDiT [[2603.29339]] uses a Wav-VAE at 11.72 Hz/64-dim, achieving PESQ 3.237 and STOI 0.967 while outperforming most discrete codecs at similar bitrate.

**Disentangled/factorized codecs (DisCodec, FACodec).** DisCodec [[2512.13251]] uses two-stage FSQ training with graduated soft orthogonality constraints to factorize content, prosody, and timbre while maintaining reconstruction fidelity. The key innovation is summing content+prosody embeddings before re-quantizing into a unified stream for LM training, enabling standard AR LM inference without multi-codebook complexity.

**Streaming causal codecs.** Qwen3-TTS [[2601.15621]] (12Hz, fully causal, 16-layer RVQ), Fish Audio S2 [[2603.08823]] (21 Hz, causal sliding-window transformer, DAC-based), and XCodec2-S [[2508.06262]] (single VQ, partial lookahead) all demonstrate that high-quality streaming codecs are achievable while maintaining semantic preservation and reconstruction quality.

## Comparison to alternatives

Continuous mel-spectrograms remain the dominant representation for non-autoregressive TTS (flow-matching, diffusion); they do not require discrete tokenization. Discrete codecs are specifically required for autoregressive language model training. The FlexiCodec-TTS results ([[2510.00981]]) show that for the AR stage, 6.25 Hz codec tokens provide no degradation vs. 12.5 Hz while providing 1.4–2.1× AR speedup, but the NAR stage still benefits from high frame rate (50 Hz mel > 12.5 Hz codec features for naturalness).

## Year-on-year trajectory

2021–2022: SoundStream, EnCodec established neural audio codecs as the standard compression tool at 50–75 Hz. 2023: SpeechTokenizer introduced semantic distillation for RVQ-1; first purpose-built TTS codecs. 2024: Mimi and DualCodec reached 12.5 Hz while maintaining semantic quality; CosyVoice 2 [[2412.10117]] demonstrated FSQ achieves 100% codebook utilization vs. 23% for VQ — establishing FSQ as the preferred quantization method for supervised semantic tokenizers. 2025: FlexiCodec ([[2510.00981]]) breaks the 12.5 Hz floor to 6.25 Hz; DisCodec [[2512.13251]] introduces two-stage disentangled FSQ for independent prosody/timbre control; XCodec2-S [[2508.06262]] demonstrates streaming single-codebook causal adaptation; MELLE [[2025.acl-long.65]] and DiTAR [[2502.03930]] challenge whether discrete codecs are necessary at all. 2026: Qwen3-TTS [[2601.15621]] demonstrates a 12.5 Hz 16-layer RVQ achieving PESQ-WB 3.21 — outperforming Mimi and FireRedTTS2 tokenizer while enabling fully causal streaming; Fish Audio S2 [[2603.08823]] demonstrates a 21 Hz streaming DAC-based codec with w2v-BERT semantic distillation and ConvNeXt V2 extensions; LongCat-AudioDiT [[2603.29339]] establishes waveform VAE at 11.72 Hz as competitive with discrete codecs for diffusion TTS, revealing that higher VAE fidelity counterintuitively does not improve downstream generation quality. The field is now tracking two competing codec futures: lower-frame-rate discrete (toward text-rate ~4.5 Hz) and continuous waveform latent (bypassing codec discretization entirely).

## Open questions

- Can dynamic frame merging be extended to sub-3 Hz without catastrophic quality loss?
- Is there a natural lower bound on frame rate below which semantic content cannot be preserved in discrete tokens?
- How does FlexiCodec's dynamic merging interact with prosodic information? [[2412.17048]] identifies paralinguistic variability (Factor C) as the main bottleneck — do dynamic tokens help with this?
- Streaming/causal encoding is not addressed by FlexiCodec; is dynamic frame merging compatible with online TTS pipelines?
- Multilingual semantic tokens require language-specific fine-tuning; can the ASR-guided merging generalize cross-lingually?
- DRI ([[2025.acl-long.1498]]) is shown to worsen with deeper RVQ layers; does this suggest the deeper-layer representation design needs fundamentally different treatment (e.g. non-contextual sub-quantizers)?
- [[2025.findings-naacl.184]] shows continuous tokens outperform RVQ on a single English benchmark; will continuous-token AR LMs scale to larger data and multilingual settings, or does the MSE regression objective become unstable?
- VocalNet [[2025.emnlp-main.989]] and other speech LLMs now use CosyVoice2 semantic tokens (25 Hz) rather than traditional RVQ codecs; does this represent a shift toward hybrid supervised-semantic tokenization as the default?
- LongCat-AudioDiT [[2603.29339]] shows higher VAE reconstruction fidelity does not improve TTS quality; what is the optimal VAE bottleneck dimension and frame rate for diffusion-based TTS?
- DisCodec [[2512.13251]] shows that content-prosody FSQ can be fused back into a single stream for LM training; does this generalize to timbre as well, enabling a single-stream codec with full factorization?
- Fish Audio S2 [[2603.08823]] uses a 21 Hz causal codec; Qwen3-TTS [[2601.15621]] uses 12.5 Hz — which frame rate offers the optimal latency/quality trade-off for production streaming?

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
| [[2412.17048]] | Why Do Speech Language Models Fail to Generate Semantically Coherent Outputs? | arXiv | 2026 | Uses HuBERT-Large discrete clusters as codec tokens in a controlled study; identifies paralinguistic variability in speech tokens (Factor C) as the dominant cause of SLM coherence failure — motivating codec designs that suppress paralinguistic variation |
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
