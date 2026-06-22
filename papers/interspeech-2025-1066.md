---
id: "interspeech-2025-1066"
title: "Score-Based Training for Energy-Based TTS Models"
authors: ["Wanli Sun", "Anton Ragni"]
organization: University of Sheffield
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-06-03
task: [TTS]
architecture: [diffusion]
conditioning: [text-conditioned]
training: [supervised]
model_size: "not reported"
codec_used: "none"
datasets_train: ["LJSpeech"]
datasets_eval: ["LJSpeech"]
metrics:
  - name: MOS
    value: 3.95
    system: Delta (1 step)
    testset: LJSpeech
  - name: MOS
    value: 3.90
    system: SSM (1 step)
    testset: LJSpeech
  - name: MOS
    value: 3.74
    system: Tacotron 2
    testset: LJSpeech
  - name: UTMOS
    value: 3.718
    system: SSM+Delta combined
    testset: LJSpeech (partial val)
  - name: UTMOS
    value: 2.959
    system: Delta only
    testset: LJSpeech (partial val)
code_available: null
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/sun25f_interspeech.html"
related_concepts: ["diffusion-tts", "flow-matching", "evaluation-metrics", "subjective-evaluation"]
related_papers: []
field_significance:
  level: moderate
  type: [architectural-novelty, conceptual-contribution]
generation:
  date: 2026-06-22
  agent: speech-generation-review-agent
  model: claude-sonnet-4-6
  commit: "24ea64d"
---

