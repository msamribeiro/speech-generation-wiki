---
slug: diffusion-tts
title: Diffusion Models for TTS
aliases: [score-based TTS, DDPM TTS, Grad-TTS, DiffSpeech, EDM speech, DiT TTS, score-matching TTS]
related_concepts: [flow-matching, transformer-enc-dec-tts, neural-codec, evaluation-metrics, zero-shot-tts, autoregressive-codec-tts, voice-conversion]
last_updated: 2026-06-13
status: established
---

## Executive Summary

> [!abstract]
> Diffusion models for TTS generate speech by learning to reverse a stochastic noising process, and have converged on the Diffusion Transformer (DiT) backbone with flow-matching objectives as the dominant non-autoregressive paradigm by 2025–2026. They offer competitive synthesis quality, explicit probabilistic control, and native support for speech editing via inpainting, but have historically trailed autoregressive codec LMs on speaker similarity. As of early 2026, waveform-latent flow-matching systems (LongCat-AudioDiT) have substantially closed the speaker similarity gap with AR systems, and the frontier questions have shifted to inference efficiency, training-inference mismatch correction, and optimal VAE latent design.

## Current Status

established — DiT-based flow-matching TTS is a major production paradigm alongside AR codec LMs, with systems at the 0.3B–3.5B scale achieving human-competitive quality on standard benchmarks; the DDPM/score-matching objective has been largely superseded by flow matching within the same DiT architecture; adoption is high and the approach is mature, though active improvement continues on inference speed, speaker similarity, and the continuous-vs-discrete representation question.

## Why This Matters

Diffusion models offer high-quality synthesis with explicit probabilistic control over the generation process. Their non-autoregressive nature (all frames generated jointly) allows faster inference than AR codec LMs for the same output length. The DiT backbone is compute-efficient and avoids the entanglement issues of skip-connection U-Net architectures. Diffusion models also naturally support inpainting (speech editing) via noisy conditioning of unmasked regions, making them useful for content editing beyond TTS.

The key challenge relative to autoregressive systems is speaker similarity: without sequential in-context learning, capturing fine-grained voice characteristics requires either strong conditioning architectures or large-scale training data.

## Core Idea

Diffusion models for TTS are generative models that learn to synthesize speech by reversing a stochastic noising process. They include score-based models (estimating the score of the data distribution), DDPM-style models (predicting the denoising residual), and more recently, flow-matching variants (which share the DiT backbone and iterative refinement but use deterministic ODE integration). In the TTS context, diffusion/flow models typically operate on mel-spectrograms, continuous codec latents, or waveform VAE latents, conditioned on text, speaker embeddings, or other conditioning signals.

The Diffusion Transformer (DiT) architecture — replacing U-Net with a pure transformer backbone and using Adaptive Layer Normalization (AdaLN) for conditioning injection — has become the dominant backbone for both diffusion and flow-matching TTS as of 2024–2025.

## Methods and Variants

**Full mel-spectrogram DDPM/score-based (NaturalSpeech 2/3 family).** Earlier systems ran diffusion over mel-spectrograms with DDPM/score-matching objectives. Quality was competitive but inference steps high (50–100 NFE).

**Continuous VAE latent diffusion (DiTAR [[2502.03930]], LongCat-AudioDiT [[2603.29339]], StyleTTS-ZS [[2025.naacl-long.242]]).** Compress audio to a continuous latent via a VAE, then run diffusion/flow matching in the compact latent space. Reduces sequence length and dimensionality while preserving reconstruction quality. DiTAR uses a VAE at 40 Hz/64-dim; LongCat-AudioDiT uses a Wav-VAE at 11.72 Hz/64-dim; StyleTTS-ZS uses a fixed-length K=50 prosody RVQ latent for the style component only.

**Discrete masked diffusion (DiSTAR [[2510.12210]], OmniVoice [[2604.00688]]).** Apply a masked diffusion (LLaDA-style) objective over discrete codec token sequences rather than continuous latents. DiSTAR applies this within AR-defined patches; OmniVoice applies it across the full sequence with full-codebook random masking and LLM initialization.

