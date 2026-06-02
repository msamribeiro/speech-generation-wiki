---
id: interspeech-2025-0203
title: "ClapFM-EVC: High-Fidelity and Flexible Emotional Voice Conversion with Dual Control from Natural Language and Speech"
authors: ["Yu Pan", "Yanni Hu", "Yuguang Yang", "Jixun Yao", "Jianhao Ye", "Hongbin Zhou", "Lei Ma", "Jianjun Zhao"]
organization: "Kyushu University / University of Tokyo / EverestAI Ximalaya"
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-06-02
task: [VC]
architecture: [flow-matching, transformer-enc-dec]
conditioning: [emotion-conditioned, prompt-conditioned, speaker-conditioned]
training: [supervised, self-supervised]
model_size: "not reported"
codec_used: "none"
datasets_train: ["Internal expressive Mandarin corpus (20h, 12,000 utterances, 7 emotion classes)"]
datasets_eval: ["Internal expressive Mandarin corpus (held-out)"]
metrics:
  - name: MOS
    value: 4.09
    system: ClapFM-EVC
    testset: Internal Mandarin (reference speech condition)
  - name: MOS
    value: 2.98
    system: MixEmo
    testset: Internal Mandarin (reference speech condition)
  - name: UTMOS
    value: 3.68
    system: ClapFM-EVC
    testset: Internal Mandarin (reference speech condition)
  - name: CER
    value: 6.76
    system: ClapFM-EVC
    testset: Internal Mandarin (reference speech condition)
code_available: null
demo_available: true
url: "https://www.isca-archive.org/interspeech_2025/pan25b_interspeech.html"
related_concepts: [voice-conversion, emotion-synthesis, flow-matching, disentanglement, self-supervised-speech, instruction-conditioned-tts]
related_papers: []
field_significance:
  level: moderate
  type: [architectural-novelty]
generation:
  date: 2026-06-02
  model: claude-sonnet-4-6
  commit: "f0299ef"
---

