---
id: interspeech-2025-2660
title: "Triadic Multi-party Voice Activity Projection for Turn-taking in Spoken Dialogue Systems"
authors: ["Mikey Elmers", "Koji Inoue", "Divesh Lala", "Tatsuya Kawahara"]
organization: Kyoto University
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-06-03
task: [SCA]
architecture: [transformer-enc-dec]
conditioning: [speaker-conditioned]
training: [supervised, self-supervised]
model_size: "not reported"
codec_used: "none"
datasets_train: ["TEIDAN (Kyoto University triadic corpus, Japanese)"]
datasets_eval: ["TEIDAN (Kyoto University triadic corpus, Japanese)"]
metrics:
  - name: WER
    value: "not reported"
    system: proposed
    testset: TEIDAN
code_available: null
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/elmers25_interspeech.html"
related_concepts: ["spoken-language-model", "speech-to-speech", "self-supervised-speech", "evaluation-metrics"]
related_papers: []
field_significance:
  level: moderate
  type: [engineering-integration, dataset-contribution]
generation:
  date: 2026-06-03
  model: claude-sonnet-4-6
  commit: "55d0339"
---

> [!abstract] Interspeech · 2025 · Conference
> **Elmers et al.** (Kyoto University) · [→ Paper](https://www.isca-archive.org/interspeech_2025/elmers25_interspeech.html) · Demo: ? · Code: ?
>
> Voice activity projection (VAP) is extended from dyadic to triadic three-person conversation, establishing that acoustic-only turn prediction generalises beyond the dyadic setting using a new Japanese multi-party corpus.

## Problem

Voice activity projection (VAP) — a self-supervised approach to predicting future voice activity from acoustic signals — has been studied exclusively in dyadic (two-speaker) settings. Real-world spoken dialogue systems, especially for embodied robots and multi-party virtual agents, must handle conversations with three or more participants, where the rules of turn-taking become substantially more complex. No prior work had demonstrated that VAP could generalise to triadic conversation, and no suitable triadic corpus with per-speaker separated audio channels existed for training or evaluation.

## Method

The system follows the established VAP architecture but replaces the dyadic two-channel processing with three-channel input, one per speaker, all recorded at 16 kHz with separate pin microphones (no diarisation required). A pre-trained contrastive predictive coding (CPC) encoder — pre-trained on LibriSpeech — extracts features from each raw audio channel independently. These are passed through a single-layer self-attention Transformer per channel, and the three channel representations are then merged by a 64-dimensional three-layer multi-channel cross-attention Transformer that models speaker interactions jointly. Two output heads perform multi-task learning: the primary VAP head predicts future voice activity for all three speakers over a 600 ms horizon, and an auxiliary voice activity detection (VAD) head detects current activity to stabilise training.

The key adaptation from dyadic VAP is the state space reduction: dyadic VAP uses four sub-state bins per speaker (yielding 2^8 = 256 states), while the triadic model uses two sub-state bins per speaker (0.2 s and 0.4 s durations, yielding 2^6 = 64 states). This reduction was necessary to avoid data sparsity and memory issues during training. The model is trained on the TEIDAN corpus — a new multi-modal triadic Japanese corpus constructed specifically for this study, comprising 287.79 minutes across 54 discussions (18 unique triads: 12 spontaneous, 6 attentive listening). Overlapping speech accounted for 31.98% of spontaneous and 21.17% of attentive listening segments.

Three model variants are evaluated: trained on spontaneous speech only (SP), attentive listening only (ATT), and the combined corpus (BOTH).

## Key Results

Next-speaker prediction accuracy (evaluated during periods of mutual silence of at least 100 ms, predicting speakers who hold the floor for at least 1 s) outperforms a last-speaker baseline for all three model variants:

- SP model: 62.43% vs. 55.56% baseline
- ATT model: 87.50% vs. 81.25% baseline
- BOTH model: 86.83% vs. 77.58% baseline

The combined BOTH model achieves the lowest cross-entropy loss (2.09) and the highest accuracy in the mixed evaluation. Performance is substantially higher on attentive listening data, where one speaker dominates, than on spontaneous discussion data, where overlapping speech is more frequent. The results confirm that VAP can distinguish next speakers during mutual silence in triadic conversation, though accuracy on spontaneous overlapping speech remains an open problem.

## Novelty Assessment

The contribution is primarily engineering integration: the dyadic VAP architecture is extended to three channels by replacing the two-channel cross-attention with a multi-channel cross-attention Transformer, and the state space is simplified to stay tractable. Neither the CPC encoder, the self-attention Transformer, nor the cross-attention mechanism is novel. The multi-task VAD auxiliary objective follows the multilingual VAP work by Inoue et al. The genuine contribution is the first empirical demonstration that VAP operates in a triadic setting, and the creation of the TEIDAN corpus as the enabling dataset. The small corpus size (under 5 hours total) and single-language (Japanese) constraint limit the strength of the empirical claim.

## Field Significance

Moderate — This paper establishes a proof of concept for multi-party turn-taking prediction using acoustic-only VAP, an important prerequisite for deploying spoken dialogue systems in group conversation scenarios. Its value to the field is primarily as a first-step demonstration and dataset contribution rather than an architectural advance; the approach is a straightforward extrapolation of existing dyadic VAP.

## Claims

- Acoustic-only voice activity projection can predict next-speaker identity in triadic conversation with accuracy above a last-speaker baseline, though gains over baseline are modest in spontaneous overlapping speech.
- The complexity of multi-party turn-taking increases faster than linearly with group size, making state-space design a fundamental constraint for VAP-style approaches beyond dyadic settings.
- The type of conversation — spontaneous discussion versus structured attentive listening — substantially affects VAP accuracy, reflecting differences in overlapping speech prevalence.
- Spoken dialogue systems operating in multi-party scenarios require turn-taking models trained on matched multi-party data, as dyadic training corpora are unlikely to transfer directly.

## Limitations and Open Questions

> [!warning]
> The TEIDAN corpus is small (under 5 hours), monolingual (Japanese), and not yet publicly released. The reported accuracy gains are measured on the same corpus used for training and testing, with no held-out language or domain. These factors limit the generalisability of the empirical claims.

The state-space reduction from 4 to 2 sub-state bins per speaker may lose prediction granularity relative to dyadic VAP; the paper does not quantify this trade-off directly. The acoustic-only constraint is explicit and acknowledged: gaze, gesture, syntax, and prosody are known turn-yielding cues that are not modelled. Scaling beyond three participants is expected to be problematic — the paper estimates 65,536 states for four-participant conversation under the original 4-bin scheme — and may require architectural alternatives to VAP. Only next-speaker prediction during mutual silence is evaluated; prediction during overlap, which is prevalent in spontaneous data, is left for future work.

## Wiki Connections

This paper informs [[spoken-language-model]] and [[speech-to-speech]] through its focus on turn-taking in live dialogue systems. The self-supervised CPC encoder connects to [[self-supervised-speech]]. The next-speaker prediction accuracy metric and the discussion of evaluation methodology on spontaneous versus attentive listening speech relate to [[evaluation-metrics]].
