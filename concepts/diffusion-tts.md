---
slug: diffusion-tts
title: Diffusion Models for TTS
aliases: [score-based TTS, DDPM TTS, Grad-TTS, DiffSpeech, EDM speech, DiT TTS, score-matching TTS]
related_concepts: [flow-matching, transformer-enc-dec-tts, neural-codec, evaluation-metrics, zero-shot-tts, autoregressive-codec-tts]
last_updated: 2026-05-29
---

# Diffusion Models for TTS

## What it is

Diffusion models for TTS are generative models that learn to synthesize speech by reversing a stochastic noising process. They include score-based models (estimating the score of the data distribution), DDPM-style models (predicting the denoising residual), and more recently, flow-matching variants (which share the DiT backbone and iterative refinement but use deterministic ODE integration). In the TTS context, diffusion/flow models typically operate on mel-spectrograms, continuous codec latents, or waveform VAE latents, conditioned on text, speaker embeddings, or other conditioning signals.

The Diffusion Transformer (DiT) architecture — replacing U-Net with a pure transformer backbone and using Adaptive Layer Normalization (AdaLN) for conditioning injection — has become the dominant backbone for both diffusion and flow-matching TTS as of 2024–2025.

## Why it matters

Diffusion models offer high-quality synthesis with explicit probabilistic control over the generation process. Their non-autoregressive nature (all frames generated jointly) allows faster inference than AR codec LMs for the same output length. The DiT backbone is compute-efficient and avoids the entanglement issues of skip-connection U-Net architectures. Diffusion models also naturally support inpainting (speech editing) via noisy conditioning of unmasked regions, making them useful for content editing beyond TTS.

The key challenge relative to autoregressive systems is speaker similarity: without sequential in-context learning, capturing fine-grained voice characteristics requires either strong conditioning architectures or large-scale training data.

## Current state of the art

As of early 2026, the leading diffusion/flow-matching non-autoregressive TTS systems:
- LongCat-AudioDiT [[2603.29339]]: 3.5B waveform-latent flow-matching model trained on 1M hours; SIM 0.818 on Seed-ZH (best diffusion NAR in corpus), WER 1.50% on Seed-EN. Key innovations: Wav-VAE latent space (vs. mel-spectrogram), training-inference mismatch correction, and Adaptive Projection Guidance (APG).
- DiTAR [[2502.03930]]: patch-based continuous AR+bidirectional-diffusion hybrid at 0.6B–1B; WER 1.78% on LibriSpeech test-clean, 3-43x lower FLOPs than competing NAR diffusion systems.
- DiSTAR [[2510.12210]]: AR+discrete masked diffusion; WER 1.32% on SeedTTS test-en at 0.3B; test-time bitrate control via RVQ layer pruning.
- StyleTTS-ZS [[2025.naacl-long.242]]: compact latent diffusion over a fixed-length RVQ prosody space; CMOS -0.032 vs. ground truth on LibriSpeech, 10-20x faster inference than prior diffusion SOTA, codec-free.
- Seed-TTS (Seed-TTSDiT variant) [[2406.02430]]: end-to-end diffusion transformer for zero-shot TTS without per-phoneme duration prediction; SIM 0.790 on Seed-TTS test-en (best among all Seed-TTS variants).

## Key variants and sub-approaches

**Full mel-spectrogram DDPM/score-based (NaturalSpeech 2/3 family).** Earlier systems ran diffusion over mel-spectrograms with DDPM/score-matching objectives. Quality was competitive but inference steps high (50–100 NFE).

**Continuous VAE latent diffusion (DiTAR [[2502.03930]], LongCat-AudioDiT [[2603.29339]], StyleTTS-ZS [[2025.naacl-long.242]]).** Compress audio to a continuous latent via a VAE, then run diffusion/flow matching in the compact latent space. Reduces sequence length and dimensionality while preserving reconstruction quality. DiTAR uses a VAE at 40 Hz/64-dim; LongCat-AudioDiT uses a Wav-VAE at 11.72 Hz/64-dim; StyleTTS-ZS uses a fixed-length K=50 prosody RVQ latent for the style component only.

