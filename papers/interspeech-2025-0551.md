---
id: interspeech-2025-0551
title: "Monotonic Attention for Robust Text-to-Speech Synthesis in Large Language Model Frameworks"
authors: [Yike Zhang, Yiming Li, Jie Chen, Qinghua Wu, Songjun Cao, Long Ma]
organization: Tencent
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: "2025-08-17"
ingested_date: "2026-05-30"
task: [TTS]
architecture: [autoregressive-LM]
conditioning: [zero-shot, speaker-conditioned, multilingual]
training: [supervised, fine-tuning]
model_size: "100M params (small), 300M params (large)"
codec_used: "not reported"
datasets_train: [LibriSpeech 960h (English), internal Chinese speech 1000h (small-scale); internal Chinese 100k hours + English 50k hours (large-scale)]
datasets_eval: [LibriSpeech test-clean (Libri-X cross-sentence), SeedTTS (Seed-ZH, Seed-ZH-Hard, Seed-EN), ELLA-V-Hard]
metrics:
  - name: WER
    value: 2.99
    system: Phn-Cosy-L-SMA (large, with SMA)
    testset: Libri-X
  - name: WER
    value: 3.12
    system: Phn-Cosy-L (large, baseline)
    testset: Libri-X
  - name: CER
    value: 8.53
    system: Phn-Cosy-L-SMA
    testset: Seed-ZH-Hard
  - name: CER
    value: 10.42
    system: Phn-Cosy-L (baseline)
    testset: Seed-ZH-Hard
  - name: WER
    value: 12.01
    system: Phn-Cosy-L-SMA
    testset: ELLA-V-Hard
  - name: WER
    value: 14.92
    system: Phn-Cosy-L (baseline)
    testset: ELLA-V-Hard
  - name: SPK-SIM
    value: 0.596
    system: Phn-Cosy-L-SMA
    testset: Libri-X
