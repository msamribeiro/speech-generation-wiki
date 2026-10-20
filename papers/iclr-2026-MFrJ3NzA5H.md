---
id: "iclr-2026-MFrJ3NzA5H"
title: "Continuous Audio Language Models"
authors: ["Simon Rouard", "Manu Orsini", "Axel Roebel", "Neil Zeghidour", "Alexandre Défossez"]
organization: "Kyutai / IRCAM (STMS, CNRS, Sorbonne Université)"
venue: ICLR
venue_type: conference
year: 2026
month: 4
published_date: "2026-04-27"
ingested_date: "2026-10-20"
task: [TTS, SCA]
architecture: [autoregressive-LM, VAE, flow-matching, hybrid]
conditioning: [text-conditioned, zero-shot, prompt-conditioned]
training: [supervised, fine-tuning, distillation]
model_size: "313M params (TTS CALM); 100M Pocket TTS incl. VAE decoder; 2.2B backbone (speech continuation, from Helium-1); 1.35B backbone (music)"
codec_used: "Causal continuous speech VAE (Mimi-derived, 32 dims, 12.5Hz, 24kHz); music VAE 128 dims, 25Hz, 32kHz"
datasets_train: ["AMI", "Earnings22", "GigaSpeech", "SPGISpeech", "TED-LIUM", "VoxPopuli", "LibriHeavy", "Emilia", "French and English speech data (Moshi recipe, not specified)", "LAION-Disco-12M (400K-song subset, music)"]
datasets_eval: ["LibriSpeech test-clean", "English speech continuation test set (not specified)", "MusicCaps", "internal music test set"]
metrics:
  - name: WER
    value: 1.81
    system: proposed (CALM w/ LSD, NFE=1, CFG=1.5)
    testset: LibriSpeech test-clean
  - name: CER
    value: 0.57
    system: proposed (CALM w/ LSD, NFE=1, CFG=1.5)
    testset: LibriSpeech test-clean
  - name: SPK-SIM
    value: 0.52
    system: proposed (CALM w/ LSD, NFE=1, CFG=1.5)
    testset: LibriSpeech test-clean
  - name: MUSHRA
    value: 61.1
    system: proposed (CALM w/ LSD, NFE=1, CFG=1.5)
    testset: LibriSpeech test-clean
  - name: WER
    value: 2.42
    system: F5-TTS (NFE=32)
    testset: LibriSpeech test-clean
  - name: SPK-SIM
    value: 0.66
    system: F5-TTS (NFE=32)
    testset: LibriSpeech test-clean
  - name: WER
    value: 1.95
    system: DSM (16 RVQ)
    testset: LibriSpeech test-clean
  - name: WER
    value: 1.84
    system: proposed (Pocket TTS, 100M)
    testset: LibriSpeech test-clean (enhanced prompts)
  - name: MOS
    value: 3.45
    system: proposed (CALM consistency 1-step, temperature 0.8, speech continuation acoustic quality)
    testset: English speech continuation test set
  - name: MOS
    value: 2.75
    system: RQ-Transformer 8 RVQ (temperature 0.8, speech continuation acoustic quality)
    testset: English speech continuation test set
  - name: PESQ
    value: 2.42
    system: proposed speech VAE (32 dims)
    testset: not reported
  - name: STOI
    value: 0.90
    system: proposed speech VAE (32 dims)
    testset: not reported
code_available: true
demo_available: true
url: "https://openreview.net/forum?id=MFrJ3NzA5H"
related_concepts: [autoregressive-codec-tts, flow-matching, zero-shot-tts, spoken-language-model, neural-codec, subjective-evaluation]
related_papers: ["2025.acl-long.313", "2509.08753", "2502.03930", "2407.08551", "2410.00037", "2209.03143", "2301.02111", "2210.13438", "2301.11325", "2310.00704", "2511.05516", "2407.05361", "2106.06909"]
field_significance:
  level: "moderate"
  type: [architectural-novelty]
