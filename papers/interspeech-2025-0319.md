---
id: interspeech-2025-0319
title: "Improving Noise Robustness of LLM-based Zero-shot TTS via Discrete Acoustic Token Denoising"
authors: ["Ye-Xin Lu", "Hui-Peng Du", "Fei Liu", "Yang Ai", "Zhen-Hua Ling"]
organization: University of Science and Technology of China
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: "2025-08-17"
ingested_date: "2026-05-30"
task: [TTS]
architecture: [autoregressive-LM]
conditioning: [zero-shot, speaker-conditioned, prompt-conditioned]
training: [supervised, fine-tuning]
model_size: "LauraTTS (pre-trained on LibriLight 60k h); codec denoiser: 12 Conformer blocks (token denoiser) + 6 Conformer blocks (embedding refiner)"
codec_used: "Encodec via FunCodeec (16 kHz, M=640, K=32 VQs, codebook C=1024, embedding D=128)"
datasets_train: ["LibriLight 60k h (LauraTTS pre-training)", "LibriTTS-R 580h + DNS Challenge 2022 noise (codec denoiser training, SNR -5 to 15 dB)"]
datasets_eval: ["LibriTTS-R test-clean + WHAM! noise (SNR 0–20 dB, utterances 3–10 s)"]
metrics:
  - name: MOS
    value: 4.02
    system: NR-LauraTTS (proposed, noisy prompt)
    testset: LibriTTS-R test-clean + WHAM! noise
  - name: MOS
    value: 3.46
    system: LauraTTS (noisy prompt, no denoising)
    testset: LibriTTS-R test-clean + WHAM! noise
  - name: MOS
    value: 4.03
    system: LauraTTS (clean prompt)
    testset: LibriTTS-R test-clean + WHAM! noise
  - name: SMOS
    value: 3.98
    system: NR-LauraTTS (proposed, noisy prompt)
    testset: LibriTTS-R test-clean + WHAM! noise
  - name: SMOS
    value: 2.80
    system: LauraTTS (noisy prompt, no denoising)
    testset: LibriTTS-R test-clean + WHAM! noise
  - name: SPK-SIM
    value: 0.827
    system: NR-LauraTTS (proposed)
    testset: LibriTTS-R test-clean + WHAM! noise
  - name: SPK-SIM
    value: 0.642
    system: LauraTTS (noisy prompt)
    testset: LibriTTS-R test-clean + WHAM! noise
  - name: WER
    value: 2.44
    system: NR-LauraTTS (proposed)
    testset: LibriTTS-R test-clean + WHAM! noise
  - name: WER
    value: 21.37
    system: LauraTTS (noisy prompt)
    testset: LibriTTS-R test-clean + WHAM! noise
