---
id: "interspeech-2025-0816"
title: "Bridging Speech and Singing: Multi-stage Speech-Prompted Singing Voice Conversion with Speaker Embedding Adaptation"
authors: ["Mingda Liu", "Jiatong Shi"]
organization: null
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-06-03
task: [singing, VC]
architecture: [diffusion]
conditioning: [speaker-conditioned, zero-shot]
training: [supervised]
model_size: "not reported"
codec_used: "none"
datasets_train: ["NUS-48E Corpus", "OpenSinger", "ACE-KiSing", "Aishell-3"]
datasets_eval: ["M4Singer", "ST-CMDS"]
metrics:
  - name: MOS
    value: 3.568
    system: SSANSVC-stage2
    testset: M4Singer→ST-CMDS (MOS-n)
  - name: SMOS
    value: 1.627
    system: SSANSVC-stage2
    testset: M4Singer→ST-CMDS (MOS-ts)
  - name: SPK-SIM
    value: 0.506
    system: SSANSVC-stage2
    testset: M4Singer→ST-CMDS
  - name: CER
    value: 36.56
    system: SSANSVC-stage2
    testset: M4Singer→ST-CMDS
code_available: null
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/liu25h_interspeech.html"
related_concepts: [voice-conversion, singing, disentanglement, self-supervised-speech, speaker-adaptation, evaluation-metrics]
related_papers: []
field_significance:
  level: moderate
  type: [architectural-novelty]
generation:
  date: 2026-06-03
  model: claude-sonnet-4-6
  commit: "55d0339"
---

