---
id: "interspeech-2025-1229"
title: "E2E-BPVC: End-to-End Background-Preserving Voice Conversion via In-Context Learning"
authors: ["Yihan Liu", "Zhengyang Chen", "Leying Zhang", "Yanmin Qian"]
organization: null
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-07-03
task: ["VC"]
architecture: ["flow-matching"]
conditioning: ["zero-shot", "prompt-conditioned"]
training: ["supervised"]
model_size: "not reported"
codec_used: "none"
datasets_train: ["LibriTTS (train-clean-100, train-clean-360, train-other-500)", "MUSAN (noise)", "MusicCaps", "MUSDB18"]
datasets_eval: ["LibriTTS (test-clean)"]
metrics:
  - name: SMOS
    value: 4.02
    system: E2E-BPVC
    testset: LibriTTS test-clean (w/ background, SS-MOS)
  - name: MOS
    value: 3.92
    system: E2E-BPVC
    testset: LibriTTS test-clean (naturalness, w/ background)
  - name: CER
    value: 6.95
    system: E2E-BPVC
    testset: LibriTTS test-clean (noise background)
code_available: null
demo_available: true
url: "https://www.isca-archive.org/interspeech_2025/liu25j_interspeech.html"
related_concepts: [voice-conversion, flow-matching, zero-shot-tts, self-supervised-speech, subjective-evaluation, evaluation-metrics]
related_papers: ["2406.02430"]
field_significance:
  level: "moderate"
  type: ["engineering-integration"]
generation:
  date: 2026-07-03
  agent: speech-generation-ingest-agent
  model: claude-sonnet-4-6
  commit: "f9e71e3"
---