code_available: false
demo_available: false
url: "https://www.isca-archive.org/interspeech_2025/lu25b_interspeech.html"
related_concepts: [zero-shot-tts, neural-codec, autoregressive-codec-tts, self-supervised-speech]
related_papers: []
---
> [!abstract] Interspeech · 2025 · Conference
> **Ye-Xin Lu et al.** (University of Science and Technology of China) · [→ Paper](https://www.isca-archive.org/interspeech_2025/lu25b_interspeech.html) · Demo: ✗ · Code: ✗
>
> NR-LauraTTS integrates a lightweight codec-domain speech denoiser — operating on discrete acoustic tokens rather than waveforms — into an LLM-based zero-shot TTS system, recovering near-clean-prompt synthesis quality from noisy audio prompts at only 1.10G additional FLOPs vs. 38.93G for a comparable waveform-domain SE model.

## Problem

LLM-based zero-shot TTS systems (VALL-E, NaturalSpeech 3, LauraTTS) use discrete acoustic tokens as audio prompt representations and exploit in-context learning to preserve the speaker's acoustic environment. This strength becomes a weakness when the audio prompt contains environmental noise: the model faithfully reproduces the noise in the synthesized output. Applying waveform-domain speech enhancement (SE) models before the TTS front-end partially mitigates this, but state-of-the-art SE models (CMGAN, MP-SENet) introduce signal artifacts that harm speaker identity information in the enhanced prompt, resulting in lower speaker similarity. Additionally, waveform-domain SE at 38.93G FLOPs is computationally expensive.

## Method

**LauraTTS backbone:** An LLM-based zero-shot TTS model using Encodec (FunCodeec implementation: 16 kHz, 32 VQs, codebook 1024, downsampling 640). LauraTTS predicts only the first two RVQ groups of the target tokens autoregressively, then reconstructs the waveform via a one-step codec vocoder conditioned on the text prompt.

**Codec Denoiser (the novel component):** A two-stage model operating entirely in the discrete acoustic token domain:

1. **Token Denoiser:** 12 Conformer blocks. Takes all K=32 groups of noisy acoustic tokens, sums their codebook embeddings, and predicts the first two groups of clean acoustic tokens via cross-entropy loss. Prediction of only the first two groups is motivated by LauraTTS's architecture (first two groups encode essential speaker properties) and by the empirical finding that predicting more groups increases complexity without improving SE metrics.

2. **Embedding Refiner:** 6 Conformer blocks. Takes the predicted first two clean token embeddings concatenated with the full noisy embedding sum, and predicts the summed acoustic embedding across all 32 VQ groups. This is supervised by an L1 + Frobenius norm loss against the clean summed embedding.

Training: teacher-forcing with random substitution of predicted tokens by clean ground-truth tokens. The codec denoiser is trained on LibriTTS-R + DNS Challenge 2022 noise (SNR −5 to +15 dB), independently of LauraTTS. At inference, the token denoiser output is fed directly as the acoustic prompt to LauraTTS.

## Key Results

**Speech enhancement task (DNSMOS P.835, 1s signals):**
| Method | SIG | BAK | OVRL | FLOPs |
|--------|-----|-----|------|-------|
| CMGAN | 3.54 | 4.05 | 3.26 | 31.68G |
| MP-SENet | 3.56 | 4.09 | 3.30 | 38.93G |
| **Codec Denoiser** | **3.62** | **4.11** | **3.36** | **9.96G** |

The codec denoiser outperforms both waveform-domain baselines in all DNSMOS dimensions at 3x lower compute.

**Noise-robust zero-shot TTS task (subjective + objective):**
| System | Prompt | MOS | SMOS | SECS | WER | +FLOPs |
|--------|--------|-----|------|------|-----|--------|
| LauraTTS | Clean | 4.03 | 3.99 | 0.827 | 2.33% | — |
| LauraTTS | Noisy | 3.46 | 2.80 | 0.642 | 21.37% | — |
| LauraTTS + MP-SENet | Noisy | 4.01 | 3.85 | 0.811 | 2.54% | 38.93G |
| **NR-LauraTTS** | **Noisy** | **4.02** | **3.98** | **0.827** | **2.44%** | **1.10G** |

NR-LauraTTS achieves SECS of 0.827 — matching LauraTTS with clean prompts exactly — while LauraTTS+MP-SENet reaches only 0.811 due to SE artifacts corrupting speaker identity. The +1.10G overhead is negligible compared to 38.93G for waveform-domain SE.

## Novelty Assessment

Operating speech enhancement in the discrete acoustic token domain is conceptually clean and practically effective. The codec bottleneck (VQ discretization) acts as an information filter that removes noise during quantization, giving the denoiser a structurally advantageous starting point. The finding that predicting only the first two RVQ groups is sufficient (and that predicting more degrades results) is well-motivated by the hierarchical information structure of RVQ codecs. The compute comparison (1.10G vs. 38.93G) is a strong practical argument. The approach is architecture-specific to LauraTTS/Encodec; generalization to other LLM-TTS systems with different codec architectures is not demonstrated but seems feasible.

## Limitations and Open Questions

- Specific to LauraTTS + Encodec (FunCodeec); adaptation to other codecs (DAC, SoundStream, EnCodec original) requires retraining.
- Training uses synthetic noise (DNS Challenge 2022 + WHAM!); real-world noise types (music, babble, room impulse responses) may not be fully covered.
- No ablation on LauraTTS model size or the effect of codec VQ count K on denoiser difficulty.
- Intelligibility (WER 2.44%) is slightly worse than clean-prompt LauraTTS (2.33%), though close.
- No evaluation of the codec denoiser on other TTS or SE tasks beyond the paired experiment.

## Wiki Connections

This paper advances [[zero-shot-tts]] by addressing the noise robustness gap of LLM-based systems, complementing prior work on noise-robust zero-shot TTS via speaker encoders. The codec denoiser design is closely related to [[neural-codec]] research on the hierarchical information structure of RVQ tokens. The LauraTTS backbone connects to [[autoregressive-codec-tts]] systems (VALL-E family). The token-domain SE finding — that the VQ bottleneck filters noise — is an underexplored property of neural codecs relevant to [[disentanglement]] research on codec representations.