**Patch-based AR+diffusion hybrid (DiTAR, DiSTAR).** A causal AR LM handles inter-patch long-range dependencies; a bidirectional diffusion/masked-diffusion model handles intra-patch local generation. This divide-and-conquer design separates temporal structure (AR) from local acoustic detail (diffusion), avoiding the bidirectionality limitations of pure AR and the sequential bottleneck of diffusion-only NAR.

**Diffusion Transformer (DiT) backbone.** The standard backbone for diffusion/flow TTS since 2024: transformer blocks with AdaLN for timestep conditioning, QK-Norm for attention stability, and RoPE positional encoding. Used by F5-TTS ([[2025.acl-long.313]]), DiTAR, LongCat-AudioDiT, DiFlow-TTS [[2509.09631]], Flamed-TTS [[2510.02848]], M3-TTS [[2512.04720]], and others.

## Major Claims

Claims are generalised propositions aggregated from paper evidence. The full claim registry with supporting paper lists is in `wiki/concepts/_evidence/diffusion-tts.yaml`.

### Strongly Supported

- The Diffusion Transformer (DiT) backbone with AdaLN conditioning is more compute-efficient and produces higher quality than U-Net-based diffusion architectures for TTS at equivalent parameter count.
  Supporting: [[2025.acl-long.313]], [[2502.03930]], [[2603.29339]]

- Flow matching has largely superseded DDPM/score-based objectives for TTS within the same DiT architecture, achieving similar or better quality with fewer inference steps and a simpler training objective.
  Supporting: [[2025.acl-long.313]], [[2603.29339]], [[2406.02430]]

### Emerging

- Waveform-latent flow-matching TTS (operating on VAE latents at 11–12 Hz) can match AR codec LM speaker similarity on standard benchmarks, substantially closing the historical SIM gap between NAR diffusion and AR systems.
  Supporting: [[2603.29339]]

- Higher VAE reconstruction fidelity does not necessarily improve downstream diffusion TTS quality, suggesting the optimal VAE bottleneck for generation differs from the optimal bottleneck for reconstruction.
  Supporting: [[2603.29339]], [[2502.03930]]

- Training-inference mismatch in Voicebox/F5-TTS style flow-matching training (caused by the OT plan assuming a specific source distribution that is not matched at inference) is a systematic but correctable degradation.
  Supporting: [[2603.29339]]

- Score-based training objectives for energy-based TTS models can reduce inference to a single MCMC step; the delta loss criterion is formally equivalent to flow matching at t=0, suggesting a deeper structural connection between energy-based and flow-based synthesis.
  Supporting: [[interspeech-2025-1066]]

- Diffusion-based voice conversion with mutual information disentanglement and inference-time expressive guidance can achieve strong emotion controllability in zero-shot settings without speaker-specific training.
  Supporting: [[interspeech-2025-1101]]

### Contested

> [!warning]
> Whether pure NAR diffusion/flow models or AR+diffusion hybrid architectures offer the best quality-efficiency trade-off at scale remains unresolved, with systems on both sides claiming competitive quality.
> Supporting: [[2603.29339]] (pure NAR NAR competitive) / Contradicting: [[2502.03930]], [[2510.12210]] (hybrid AR+diffusion advantageous)

## Relationship to Other Concepts

### Replaces or Supersedes
- [[transformer-enc-dec-tts]] — in zero-shot TTS, DiT-based diffusion/flow models have largely displaced FastSpeech2-style encoder-decoder systems because they do not require explicit duration modeling and naturally handle variable-length output distributions

### Extends or Builds On
- [[neural-codec]] — continuous VAE variants (DiTAR, LongCat-AudioDiT) use codec-like VAE latents as the generation space; discrete masked diffusion variants (DiSTAR, OmniVoice) operate on neural codec token sequences; the two paradigms share the codec as a compression backbone

