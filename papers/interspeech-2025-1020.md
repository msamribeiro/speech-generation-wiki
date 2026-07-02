---
id: "interspeech-2025-1020"
title: "Learning Optimal Prosody Embedding Codebook based on F0 and Energy"
authors: ["David Portes", "Ales Horak"]
organization: "Masaryk University"
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: "2025-08-17"
ingested_date: "2026-07-03"
task: ["TTS", "evaluation"]
architecture: ["VAE"]
conditioning: ["prosody-conditioned"]
training: ["supervised"]
model_size: "not reported"
codec_used: "none"
datasets_train: ["LibriTTS train-clean-100"]
datasets_eval: ["LibriTTS test-clean"]
metrics:
  - name: F0-RMSE
    value: "FFE20=0.49% (interpolated)"
    system: FI strategy
    testset: LibriTTS test-clean
  - name: F0-RMSE
    value: "FFE20=3.30% (non-interpolated)"
    system: FN+EN+VM strategy
    testset: LibriTTS test-clean
code_available: true
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/portes25_interspeech.html"
related_concepts: ["prosody-control", "evaluation-metrics", "neural-codec"]
related_papers: []
field_significance:
  level: "low"
  type: ["empirical-benchmark", "engineering-integration"]
generation:
  date: "2026-07-03"
  agent: speech-generation-ingest-agent
  model: claude-sonnet-4-6
  commit: "d8ecfd4"
---