> [!abstract] Interspeech · 2025 · Conference
> **Sun and Ragni** (University of Sheffield) · [→ Paper](https://www.isca-archive.org/interspeech_2025/sun25f_interspeech.html) · Demo: ? · Code: ?
>
> A new score-based training criterion for energy-based TTS models — delta loss — that encourages scores pointing toward reference speech along linear paths, enabling high-quality synthesis with a single inference step and revealing a formal connection to flow matching objectives.

## Problem

Energy-based models (EBMs) for TTS have relied on noise contrastive estimation (NCE) to circumvent their intractable normalisation terms, but NCE is sensitive to the quality of negative samples and requires elaborate sampling schemes. Score matching alternatives — including sliced score matching (SSM) — have not been applied to EBMs in TTS and ignore the shape of the underlying log-likelihood, making them poorly suited for the Langevin-style first-order MCMC inference that EBMs use. No prior work had studied whether score-based training objectives could simultaneously reduce this inference-unfriendliness and outperform NCE.

## Method

The paper works with mel-spectrogram-level EBMs whose log-likelihood is defined as a Boltzmann distribution with energy function E_θ(x, Y) and an intractable normaliser. Inference proceeds by iterative Langevin MCMC, initialised from hypotheses produced by a pretrained Tacotron 2 acoustic model.

Two score-based training objectives are investigated. The first is sliced score matching (SSM), previously used in image and imitation-learning EBMs but not in TTS. SSM replaces the expensive full Hessian trace in ordinary score matching with projections onto random directions, requiring only two gradient operations per step. The second is a new objective called delta loss. Rather than minimising the distance between model scores and data scores regardless of their geometry, delta loss trains the model to point noisy-sample scores in the direction of (Y^+ - Y^-) — the vector from the noisy hypothesis to the reference. This ensures scores lie on linear paths between hypothesis and reference, enabling the MCMC update to reach the reference in approximately one step with the right learning rate.

The paper shows that delta loss in equation (9) is formally equivalent to the flow matching objective (equation 10) when t=0, connecting an EBM training criterion to the continuously-interpolated vector field used in rectified-flow-style TTS systems, despite arising from different motivations.

The score models use a U-Net architecture identical to Matcha-TTS, with the time/iteration conditioning removed to yield time-independent scores. Models are trained for 50K steps (Adam, lr 1e-4, batch 10) on a single NVIDIA 3090. HiFi-GAN serves as the vocoder.

## Key Results

On LJSpeech objective evaluation (full validation set, 100 inference steps), SSM and SSM+Delta achieve MCD 5.312 and SpeechBERTScore 0.905 compared to Tacotron 2 at MCD 5.770 / S-BERT 0.882, showing clear gains. Delta alone scores MCD 5.399 but lags SSM on UTMOSv2 (2.959 vs. 3.710). Combining SSM and delta losses provides marginal improvement at best.

Critically, an inference-step analysis (Table 2 and Table 3) reveals that both SSM and delta loss EBMs obtain their best results after just 1 inference step rather than the 100 steps typical of NCE-trained EBMs. Delta loss performance degrades markedly at 100 steps (UTMOSv2 drops from 3.703 to 2.937), while SSM is more stable but also peaks early.

In subjective MOS (100 test clips, 3 listeners via Amazon Mechanical Turk), Delta at 1 step achieves 3.95 ± 0.05 and SSM at 1 step achieves 3.90 ± 0.06, both outperforming Tacotron 2 (3.74 ± 0.08). Ground truth (vocoded) scores 4.15. Notably, the detailed score breakdown shows that delta loss produces far fewer low ratings (1–3) and more high ratings (4–5) than SSM, despite comparable mean MOS.

NCE-trained EBMs (from prior work, [3]) score MCD 5.762, S-BERT 0.875, UTMOSv2 2.572 at 100 steps — substantially worse than either score-based approach at 1 step.

> [!note]
> The MOS study uses only 3 listeners and 100 test clips. UTMOSv2 and MOS appear inconsistent for delta loss (low UTMOSv2 but competitive MOS), suggesting the automatic predictor may not track the specific quality dimensions where delta loss excels.

## Novelty Assessment

The delta loss is a genuinely new training objective. Its derivation — enforcing linearity of score paths rather than minimising distributional score divergence — is conceptually clean and the link to flow matching is a non-obvious theoretical finding. The first application of SSM to EBMs in TTS is a solid empirical contribution, though SSM itself is a known technique.

The practical gain is also real: reducing EBM inference from 100 MCMC steps to 1 step changes the computational profile of EBM-based TTS fundamentally. The limitation is that experiments are conducted only on single-speaker LJSpeech with a small listener panel, and it is unclear whether the approach scales to multi-speaker or neural-codec-based architectures. The backbone (Tacotron 2 + mel + HiFi-GAN) is dated by 2025 standards, which limits the claim's direct practical relevance to modern pipelines.

## Field Significance

Moderate — This paper introduces a well-motivated training criterion for EBMs and provides the first comparison of score-matching and NCE training in TTS. Its most significant contribution is the formal connection between delta loss and flow matching, which positions EBMs as a theoretical bridge between energy-based and flow-based synthesis. In practical terms, the one-step inference result is notable, though the single-speaker evaluation limits generalizability.

## Claims

- Score-based training objectives for energy-based models can reduce TTS inference to a single step, bypassing the many-iteration MCMC requirement of NCE-trained EBMs. *(§4.2, Table 2, Table 3)*
- The choice of training objective for score-based generative models materially affects the geometry of learned score functions and their suitability for first-order iterative inference. *(§3.2)*
- Sliced score matching and flow matching objectives are formally equivalent at a single interpolation timestep, suggesting a deeper structural connection between energy-based and flow-based TTS training regimes. *(§3.3)*
- Automatic MOS predictors and subjective listening scores can diverge for EBM-based TTS systems, limiting the reliability of UTMOSv2 as a sole quality proxy in non-standard synthesis paradigms. *(§4.2, §4.3)*

## Limitations and Open Questions

> [!warning]
> The subjective evaluation uses only 3 listeners and 100 test clips on single-speaker LJSpeech. The statistical power is insufficient to draw strong conclusions about relative naturalness, and the listener pool is too small to assess inter-rater reliability. Results should be treated as indicative rather than conclusive.

The experimental setup uses a Tacotron 2 backbone (mel spectrograms, HiFi-GAN vocoder) that is substantially behind current practice; it is open whether delta loss advantages persist in codec-based or end-to-end architectures. The inconsistency between UTMOSv2 and MOS for delta loss is unexplained and warrants further investigation. The delta loss formulation requires noisy-sample hypotheses at training time, so training cost depends on the quality and source of those hypotheses. Multi-speaker generalisation is untested.

## Wiki Connections

- [[diffusion-tts]]
- [[flow-matching]]
- [[evaluation-metrics]]
- [[subjective-evaluation]]
