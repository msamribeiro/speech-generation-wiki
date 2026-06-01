---
id: interspeech-2025-0669
title: "PAST: Phonetic-Acoustic Speech Tokenizer"
authors: [Nadav Har-Tuv, Or Tal, Yossi Adi]
organization: Hebrew University of Jerusalem
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-05-30
task: [codec, TTS]
architecture: [hybrid]
conditioning: []
training: [supervised, self-supervised]
model_size: "185M params"
codec_used: "EnCodec (backbone, extended)"
datasets_train: [LibriSpeech, TIMIT]
datasets_eval: [LibriSpeech]
metrics:
  - name: PESQ
    value: 3.55
    system: PAST
    testset: LibriSpeech test-clean
  - name: STOI
    value: not reported
    system: PAST
    testset: LibriSpeech test-clean
  - name: WER
    value: 15.7
    system: PAST
    testset: LibriSpeech test-clean (DASB benchmark)
  - name: WER
    value: 36.8
    system: PAST
    testset: LibriSpeech test-other (DASB benchmark)
  - name: PESQ
    value: 3.88
    system: EnCodec (acoustic topline)
    testset: LibriSpeech test-clean
  - name: WER
    value: 18.5
    system: SpeechTokenizer (baseline)
    testset: LibriSpeech test-clean (DASB)
code_available: true
demo_available: true
url: https://www.isca-archive.org/interspeech_2025/hartuv25_interspeech.html
related_concepts: [neural-codec, self-supervised-speech, autoregressive-codec-tts, streaming-tts]
related_papers: []
---
> [!abstract] Interspeech · 2025 · Conference
> **Nadav Har-Tuv et al.** (Hebrew University of Jerusalem) · [→ Paper](https://www.isca-archive.org/interspeech_2025/hartuv25_interspeech.html) · Demo: ✓ · Code: ✓
>
> PAST is an end-to-end speech tokenizer that achieves state-of-the-art phonetic representation and speech reconstruction without relying on any external SSL model, by integrating supervised phoneme and character-level CTC losses directly into an EnCodec-based RVQ architecture augmented with a transformer encoder.

## Problem

Existing hybrid speech tokenizers (SpeechTokenizer, X-Codec, Mimi) that aim to capture both phonetic and acoustic information all depend on pretrained self-supervised learning (SSL) models (HuBERT, WavLM) as phonetic teachers. This creates several problems: (1) SSL pseudo-labels are not explicitly optimized for phonetic discrimination, so the distilled phonetic content may be noisy or semantically incomplete; (2) SSL model training requires vast unlabeled data, is computationally expensive, and creates dependency on external models; (3) the distillation process may encode cross-vector properties irrelevant to phonetics. Purely acoustic tokenizers (EnCodec, SoundStream) achieve high reconstruction fidelity but encode minimal phonetic structure, making them poor foundations for speech language models.

## Method

PAST builds on EnCodec's convolutional encoder-decoder with RVQ quantization and adds two components:

**Transformer encoder.** Inserted between the convolutional encoder and the RVQ module, an 8-layer transformer encoder (hidden size 768, 16 attention heads, feed-forward 2048) models sequence-level dependencies. To stabilize training and prevent the transformer from being bypassed, a stochastic skip-connection is used: during training the input to the RVQ is sampled from (a) transformer output only (p=0.3), (b) skip (encoder output, p=0.1), or (c) their average (p=0.6). During inference, only the average is used.

**Supervised auxiliary heads on the first RVQ codebook (VQ1).** Two auxiliary loss heads operate on the first-codebook output ẑ₁:
- CTC character match: a BiLSTM head with linear projections applies CTC loss against character transcriptions obtained via Wav2Vec2 alignment. CTC is shown to be the dominant phonetic supervision signal in ablations.
- Phoneme classification: a linear projection applies cross-entropy against TIMIT phoneme labels (10% of training batch).

Training data is LibriSpeech (960h) + TIMIT (5h phoneme-labeled). Training uses 2 × A100 GPUs for 400k steps. The model has 185M parameters total (68M in the transformer, 4M in auxiliary heads), with 8 RVQ codebooks at 50Hz frame rate.

**Streamable variant.** PAST-Streamable replaces the transformer with a causal self-attention and unidirectional LSTM, uses left-only padding, and operates with a 20ms look-ahead window. This enables real-time applications.

## Key Results

**Phonetic metrics** (PNMI, ABX, WER on discrete tokens, DASB benchmark): PAST achieves PNMI=0.75 (vs. SpeechTokenizer 0.72, X-Codec 0.40), ABX-within 2.82 (vs. SpeechTokenizer 3.43), WER test-clean 15.7% (vs. SpeechTokenizer 18.5%, X-Codec 17.1%), WER test-other 36.8% (vs. SpeechTokenizer 41.3%). PAST-Streamable also surpasses baselines: PNMI=0.74, ABX-within 3.05, WER clean 14.3%.

**Reconstruction** (LibriSpeech test-clean): PAST SISNR=4.84, VISQOL=4.40, PESQ=3.55 — all substantially better than SpeechTokenizer (SISNR=0.44, PESQ=3.15) and X-Codec (SISNR=-7.12). Only EnCodec (the pure acoustic topline) exceeds PAST on reconstruction (SISNR=7.49, PESQ=3.88).

**Speech language modeling** (sWUGGY, identical 300M LM trained per tokenizer): PAST achieves 71.8/57.5 (Inter/OOV) vs. SpeechTokenizer 63.7/55.6, X-Codec 55.1/52.9, and even k-means HuBERT 67.9/55.4.

Ablation over components: CTC loss is the single most important addition (ABX-within drops from 21.2 to 4.42 with CTC added alone); phoneme classification adds further modest gains; the transformer encoder is necessary to avoid reconstruction degradation when CTC is added.

## Novelty Assessment

The key insight — that supervised phonetic data (CTC on characters + phoneme classification) can replace SSL pseudo-label distillation in hybrid tokenizers — is well-supported and practically impactful. It simplifies the pipeline (no external SSL model needed), potentially lowers cost for low-resource languages (only 965h of labeled data needed), and achieves better performance. The stochastic skip-connection to prevent transformer bypass is a practical training-stability contribution. The streamable variant is a useful engineering contribution for real-time applications. The approach is a direct challenge to the prevailing assumption that SSL pseudo-labels are necessary for hybrid tokenization.

## Limitations and Open Questions

PAST requires labeled phoneme/character data, which constrains multilingual scalability — the paper explicitly acknowledges this and targets it as future work. PAST's reconstruction quality (SISNR=4.84) falls below pure EnCodec (SISNR=7.49), reflecting the phonetic-acoustic trade-off. The model is 185M parameters, larger than some baseline tokenizers, partly due to the transformer encoder. Evaluation is English-only on clean speech; robustness to noise, accents, and spontaneous speech is not assessed.

## Wiki Connections

PAST is a direct contribution to the [[neural-codec]] concept, advancing the state of art in hybrid tokenizers. It competes with and improves over SpeechTokenizer (a key prior work in [[neural-codec]]). The CTC and phoneme supervision approach connects to [[self-supervised-speech]] — PAST argues for replacing SSL with explicit supervision. The streamable variant extends [[streaming-tts]] applicability to codec-level tokenization. As a foundation for speech language models, it is foundational to the [[autoregressive-codec-tts]] paradigm.
