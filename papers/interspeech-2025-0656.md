---
id: "interspeech-2025-0656"
title: "EEG-based Voice Conversion: Hearing the Voice of Your Brain"
authors: ["Yizhong Geng", "Wenxin Fu", "Qihang Lu", "Bingsong Bai", "Cong Wang", "Yingming Gao", "Ya Li"]
organization: "Beijing University of Posts and Telecommunications"
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: "2025-08-17"
ingested_date: "2026-07-02"
task: ["VC"]
architecture: ["hybrid"]
conditioning: ["speaker-conditioned", "zero-shot"]
training: ["supervised", "fine-tuning"]
model_size: "not reported"
codec_used: "none"
datasets_train: ["VCTK", "Single-Word-Production Dutch-iBIDS"]
datasets_eval: ["Single-Word-Production Dutch-iBIDS"]
metrics:
  - name: MOS
    value: 4.15
    system: "EEG-based VC (seen speakers)"
    testset: "Single-Word-Production Dutch-iBIDS"
  - name: MOS
    value: 4.00
    system: "EEG-based VC (unseen speakers)"
    testset: "Single-Word-Production Dutch-iBIDS"
code_available: null
demo_available: true
url: "https://www.isca-archive.org/interspeech_2025/geng25b_interspeech.html"
related_concepts: [voice-conversion, zero-shot-tts, disentanglement, evaluation-metrics, subjective-evaluation, self-supervised-speech]
related_papers: []
field_significance:
  level: "moderate"
  type: ["architectural-novelty", "engineering-integration"]
generation:
  date: "2026-07-02"
  agent: speech-generation-ingest-agent
  model: claude-sonnet-4-6
  commit: "d8ecfd4"
---

> [!abstract] Interspeech · 2025 · Conference
> **Geng et al.** (Beijing University of Posts and Telecommunications) · [→ Paper](https://www.isca-archive.org/interspeech_2025/geng25b_interspeech.html) · Demo: ✓ · Code: ?
>
> The first voice conversion system conditioned on EEG brain signals rather than voice samples, enabling zero-shot speaker identity transfer driven entirely by neural activity.

## Problem

Prior EEG-to-speech research focused on reconstructing speech content or basic sound features from neural signals, but largely ignored the timbral qualities that define individual voice identity. Brain-computer interface (BCI) systems capable of restoring a person's natural-sounding voice, rather than generic synthesised speech, remained out of reach. Existing voice conversion work operates exclusively on speech-to-speech pipelines; no prior system demonstrated that EEG signals alone can serve as a speaker conditioning source for voice conversion.

## Method

The system frames voice conversion as a cross-modal conditioning problem: source speech is converted to match the speaker identity encoded in a target speaker's EEG signals, with no voice sample from the target speaker required at inference time.

The architecture has three main components. The EEG encoder first reduces high-dimensional multichannel EEG data with PCA (to 25 principal components) and then processes the result with a CNN-Transformer hybrid (EEG Conformer architecture) that captures both local spatiotemporal patterns and global sequence dependencies. Its output feeds an EEG-voice feature alignment module, consisting of two parallel branches: a feature projection branch that maps EEG representations to the speaker embedding space via adaptive pooling and an MLP, and a speaker classification branch that predicts speaker identity with softmax output. A speaker encoder extracts conventional speaker embeddings from speech (used during training). The generator is a modified FreeVC framework with a content encoder for semantic extraction and a speech decoder for waveform synthesis.

Training proceeds in three stages. Stage I pre-trains the generator and speaker encoder on VCTK speech data using standard voice conversion objectives, establishing robust speech-conditioned VC. Stage II freezes the generator and trains only the EEG encoder, aligning EEG-derived embeddings to pre-trained speaker embeddings via an embedding fitting loss and speaker classification loss (weighted lambda_1=0.9, lambda_2=0.1). Stage III fine-tunes both the generator and EEG encoder jointly, replacing the speech-derived speaker embedding with the EEG-derived one for end-to-end conversion.

![Overview of the proposed system architecture and training strategy. The upper section outlines the three-stage training process: I) Generator and speaker encoder pre-training with speech data; II) EEG-speaker embedding alignment; III) Generator finetuning for EEG-based synthesis. The bottom section highlights the EEG encoder, which includes a feature extraction module and an EEG-voice feature alignment module.](assets/interspeech-2025-0656/figure-2.png)

## Key Results

Evaluation uses the Single-Word-Production Dutch-iBIDS dataset (10 speakers, paired EEG and audio), with approximately 1,500 generated samples per condition. Metrics include speaker embedding cosine similarity (Homogeneity, Consistency obj), a subjective consistency MOS (Consistency sub, 5-point scale, 20 testers, 16 audio groups), and a naturalness MOS.

For seen speakers, the EEG-based system achieves Homogeneity 0.9465 and Consistency (obj) 0.8391, outperforming the FreeVC speech-only baseline (0.9371 / 0.8216). Naturalness MOS reaches 4.15 versus 3.95 for FreeVC, with Consistency (sub) MOS 3.68 versus 3.59. For unseen speakers (zero-shot, k-fold cross-validation), Homogeneity is 0.9437, Consistency (obj) 0.8026, Consistency (sub) MOS 3.45, and Naturalness MOS 4.00. A t-SNE visualization confirms that synthesised speech clusters tightly with reference audio in speaker embedding space across all 10 speakers.