### Competes With
- [[flow-matching]] — flow matching (a deterministic ODE variant using the same DiT backbone) has largely supplanted stochastic diffusion (DDPM/score-based) for TTS due to fewer inference steps, simpler training objective, and compatibility with CFG. The two are architecturally convergent: modern "diffusion TTS" papers typically use flow matching objectives while retaining the "diffusion" name for the iterative refinement process. Pure AR codec LMs (VALL-E family) achieve stronger speaker similarity via in-context learning but require sequential decoding; diffusion/flow NAR models are faster per utterance but historically trailed in SIM. LongCat-AudioDiT [[2603.29339]] narrows this gap substantially via waveform-latent flow matching.
- [[autoregressive-codec-tts]] — AR codec LMs achieve stronger speaker similarity via sequential in-context learning; diffusion NAR systems are faster per utterance but have historically trailed on SIM; LongCat-AudioDiT [[2603.29339]] narrows this gap substantially

### Commonly Paired With
- [[zero-shot-tts]] — all major diffusion/flow TTS systems in the corpus target zero-shot speaker adaptation; the standard evaluation is Seed-TTS-Eval (zh/en) or LibriSpeech test-clean with SPK-SIM and WER as primary metrics
- [[evaluation-metrics]] — diffusion TTS papers uniformly use MOS/UTMOS, WER, and SPK-SIM as the evaluation triple; the NFE (number of function evaluations) is additionally reported as an efficiency metric specific to iterative models

## Representative Papers

### Foundational
- [[2207.12598]] — Classifier-free guidance: eliminates the external classifier from guided diffusion sampling by jointly training for conditional and unconditional generation; the conditioning mechanism adopted by virtually all subsequent conditional diffusion and flow-matching TTS systems
- [[2406.02430]] — Seed-TTSDiT variant: end-to-end diffusion transformer for zero-shot TTS without per-phoneme duration prediction; established that diffusion models are viable for large-scale zero-shot TTS
- [[2025.acl-long.313]] — F5-TTS: demonstrates DiT backbone superiority over U-Net for flow-matching TTS; establishes DiT+AdaLN as the standard architecture

### Influential
- [[2502.03930]] — DiTAR: patch-based continuous VAE AR+bidirectional-DiT hybrid; 3-43x lower FLOPs than competing NAR diffusion systems; establishes patch-based AR+diffusion as a synthesis paradigm
- [[2510.12210]] — DiSTAR: extends patch-based hybrid to discrete token space with test-time bitrate control via RVQ layer pruning
- [[2025.naacl-long.242]] — StyleTTS-ZS: compact fixed-length latent diffusion with one-step distillation; 10-20x faster than prior diffusion SOTA while maintaining quality; demonstrates distillation-friendly design

### Recent Highlights
- [[2603.29339]] — LongCat-AudioDiT: 3.5B waveform-latent flow-matching achieving SIM 0.818 on Seed-ZH (best diffusion NAR in corpus); identifies training-inference mismatch and introduces Adaptive Projection Guidance
- [[2603.26364]] — LLaDA-TTS: masked discrete diffusion replacing the AR stage in a CosyVoice-style pipeline, enabling parallel generation with step-count-independent cost

### Cautionary / Negative Evidence
- [[interspeech-2025-0063]] — DLPO shows that RLHF for diffusion TTS requires architecture-specific formulations (DDPM loss as reward penalty); standard image-domain RL methods fail for audio diffusion due to temporal incoherence

## Open Questions

- Can discrete masked diffusion (DiSTAR, OmniVoice) be accelerated to the same NFE range as continuous flow matching?
- Does the training-inference mismatch identified by LongCat-AudioDiT [[2603.29339]] affect all Voicebox/F5-TTS style models, and how easily can it be corrected post-hoc in pre-trained models?
- What is the optimal VAE latent dimensionality and frame rate for diffusion TTS? LongCat-AudioDiT shows a counterintuitive trade-off: higher VAE fidelity does not help downstream TTS quality.
- Can DiTAR [[2502.03930]] scaling laws (data and model scale both improve quality monotonically) be replicated for StyleTTS-ZS and other compact latent diffusion approaches?
- [[interspeech-2025-1066]] shows that delta loss EBMs peak at 1 inference step and degrade at 100 steps, while automatic MOS predictors (UTMOSv2) and human MOS diverge for these systems — does this UTMOS divergence extend to other non-standard synthesis paradigms?
- Can diffusion-based EVC with zero-shot generalisation ([[interspeech-2025-1101]]) preserve naturalness comparable to seen-speaker diffusion methods, or is the naturalness gap at the zero-shot boundary fundamental?

