---
id: "iclr-2025-hQvX9MBowC"
title: "DiTTo-TTS: Diffusion Transformers for Scalable Text-to-Speech without Domain-Specific Factors"
authors: ["Keon Lee", "Dong Won Kim", "Jaehyeon Kim", "Seungjun Chung", "Jaewoong Cho"]
organization: "KRAFTON"
venue: "ICLR"
venue_type: "conference"
year: 2025
month: 4
published_date: "2025-04-24"
ingested_date: "2026-06-30"
task: ["TTS"]
architecture: ["diffusion", "transformer-enc-dec"]
conditioning: ["text-conditioned", "speaker-conditioned", "zero-shot", "multilingual", "prompt-conditioned"]
training: ["supervised"]
model_size: "740M params (DiTTo-en-XL), 790M params (DiTTo-multi)"
codec_used: "Mel-VAE (10.76 Hz, fine-tuned for text-speech semantic alignment)"
datasets_train: ["MLS (55K hours, English)", "82K-hour 9-language mix (English, Korean, German, Dutch, French, Spanish, Italian, Portuguese, Polish)", "LibriSpeech train-clean-100/360, train-other-500"]
datasets_eval: ["LibriSpeech test-clean", "multilingual test sets (100 examples per language)", "RAVDESS (out-of-domain prompts)"]
metrics:
  - name: WER
    value: 1.78
    system: DiTTo-en-XL
    testset: LibriSpeech test-clean (continuation)
  - name: WER
    value: 2.56
    system: DiTTo-en-XL
    testset: LibriSpeech test-clean (cross-sentence)
  - name: SIM-r
    value: 0.6554
    system: DiTTo-en-XL
    testset: LibriSpeech test-clean (cross-sentence)
  - name: SMOS
    value: 3.91
    system: DiTTo-en-XL
    testset: LibriSpeech test-clean (cross-sentence, human eval, n=40)
  - name: CMOS
    value: 0.0
    system: DiTTo-en-XL
    testset: LibriSpeech test-clean (cross-sentence, human eval, reference point)
code_available: false
demo_available: true
url: "https://openreview.net/forum?id=hQvX9MBowC"
related_concepts: ["diffusion-tts", "zero-shot-tts", "neural-codec", "multilingual-tts", "transformer-enc-dec-tts"]
related_papers: ["2301.02111", "2403.03100", "2304.09116", "2206.04658", "2406.02430"]
field_significance:
  level: "high"
  type: ["architectural-novelty", "empirical-benchmark"]
generation:
  date: "2026-06-30"
  agent: "speech-generation-ingest-agent"
  model: "claude-sonnet-4-6"
  commit: "f406e94"
---

