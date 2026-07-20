---
id: "interspeech-2025-1747"
title: "FasterVoiceGrad: Faster One-step Diffusion-Based Voice Conversion with Adversarial Diffusion Conversion Distillation"
authors: ["Takuhiro Kaneko", "Hirokazu Kameoka", "Kou Tanaka", "Yuto Kondo"]
organization: "NTT, Inc."
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: "2025-08-17"
ingested_date: "2026-07-03"
task: ["VC"]
architecture: ["diffusion", "GAN"]
conditioning: ["zero-shot", "speaker-conditioned"]
training: ["distillation"]
model_size: "not reported"
codec_used: "none"
datasets_train: ["VCTK", "LibriTTS"]
datasets_eval: ["VCTK", "LibriTTS"]
metrics:
  - name: UTMOS
    value: 4.03
    system: proposed
    testset: VCTK
  - name: DNSMOS
    value: 3.82
    system: proposed
    testset: VCTK
  - name: CER
    value: 1.2
    system: proposed
    testset: VCTK
  - name: SECS
    value: 0.853
    system: proposed
    testset: VCTK
  - name: MOS
    value: 3.81
    system: proposed
    testset: VCTK
code_available: null
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/kaneko25_interspeech.html"
related_concepts: [voice-conversion, diffusion-tts, gan-vocoder, speaker-adaptation, subjective-evaluation, evaluation-metrics]
related_papers: []
field_significance:
  level: "moderate"
  type: ["architectural-novelty"]
generation:
  date: "2026-07-03"
  agent: speech-generation-ingest-agent
  model: claude-sonnet-4-6
  commit: "49d73f8"
---