The comparison baseline (FreeVC) is not a fully fair match: it uses a conventional voice prompt rather than EEG, so differences conflate the conditioning modality with signal quality. The evaluation dataset is small (10 speakers, single-word utterances).

## Novelty Assessment

The primary novelty is the cross-modal conditioning concept: using EEG signals as the sole source of speaker identity for voice conversion, bypassing any voice sample from the target speaker. This is a genuine first within VC literature. The underlying components are largely established: EEG Conformer for neural feature extraction, FreeVC as the VC backbone, and standard contrastive alignment via embedding MSE and classification loss. The three-stage curriculum is a sensible engineering solution to the data scarcity and modality gap problems, but each stage individually uses known techniques. The contribution is best characterised as an engineering integration of existing VC and EEG components with a novel cross-modal conditioning scheme, enabled by a curriculum training strategy.

The evaluation is constrained by dataset scale (10 speakers, single words in Dutch), and there are no comparisons to other EEG-to-speech reconstruction systems. The paper acknowledges that larger datasets would likely improve performance.

## Field Significance

Moderate — This paper introduces a genuinely new conditioning modality for voice conversion (EEG brain signals) and demonstrates feasibility at a small scale, with results that match or slightly exceed a conventional speech-conditioned VC baseline. Its significance lies primarily in opening the problem space for BCI-driven voice identity restoration, particularly for individuals with speech impairments. The contribution is exploratory and limited by data scale; establishing the approach's robustness across larger speaker sets and more naturalistic EEG recordings remains future work.

## Claims

- **supports:** Cross-modal feature alignment between neural signals and speaker embeddings can enable voice conversion without any target-speaker voice data.
  > *Evidence:* The EEG-voice feature alignment module, trained with embedding MSE and speaker classification losses, produces speaker embeddings from EEG that drive FreeVC-based conversion to zero-shot quality (Naturalness MOS 4.00, Consistency obj 0.8026 for unseen speakers). *(§3.1.2, §4.4)*

- **supports:** Non-speech biometric signals can encode speaker-identity information sufficient to guide voice timbre conversion.
  > *Evidence:* t-SNE visualisation shows synthesised speech clusters align with reference audio per speaker, and Homogeneity scores (0.9437–0.9465) exceed the FreeVC baseline (0.9371), indicating EEG features encode timbre-discriminative information. *(§4.4.1, Figure 2)*

- **complicates:** Zero-shot voice conversion from EEG signals requires a large-scale speech-only pre-training stage to compensate for the scarcity and noise of paired EEG-speech data.
  > *Evidence:* The three-stage curriculum first pre-trains on VCTK (Stage I, speech only), then aligns EEG to pre-trained speaker embeddings (Stage II), before joint fine-tuning (Stage III). The authors explicitly attribute feasibility to leveraging the pre-trained VC model's representations. *(§3.2)*

- **complicates:** Evaluation of EEG-driven voice conversion is fundamentally limited by the availability of paired EEG-speech corpora at the scale needed for generalisation.
  > *Evidence:* The entire EEG evaluation uses the Single-Word-Production Dutch-iBIDS dataset (10 speakers, single-word utterances). Seen-speaker training uses 80% of this data; unseen-speaker evaluation uses k-fold over 10 speakers. The authors note that "more data will improve performance." *(§4.1, §5)*

## Limitations and Open Questions

> [!warning]
> The evaluation dataset contains only 10 speakers producing single words in Dutch. Results on connected speech, diverse languages, and larger speaker populations are entirely untested. Generalisability claims should be treated as preliminary.

The baseline comparison (FreeVC) is not an equivalent system: FreeVC uses a target-speaker voice prompt while the proposed system uses EEG, so observed differences in quality scores may partly reflect the inherent difficulty of the EEG conditioning signal rather than architectural superiority. The paper does not report intelligibility metrics (WER/CER), making it impossible to assess whether semantic content is preserved faithfully. The EEG signals in the dataset come from intracranial recordings (high SNR), which may not transfer to consumer-grade scalp EEG equipment used in practical BCI deployments.

## Wiki Connections

- [[voice-conversion|Voice Conversion]] — introduces EEG signals as a novel non-speech conditioning modality for VC, extending the task beyond speech-to-speech paradigms.
- [[zero-shot-tts|Zero-Shot TTS]] — demonstrates zero-shot speaker identity transfer without any voice sample from the target speaker at inference, using only EEG data.
- [[disentanglement|Disentanglement]] — the alignment module explicitly maps EEG features into a shared speaker embedding space, separating neural activity from speech content in a cross-modal disentanglement scheme.
- [[evaluation-metrics|Evaluation Metrics]] — reports objective speaker-embedding similarity metrics (Homogeneity, Consistency obj) via cosine similarity, alongside a FreeVC speech-only baseline comparison for both seen and unseen speakers.
- [[subjective-evaluation|Subjective Evaluation]] — reports subjective MOS scores for both naturalness and consistency, evaluated by 20 listeners across 16 audio sample groups.
- [[self-supervised-speech|Self-Supervised Speech]] — the FreeVC backbone used in Stages I and III relies on self-supervised speech representations (ContentVec/HuBERT-derived content encoder) for semantic extraction in the VC pipeline.