generation:
  schema_version: 2
  date: "2026-10-20"
  runtime: "claude-code"
  provider: "anthropic"
  agent: speech-generation-ingest-agent
  model: "claude-opus-5-5"
  commit: "fb8dafe"
---

> [!abstract] ICLR · 2026 · Conference
> **Rouard et al.** (Kyutai / IRCAM (STMS, CNRS, Sorbonne Université)) · [→ Paper](https://openreview.net/forum?id=MFrJ3NzA5H) · Demo: ✓ · Code: ✓
>
> Replaces the RVQ token stack and its RQ-Transformer sampler with autoregressive prediction of continuous VAE latents through a one-step consistency (or Lagrangian self-distillation) MLP head, yielding audio language models that beat matched discrete baselines on quality while sampling faster, and a 100M-parameter TTS model (Pocket TTS) that runs faster than real time on a laptop CPU.

## Problem

Discrete audio language models inherit the lossy bitrate ceiling of their RVQ codecs: higher fidelity requires deeper token hierarchies, and because tokens within a frame are strongly dependent, the extra depth must be modelled sequentially (flattening, delay patterns, or an RQ-Transformer depth model), which inflates inference cost. Continuous autoregressive alternatives existed (MAR in vision, and MELLE, SALAD, and DiTAR for TTS), but they were limited to TTS on smaller or domain-specific data, needed many diffusion steps per frame, and in the authors' preliminary experiments a MAR-style model diverged quickly when applied to long, diverse music because of error accumulation. Whether continuous modelling could handle text-free speech continuation and full-mix music, and do so at lower cost than RVQ, was open.

## Method

CALM (Continuous Audio Language Models) operates on the latents of a causal VAE-GAN derived from Mimi: the RVQ bottleneck is replaced by a Gaussian VAE bottleneck with KL regularization, the encoder and decoder keep Mimi's convolution-plus-Transformer design, and for speech WavLM is distilled into the whole latent (not only a first codebook) via a cosine loss. The speech VAE produces 32-dimensional latents at 12.5 Hz from 24 kHz audio and is trained with adversarial, feature-matching, KL, and distillation losses but no reconstruction loss; the music VAE uses 128 dimensions at 25 Hz from 32 kHz audio.

A causal backbone Transformer reads previous latents and emits a context vector per frame, which conditions a small MLP head that generates the next continuous latent. Instead of MAR's multi-step diffusion head, the head is a continuous-time consistency model (TrigFlow parameterisation, following Lu and Song) that maps noise to a latent in a single step; for TTS the authors also use Lagrangian Self-Distillation (LSD), a one-step flow-map method trained with a mix of flow-matching (75% of a batch) and self-distillation (25%) losses. Two additions target error accumulation in music: variance-preserving noise injection into the backbone's long-context inputs at training time, and a short-context causal Transformer over the last K=10 clean latents whose output is summed with the backbone output before the head. Neither addition helped speech, so the speech models omit both.

![Figure 1: Overview of our model. During training, latent vectors x s are noised to encourage the backbone Transformer to focus on coarse structure. The consistency head is a consistency model conditioned on the latent variable z s long produced by the backbone, as well as a short-term context vector z s short computed from a short-context Transformer applied to the most recent clean latent tokens.](assets/iclr-2026-MFrJ3NzA5H/figure-1.png)

Four further techniques make the model practical. A head batch multiplier reuses each backbone output for N=8 independently noised head losses per step, amortising the backbone cost. Gaussian temperature sampling scales the head's input noise standard deviation by √τ, a heuristic analogue of discrete temperature (0.8 for speech continuation, 0.7 for TTS). Latent classifier-free guidance applies CFG to the conditioning vector fed to the head, since one-step consistency has no sampling trajectory to guide. Latent distillation then trains a student backbone with an ℓ2 loss to reproduce the CFG-guided teacher latent while reusing the teacher's head unchanged, which removes the doubled batch at inference and can shrink the backbone.

The speech continuation model starts from the 2B Helium-1 text LM (2.2B backbone), trains on French and English speech following the Moshi recipe, and keeps Moshi's inner monologue: a time-aligned text stream predicted from the backbone output, with audio delayed two frames (160 ms) behind text. The TTS model has a 302M backbone and a 10M head (313M total), takes SentencePiece text (4k vocabulary) as a prefix with an audio prompt, and trains on 88k hours of public data (AMI, Earnings22, GigaSpeech, SPGISpeech, TED-LIUM, VoxPopuli, LibriHeavy, Emilia). Pocket TTS distills this 24-layer teacher at CFG α=1.5 into a 6-layer student, giving a 90M generator plus a 20M VAE.

## Key Results

**Speech VAE vs. Mimi.** The 32-dim VAE roughly matches 8-RVQ Mimi on MOSNet (3.15 vs. 3.11) and improves ABX (8.1% vs. 9.4%), PESQ (2.42 vs. 2.13), STOI (0.90 vs. 0.87), and a MUSHRA acoustic-quality score (66.0 vs. 57.7) *(Table 1)*.

**Speech continuation.** Against an RQ-Transformer over 8-RVQ Mimi tokens with the same setup, the one-step consistency CALM at temperature 0.8 improves perplexity (23.8 vs. 26.8), VERT diversity (31.2 vs. 33.1), human acoustic quality on a 1-5 scale (3.45 vs. 2.75), and meaningfulness Elo (2023 vs. 1870). The head samples 12.3 times faster, but because the backbone dominates, end-to-end inference for 30 s is only 1.3 times faster *(Table 2)*. Gaussian temperature matters: at temperature 1.0 CALM acoustic quality falls to 2.82. The authors expected parity on meaningfulness given the text stream and offer only a speculative explanation for the gain. The only baseline here is the authors' own RQ-Transformer.

**TTS on LibriSpeech test-clean (F5-TTS protocol).** The 313M CALM with one-step LSD and CFG 1.5 reaches WER 1.81 and CER 0.57, below F5-TTS (WER 2.42, 32 NFE), DSM (1.95), and DiTAR (2.39, reported from its paper), and a MUSHRA-style acoustic quality of 61.1 vs. 54.7 for F5-TTS and 60.2 for DSM. WavLM speaker similarity is the weakest of the group (0.52 vs. 0.66-0.67 for F5-TTS, DSM, DiTAR); passing the reference itself through the VAE drops similarity to 0.57, which the authors read as a measurement artefact. A human speaker-similarity Elo places CALM (1966) slightly above the ground-truth reference (1953) but below F5-TTS (2032) and DSM (2112) *(§5.2, Table 3)*. In the TTS ablation, plain consistency at the same scale is clearly worse than LSD (MUSHRA 46.8 vs. 61.1, SIM 0.40 vs. 0.52), and removing latent CFG raises WER from 1.81 to 2.39 *(§E.6, Table 10)*.

**Pocket TTS.** On test-clean with Adobe-enhanced prompts, the 100M Pocket TTS reaches WER 1.84 (tied with DSM, below F5-TTS 2.21 and Kokoro 1.93), an audio-quality Elo of 2016 (above F5-TTS and DSM, below Chatterbox Turbo at 2055), and the lowest speaker-similarity Elo among voice-cloning systems (1898), while being the only voice-cloning model in the comparison that runs faster than real time on a laptop CPU *(§G, Table 13)*.

**Music.** On 30 s music continuation with a shared 1.35B backbone, one-step consistency CALM beats the 32-RVQ RQ-Transformer on FAD (0.83 vs. 1.06) and matches it on human ratings while being 2.2 times faster overall; a 100-step TrigFlow head is best on quality but is three times slower than the baseline *(Table 4)*. Removing the short-context Transformer, noise injection, and head batch multiplier together (approximately MAR) raises FAD from 0.93 to 8.38 *(§E.1, Table 6)*. Below 10 steps, where real-time factor stays under 1, only consistency produces usable music; TrigFlow FAD is 28.83 at 4 steps *(§E.4, Table 8)*.

## Novelty Assessment

Continuous autoregressive TTS was not new (MELLE, SALAD, DiTAR precede it), and the backbone-plus-MLP-head structure is MAR. The genuine contributions are the one-step consistency/LSD head as a direct replacement for an RQ-Transformer depth model, the combination of noisy long context with a clean short-context Transformer (with strong ablation support, though only on music), and a set of practical inference tools for one-step heads: Gaussian temperature, latent CFG (introduced earlier in SoundReactor, as the paper acknowledges), and latent distillation of the guided backbone into a smaller student. The most useful aspect of the evaluation is the matched comparison: discrete and continuous variants share backbones, data, and setup for speech continuation and music, which isolates the representation choice better than most cross-paper comparisons. The TTS comparison is weaker: DiTAR and SALAD numbers are copied from their papers, speaker similarity lags, and the speech-side models drop two of the paper's headline architectural components, so the speech gains come mainly from the continuous VAE, consistency/LSD head, temperature, and CFG.

## Field Significance

Moderate — Demonstrates under matched backbones that a continuous VAE latent with a one-step consistency or LSD head can outperform RVQ token prediction on both quality and sampling cost for speech continuation and music, not only for TTS, and shows that the approach compresses to a CPU-real-time 100M TTS model with open code. It provides a concrete recipe (noise injection, short context, latent CFG, Gaussian temperature, latent distillation) for making continuous audio LMs stable and controllable. Continuous autoregressive TTS itself predates it (MELLE, DiTAR), and its TTS speaker similarity trails the strongest baselines, so the contribution is the domain-general comparison and recipe rather than a new TTS capability.

## Claims

- **supports:** Autoregressive models over continuous VAE latents can match or exceed RVQ-token language models in perceived quality while reducing per-frame sampling cost, when a single-step generative head replaces the depth-wise token sampler.
  > *Evidence:* With the same backbone and data, a one-step consistency head outperforms an 8-RVQ RQ-Transformer on perplexity, human acoustic quality (3.45 vs. 2.75), and meaningfulness Elo for speech continuation, with a 12.3x faster sampler; on music it improves FAD (0.83 vs. 1.06) at 2.2x overall speedup. *(§5.1, Table 2; §5.3, Table 4)*
- **complicates:** Faster sampling heads yield limited end-to-end speedups when the autoregressive backbone dominates inference cost.
  > *Evidence:* The consistency head is 12.3x faster than the RQ-Transformer for speech continuation, but full 30 s generation is only 1.3x faster because the sampler accounted for 26.7% of baseline inference time. *(§5.1, Table 2)*
- **supports:** Continuous autoregressive audio models are vulnerable to error accumulation on long, diverse sequences, and clean local context combined with noise-robust long context mitigates it.
  > *Evidence:* A MAR-like music configuration without noise injection, short-context Transformer, or head batch multiplier reaches FAD 8.38 vs. 0.93 for the full model; removing only the short-context Transformer gives 4.03. Speech models did not benefit from either component. *(§4.3; §E.1, Table 6; §5.1)*
- **refines:** Temperature-style truncation and classifier-free guidance, standard in discrete token LMs, can be transferred to one-step continuous heads by acting on the head's input noise and conditioning latent respectively, with large effects on intelligibility and quality.
  > *Evidence:* Lowering Gaussian temperature to 0.8 raises speech-continuation acoustic quality from 2.82 to 3.45; removing latent CFG in TTS raises WER from 1.81 to 2.39 and lowers MUSHRA from 61.1 to 55.5. *(§4.5, §4.6, Table 2; §E.6, Table 10)*
- **complicates:** Automatic speaker-similarity scores can understate voice preservation for systems built on a lossy latent autoencoder, and need human verification.
  > *Evidence:* CALM TTS scores WavLM SIM 0.52 vs. 0.66-0.67 for baselines, but the ground-truth reference passed through the same VAE only scores 0.57, and a human speaker-similarity Elo rates CALM (1966) above the reference (1953), though still below F5-TTS and DSM. *(§5.2, Table 3)*

## Limitations and Open Questions

- Speaker similarity remains the weak point for TTS: WavLM SIM is lowest among compared systems, the human similarity Elo trails F5-TTS and DSM, and Pocket TTS has the lowest speaker-similarity Elo among voice-cloning systems in Table 13. The VAE-roundtrip explanation is plausible but does not account for the human-rated gap.
- The noise-injection and short-context components, and nearly all architectural ablations (Tables 6-9), are validated only on music; the speech models omit them, so their value for speech is untested beyond the authors' remark that they gave no gain.
- Speech continuation is compared only against the authors' own RQ-Transformer, not against other published speech LMs, and the evaluation data is described only as an English test set.
- TTS baselines DiTAR and SALAD are reported from their papers rather than re-run; Pocket TTS uses Adobe-enhanced prompts, so its numbers are not directly comparable with Table 3.
- Scaling evidence is a single 3B music backbone; the paper leaves a full scaling study, and semantic distillation for the music VAE, to future work. Speech-continuation training data beyond "French and English speech data" is not described.

## Wiki Connections

- [[autoregressive-codec-tts|Autoregressive Codec TTS]] — Continues the codec-free branch of autoregressive TTS, replacing RVQ tokens and the RQ-Transformer with continuous VAE latents and a one-step head.
- [[flow-matching|Flow Matching]] — The TTS head uses Lagrangian Self-Distillation trained partly with a flow-matching loss, and the consistency head uses the TrigFlow formulation, to obtain one-step sampling.
- [[zero-shot-tts|Zero-Shot TTS]] — Evaluates prompt-based voice cloning on LibriSpeech test-clean under the F5-TTS protocol, with strong WER but weaker speaker similarity.
- [[spoken-language-model|Spoken Language Model]] — Adapts the Helium-1 text LM into a continuous-latent speech continuation model with a Moshi-style inner monologue text stream.
- [[neural-codec|Neural Audio Codec]] — Trains a Mimi-derived causal VAE that outperforms 8-RVQ Mimi on ABX, PESQ, STOI, and MUSHRA, arguing for continuous rather than quantized audio representations.
- [[subjective-evaluation|Subjective Evaluation]] — Uses 50-rater listening tests with 1-5 quality ratings, MUSHRA, and Bayesian Bradley-Terry Elo scores from pairwise meaningfulness and speaker-similarity judgments.
- [[2025.acl-long.313|F5-TTS]] — Primary TTS baseline and source of the evaluation protocol; CALM and Pocket TTS achieve lower WER with a single function evaluation.
- [[2509.08753|DSM]] — Kyutai's RVQ-based streaming TTS baseline, strongest on human speaker similarity in both TTS comparisons.
- [[2502.03930|DiTAR]] — Continuous autoregressive TTS with a diffusion-transformer patch head; its patch-level local context parallels CALM's short-context Transformer, and its reported results serve as a baseline.
- [[2407.08551|MELLE]] — Earlier autoregressive TTS without vector quantization on mel-spectrograms, cited as related continuous-domain work.
- [[2410.00037|Moshi]] — Supplies the Mimi codec architecture behind the VAE, the speech training recipe, and the inner monologue with acoustic delay used for speech continuation.
- [[2511.05516|Ming-UniAudio]] — Cited as evidence that continuous speech language models scale to large mixture-of-experts LLMs.
- [[2209.03143|AudioLM]] — Representative discrete-token audio LM whose flattened RVQ hierarchy motivates the cost argument for continuous modelling.
- [[2407.05361|Emilia]] — One of the eight public corpora in the 88k-hour TTS training mix.