> [!abstract] Interspeech · 2025 · Conference
> **Kaneko et al.** (NTT, Inc.) · [→ Paper](https://www.isca-archive.org/interspeech_2025/kaneko25_interspeech.html) · Demo: ? · Code: ?
>
> FasterVoiceGrad simultaneously distils both the main reverse diffusion module and content encoder in a diffusion-based voice conversion system using a novel adversarial diffusion conversion distillation (ADCD) method, achieving 6.6-6.9x GPU speedup over its predecessor FastVoiceGrad while maintaining competitive conversion quality.

## Problem

Multi-step diffusion-based VC models such as VoiceGrad achieve strong speech quality and speaker similarity but are slow owing to iterative sampling. FastVoiceGrad addressed the iterative sampling cost by distilling VoiceGrad into a one-step model, but left the content encoder intact. That encoder, a Conformer-based bottleneck encoder (BNE) required for content-speaker disentanglement, is computationally expensive and becomes the dominant inference bottleneck once the diffusion sampling steps are eliminated. Replacing it with a lighter module using the same reconstruction-based distillation that FastVoiceGrad uses fails because the trainable encoder can trivially learn an identity mapping through the reconstruction path, bypassing the diffusion process entirely.

## Method

FasterVoiceGrad proposes adversarial diffusion conversion distillation (ADCD), which simultaneously distils both the main reverse diffusion module and the content encoder by conducting distillation in the conversion process rather than the reconstruction process. This change forces the content encoder to learn a useful intermediate representation, because the conversion target (using a different speaker embedding) cannot be reconstructed via an identity path.

The frozen Conformer-based BNE from FastVoiceGrad is replaced with a trainable three-layer 1D CNN with 512 hidden channels, instance normalisation, and weight normalisation. The main diffusion module retains the U-Net architecture (12-layer 1D CNN, 512 hidden channels, GLU, two down/up-sampling layers). A pre-trained HiFi-GAN V1 is used as the vocoder. Multi-period and multi-resolution discriminators provide adversarial supervision.

The ADCD training objective combines three mechanisms on top of conversion-form adversarial and score distillation losses. Reconversion score distillation applies a score distillation loss to a twice-converted mel-spectrogram: the one-step converted output is converted a second time (with a fresh target speaker embedding), and the resulting mel-spectrogram is required to match the teacher model's output with the original content embedding. This round-trip enforces content fidelity without requiring parallel data. Inverse score distillation pushes the converted output away from non-target speakers' denoised mel-spectrograms contrastively, sharpening speaker identity without direct target-speaker supervision.

![Comparison of training processes between (a) FastVoiceGrad (previous) and (b) FasterVoiceGrad (proposed). FasterVoiceGrad distills not only the main reverse diffusion module but also the content encoder in the conversion process, preventing identity-mapping collapse.](assets/interspeech-2025-1747/figure-1.png)

## Key Results

On VCTK (unseen-to-unseen, one-shot VC), FasterVoiceGrad outperforms or matches FastVoiceGrad on all four objective metrics: UTMOS 4.03 vs. 3.96, DNSMOS 3.82 vs. 3.77, CER 1.2 vs. 1.3, and SECS 0.853 vs. 0.847. GPU RTF improves from 0.00560 to 0.00085 (6.6x), and CPU RTF from 0.1347 to 0.0747 (1.8x) (Table 3). On LibriTTS, similar trends hold with a 6.9x GPU speedup (Table 5).

Subjective MOS tests (11 participants, 90 speaker/sentence pairs) show FasterVoiceGrad exceeds FastVoiceGrad on speech quality (qMOS 3.81 vs. 3.60) but falls short on speaker similarity (sMOS 2.66 vs. 2.84). The authors attribute the sMOS gap to FasterVoiceGrad's CNN content encoder not explicitly removing source speaker characteristics, leaving residual source voice features that are perceptible to humans but not detected by the speaker-encoder-based SECS metric (Table 4).

Ablation confirms the contribution of each ADCD component. Using FastVoiceGrad's reconstruction-based distillation with the trainable CNN baseline yields UTMOS 3.45 and SECS 0.718. Switching to conversion-based adversarial and score distillation raises UTMOS to 3.98 and SECS to 0.847. Reconversion score distillation further improves UTMOS to 4.03. Inverse score distillation recovers SECS to 0.853 while maintaining content fidelity (Table 1). Direct distillation (aligning the CNN's output to BNE's output within reconstruction training) performs worse than ADCD at all model sizes, most severely at one layer (UTMOS 3.35 vs. 4.01) (Table 2).

## Novelty Assessment

The core novelty is the insight that distillation must be performed in the conversion process, not the reconstruction process, when the content encoder is simultaneously being distilled. This reasoning is clear and the failure mode it addresses (identity-mapping collapse through a trainable content encoder) is well-motivated. The two auxiliary losses, reconversion score distillation and inverse score distillation, are original training objectives specific to this setting. The architecture itself (U-Net diffusion module, CNN content encoder, HiFi-GAN vocoder) recombines known components; the training procedure is the contribution.

The work is narrowly scoped to follow-up on FastVoiceGrad within the same NTT research group. It does not evaluate against a broader range of VC baselines beyond DiffVC-30 as an anchor, and relies on automated MOS predictors (UTMOS, DNSMOS) as primary objective metrics. The subjective tests reveal a genuine quality-similarity trade-off that the objective metrics do not capture fully, which is an honest finding but also a limitation of the proposed approach.

## Field Significance

Moderate — FasterVoiceGrad advances the line of distillation-based diffusion VC by demonstrating that the content encoder is not a fixed cost once iterative sampling is eliminated, and that both components can be jointly distilled without quality loss. The conversion-process distillation strategy and the reconversion/inverse score distillation losses are principled additions to the distillation toolbox for voice conversion. The trade-off between automatic speaker similarity and perceptual sMOS is a notable finding for practitioners choosing evaluation strategies.

## Claims

- **supports:** Performing knowledge distillation in the conversion process rather than the reconstruction process is necessary to avoid identity-mapping collapse when the content encoder is simultaneously trainable.
  > *Evidence:* Baseline reconstruction-distillation with trainable CNN (FastVoiceGrad + pϕ) achieves UTMOS 3.45 and SECS 0.718; switching to conversion-based ADCD raises these to 4.03 and 0.853, respectively. Direct distillation (an alternative that avoids conversion-process training) degrades to UTMOS 3.35 with a 1-layer encoder, confirming the mechanism failure. *(§4.2, Table 1, Table 2)*

- **supports:** Simultaneously distilling both the diffusion backbone and the content encoder of a one-step diffusion VC model can reduce inference cost substantially with little or no quality degradation relative to distilling only the diffusion module.
  > *Evidence:* FasterVoiceGrad (joint distillation) achieves UTMOS 4.03 and SECS 0.853 vs. FastVoiceGrad's 3.96 and 0.847, while delivering 6.6x GPU RTF and 1.8x CPU RTF improvement across both VCTK and LibriTTS evaluations. *(§4.3, Table 3, Table 5)*

- **complicates:** Neural speaker-encoder-based similarity metrics may not reflect perceptual speaker identity as judged by human listeners, particularly when source speaker residuals remain after conversion.
  > *Evidence:* FasterVoiceGrad scores higher SECS (0.853 vs. 0.847) than FastVoiceGrad but lower sMOS (2.66 vs. 2.84); the authors attribute the discrepancy to the CNN content encoder not explicitly removing source speaker characteristics, which humans but not speaker encoders detect. *(§4.3, Table 3, Table 4)*

- **refines:** Contrastive speaker emphasis via inverse score distillation mitigates the trade-off between content preservation and speaker conversion when joint distillation is used.
  > *Evidence:* Adding inverse score distillation on top of reconversion-based distillation improves SECS from 0.844 to 0.853 without degrading CER (1.2) or UTMOS (4.03), recovering the speaker similarity lost by reconversion's content preservation bias. *(§4.2, Table 1)*

## Limitations and Open Questions

> [!warning]
> The CNN content encoder does not explicitly separate source speaker identity from linguistic content. Subjective sMOS scores (2.66) are measurably lower than FastVoiceGrad's (2.84), suggesting residual source voice characteristics persist and limit perceptual speaker conversion quality despite competitive automatic metrics.

Subjective evaluation involved only 11 participants and 90 speaker/sentence pairs, which constrains the statistical power of the sMOS comparisons. The GPU speedup (6.6x) is larger than the CPU speedup (1.8x), reflecting architectural factors that may affect deployment on edge hardware differently. The study does not evaluate against modern flow-matching or large-scale VC baselines, leaving the system's position in the broader landscape uncharacterised. Future directions suggested include accent conversion and real-time VC applications.

## Wiki Connections

- [[voice-conversion|Voice Conversion]] — this paper proposes a new distillation-based method for one-shot any-to-any VC that targets inference efficiency without sacrificing conversion quality.
- [[diffusion-tts|Diffusion TTS]] — FasterVoiceGrad builds directly on diffusion-based VC (VoiceGrad, FastVoiceGrad) and extends the one-step distillation paradigm to jointly compress both the diffusion module and the content encoder.
- [[gan-vocoder|GAN Vocoder]] — the system uses a pre-trained HiFi-GAN V1 vocoder and incorporates multi-period and multi-resolution discriminators as part of the ADCD adversarial training objective.
- [[speaker-adaptation|Speaker Adaptation]] — the one-shot any-to-any VC setting evaluates conversion to unseen target speakers using only a single reference utterance, testing adaptation to novel speaker identities at inference time.
- [[subjective-evaluation|Subjective Evaluation]] — MOS listening tests with human raters (qMOS and sMOS) reveal a quality-similarity trade-off not captured by automatic metrics, highlighting the importance of subjective evaluation for VC systems.
- [[evaluation-metrics|Evaluation Metrics]] — the paper uses UTMOS and DNSMOS as predicted MOS proxies alongside human MOS, CER (via Whisper-large-v3), and speaker encoder cosine similarity (SECS), comparing how automatic and perceptual metrics diverge on speaker similarity.
