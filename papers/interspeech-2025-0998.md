---
id: "interspeech-2025-0998"
title: "Voice-ENHANCE: Speech Restoration using a Diffusion-based Voice Conversion Framework"
authors: ["Kyungguen Byun", "Jason Filos", "Erik Visser", "Sunkuk Moon"]
organization: "Qualcomm Technologies Inc."
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-07-02
task: [VC]
architecture: [diffusion, GAN]
conditioning: [speaker-conditioned]
training: [supervised, self-supervised]
model_size: "209M params (GSR 40M + VC 169M)"
codec_used: "none"
datasets_train: ["proprietary speech enhancement dataset (1750 hours)", "LibriTTS (585 hours)"]
datasets_eval: ["VCTK-DEMAND", "UNIVERSE"]
metrics:
  - name: UTMOS
    value: 4.12
    system: proposed (GSR+VC)
    testset: VCTK-DEMAND
  - name: UTMOS
    value: 4.10
    system: proposed (GSR+VC)
    testset: UNIVERSE
  - name: DNSMOS
    value: 3.31
    system: proposed (GSR+VC)
    testset: VCTK-DEMAND
  - name: DNSMOS
    value: 3.26
    system: proposed (GSR+VC)
    testset: UNIVERSE
code_available: false
demo_available: false
url: "https://www.isca-archive.org/interspeech_2025/byun25_interspeech.html"
related_concepts: ["voice-conversion", "diffusion-tts", "self-supervised-speech", "gan-vocoder", "evaluation-metrics"]
related_papers: []
field_significance:
  level: moderate
  type: [engineering-integration]
generation:
  date: 2026-07-02
  agent: speech-generation-ingest-agent
  model: claude-sonnet-4-6
  commit: "d8ecfd4"
---