> [!abstract] Interspeech · 2025 · Conference
> **Mingda Liu et al.** · [→ Paper](https://www.isca-archive.org/interspeech_2025/liu25h_interspeech.html) · Demo: ? · Code: ?
>
> SSANSVC introduces a Speaker-Singer Adaptation Network (SSAN) that bridges the embedding-space gap between speech and singing modalities to enable zero-shot speech-prompted singing voice conversion, paired with a multi-stage cycle training strategy that sidesteps the scarcity of paired speech-singing data.

## Problem

Singing voice conversion has largely focused on singer-to-singer timbre transfer, where both source and target audio come from the singing domain. Converting a target speaker's speech timbre into singing is substantially harder: speech and singing produce different acoustic characteristics and occupy different regions of the speaker embedding space, so standard SVC conditioning mechanisms fail to generalise across modalities. Paired speech-singing data from the same person is also scarce, making direct supervised cross-domain training impractical.

## Method

The system has two main components: a Speaker-Singer Adaptation Network (SSAN) and a consistency-model SVC backbone, together called SSANSVC.

The SSAN is a Transformer-based encoder (6 layers, 8 attention heads, 512-dimensional FFN, 192-dimensional attention) that aligns speaker embeddings extracted from speech with singer embeddings extracted from singing. It is trained with a cosine embedding loss to pull matched speaker-singer pairs closer and push apart mismatched ones, supplemented by separate reconstruction losses that prevent the aligned embeddings from drifting away from their original representations. The training process randomly constructs matched and mismatched speaker-singer pairs from the NUS-48E corpus, which provides paired speech and singing from the same individuals.

![The framework of the SSAN.](assets/interspeech-2025-0816/figure-1.png)

The SVC model is built on the CoMoSVC teacher architecture (a consistency model), with the singer-identity conditioning replaced by the adapted speaker timbre feature from SSAN. Feature extraction follows the standard singing synthesis pipeline: content features from ContentVec (12th layer), pitch from the DIO F0 estimator, loudness from squared amplitude, and the 192-dimensional SSAN timbre embedding. All features are projected to 256 dimensions, concatenated, and fed as conditional input to the decoder, which generates mel-spectrograms converted to audio by a pre-trained singing vocoder.

Training proceeds in two stages. Stage 1 replicates standard SVC training, reconstructing audio from same-speaker same-domain features. Stage 2 introduces cross-domain cycle training: the model takes singing content from one speaker paired with speech timbre from another, generates an intermediate output, and then uses that output as the timbre input for a second conversion pass — creating a cycle that simulates paired speech-singing data without requiring it. All audio is processed at 24 kHz with 80-bin mel-spectrograms.

## Key Results

In the primary cross-domain task (speech-prompted singing conversion, M4Singer source songs to ST-CMDS target speakers), SSANSVC-stage2 achieves SIM 0.506, SingMOS 3.871, and CER 36.56, compared to the CoMoSVC baseline's SIM 0.448, SingMOS 3.820, and CER 40.43. The gain in speaker similarity (+0.058) is the most substantial improvement. In subjective evaluation by 10 listeners, SSANSVC-stage2 scores MOS-n 3.568 ± 0.066 vs. CoMoSVC's 3.511 ± 0.065, and MOS-ts 1.627 ± 0.057 vs. 1.597 ± 0.055 — modest margins.

> [!note]
> Subjective evaluation used only 10 volunteers, and confidence intervals overlap between SSANSVC-stage2 and SSANSVC-stage1 for both naturalness and timbre similarity. The improvements over the baseline are statistically borderline at this listener count.

On in-domain tasks (singing-to-singing, speech-to-speech), SSANSVC-stage2 matches or slightly exceeds CoMoSVC on similarity metrics while showing minor regressions on CER in some conditions, indicating that the SSAN adaptation does not harm normal-domain conversion.

## Novelty Assessment

The core contribution is the SSAN, a relatively straightforward metric-learning module using cosine embedding loss to align speech and singing speaker representations. The SVC backbone is adopted unchanged from CoMoSVC; the cycle training strategy is explicitly adapted from prior multilingual VC work (MulliVC). The novelty is therefore incremental: the authors combine an existing consistency-model SVC with a new cross-domain embedding alignment module and a training recipe drawn from adjacent work. The technical choices are well-motivated and the empirical improvements are real, but none of the components represents a departure from established practice.

## Field Significance

Moderate — This paper addresses a genuine and underexplored gap: SVC systems that accept speech (rather than singing) as the target timbre reference. The SSAN alignment approach offers a transferable pattern for other cross-domain voice conversion problems where embedding spaces diverge across modalities. Its contribution is primarily architectural in the narrow sense of a new conditioning module, but the improvements over the baseline are modest and the evaluation panel is small.

## Claims

- Converting speech timbre to singing requires cross-modal speaker embedding alignment, and standard singer-identity conditioning generalises poorly across the speech-singing domain boundary.
- Cycle training strategies that simulate paired cross-domain data can compensate for the scarcity of matched speech-singing corpora in voice conversion training.
- Zero-shot singing voice conversion with speech prompts achieves lower timbre similarity scores than same-domain (singing-to-singing) conversion, indicating that the cross-modal gap is not fully closed by embedding alignment alone.
- Automated speaker similarity metrics capture relative improvements from cross-domain adaptation that are not clearly reflected in small-panel subjective timbre similarity ratings.

## Limitations and Open Questions

> [!warning]
> The subjective evaluation relies on only 10 volunteers, producing confidence intervals that overlap between all three systems on both MOS-n and MOS-ts. The claimed superiority of SSANSVC-stage2 over CoMoSVC in naturalness and similarity is not statistically robust at this sample size.

The model's loss function addresses only mel reconstruction; the authors note that timbre loss and lyrics recognition loss (to reduce CER) are absent and represent the primary direction for future improvement. The two-stage training procedure also introduces significant complexity and training cost compared to the CoMoSVC baseline. Evaluation is restricted to Mandarin speech and singing datasets, so generalisation to other languages and vocal styles is untested. The dependency on NUS-48E — one of few available paired speech-singing corpora — limits reproducibility in languages where such data does not exist.

## Wiki Connections

Related concepts: [[voice-conversion]] · [[singing]] · [[disentanglement]] · [[self-supervised-speech]] · [[speaker-adaptation]] · [[evaluation-metrics]]