**Discrete masked diffusion (DiSTAR [[2510.12210]], OmniVoice [[2604.00688]]).** Apply a masked diffusion (LLaDA-style) objective over discrete codec token sequences rather than continuous latents. DiSTAR applies this within AR-defined patches; OmniVoice applies it across the full sequence with full-codebook random masking and LLM initialization.

**Patch-based AR+diffusion hybrid (DiTAR, DiSTAR).** A causal AR LM handles inter-patch long-range dependencies; a bidirectional diffusion/masked-diffusion model handles intra-patch local generation. This divide-and-conquer design separates temporal structure (AR) from local acoustic detail (diffusion), avoiding the bidirectionality limitations of pure AR and the sequential bottleneck of diffusion-only NAR.

**Diffusion Transformer (DiT) backbone.** The standard backbone for diffusion/flow TTS since 2024: transformer blocks with AdaLN for timestep conditioning, QK-Norm for attention stability, and RoPE positional encoding. Used by F5-TTS ([[2025.acl-long.313]]), DiTAR, LongCat-AudioDiT, DiFlow-TTS [[2509.09631]], Flamed-TTS [[2510.02848]], M3-TTS [[2512.04720]], and others.

## Comparison to alternatives

Flow matching (a deterministic ODE variant using the same DiT backbone) has largely supplanted stochastic diffusion (DDPM/score-based) for TTS due to fewer inference steps, simpler training objective, and compatibility with CFG. The two are architecturally convergent: modern "diffusion TTS" papers typically use flow matching objectives while retaining the "diffusion" name for the iterative refinement process. Pure AR codec LMs (VALL-E family) achieve stronger speaker similarity via in-context learning but require sequential decoding; diffusion/flow NAR models are faster per utterance but historically trailed in SIM. LongCat-AudioDiT [[2603.29339]] narrows this gap substantially via waveform-latent flow matching.

## Year-on-year trajectory

2022–2023: NaturalSpeech 2/3 established diffusion TTS with mel-spectrogram DDPM/score-based objectives; introduced factorized codecs and DiT backbones. 2024: Flow matching superseded DDPM as the training objective for most new systems; E2-TTS, F5-TTS, Voicebox established flow matching as competitive with AR on zero-shot TTS. 2025: DiTAR [[2502.03930]] proposed patch-based continuous AR+diffusion as a new synthesis paradigm; StyleTTS-ZS [[2025.naacl-long.242]] demonstrated compact latent diffusion with one-step distillation; Seed-TTSDiT [[2406.02430]] demonstrated end-to-end diffusion for zero-shot TTS at scale; DiSTAR [[2510.12210]] extended patch-based hybrid to discrete token space; LongCat-AudioDiT [[2603.29339]] shifted the representation from mel-spectrogram to waveform latent, achieving SOTA SIM on Seed-ZH and identifying a training-inference mismatch previously overlooked. 2026: The SIM gap between diffusion NAR and AR systems has largely closed; the frontier questions are now about inference efficiency (distillation, fewer NFE) and training data scale.

## Open questions

- Can discrete masked diffusion (DiSTAR, OmniVoice) be accelerated to the same NFE range as continuous flow matching?
- Does the training-inference mismatch identified by LongCat-AudioDiT [[2603.29339]] affect all Voicebox/F5-TTS style models, and how easily can it be corrected post-hoc in pre-trained models?
- What is the optimal VAE latent dimensionality and frame rate for diffusion TTS? LongCat-AudioDiT shows a counterintuitive trade-off: higher VAE fidelity does not help downstream TTS quality.
- Can DiTAR [[2502.03930]] scaling laws (data and model scale both improve quality monotonically) be replicated for StyleTTS-ZS and other compact latent diffusion approaches?