> [!abstract] Interspeech · 2025 · Conference
> **Portes and Horak** (Masaryk University) · [→ Paper](https://www.isca-archive.org/interspeech_2025/portes25_interspeech.html) · Demo: ? · Code: ✓
>
> A large-scale hyperparameter study over VQ-VAE configurations produces joint F0 and Energy prosody embeddings that outperform prior F0-only embeddings and are released as pre-trained models on HuggingFace.

## Problem

Fundamental frequency and energy are standard prosody features used in TTS, speech resynthesis, and prosody transfer, but there are no freely available pre-trained discrete vector representations of these features. Existing systems (FastSpeech 2, Daft-Exprt, etc.) each define their own task-specific encoding schemes that cannot be reused off-the-shelf in modern deep-learning architectures such as multimodal LLMs. Frame-level energy embeddings in particular have not been jointly encoded with F0 in prior work.

## Method

The system uses a Vector Quantized Variational Autoencoder (VQ-VAE) to jointly encode frame-level F0 and Energy features into a single stream of discrete codebook vectors. F0 is extracted at 200 Hz using the YAAPT algorithm and energy at matching resolution using PRAAT. The encoder downsamples by a factor of 16 via 1D convolution (from 200 Hz to 12.5 Hz), then nearest-neighbor vector quantization maps each latent vector to its closest codebook entry. F0 and Energy are encoded in separate convolutional channels with MSE reconstruction loss.

Three strategies for handling unvoiced F0 regions are explored. F0 interpolation (FI) smooths over unvoiced gaps so the signal does not descend to zero. Voiced F0 normalization with energy normalization and a voicedness mask (FN+EN+VM) speaker-normalises voiced frames, adds a third channel for the voicedness binary mask, and recovers unvoiced regions at reconstruction. A combined strategy (FN+EN+FI) applies normalization but interpolates rather than masking. The hyperparameter search with Optuna covers codebook size (20-320 bins), embedding vector size (32-1024), batch size (32-256), and learning rate (0.0001-0.001), totalling over 150 experiments on LibriTTS train-clean-100.

## Key Results

The best FI model achieves FFE 20 % of 0.49% on the test set, improving on the prior F0-only reference baseline of 1.60% FFE 20 %. The FN+EN+VM strategy (which preserves unvoiced regions) reaches 3.30% FFE 20 , improving on the prior baseline of 4.29% FFE 20 , while also embedding energy with an RMSE of 3.00 dB at 36.3 dB scale. The number of codebook bins is the dominant factor for reconstruction quality; larger bins consistently yield lower FFE. Embedding vector size is surprisingly unimportant: smaller sizes often match or exceed larger ones, which reduces model footprint without quality loss. Normalization strategies improve energy reconstruction (RMSE around 3 dB) relative to FI only (RMSE around 6.3 dB), but at the cost of higher FFE on F0.

## Novelty Assessment

The core contribution is engineering integration: VQ-VAE is an established architecture and F0 embeddings via VQ-VAE were explored in prior work by Polyak et al. and improved by the same first author in an earlier paper. The novelty here is the addition of Energy as a jointly encoded feature and the scale of the hyperparameter search (150+ experiments via Optuna). Releasing pre-trained models on HuggingFace is the practical differentiator. The paper does not introduce a new architecture, training objective, or conditioning mechanism; it characterises an existing approach across a wider parameter space and extends it to a second feature dimension. The evaluation is clean and the comparisons are direct, but the scope is narrow (English LibriTTS only, no downstream TTS evaluation).

## Field Significance

Low: The paper provides a well-characterised, reusable resource for prosody representation but does not advance the architecture or theory of prosody encoding. Its contribution is primarily empirical (hyperparameter sensitivity analysis) and resource-oriented (pre-trained codebook release). The finding that codebook size dominates and embedding dimension has little effect is useful design guidance, but the work remains narrow in scope with no downstream TTS evaluation to validate real-world utility.

## Claims

- **supports:** Larger codebook sizes are the primary lever for improving discrete prosody representation quality, with embedding vector dimensionality having minimal independent effect.
  > *Evidence:* Across 150+ experiments, FFE improves monotonically with codebook bin count (20 to 320), while embedding size had "surprisingly insignificant" effect with smaller sizes often outperforming larger ones. *(§5)*

- **supports:** Joint discrete encoding of F0 and Energy is achievable with minimal reconstruction trade-off relative to F0-only encoding, when codebook capacity is matched.
  > *Evidence:* The FI strategy reaches 0.49% FFE 20 % while simultaneously embedding Energy at 6.33 dB RMSE, outperforming the F0-only reference (1.60% FFE 20 %) from prior work. *(§5, Table 5)*

- **complicates:** Speaker normalisation of F0 improves energy reconstruction but degrades F0 reconstruction accuracy when unvoiced regions must be preserved.
  > *Evidence:* FN+EN+VM achieves energy RMSE of 3.00 dB vs 6.33 dB for FI, but FFE 20 % rises to 3.30% vs 0.49% for FI, indicating a direct quality trade-off between unvoiced-region handling and energy embedding fidelity. *(§5, Tables 5-6)*

- **complicates:** Downstream utility of prosody embeddings in TTS or speech LM systems is unverified; embedding quality is measured only through reconstruction accuracy rather than impact on synthesis naturalness.
  > *Evidence:* Evaluation is limited to VDE and FFE reconstruction metrics on LibriTTS; no TTS or downstream speech generation experiments are reported. *(§4, §6)*

## Limitations and Open Questions

No downstream TTS evaluation is provided, so the practical benefit of these embeddings for synthesis quality is undemonstrated. The evaluation is limited to English multispeaker speech (LibriTTS), and it is unclear how well the learned codebooks would transfer to other languages or acoustic conditions. The paper excludes audio files shorter than one second from training, which may bias the codebook away from short utterances common in conversational speech. Future work proposed by the authors involves prosody-conditioned text generation, which would serve as the first downstream validation of the released embeddings.

## Wiki Connections

- [[prosody-control|Prosody Control]] -- this paper provides pre-trained discrete prosody embeddings designed to make F0 and Energy straightforwardly usable as conditioning signals in downstream TTS and speech LM systems.
- [[evaluation-metrics|Evaluation Metrics]] -- the paper applies VDE and FFE reconstruction metrics to evaluate prosody embedding fidelity, with results showing codebook size as the dominant factor and embedding dimension as largely irrelevant.
- [[neural-codec|Neural Audio Codec]] -- the VQ-VAE codebook mechanism is shared with neural codec approaches; this work specialises it for prosodic features (F0 and Energy) rather than full-band audio, demonstrating that the same quantisation principles apply at the feature level.
