---
id: "interspeech-2025-cho25c_interspeech"
title: "Unleashing the Inner Monster: Demonstrating High-Fidelity Human to Non-Human Voice Conversion"
authors: ["Namhyun Cho", "Sunmin Kim", "Minsu Kang", "Seolhee Lee", "Choonghyeon Lee", "Yangsun Lee"]
organization: "NC AI Co., Ltd"
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-07-13
task: [VC]
architecture: [hybrid]
conditioning: [speaker-conditioned]
training: [supervised]
model_size: "not reported"
codec_used: "none"
datasets_train: ["not reported"]
datasets_eval: ["not reported"]
metrics: []
code_available: false
demo_available: true
url: "https://www.isca-archive.org/interspeech_2025/cho25c_interspeech.html"
related_concepts: [voice-conversion, disentanglement, self-supervised-speech]
related_papers: [interspeech-2025-0433]
field_significance:
  level: low
  type: [engineering-integration]
generation:
  date: 2026-07-13
  agent: speech-generation-ingest-agent
  model: claude-sonnet-4-6
  commit: "b035ebb"
---

> [!abstract] Interspeech · 2025 · Conference
> **Cho et al.** (NC AI Co., Ltd) · [→ Paper](https://www.isca-archive.org/interspeech_2025/cho25c_interspeech.html) · Demo: ✓ · Code: ✗
>
> A demonstration of a human-to-non-human (H2NH) voice conversion system that transforms live user speech into high-fidelity monster vocalizations in real-time, using a CVAE-flow architecture adapted for 44.1 kHz audio bandwidth and transient-rich non-human acoustic targets.

## Problem

Professional monster voice production for massively multiplayer online roleplaying games requires voice actors capable of approximating non-human timbres, followed by extensive post-production using specialised tools such as Dehumaniser. This process is costly in both time and personnel. Existing voice conversion models cannot adequately substitute for this workflow because they are trained exclusively on human speech and use audio features tuned to human vocal characteristics, which span a narrower frequency range and lack the unvoiced and extreme-vocal elements (breathing, screaming) that define monster sounds. No publicly available technology existed for converting human voices into monster or non-human timbres at the time of publication.

## Method

The H2NH model operates at 44.1 kHz with a 20 ms window and 5 ms hop-length STFT, covering Mel spectrograms over 0-22.05 kHz to accommodate the broader frequency content of monster vocalizations. A reference encoder extracts a global style vector from the target monster's Mel spectrogram at inference time. Because pitch estimation is unreliable for non-human vocalizations, frame-level energy replaces f0 as the primary prosodic conditioning signal.

The architecture combines a CVAE-based posterior encoder, decoder, prior encoder, and a normalizing flow module (similar in structure to VITS-family models). A key design choice is that the style vector is applied exclusively within the prior and flow components rather than the posterior encoder, explicitly separating content and style pathways. The prior encoder integrates style embeddings, energy inputs, and SSL-derived linguistic features (captured via self-supervised perturbations across pitch and formant variations) to estimate latent distribution parameters; the flow module then refines the sampled latent variable.

Training uses a combined generator and discriminator objective including KL divergence, feature matching, LS-GAN adversarial loss, and reconstruction loss. To handle the transient-rich nature of non-human audio, the model incorporates frequency discriminators with random lags (FDRL) at multiple hop lengths and a DAC discriminator for enhanced temporal resolution.

The system is packaged as a Linux virtual machine with a Windows-based web interface; users select a target monster voice, record via a Web Audio API AudioWorklet, and receive the converted output immediately on the same page.

## Key Results

This is a demonstration paper. Quantitative evaluation metrics are not reported here; comparative results against baselines including DDDM-VC are contained in the companion full research paper (Kang et al., Interspeech 2025, DOI: 10.21437/Interspeech.2025-433). Audio conversion samples generated from a public speech corpus are available at https://nc-ai.github.io/speech/publications/nonhuman-vc. The real-time interactive demo is described as producing perceptibly monster-like conversions from live user recordings.

## Novelty Assessment

The contribution is engineering integration rather than architectural novelty. The H2NH model adapts a VITS-like CVAE-flow VC pipeline to a non-human target domain through three specific choices: extending audio bandwidth to 44.1 kHz, substituting energy for f0 as the prosodic cue, and integrating higher-resolution frequency discriminators. Each of these adaptations is well-motivated by the acoustic properties of the target domain, but none represents a methodological advance over existing VC architectures. The paper's primary novelty is the application domain (non-human gaming audio) and the real-time interactive deployment; the detailed technical evaluation and comparison against baselines are deferred to the companion research paper.

## Field Significance

Low — This demo paper validates proof-of-concept for interactive human-to-non-human voice conversion in a gaming context, accompanying a separate full research paper that provides the quantitative results. It contributes to the field primarily by documenting the domain-specific engineering adaptations needed when human-speech VC pipelines are applied to non-human acoustic targets.

## Claims

- **complicates:** Voice conversion models trained exclusively on human speech are insufficient for targets with broader-than-human frequency range and non-periodic vocalizations.
  > *Evidence:* Monster sounds span a wider frequency range than human speech, contain intricate audio-effect details, and include non-speech elements (unvoiced breathing, extreme vocal expressions) absent from standard human-speech training corpora; standard VC models trained on these corpora fail to reproduce the target timbres. *(§3)*

- **supports:** Replacing f0 estimation with frame-level energy as the primary prosodic cue enables voice conversion for target voices where pitch periodicity cannot be reliably estimated.
  > *Evidence:* The H2NH prior encoder substitutes frame-level energy for f0 because f0 estimation degrades severely for non-human monster sounds; the system still produces satisfactory conversions in the real-time demonstration. *(§4)*

- **supports:** Extending audio bandwidth to 44.1 kHz and incorporating high-temporal-resolution discriminators improves voice conversion quality for transient-rich non-human acoustic targets.
  > *Evidence:* The H2NH model uses a 5 ms hop-length STFT, Mel spectrograms covering 0-22.05 kHz, FDRL with multiple hop lengths, and a DAC discriminator specifically to capture fine temporal details and broad frequency content of monster vocalizations that standard 22.05 kHz pipelines cannot reproduce. *(§4)*

## Limitations and Open Questions

> [!warning]
> This paper contains only a surface-level architectural description; training data, hyperparameters, speaker or monster coverage, and quantitative evaluation metrics are all deferred to the companion Interspeech 2025 research paper. Claims about conversion fidelity cannot be independently verified from this paper alone.

The system targets a predefined set of monster voices for a specific gaming title; generalisation to arbitrary non-human voices, varied recording conditions, or diverse languages is not assessed. No ablation studies, failure cases, or degradation conditions are described. The demonstration runs locally within a VM environment, and no information is given about latency, throughput, or hardware requirements that would inform deployment feasibility.

## Wiki Connections

- [[voice-conversion|Voice Conversion]] — this paper extends voice conversion to non-human target domains (monster vocalizations for gaming), documenting the domain-specific adaptations required beyond standard human-speech VC pipelines.
- [[disentanglement|Disentanglement]] — the H2NH architecture separates style (extracted by a reference encoder and applied exclusively in the prior and flow modules) from SSL-derived linguistic content in the posterior encoder.
- [[self-supervised-speech|Self-Supervised Speech]] — SSL-derived linguistic features serve as a core input to the prior encoder, enabling content-style separation without relying on a text encoder or phoneme aligner.
- [[interspeech-2025-0433|When Humans Growl and Birds Speak]] — the companion full research paper presenting the same H2NH system with quantitative evaluation and comparisons against DDDM-VC and other baselines.