## Papers

| ID | Title | Venue | Year | Key use of this concept |
|----|-------|-------|------|------------------------|
| [[2025.acl-long.313]] | F5-TTS: A Fairytaler that Fakes Fluent and Faithful Speech with Flow Matching | ACL | 2025 | Uses Diffusion Transformer (DiT) backbone with adaLN-zero for flow matching TTS; shows that DiT is more compute-efficient than flat U-Net Transformer (lower GFLOPs) and avoids the entanglement issues of skip-connection architectures |
| [[2025.naacl-long.242]] | StyleTTS-ZS: Efficient High-Quality Zero-Shot TTS with Distilled Time-Varying Style Diffusion | NAACL | 2025 | Fixed-length RVQ prosody latent diffusion model; one-step distillation from 10k samples; CMOS -0.032 vs. ground truth at 10-20x faster inference than prior diffusion SOTA; codec-free design |
| [[2406.02430]] | Seed-TTS: A Family of High-Quality Versatile Speech Generation Models | arXiv | 2024 | Seed-TTSDiT variant: end-to-end diffusion transformer for zero-shot TTS without per-phoneme duration prediction; highest SIM (0.790) among Seed-TTS variants; supports speech editing via masked infilling |
| [[2510.12210]] | DiSTAR: Diffusion over a Scalable Token Autoregressive Representation | arXiv | 2025 | Discrete masked diffusion (LLaDA-style) applied within AR-drafted patches in RVQ code space; test-time bitrate control via RVQ layer pruning; WER 1.32% on SeedTTS test-en at 0.3B |
| [[2502.03930]] | DiTAR: Diffusion Transformer Autoregressive Modeling for Speech Generation | arXiv | 2025 | Patch-based continuous VAE AR+bidirectional-DiT hybrid; 3-43x lower FLOPs than competing NAR diffusion systems; WER 1.685% on Seed-EN at 1B; scales monotonically with data and model size |
| [[2603.29339]] | LongCat-AudioDiT: High-Fidelity Diffusion TTS in the Waveform Latent Space | arXiv | 2026 | Waveform-latent flow-matching NAR (Wav-VAE 11.72 Hz/64-dim); SIM 0.818 on Seed-ZH (best diffusion NAR in corpus); identifies and corrects training-inference mismatch in Voicebox/F5-TTS style training; introduces APG to replace standard CFG |
| [[2512.14291]] | GLM-TTS Technical Report | arXiv | 2025 | Two-stage architecture with autoregressive LM followed by diffusion token-to-waveform model; demonstrates 4-reward GRPO RL alignment for diffusion-based TTS pipeline |
| [[2025.emnlp-main.40]] | Towards Controllable Speech Synthesis in the Era of LLMs: A Systematic Survey | EMNLP | 2025 | Surveys diffusion-based controllable TTS methods (NaturalSpeech 2/3, PromptTTS 2) alongside AR and flow-matching systems; provides historical taxonomy from DDPM to flow matching |
| [[2025.coling-main.352]] | DiffStyleTTS: Diffusion-based Hierarchical Prosody Modeling for Text-to-Speech | workshop | 2025 | Replaces variance adaptor in FastSpeech2 with a conditional diffusion module modeling coarse-grained implicit style and fine-grained explicit attributes; MOS 4.18 on Mandarin outperforming Grad-TTS, Guided-TTS, and DiffProsody |
| [[2603.26364]] | LLaDA-TTS: Unifying Speech Synthesis and Zero-Shot Editing via Masked Discrete Diffusion | arXiv | 2026 | Replaces the AR LM stage of a CosyVoice-style TTS pipeline with a masked discrete diffusion model (LLaDA-based); achieves CER 0.98% (zh) on Seed-TTS-Eval while enabling parallel generation with step-count-independent cost |
