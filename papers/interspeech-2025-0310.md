---
id: interspeech-2025-0310
title: "Exploring the Effect of Segmentation and Vocabulary Size on Speech Tokenization for Speech Language Models"
authors: ["Shunsuke Kando", "Yusuke Miyao", "Shinnosuke Takamichi"]
organization: null
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: "2025-08-17"
ingested_date: "2026-05-30"
task: [TTS, codec]
architecture: [autoregressive-LM]
conditioning: []
training: [self-supervised, supervised]
model_size: "not reported (SLM uses OPT architecture, 12 layers, 16 heads, 1024 hidden, 4096 FFN)"
codec_used: "HuBERT layer 9 + K-means (variable K from 128 to 16384)"
datasets_train: ["LibriSpeech 960h (SLM training)", "LibriSpeech 100h subset (K-means training)"]
datasets_eval: ["sBLIMP", "sWUGGY", "ProsAudit (pros-syntax, pros-lexical)", "TopicStoryCloze (tSC)"]
metrics:
  - name: UAR
    value: 0.67
    system: (N=80, K=16384) fixed-width segmentation — best average across 5 SLU tasks
    testset: zero-shot SLU benchmarks (average)
  - name: UAR
    value: 0.65
    system: (N=20, K=128) baseline — GSLM-style original resolution
    testset: zero-shot SLU benchmarks (average)
code_available: true
demo_available: false
url: "https://www.isca-archive.org/interspeech_2025/kando25_interspeech.html"
related_concepts: [neural-codec, self-supervised-speech, spoken-language-model, autoregressive-codec-tts]
related_papers: []
---
> [!abstract] Interspeech · 2025 · Conference
> **Shunsuke Kando et al.** · [→ Paper](https://www.isca-archive.org/interspeech_2025/kando25_interspeech.html) · Demo: ✗ · Code: ✓
>
> A systematic 64-configuration grid search over segmentation width (N ∈ {20,…,280} ms) and K-means vocabulary size (K ∈ {128,…,16384}) for HuBERT-based speech tokenization demonstrates that moderately coarse segmentation (N=80 ms) paired with large vocabulary (K=16384) outperforms GSLM-style original-resolution tokenization while reducing training data by 50% and training runtime by 70%.

## Problem

Speech language models (SLMs) depend critically on their discrete unit vocabulary, yet the design space of segmentation width and cluster size has not been systematically explored. Prior work either uses original-resolution HuBERT features with small K (GSLM: K=50–200) or very coarse segmentation with large K (syllable-level), but does not compare these jointly. The consequence is that practitioners lack guidance on the optimal tokenization tradeoff between sequence length (computational cost), vocabulary expressiveness, and downstream SLU performance.

## Method

The pipeline is: (1) extract HuBERT layer-9 representations; (2) segment by fixed width N ms and mean-pool within segments; (3) apply K-means with vocabulary size K; (4) deduplicate consecutive identical units. This yields 8×8 = 64 tokenization configurations.

The SLM is an OPT-style decoder-only Transformer (12 layers, 16 attention heads, hidden dim 1024, FFN 4096), trained on LibriSpeech 960h (or smaller after pooling). Training follows GSLM hyperparameters, with sequences chunked to 2048 tokens, batch size 16, up to 50k steps with early stopping, on a single A100. Results are averages across 3 random seeds.

Evaluation uses five zero-shot spoken language understanding benchmarks:
- **sBLIMP**: grammatical knowledge (12 phenomena)
- **sWUGGY**: lexical knowledge (real vs. nonce word)
- **pros-syntax / pros-lexical**: prosodic phrase/word boundary perception (ProsAudit)
- **tSC (TopicStoryCloze)**: commonsense/semantic coherence

Variable-width segmentation (phoneme via UnsupSeg, syllable via Sylber, word via GradSeg) is also evaluated against fixed-width baselines of matched median segment duration.

## Key Results

Best fixed-width configuration: (N=80, K=16384), average accuracy 0.67 across 5 tasks.
vs. (N=20, K=128) GSLM-style baseline: average accuracy 0.65, training runtime 12.4h, 87M tokens.
Best (N=80, K=16384): training runtime 8.3h, 42M tokens — 50% data reduction, 70% runtime reduction.

Task-specific patterns:
- pros-syntax achieves very high accuracy even at large N, because the 400 ms inserted pauses greatly exceed N.
- sBLIMP hovers near chance (0.5) for all configurations, consistent with prior literature on SLM grammatical limitations.
- Some sBLIMP subtasks (ellipsis, quantifiers) show above-chance accuracy with unique optimal settings, suggesting multi-token ensembling could help.
- Variable-width segmentation (phoneme, syllable, word levels) generally matches but does not exceed fixed-width, with syllable showing slight gains on sBLIMP and pros-syntax at the cost of added unsupervised segmentation compute.

Qualitative analysis: at (80, 128) — large N, small K — the model cannot distinguish phonetically similar tokens because the vocabulary is too small to reflect the increased acoustic diversity at 80ms resolution. Increasing K to 16384 restores discriminability (illustrated with the yonder/zonder sWUGGY example).

## Novelty Assessment

The contribution is empirical and systematic rather than architectural: a comprehensive ablation over a large tokenization grid that had not been done before. The finding that moderately coarse (80ms) fixed-width tokenization with large K beats fine-grained (20ms) tokenization is practically important for SLM efficiency without hurting accuracy. The phoneme-morpheme analogy (combining fewer, broader segments requires more codewords) is an intuitive explanation for the observed tradeoff. The study does not address speech synthesis (generation quality) — only spoken language understanding — which limits its relevance to TTS directly, though the tokenization findings apply to codec-based TTS architectures using HuBERT-like semantic tokens.

## Limitations and Open Questions

- Evaluation is exclusively on SLU (understanding) tasks; no speech generation quality assessment.
- Training data (LibriSpeech 960h) is small by current SLM standards; findings may differ at larger scale (LibriLight 60k).
- Only HuBERT layer-9 as the SSL backbone; other models (WavLM, wav2vec 2.0) may yield different tradeoff curves.
- Variable-width segmentation using predicted (rather than oracle) boundaries introduces inaccuracy; learned segment representations (as in Sylber) may be better than raw pooling.
- Optimal tokenization varies per benchmark, motivating multi-token ensemble strategies not explored here.

## Wiki Connections

This paper directly informs [[neural-codec]] and [[self-supervised-speech]] by characterizing the design space of K-means-based discrete speech representations. The SLM training framework connects to [[spoken-language-model]] research. The codec-LM interface question — how to tokenize continuous SSL features for language model training — is also central to [[autoregressive-codec-tts]] systems (VALL-E family uses EnCodec acoustic tokens, but semantic token design follows similar tradeoffs). The efficiency gains (50% data, 70% runtime) are practically relevant across SCA and TTS pipelines that rely on HuBERT tokenization.