> [!abstract] Interspeech · 2025 · Conference
> **Byun et al.** (Qualcomm Technologies Inc.) · [→ Paper](https://www.isca-archive.org/interspeech_2025/byun25_interspeech.html) · Demo: ✗ · Code: ✗
>
> A two-stage speech restoration framework that chains a speaker-agnostic generative speech restoration model with a diffusion-based voice conversion module guided by clean speaker embeddings, achieving objective quality scores competitive with larger state-of-the-art systems.

## Problem

VC models trained on clean data are vulnerable to degraded input conditions: noise, reverberation, clipping, packet loss, and codec artefacts all impair the content features that VC relies on. At the same time, conventional masking-based speech enhancement struggles in low-SNR scenarios where speech and noise overlap spectrally, producing muffled or choppy output. No prior system had cleanly integrated a speaker-agnostic, multi-distortion restoration front-end with a speaker-guided generative VC back-end to achieve studio-level quality without requiring transcript data.

## Method

Voice-ENHANCE is a two-stage pipeline. In the first stage, a Generative Speech Restoration (GSR) module processes the degraded input regardless of who is speaking. GSR is based on the ResU-Net analysis network from VoiceFixer, with a key objective change: instead of multiplicative masking, it uses additive mel-spectrogram restoration, allowing the network to complete missing time-frequency bins (bandwidth extension and in-painting). HiFi-GAN serves as the neural vocoder in this stage, and training uses GAN, feature matching, and mel-spectrogram losses. GSR is trained on a proprietary dataset of 1,750 hours covering a wide SNR range and including codec artefacts and simulated packet drops; the module totals 40M parameters.

In the second stage, a diffusion-based voice conversion model (derived from Diff-VC) takes the GSR output and generates a refined restoration conditioned on a clean speaker embedding from the same speaker. The key modification versus Diff-VC is replacing the original phoneme-wise spectrogram averaging content encoder with a pre-trained HuBERT encoder followed by vector quantisation (VQ, 2,000 clusters). The VQ discrete units feed a transformer-based content encoder (6-layer, 192 channels, 4 attention heads) that produces a coarse mel-spectrogram. A U-Net decoder then runs the diffusion process conditioned on this coarse spectrogram and a speaker embedding from ECAPA-TDNN. Classifier-free guidance is applied during training (p=0.1 dropout for speaker and content). Inference uses 30 diffusion steps with guidance scales of 0.25 (speaker) and 1.0 (content). The VC module adds 169M parameters for a combined system size of 209M.

The pipeline is designed so that the GSR stage handles noise suppression before VC is applied, working around VC's sensitivity to degraded input. The clean enrollment utterance for the speaker embedding need not be content-aligned with the input.

## Key Results

On VCTK-DEMAND, the proposed GSR+VC system achieves NISQA 4.24, UTMOS 4.12, WV-MOS 4.34, DNSMOS 3.31, competitive with FINALLY (NISQA 4.48, UTMOS 4.32) which has 454M parameters, more than twice the size. On the UNIVERSE validation set, the system reaches NISQA 4.21, UTMOS 4.10, WV-MOS 4.23, DNSMOS 3.26, outperforming UNIVERSE (the system) on 3 of 4 metrics and matching FINALLY on 2 of 4, while being substantially smaller (209M vs 454M and 189M respectively) (Tables 1 and 2).

The ablation study (Table 3) shows that combining GSR and VC yields large gains over either stage alone, particularly on the more challenging UNIVERSE set. The HuBERT+VQ content representation outperforms direct mel-spectrogram input to the VC module, especially under severe degradation, confirming that SSL-derived discrete features are more robust to noise than raw spectrograms. The GSR module on its own outperforms VoiceFixer across all metrics on both datasets despite being smaller (40M vs 117M), attributed to more training data, additional augmentations, and the switch from TFGAN to HiFi-GAN.

## Novelty Assessment

The contribution is primarily engineering integration: the GSR and VC components are both adapted from prior work (VoiceFixer and Diff-VC respectively), and the two-stage enhancement-then-VC pipeline design has been explored before. The key technical modification is replacing Diff-VC's content encoder with HuBERT+VQ, which the ablation shows provides a meaningful robustness benefit under severe degradation. The conditioning mechanism (same-speaker clean enrollment) is straightforward and the system assumes such enrollment audio is available, which limits applicability in unconstrained scenarios. The results are competitive but fall short of FINALLY on most metrics, and a direct comparison with Miipher (the most similar prior system) was not possible due to unavailability of Miipher's model and data.

## Field Significance

Moderate — Voice-ENHANCE demonstrates that chaining a speaker-agnostic restoration front-end with a diffusion-based VC back-end provides a practical and efficient path to studio-level speech quality, achieving performance competitive with larger specialist systems. The use of HuBERT+VQ as the VC content representation for noise-robust restoration is a useful design insight, though the overall system design is incremental relative to the components it builds on.

## Claims

- **supports:** Multi-stage speech restoration pipelines that separate noise suppression from speaker-guided generation outperform single-stage generative approaches under severe degradation conditions.
  > *Evidence:* GSR+VC substantially outperforms standalone GSR or standalone VC across all metrics on both VCTK-DEMAND and UNIVERSE; the gain is largest on the UNIVERSE set, which simulates more severe distortions including band-limiting, reverberation, codec artefacts, and packet drops. *(§4.3, Table 3)*

- **supports:** Self-supervised discrete speech representations provide more robust content features for voice conversion than raw mel-spectrograms when the input speech is degraded.
  > *Evidence:* VC (SSL) using HuBERT+VQ consistently outperforms VC (Mel) using direct mel-spectrogram input, with the gap widening on the more challenging UNIVERSE dataset where VC (Mel) shows significant quality degradation. *(§4.3, Table 3)*

- **complicates:** Diffusion-based voice conversion models cannot reliably handle degraded input without a dedicated pre-processing stage, even when conditioned on clean speaker embeddings.
  > *Evidence:* VC (SSL) in standalone mode achieves lower scores than GSR+VC on both evaluation sets; the VC module performs markedly worse when the input is noisy without the GSR front-end, confirming that speaker-embedding guidance alone does not compensate for noisy content features. *(§4.3, Table 3)*

- **complicates:** Enrollment-dependent speaker guidance for speech restoration limits applicability to settings where clean reference speech from the same speaker is available in advance.
  > *Evidence:* The system assumes short, uncorrelated segments of clean speech are obtained beforehand for speaker embedding extraction; the paper does not evaluate performance when such enrollment audio is unavailable or mismatched. *(§3, §4.1)*

## Limitations and Open Questions

The system requires a clean enrollment utterance from the target speaker, which may not be available in all real-world scenarios. No comparison with Miipher could be performed due to unavailability of that model, leaving the relationship between this approach and the closest prior work unquantified. Evaluation uses only automated perceptual quality metrics (NISQA, UTMOS, WV-MOS, DNSMOS) without any human listening tests, so it is unclear whether the metric gains translate to perceived quality improvements. Both the GSR and VC models are trained on separate datasets, and the system has not been evaluated on the joint training configuration proposed as future work. Performance on languages other than English is not examined.

## Wiki Connections

- [[voice-conversion|Voice Conversion]] — applies diffusion-based VC techniques in a speech restoration context, using same-speaker conditioning rather than cross-speaker conversion.
- [[diffusion-tts|Diffusion TTS]] — the VC restoration module uses a diffusion-based U-Net decoder derived from Diff-VC, with score-based generation conditioned on speaker and content embeddings.
- [[self-supervised-speech|Self-Supervised Speech]] — the system uses a pre-trained HuBERT encoder with vector quantisation as the content representation for the VC stage, and the ablation confirms this improves robustness over mel-spectrogram features under noise.
- [[gan-vocoder|GAN Vocoder]] — HiFi-GAN serves as the neural vocoder in both the GSR stage (for initial restoration) and the final synthesis step after VC, with GAN training losses applied in the GSR module.
- [[evaluation-metrics|Evaluation Metrics]] — evaluation relies entirely on automatic perceptual metrics (NISQA, UTMOS, WV-MOS, DNSMOS) without human listening tests, which raises questions about the fidelity of metric-based comparisons.