## Trend Summary

2022–2023: NaturalSpeech 2/3 established diffusion TTS with mel-spectrogram DDPM/score-based objectives; introduced factorized codecs and DiT backbones. 2024: Flow matching superseded DDPM as the training objective for most new systems; E2-TTS, F5-TTS, Voicebox established flow matching as competitive with AR on zero-shot TTS. 2025: DiTAR [[2502.03930]] proposed patch-based continuous AR+diffusion as a new synthesis paradigm; StyleTTS-ZS [[2025.naacl-long.242]] demonstrated compact latent diffusion with one-step distillation; Seed-TTSDiT [[2406.02430]] demonstrated end-to-end diffusion for zero-shot TTS at scale; DiSTAR [[2510.12210]] extended patch-based hybrid to discrete token space; LongCat-AudioDiT [[2603.29339]] shifted the representation from mel-spectrogram to waveform latent, achieving SOTA SIM on Seed-ZH and identifying a training-inference mismatch previously overlooked. 2026: The SIM gap between diffusion NAR and AR systems has largely closed; the frontier questions are now about inference efficiency (distillation, fewer NFE) and training data scale.

## All Papers

| ID | Title | Venue | Year | Key use of this concept |
|----|-------|-------|------|------------------------|
| [[2207.12598]] | Classifier-Free Diffusion Guidance | arXiv | 2022 | Introduces CFG — the standard quality-diversity guidance mechanism adopted by all subsequent conditional diffusion and flow-matching TTS systems; eliminates the external classifier requirement |
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
| [[interspeech-2025-0063]] | Fine-Tuning TTS Diffusion Models Using Reinforcement Learning with Human Feedback | Interspeech | 2025 | DLPO: first RLHF framework specifically designed for non-autoregressive diffusion TTS; original DDPM noise-prediction loss used as reward penalty to maintain temporal coherence; UTMOS 3.65 vs. 2.90 base, preferred 67% of pairwise comparisons |
| [[2508.01796]] | Enhancing Spectrogram Realism in Singing Voice Synthesis via Explicit Bandwidth Extension | arXiv | 2025 | DiT-based diffusion model for linear spectrogram estimation (LSE) as an intermediate step between acoustic model and vocoder; time-axis-only self-attention with per-frequency embeddings; improves singing synthesis MOS from 3.576 to 4.176 |
| [[interspeech-2025-1066]] | Score-Based Training for Energy-Based TTS Models | Interspeech | 2025 | Introduces delta loss for EBMs — a score-based criterion equivalent to flow matching at t=0 — enabling single-step EBM inference; first comparison of SSM and NCE training for TTS EBMs |
| [[interspeech-2025-1101]] | ZSDEVC: Zero-Shot Diffusion-based Emotional Voice Conversion with Disentangled Mechanism | Interspeech | 2025 | Diffusion-based zero-shot EVC with vCLUB MI disentanglement and expressive guidance; demonstrates zero-shot emotional VC with substantially higher ECA (0.53) than seen-speaker diffusion baselines (0.256) |
| [[2508.19205]] | VibeVoice Technical Report | arXiv | 2025 | Next-token diffusion head conditioned on LLM hidden states for continuous VAE feature generation; 10-step DPM-Solver++ at 7.5 Hz; outperforms discrete AR systems on long-form subjective quality |
| [[2304.09116]] | NaturalSpeech 2: Latent Diffusion Models are Natural and Zero-Shot Speech and Singing Synthesizers | arXiv | 2023 | Introduces latent diffusion over codec continuous vectors for zero-shot TTS; avoids AR repetition/skipping errors; first diffusion TTS also demonstrated for singing synthesis; CMOS competitive with ground truth |
| [[2305.07243]] | Better speech synthesis through scaling (Tortoise TTS) | arXiv | 2023 | AR+diffusion cascade TTS; diffusion decoder conditioned on continuous AR latents rather than discrete tokens substantially improves output quality |
| [[2106.15561]] | A Survey on Neural Speech Synthesis | arXiv | 2021 | Historical survey covering diffusion-based TTS and vocoders; documents the transition from iterative diffusion toward faster sampling methods |