code_available: null
demo_available: true
url: "https://www.isca-archive.org/interspeech_2025/zhang25c_interspeech.html"
related_concepts: [autoregressive-codec-tts, zero-shot-tts, evaluation-metrics]
related_papers: []
---
> [!abstract] Interspeech · 2025 · Conference
> **Yike Zhang et al.** (Tencent) · [→ Paper](https://www.isca-archive.org/interspeech_2025/zhang25c_interspeech.html) · Demo: ✓ · Code: ?
>
> A stepwise monotonic attention algorithm applied selectively to automatically identified alignment heads in decoder-only LLM-based TTS, reducing word repetitions and omissions without requiring external forced aligners, and scaling to 300M parameters and 150k hours of training data.

## Problem

Decoder-only autoregressive LLMs used for TTS (VALL-E family, CosyVoice, VoiceCraft) lack explicit monotonic alignment constraints between text tokens and acoustic tokens. This causes hard-to-control failure modes: repetition (the same word synthesized multiple times), omission (words skipped), and misalignment (wrong phoneme attended to at synthesis time). Monotonic attention methods exist for encoder-decoder architectures but do not transfer directly to decoder-only models. VALL-E R addresses this with aligned phoneme-acoustic token interleaving, but this requires a pretrained forced aligner — unreliable on diverse, large-scale training corpora. The transducer-based approach (VALL-T) is memory-prohibitive at scale.

## Method

The system extends CosyVoice by replacing the Whisper BPE text tokenizer with a phoneme converter, producing LLM inputs as: [⟨S⟩, speaker embedding v, phonemes p₁…pₙ, ⟨T⟩, acoustic tokens a₁…aₜ, ⟨E⟩].

**Stepwise Monotonic Attention (SMA):** At each acoustic token generation step i, the model computes selection probabilities over phoneme positions using sigmoid-activated energies from the self-attention mechanism. A causal mask ensures each acoustic token can only attend to phonemes at or before the previous phoneme's position. A recursive formula aggregates these probabilities into monotonic attention scores — analogous to monotonic multihead attention in encoder-decoder models (Raffel et al. 2017, Ma et al. 2019) but adapted for decoder-only self-attention. Weight normalization and Gaussian noise injection encourage binary selection probabilities.

**Automatic alignment head selection:** Not all attention heads in an LLM exhibit alignment behavior. A diagonal ratio γ is computed over each head — the fraction of attention mass that falls on a diagonal band expected for monotonic phoneme-to-acoustic alignment. The top 2–3 heads per model (by summed diagonal ratio across training batches) are selected. SMA fine-tuning (with a constant lr=1e-5) is applied only to these heads; other heads use standard attention.

**Parallel batch processing:** Since variable phoneme counts shift the starting acoustic token position in the sequence, a circular shift operation allows Equation 5 (the monotonic recursion) to be applied uniformly across a batch.

**Models:** Phn-Cosy-S: 7 transformer blocks, hidden 2048, 8 heads, ~100M params. Phn-Cosy-L: 14 blocks, hidden 4096, 16 heads, ~300M params. Small-scale: 1K Chinese + 960h English. Large-scale: 100K Chinese + 50K English.

## Key Results

On hard-case sets (where robustness failures concentrate): Phn-Cosy-L-SMA reduces CER on Seed-ZH-Hard from 10.42% to 8.53%, and WER on ELLA-V-Hard from 14.92% to 12.01%, vs. baseline Phn-Cosy-L. On general sets (Libri-X, Seed-ZH, Seed-EN), SMA has minimal negative impact (Libri-X WER: 3.12% → 2.99%). Speaker similarity is largely unchanged across all test sets (0.596 vs. 0.580 on Libri-X). Detailed error analysis (Table 2) shows SMA reduces deletion (1.64%→1.14%) and insertion (1.63%→0.24%) errors on Seed-ZH-Hard, and deletion (4.07%→2.36%) and insertion (2.29%→1.73%) on ELLA-V-Hard. VALL-E R (via Phn-Cosy-S-Ali) significantly raises general test WER (Seed-ZH CER: 4.03%→5.08%, Seed-EN WER: 5.32%→6.40%) indicating poor out-of-domain generalization, while SMA avoids this degradation.

## Novelty Assessment

The method transfers monotonic attention from encoder-decoder models to decoder-only transformers in a technically principled way. The alignment head selection procedure is novel and practically important — applying SMA to non-alignment heads severely degrades accuracy. The focus-vs-completeness trade-off in head selection (diagonal ratio captures both, unlike prior focus-rate metrics) is a concrete algorithmic contribution. The method is alignment-free (no external aligner needed) and memory-efficient (near-zero additional cost). The demonstration at 300M / 150K-hour scale, and on both Chinese and English, is meaningful for production applicability. The limitation is that it only partially solves the robustness problem (hard-case errors are reduced but not eliminated).

## Limitations and Open Questions

The method fine-tunes a pre-trained model; it remains unclear whether SMA should be applied from scratch or only as a fine-tuning step. Hard-case CER/WER remains substantially above general test performance even after SMA (e.g., Seed-ZH-Hard CER 8.53% vs. Seed-ZH CER 2.33%), meaning severe repetition/omission cases are not fully resolved. The selection of alignment heads via the diagonal ratio heuristic is validated empirically but not theoretically. Extension to streaming/online inference (where the monotonic state must be maintained across steps) is not discussed. No subjective (MOS/SMOS) evaluation is reported, making naturalness trade-offs unclear.

## Wiki Connections

This paper directly advances [[autoregressive-codec-tts]] by addressing a critical robustness failure mode of LLM-based systems without sacrificing naturalness or speaker similarity. It connects to [[zero-shot-tts]] via CosyVoice as the base model and multilingual evaluation. The evaluation methodology (WER, CER, SPK-SIM across multiple hard-case test sets) informs [[evaluation-metrics]]. No in-corpus references found; primary baselines are VALL-E, VALL-E R, CosyVoice, VoiceCraft, Seed-TTS (all out-of-corpus).