> [!abstract] Interspeech · 2025 · Conference
> **Liu et al.** · [→ Paper](https://www.isca-archive.org/interspeech_2025/liu25j_interspeech.html) · Demo: ✓ · Code: ?
>
> The first end-to-end system for background-preserving voice conversion that converts speaker timbre while retaining background sounds through in-context learning, eliminating the need for a separate speech separation or denoising module.

## Problem

Voice conversion systems are trained on clean speech and perform poorly when source audio contains background sounds such as noise or music. Prior work on background-preserving voice conversion (BPVC) relies on a three-step cascade: a denoising or separation module extracts background from speech, voice conversion is applied to the cleaned signal, and the background is then recombined with the converted output. This approach introduces additional system complexity, potential distortion from the separation step, and energy mismatches across stages that degrade overall audio quality.

## Method

E2E-BPVC frames background-preserving voice conversion as a single in-context learning (ICL) task. Rather than explicitly separating speech from background, the model learns to simultaneously transform speaker timbre and reproduce the source background by conditioning on multiple correlated input streams within one generative model.

The generative backbone is a flow-matching model built on an 8-layer Transformer encoder with 768-dimensional hidden size, trained to predict mel spectrograms via the ODE formulation of flow matching. Content information is disentangled from speaker identity and background using semantic tokens extracted from the 9th layer of a pretrained WavLM model, clustered into 2000 k-means categories. The k-means model is trained on noisy speech to improve robustness to background variation. BigVGAN serves as the vocoder for both encoding input audio into mel spectrograms and decoding predicted spectrograms back to waveforms.

Training uses a mask-and-reconstruct task to elicit ICL capability. Given two speech segments from a target speaker A and a source speaker B (both clean), both are mixed with the same randomly sampled background sound to produce noisy pairs. The model receives: the partially masked mel spectrogram of A's clean speech (providing timbre context from unmasked regions), semantic tokens from A's noisy speech (providing content), and the full mel spectrogram of B's noisy speech (providing background). The objective is to reconstruct A's noisy mel spectrogram, forcing the model to simultaneously extract timbre from the unmasked reference context, content from semantic tokens, and background from the noisy source.

At inference, a 10-second reference clip from the target speaker and the noisy source audio are aligned to a shared temporal layout via zero-padding and repetition. The model integrates the flow ODE over 32 steps to produce the converted mel spectrogram. Training data combines LibriTTS (train-clean-100, train-clean-360, train-other-500, totalling 2,311 speakers) with background sounds from MUSAN, MusicCaps, and MUSDB18 at SNRs of 5 to 15 dB (noise) and 10 to 15 dB (music).

## Key Results

Subjective evaluations with 12 human raters compare E2E-BPVC against two baselines: ICL-VC (a clean-output-only ICL voice conversion system) and Denoise-VC II (ICL-VC preceded by BSRNN denoising, with background recombined post-conversion). Tests use LibriTTS test-clean mixed with background at 7 dB and 12 dB SNR, across noise and music background types (Table 2):

- Speaker similarity MOS (SS-MOS): E2E-BPVC 4.02, Denoise-VC II 4.07, ICL-VC 4.01
- Naturalness MOS (N-MOS): E2E-BPVC 3.92, Denoise-VC II 4.00, ICL-VC 3.91
- Background similarity MOS (BS-MOS): E2E-BPVC 4.60, Denoise-VC II 4.65, ICL-VC 0.70

E2E-BPVC achieves background preservation comparable to Denoise-VC II (BS-MOS 4.60 vs. 4.65) while using a single model with no denoising component. ICL-VC, which always outputs clean speech, scores 0.70 on BS-MOS, confirming that background preservation requires an explicit design choice.

On objective metrics (Table 1), E2E-BPVC is modestly weaker than baselines on speaker similarity (SIM) and character error rate (CER) in clean conditions (SIM 0.849 vs. ICL-VC 0.873, CER 3.18 vs. 2.37). The authors note that these metrics are degraded by background sound and do not fully reflect background-preserving system capabilities.

The ablation confirms that WavLM outperforms HuBERT as the semantic token backbone in noisy conditions (CER 7.99 vs. 10.27), and that k-means trained on noisy speech outperforms clean-data k-means training (CER 7.99 vs. 9.22, Table 3).

## Novelty Assessment

The contribution is engineering integration: ICL, flow matching, and WavLM semantic token extraction are all established techniques, applied here to a new task formulation (BPVC without a denoising step). The mask-and-reconstruct training design that conditions the model on masked reference mel, semantic tokens, and noisy source mel simultaneously is a thoughtful task-specific construction, but the components themselves are not novel. The claim of being end-to-end is accurate in the sense that no separate denoising module is required at inference, though the system still depends on a pre-trained WavLM backbone and a BigVGAN vocoder.

The system reaches parity with, rather than improvement over, the cascade baseline on background preservation. The evaluation relies on synthetic noise conditions and a single clean speech corpus, so performance on real-world recordings with natural backgrounds is unknown.

## Field Significance

Moderate — the paper provides proof-of-concept that ICL with flow matching can absorb background preservation into the core voice conversion model, eliminating the denoising cascade. The result is practically relevant for latency-sensitive and distortion-sensitive applications such as real-time voice modification, but the evaluation scope (synthetic noise, LibriTTS only, two baselines from the same model family) limits how far the finding can be generalised. The task of background-preserving VC itself remains a niche subfield.

## Claims

- **supports:** In-context learning with flow matching can enable voice conversion systems to simultaneously transform speaker timbre and preserve background sounds without an explicit separation step.
  > *Evidence:* E2E-BPVC achieves BS-MOS 4.60 and SS-MOS 4.02 comparable to Denoise-VC II (BS-MOS 4.65, SS-MOS 4.07) using a single model without a denoising module, validated by 12 human raters on LibriTTS test-clean with synthetic noise and music backgrounds at 7 and 12 dB SNR. *(§4.2, Table 2)*

- **complicates:** Standard voice conversion evaluation metrics (speaker similarity, character error rate, speech quality) are insufficient for assessing systems that operate on speech with background sounds.
  > *Evidence:* ECAPA-TDNN speaker similarity and ASR-based CER are degraded by background sound in both source and converted audio, causing clean-output systems to appear relatively stronger on objective metrics despite failing entirely on background preservation (ICL-VC BS-MOS 0.70). The authors explicitly note that objective metrics "do not adequately reflect the capabilities" of background-preserving systems. *(§4.1, Table 1)*

- **supports:** Noise-robust self-supervised speech representations improve content disentanglement in voice conversion systems trained on speech with background sounds.
  > *Evidence:* Replacing HuBERT with WavLM as the semantic token backbone reduces CER from 10.27 to 7.99 under noisy evaluation conditions, and training k-means on noisy speech (rather than clean speech) further reduces CER to 7.99 versus 9.22, demonstrating that noise robustness at the representation level propagates to improved content disentanglement. *(§4.3, Table 3)*

- **complicates:** Designing a single voice conversion model to handle background preservation introduces a competing objective that slightly degrades clean-speech conversion quality relative to a clean-speech-only system.
  > *Evidence:* E2E-BPVC achieves SIM 0.849 and CER 3.18 on clean-speech conversion, modestly below ICL-VC (SIM 0.873, CER 2.37), reflecting a trade-off introduced by the shared background-preservation training objective. *(§4.1, Table 1)*

## Limitations and Open Questions

The evaluation relies exclusively on LibriTTS with synthetically added noise and music at controlled SNRs (7 dB and 12 dB), without testing on real-world recordings where backgrounds are natural, non-stationary, or below 7 dB SNR. The comparison set is limited to two baselines sharing the same ICL-VC foundation, so performance relative to independent VC architectures is unknown. The paper also acknowledges that the current experiments use small-scale training data, and that scaling up is needed to improve robustness. The model offers no controllability over whether to preserve or suppress background sounds, though the authors identify this as future work.

## Wiki Connections

- [[voice-conversion|Voice Conversion]] — proposes the first end-to-end approach to background-preserving voice conversion, eliminating the cascade of separate denoising and VC modules.
- [[flow-matching|Flow Matching]] — uses flow matching as the generative backbone for mel-spectrogram prediction in the single-model BPVC framework.
- [[zero-shot-tts|Zero-Shot TTS]] — achieves zero-shot voice conversion using a 10-second reference clip from the target speaker without per-speaker fine-tuning or pre-enrolled speaker embeddings.
- [[self-supervised-speech|Self-Supervised Speech]] — relies on WavLM intermediate representations with k-means clustering as the content disentanglement mechanism; the noise robustness of WavLM over HuBERT is central to the ablation findings.
- [[subjective-evaluation|Subjective Evaluation]] — introduces background similarity MOS (BS-MOS) as a dedicated subjective metric alongside SS-MOS and N-MOS to evaluate background preservation quality with 12 human raters.
- [[evaluation-metrics|Evaluation Metrics]] — argues that conventional VC metrics (speaker similarity, CER, NISQA) are unreliable for background-inclusive systems, motivating background-specific evaluation protocols.
- [[2406.02430|Seed-TTS]] — cited as a high-quality versatile speech generation system representing the broader ICL-based generation line that motivates the approach.