> [!abstract] Interspeech · 2025 · Conference
> **Yu Pan et al.** (Kyushu University / University of Tokyo / EverestAI Ximalaya) · [→ Paper](https://www.isca-archive.org/interspeech_2025/pan25b_interspeech.html) · Demo: ✓ · Code: ?
>
> ClapFM-EVC couples a contrastive language-audio pretraining model (EVC-CLAP) with a flow-matching decoder to enable emotional voice conversion steered by either reference speech or free-form natural language prompts, with adjustable emotion intensity.

## Problem

Emotional voice conversion systems have historically relied on either reference speech clips or fixed categorical labels to specify the target emotion. Both paradigms constrain flexibility: categorical labels cover only a small discrete set of states, while reference speech requires the user to find an appropriate exemplar. Beyond the conditioning limitation, GAN- and autoencoder-based EVC systems exhibit inconsistencies in emotion diversity, speech naturalness, and overall audio quality that diffusion- and flow-matching-based approaches in adjacent TTS work have since addressed. The additional challenge of intensity control — allowing users to dial in how strongly an emotion is expressed — has been handled only partially in prior work, typically by retrofitting a scalar multiplier into an already-constrained representation.

## Method

ClapFM-EVC is a two-stage, any-to-one EVC framework. The first stage trains EVC-CLAP, an emotional contrastive language-audio pretraining model that aligns speech and text emotional representations in a shared 512-dimensional latent space. Audio is encoded with a pre-trained HuBERT model and text with XLM-RoBERTa. Rather than using hard one-hot labels, EVC-CLAP is trained with a symmetric KL-divergence loss (symKL-loss) against soft labels that blend categorical emotion labels and natural language prompt labels. This allows the model to represent emotional nuance across a continuum rather than a discrete set.

The second stage is AdaFM-VC, the voice conversion backbone. Content representations are extracted from source speech via a pre-trained ASR model (HybridFormer) as Phonetic Posteriorgrams (PPGs). A FuEncoder module fuses PPGs with emotional embeddings from EVC-CLAP. Central to FuEncoder is an adaptive intensity gate (AIG), a learnable scalar applied to the emotional embedding before fusion, giving the user direct control over expression strength. The fused features are passed to a conditional flow-matching (CFM) decoder — 6 CFM blocks using ResNet, multi-head self-attention, and FiLM conditioning — that generates the target Mel-spectrogram from Gaussian noise via an optimal-transport flow. A pre-trained BigVGAN vocoder converts the spectrogram to audio.

![Overall training architecture of the proposed ClapFM-EVC framework.](assets/interspeech-2025-0203/figure-1.png)

At inference, the target emotion embedding can be obtained in three ways: from a reference speech clip directly, from a natural language prompt, or via retrieval from a pre-built reference corpus using a natural language query. This three-mode design means the system degrades gracefully when reference audio is unavailable.

## Key Results

Against three baselines (StarGAN-EVC, Seq2seq-EVC, MixEmo) on an internal Mandarin test set, ClapFM-EVC substantially outperforms all comparators across both speech quality and emotion similarity dimensions (Table 1). The strongest prior system, MixEmo, achieves nMOS 2.98 and EECS 0.65; ClapFM-EVC reaches nMOS 4.09 (±0.09) and EECS 0.82 — a 37.2% relative improvement in naturalness and 26.2% in automatic emotion similarity. UTMOS rises from 2.09 to 3.68, a 49.2% relative gain. CER drops from 8.93% to 6.76%.

An ABX preference test comparing reference-speech-driven versus prompt-driven conversion found that 57.4% of 47 participants expressed no preference between the two modes, with 19.1% favouring the prompt-driven output, suggesting the natural language conditioning is effective without a meaningful quality penalty.

Ablation results confirm that soft-label training (categorical labels + symKL-loss) and the AIG module are each individually necessary: removing categorical labels causes EECS to drop from 0.79 to 0.66; replacing symKL with KL causes a 10.1% EECS decline; removing AIG degrades both nMOS (4.01 to 3.62) and EECS (0.79 to 0.74).

> [!note]
> All evaluations use a proprietary internal Mandarin corpus not publicly available. Baseline comparisons are restricted to systems that accept reference speech; the prompt-driven comparison is conducted only as a within-system ABX test against ClapFM-EVC's own reference mode. Cross-system comparisons are not conducted in the natural language prompt setting.

## Novelty Assessment

The core ideas — CLAP-style contrastive alignment, flow-matching decoders, and PPG-based content extraction — are all established techniques. The novelty lies in their systematic composition for EVC: adapting CLAP-style training with symKL-loss and soft multi-source labels to the emotion alignment problem, and introducing the AIG module as an explicit intensity knob within the FuEncoder. The three-mode inference (reference, prompt, retrieval) is a practical contribution that adds usability without requiring new training. The architecture is a reasonable and well-motivated combination of existing components rather than a fundamentally new model family. The primary contribution is engineering integration with a targeted training-objective refinement (symKL + soft labels).

## Field Significance

Moderate — ClapFM-EVC advances the case that natural language prompts can replace or complement reference speech as a conditioning signal in emotional voice conversion, supporting a broader trend of instruction-conditioned speech manipulation. The flow-matching backbone brings EVC quality noticeably above prior GAN and autoencoder baselines on the authors' benchmark. However, the evaluation is limited to a single proprietary Mandarin speaker corpus and the baseline set does not include more recent diffusion- or flow-matching-based EVC systems, limiting the generalisability of the quantitative claims.

## Claims

- Natural language prompts can control emotional voice conversion at parity with reference speech for the majority of listeners, reducing reliance on hard-to-source reference audio.
- Flow matching produces noticeably higher speech naturalness and audio quality in emotional voice conversion than GAN and autoencoder baselines.
- Combining categorical emotion labels with free-form prompt labels through soft-label contrastive training improves emotion embedding quality over prompt-only or label-only training.
- An explicit scalar intensity gate applied to emotional embeddings before content-emotion fusion improves both naturalness and emotion similarity in converted speech.

## Limitations and Open Questions

> [!warning]
> The system is trained and evaluated entirely on a proprietary internal Mandarin corpus. No open-source data or model weights are released, and no cross-lingual or multi-speaker generalisation is tested.

Comparisons are restricted to older GAN and autoencoder baselines (StarGAN-EVC, Seq2seq-EVC, MixEmo); no diffusion-based or recent flow-matching EVC systems are included, so the claimed state-of-the-art position cannot be verified against the most competitive contemporaries. The evaluation is any-to-one (fixed target speaker identity), leaving any-to-any EVC performance unaddressed. Emotion coverage is limited to seven categorical classes; whether the natural language conditioning generalises to subtler or blended emotional states is untested.

## Wiki Connections

This paper advances [[voice-conversion]] and [[emotion-synthesis]] by introducing a dual-modality conditioning framework. The flow-matching decoder places it within the [[flow-matching]] paradigm. The CLAP-based alignment of speech and text emotional features connects to [[self-supervised-speech]] representation learning. The natural language prompt interface is closely related to [[instruction-conditioned-tts]]. The AIG module addresses intensity control within [[disentanglement]] of content, speaker, and affective attributes.
