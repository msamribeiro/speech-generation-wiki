---
id: interspeech-2025-0455
title: "APTTS: Adversarial Post-training in Latent Flow Matching for Fast and High-fidelity Text-to-Speech"
authors: [Hyungchan Yoon, Chanwoo Lee, Hoodong Lee, Stanley Jungkyu Choi]
organization: LG AI Research
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: "2025-08-17"
ingested_date: "2026-05-30"
task: [TTS]
architecture: [flow-matching, VAE, hybrid]
conditioning: [zero-shot, speaker-conditioned]
training: [supervised, distillation]
model_size: "not reported"
codec_used: "none"
datasets_train: [LibriTTS (train+dev), VCTK, Hi-Fi TTS, Expresso read (~918 hours, 2507 speakers)]
datasets_eval: [LibriSpeech test-clean, VCTK]
metrics:
  - name: MOS
    value: 3.58
    system: APTTS (4 steps)
    testset: LibriSpeech test-clean (39 samples)
  - name: SMOS
    value: 3.65
    system: APTTS (4 steps)
    testset: LibriSpeech test-clean (39 samples)
  - name: WER
    value: 2.08
    system: APTTS (4 steps)
    testset: LibriSpeech test-clean (39 samples)
  - name: WER
    value: 1.73
    system: APTTS (4 steps)
    testset: LibriSpeech test-clean (full, 4-10s)
  - name: MOS
    value: 3.66
    system: F5-TTS (32 steps)
    testset: LibriSpeech test-clean (39 samples)
  - name: WER
    value: 2.61
    system: F5-TTS (32 steps)
    testset: LibriSpeech test-clean (full)
code_available: null
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/yoon25_interspeech.html"
related_concepts: [flow-matching, zero-shot-tts, gan-vocoder, evaluation-metrics]
related_papers: []
---

# APTTS: Adversarial Post-training in Latent Flow Matching for Fast and High-fidelity Text-to-Speech

**Paper:** [https://www.isca-archive.org/interspeech_2025/yoon25_interspeech.html](https://www.isca-archive.org/interspeech_2025/yoon25_interspeech.html)

**One-sentence contribution:** APTTS introduces adversarial post-training (AP) on a latent flow matching TTS framework that reduces inference to 4 ODE steps while achieving state-of-the-art intelligibility (WER 1.73%) and real-time synthesis (RTF 0.052) on zero-shot TTS, using only ~900 hours of training data.

## Problem

Flow matching TTS models achieve high-quality synthesis but require many ODE solver steps (typically 20–64), making real-time deployment difficult. Existing acceleration methods (consistency distillation, reflow, distribution matching distillation) address mel-spectrogram-domain models. Prior work has not explored latent flow matching for TTS — operating over a learned continuous latent space rather than fixed mel-spectrograms — nor applied adversarial post-training to the intermediate representation domain of TTS. The training-inference mismatch in few-step ODE solvers also degrades quality.

## Method

APTTS is trained in three stages.

**Stage 1 — VAE pre-training:** A VAE based on PeriodVITS encodes mel-spectrograms into a 64-dimensional continuous latent space. Explicit F0 and voicing flags are extracted separately and fed to the VAE decoder to improve pitch reconstruction. A multi-period discriminator (MPD) and multi-resolution discriminator (MRD) with truncated pointwise relativistic loss improve audio fidelity.

**Stage 2 — Flow Matching Decoder + Pitch Predictor:** A Diffusion Transformer (DiT) with 24 layers, 16 attention heads, 1024/2048 dimensions, and RoPE embeddings is trained as the FM decoder. A transformer-based text encoder with SC-CNN style injection and a NeXt-TDNN style encoder extract conditioning from text and speech prompt. Zero-shot TTS is framed as speech infilling: a binary mask covers 70–100% of the latent, and the decoder reconstructs masked frames. Dual classifier-free guidance (CFG) drops context embeddings and prompt independently at 5/10/10% rates during training.

**Stage 3 — Adversarial Post-training (AP):** The trained FM decoder is repurposed as a 4-step generator using a fixed Euler ODE with time steps [0, 0.08, 0.29, 0.62] (emphasizing earlier steps). A joint conditional-unconditional (JCU) discriminator, conditioned on a time-averaged prompt embedding and using dilated convolutions, provides adversarial and feature-matching losses. The AP loss combines L1 reconstruction loss, JCU adversarial loss, and feature matching loss, applied only to masked frames. A hybrid CFG technique addresses the training-inference mismatch: during AP the decoder is always fully conditioned, while at inference the unconditional terms are derived from the pre-AP base model rather than the few-step generator, enabling stable guidance without the error accumulation from a fixed large guidance scale.

The system is trained on ~918 hours from LibriTTS, VCTK, Hi-Fi TTS, and Expresso. Inference uses 22050 Hz audio, 80-band mel-spectrograms.

## Key Results

On 39-sample LibriSpeech test-clean evaluation against public models: APTTS (4 steps) achieves MOS 3.58, SMOS 3.65, WER 2.08% vs. F5-TTS (32 steps) at MOS 3.66, SMOS 3.60, WER 2.52%. RTF is 0.052 vs. F5-TTS's 0.217, a 4× speedup. On the full LibriSpeech test-clean (4–10s), APTTS achieves WER 1.73% and CER 0.40% — the lowest of all baselines including models trained on 60–100K hours (VALL-E at WER 5.9%, Voicebox at WER 1.9%, FlashSpeech at WER 2.7%). SIM-o is 0.55, below Voicebox (0.66) and F5-TTS (0.64), attributed to the 100× smaller training set. AP reduces inference by more than 5× vs. the 32-step base model with minimal quality loss.

Scalability is validated on Matcha-TTS (VCTK, multi-speaker): AP at 2 steps surpasses MAT-10 in both MOS (3.96 vs. 3.71) and WER (4.8% vs. 6.81%), confirming the strategy generalizes to mel-spectrogram-domain FM models.

## Novelty Assessment

The primary architectural novelty is the combination of latent flow matching with adversarial post-training for TTS, specifically the prompt-conditioned JCU discriminator applied in the latent domain and the hybrid CFG technique. The APTTS system achieves competitive zero-shot quality with dramatically less training data (0.9K vs. 60–100K hours for comparisons), which is a practically important result. The AP strategy's generalization to Matcha-TTS is a meaningful scalability demonstration. The distillation-like AP procedure is analogous to adversarial score distillation in image generation, adapted here with novel masking and discriminator design choices for TTS.

## Limitations and Open Questions

SIM-o scores lag behind large-scale trained models, suggesting prompt fidelity would improve with more data. The hybrid CFG requires inference calls to both the post-trained and base FM models, adding complexity. The fixed ODE time steps (empirically chosen) may not be optimal across all input lengths. The system is English-only; multilingual extension is unexplored.

## Wiki Connections

APTTS directly advances [[flow-matching]] by introducing latent-domain FM and adversarial acceleration, and [[zero-shot-tts]] by demonstrating competitive zero-shot quality with only 918 hours. The VAE component connects to [[disentanglement]] (pitch separation). The adversarial training at inference time connects to patterns from [[gan-vocoder]] literature. No in-corpus references identified; primary comparisons are to F5-TTS, Voicebox, VALL-E, FlashSpeech, and Matcha-TTS (all out-of-corpus).