> [!abstract] ICLR · 2025 · Conference
> **Lee et al.** (KRAFTON) · [→ Paper](https://openreview.net/forum?id=hQvX9MBowC) · Demo: ✓ · Code: ✗
>
> DiTTo-TTS shows that a Diffusion Transformer operating on text and speech latents through cross-attention alone, without phonemes or phoneme-level durations, can match or exceed state-of-the-art autoregressive and diffusion TTS baselines at scale.

## Problem

Latent diffusion models (LDMs) achieve strong results in image, audio, and video generation by operating directly on raw conditioning signals, but TTS applications of LDMs have continued to rely on domain-specific factors: phonemes and phoneme-level durations supplied by external aligners. This dependency complicates data preparation, restricts which datasets can be used for training, and limits scalability relative to other generative domains where LDMs need no such auxiliary annotation. Prior attempts to remove these factors (E3 TTS, Simple-TTS) demonstrated the idea was feasible but reported clearly suboptimal performance relative to phoneme-duration-based systems, leaving open the question of whether the gap was inherent or an artifact of suboptimal design choices.

## Method

DiTTo-TTS conditions a Diffusion Transformer (DiT) on text purely through cross-attention, with no explicit phoneme or duration input. Speech is represented in the latent space of Mel-VAE, a neural audio codec compressed to a 10.76 Hz frame rate (roughly 7-8x more compressed than EnCodec). The codec is fine-tuned with an auxiliary causal language-modeling loss that forces its latent space toward semantic alignment with a pretrained text encoder's embedding space, on the premise that closer text-speech latent alignment improves cross-attention-based conditioning. The diffusion backbone is trained with a v-prediction objective and random span masking (following Voicebox-style infilling) to support zero-shot audio prompting.

Three design choices are identified as critical through ablation. First, DiT with minimal modifications (long skip connections, global adaptive layer normalization, gated-GELU feedforward, rotary position embeddings) outperforms the U-Net backbones used by prior phoneme-free LDM-TTS systems, particularly once Mel-VAE's higher compression ratio is in play. Second, because no phoneme-level durations are available, total utterance length must be predicted separately: a Speech Length Predictor (an encoder-decoder transformer trained with cross-entropy loss, separate from the diffusion model) estimates total token count from text and an optional speech prompt at inference time, replacing the padding-based fixed-length schemes used in E3 TTS and Simple-TTS. Third, the degree of semantic alignment between text and speech embeddings (achieved by using a jointly-trained text-speech encoder, a semantically fine-tuned codec, or both) correlates directly with downstream TTS quality, independent of model or data scale.

![An overview of DiTTo-TTS. (middle) The LDM backbone is trained to denoise a span-masked noisy segment given its contextual counterpart, without utilizing phoneme and phoneme-level duration. (left) The inner structure of the DiT block incorporates multi-head cross-attention with global AdaLN. (right) The speech length predictor is based on causal transformer blocks. Both DiT blocks and the speech length predictor employ cross-attention to condition on text representation.](assets/iclr-2025-hQvX9MBowC/figure-1.png)

At inference, the Speech Length Predictor first determines target length from text and an optional speech prompt; the DiT backbone then generates the latent of that length conditioned on text and prompt; the result is decoded to mel-spectrograms via the Mel-VAE decoder and converted to waveform with a pretrained BigVGAN vocoder. Four sizes are trained (S: 42M, B: 152M, L: 508M, XL: 740M parameters), plus a 790M-parameter multilingual variant (DiTTo-multi) trained on 82K hours across nine languages.

## Key Results

On LibriSpeech test-clean continuation and cross-sentence tasks, DiTTo-en-B (152M params) outperforms CLaM-TTS, a state-of-the-art autoregressive codec-LM baseline, on WER, CER, and speaker similarity while running 4.6x faster at 3.84x smaller model size (Table 1). The largest variant, DiTTo-en-XL (740M params), reaches WER 1.78 / SIM-r 0.6075 on continuation and WER 2.56 / SIM-r 0.6554 on cross-sentence, both competitive with or ahead of Voicebox, VALL-E, and CLaM-TTS, none of which require phoneme/duration factors at this performance level (Tables 1-2). Human evaluation on 40 LibriSpeech cross-sentence samples gives DiTTo-en-XL an SMOS of 3.91 ± 0.16 and CMOS of 0.0 relative to itself, against Ground Truth at SMOS 4.08 ± 0.14, closing most of the gap to ground-truth speaker similarity and naturalness, and clearly ahead of Simple-TTS (SMOS 2.15) and CLaM-en (SMOS 3.42) (Table 3).

Ablations isolate the source of these gains. Replacing DiT with a U-Net backbone under identical conditions raises WER from 2.93 to 3.7 and drops SIM-r from 0.588 to 0.389 (Table 4), evidencing that backbone choice, not codec or data, is the binding constraint once domain-specific factors are removed. Variable-length modeling via the Speech Length Predictor reduces WER from 6.81-8.89 (fixed-length variants) to 5.36-5.58 (Table 5). Mel-VAE outperforms EnCodec and DAC as the diffusion target despite EnCodec having a smaller latent dimension and DAC scoring higher on codec-only PESQ/ViSQOL, because Mel-VAE's shorter latent sequence length dominates training and inference efficiency (Table 7). On the multilingual side, DiTTo-multi shows performance comparable to or better than CLaM-TTS across the nine trained languages (Appendix).

## Novelty Assessment

The contribution is a focused empirical and architectural one: the paper does not invent diffusion transformers, cross-attention text conditioning, or variable-length generation as primitives, but it is the first to combine them carefully enough to close the performance gap that previously separated phoneme-free LDM-TTS (E3 TTS, Simple-TTS) from phoneme-and-duration-based state of the art. The three identified levers (DiT over U-Net, learned variable length over fixed-length padding, and explicit semantic alignment between text and speech latents) are individually plausible engineering choices, but the ablations (Tables 4-7) make a credible case that all three are necessary rather than incidental, which is the paper's main evidentiary contribution. The semantic-alignment finding is the most transferable insight: a codec fine-tuned with an auxiliary language-modeling loss against a frozen text encoder improves downstream generation quality regardless of which text encoder or codec pairing is used, suggesting the property is generic to cross-attention-conditioned latent generation rather than specific to this implementation. The comparisons to closed baselines (Voicebox, VALL-E, NaturalSpeech 2/3) rely on numbers reported in their original papers rather than reproduced runs, which the paper discloses but which limits independent verification of the head-to-head claims.

## Field Significance

> [!tip]
> High — DiTTo-TTS provides the first strong empirical demonstration that latent diffusion TTS can match autoregressive and duration-based diffusion state of the art without phonemes or duration labels, removing what had been treated as a near-necessary scaffold for text-speech alignment in LDM-based TTS.

This closes a gap that prior phoneme-free LDM-TTS work (E3 TTS, Simple-TTS) had left open, and its semantic-alignment and variable-length-modeling findings give later phoneme-free systems a validated recipe rather than an unproven direction. The paper's scaling experiments (across model size and up to 82K training hours) also provide one of the more systematic looks at how LDM-TTS quality scales when domain-specific alignment supervision is removed entirely.

## Claims

- **supports:** Diffusion Transformer backbones are better suited to TTS than U-Net backbones once domain-specific conditioning factors (phonemes, durations) are removed.
  > *Evidence:* Under matched training conditions, replacing the DiT backbone with a U-Net (and a U-Net variant without down/up-sampling) increases WER from 2.93 to 3.7 and drops SIM-r from 0.588 to 0.389 on the English cross-sentence task. *(§5.2, Table 4)*
- **supports:** Predicting total target length and generating variable-length sequences outperforms fixed-length generation with padding in diffusion-based TTS.
  > *Evidence:* Fixed-length modeling with padding reaches WER 6.81-8.89, while a learned speech length predictor with variable-length generation reaches WER 5.36-5.58 under otherwise identical settings. *(§5.2, Table 5)*
- **supports:** Aligning text and speech latent representations improves cross-attention-conditioned generation quality, independent of model or training-data scale.
  > *Evidence:* A speech codec fine-tuned with an auxiliary language-modeling loss against a frozen text encoder (Mel-VAE++) improves WER/SIM over the unaligned codec regardless of which text encoder (ByT5 or SpeechT5) is paired with it, and a jointly text-speech-trained text encoder (SpeechT5, 85M params) outperforms a larger text-only encoder (ByT5-base, 415M params) trained on more data. *(§5.2, Tables 6-7)*
- **complicates:** Removing domain-specific alignment factors from LDM-based TTS narrows but does not eliminate the gap to phoneme-duration-based systems in speaker similarity.
  > *Evidence:* DiTTo-en-XL reaches SIM-r 0.6554 on the cross-sentence task, below Voicebox's reported 0.681 (a phoneme/duration-based non-autoregressive model), even though DiTTo-en-XL is faster and matches or exceeds Voicebox on WER. *(§5.1, Table 2)*
- **complicates:** Codec compression ratio, not codec-reconstruction quality alone, determines suitability as a diffusion target for variable-length TTS.
  > *Evidence:* DAC achieves higher PESQ and ViSQOL codec-reconstruction scores than Mel-VAE, but its 7-8x longer latent sequences make training and inference substantially less efficient and degrade end-to-end WER/SIM relative to the more compressed but lower-fidelity Mel-VAE. *(§5.2, Table 7)*

## Limitations and Open Questions

> [!warning]
> Code and pretrained weights are not released at publication (only demo samples), and most baseline comparisons (Voicebox, VALL-E, NaturalSpeech 2/3) use numbers copied from the original papers' reported results rather than reproduced under DiTTo's own evaluation pipeline, limiting independent verification of head-to-head rankings.

The speaker-similarity gap to phoneme-duration-based systems like Voicebox persists even at the largest model scale tested, suggesting that removing domain-specific factors trades off some speaker-identity preservation for simplicity and speed. The paper also does not explore natural-language instruction following or fine-grained prosody control, both noted as future directions rather than addressed in this work. Multilingual results are reported on only 100 held-out examples per language, which is a narrow evaluation slice given nine languages with very different phonological properties.

## Wiki Connections

- [[diffusion-tts|Diffusion TTS]] — demonstrates that a Diffusion Transformer conditioned purely via cross-attention can match phoneme-and-duration-based diffusion TTS systems without explicit alignment supervision.
- [[zero-shot-tts|Zero-Shot TTS]] — evaluates zero-shot speaker similarity and naturalness via audio prompting on continuation and cross-sentence tasks against established zero-shot baselines.
- [[neural-codec|Neural Audio Codec]] — shows that codec compression ratio and semantic alignment with text embeddings, not codec reconstruction fidelity alone, drive downstream TTS quality when the codec's latent is the diffusion target.
- [[multilingual-tts|Multilingual TTS]] — trains a 790M-parameter variant on 82K hours across nine languages and reports comparable or better results than a multilingual autoregressive baseline.
- [[transformer-enc-dec-tts|Transformer Encoder-Decoder TTS]] — uses an encoder-decoder transformer as the speech length predictor that replaces explicit phoneme-duration alignment.
- [[2301.02111|VALL-E]] — used as the primary autoregressive codec-LM baseline for WER, speaker similarity, and inference speed comparisons across both English-only tasks.
- [[2403.03100|NaturalSpeech 3]] — cited as a phoneme-and-duration-based diffusion TTS baseline that DiTTo-TTS approaches without relying on the same alignment factors.
- [[2304.09116|NaturalSpeech 2]] — referenced as a prior LDM-based TTS system that still depends on phoneme and duration conditioning, motivating DiTTo-TTS's domain-factor-free approach.
- [[2206.04658|BigVGAN]] — used as the pretrained vocoder that converts DiTTo-TTS's generated mel-spectrograms into raw waveform at inference time.
- [[2406.02430|Seed-TTS]] — discussed as concurrent work in the appendix, situating DiTTo-TTS among contemporaneous large-scale TTS systems.
