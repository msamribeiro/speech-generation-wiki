---
id: "interspeech-2025-0438"
title: "LinearVC: Linear Transformations of Self-Supervised Features Through the Lens of Voice Conversion"
authors: ["Herman Kamper", "Benjamin van Niekerk", "Julian Zaïdi", "Marc-André Carbonneau"]
organization: null
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-07-02
task: ["VC"]
architecture: ["hybrid"]
conditioning: ["speaker-conditioned", "zero-shot"]
training: ["supervised"]
model_size: "not reported"
codec_used: "none (HiFi-GAN vocoder on WavLM features)"
datasets_train: ["LibriSpeech train-clean-100"]
datasets_eval: ["LibriSpeech test-clean", "LibriSpeech dev-clean"]
metrics:
  - name: WER
    value: 4.9
    system: LinearVC
    testset: LibriSpeech test-clean
  - name: EER
    value: 33.6
    system: LinearVC
    testset: LibriSpeech test-clean
  - name: MUSHRA
    value: 62.5
    system: LinearVC
    testset: LibriSpeech test-clean
code_available: true
demo_available: true
url: "https://www.isca-archive.org/interspeech_2025/kamper25_interspeech.html"
related_concepts: ["voice-conversion", "self-supervised-speech", "disentanglement", "evaluation-metrics", "subjective-evaluation"]
related_papers: []
field_significance:
  level: "moderate"
  type: ["architectural-novelty", "conceptual-contribution"]
generation:
  date: 2026-07-02
  agent: speech-generation-ingest-agent
  model: claude-sonnet-4-6
  commit: "d8ecfd4"
---

> [!abstract] Interspeech · 2025 · Conference
> **Kamper et al.** · [→ Paper](https://www.isca-archive.org/interspeech_2025/kamper25_interspeech.html) · Demo: ✓ · Code: ✓
>
> LinearVC shows that a single linear projection of WavLM SSL features suffices for competitive voice conversion, and uses constrained transformation experiments and SVD-based factorization to reveal that content and speaker information occupy orthogonal low-dimensional subspaces within the same SSL layer.

## Problem

Most SSL-based voice conversion methods apply nonlinear transformations (nearest-neighbour mapping, complex decoders) to convert speech across speakers, offering limited insight into why SSL features support voice conversion so well. Prior work has established that phonetic and speaker information tends to be encoded in orthogonal directions within SSL representations, but the practical implications of this geometry for conversion quality and the structure of content subspaces have not been systematically probed.

## Method

LinearVC operates on WavLM-Large (layer 6) features, producing 1024-dimensional frames at 20 ms intervals. During training, source and target speech utterances are encoded with WavLM; for each source frame, its single nearest neighbour (cosine distance) in the target speaker's frame set is found. A projection matrix W is then learned by solving the least-squares multivariate regression: Y ≈ XW, where X and Y are the matched source and target frame matrices. At inference, each frame of an unseen source utterance is linearly projected by W to produce the converted representation; a pre-trained HiFi-GAN vocoder operating on WavLM features then synthesises the waveform.

![LinearVC: At learning time (top), source speech frames are paired with target frames by finding their single nearest neighbours. A projection matrix is learned by finding the least squares solution for mapping from source to target frames. At inference time (bottom), frames from an unseen source utterance are linearly projected and then vocoded to get speech in the target speaker's voice.](assets/interspeech-2025-0438/figure-2.png)

To probe the feature space geometry, the paper runs a series of constrained transformation experiments: translation only (bias vector), rotation and reflection only (orthogonal Procrustes), and unconstrained affine. This reveals that rotation and reflection account for nearly all of the speaker similarity gain, while translation alone changes the voice but does not reliably mimic the target.

A second contribution is a factorized variant using singular value decomposition (SVD). Features from K speakers sharing matched content are arranged into a block matrix X; SVD decomposes this into a shared content matrix C and speaker-specific transformations S_k. Voice conversion is then performed by projecting the source into the content subspace using the pseudoinverse of the source speaker's transformation and applying the target speaker's transformation. The rank r of the factorization is a hyperparameter: content can be encoded at rank 16 with low CER, but speaker similarity (EER) continues to improve up to rank ~100; beyond rank 200, source speaker information begins leaking into the content representation.

## Key Results

On LibriSpeech test-clean (7,800 conversions across 40 speakers), LinearVC achieves WER 4.9%, CER 2.6%, EER 33.6%, and a MUSHRA-like naturalness of 62.5 ± 3.5 and similarity of 67.5 ± 2.6. These results are statistically indistinguishable from kNN-VC (WER 5.7%, EER 38.9%, naturalness 60.6, similarity 67.2) and SoundStorm (WER 4.6%, EER 30.2%, naturalness 58.6, similarity 68.6). FreeVC achieves higher naturalness (71.1) but at the cost of substantially lower speaker similarity (EER 10.5%), indicating that it over-smooths the output.

The factorized variant (rank 100) is also statistically indistinguishable from the non-factorized LinearVC, achieving WER 4.7%, EER 35.2%, naturalness 62.3, and similarity 64.2.

In the constrained transformation experiments on dev-clean, orthogonal-only transformations reach EER 27.7% vs. 31.8% for unconstrained, showing that rotation and reflection capture most of the conversion capability. Translation alone reaches only EER 7.7%, confirming that merely shifting the feature distribution does not produce reliable speaker mimicry.

> [!note]
> Subjective evaluations (MUSHRA-like) use at least 15 raters per conversion, but are conducted only on ten target speakers (five same-gender, five cross-gender) from 40 utterances of roughly five seconds each. Statistical significance is assessed with Friedman tests followed by Wilcoxon signed-rank tests with Bonferroni correction.

## Novelty Assessment

The LinearVC method itself is a deliberate simplification of kNN-VC: replacing the nonlinear nearest-neighbour lookup at inference with a linear projection learned offline. This architectural simplification is principled and produces a comparable system while enabling the geometric analysis that is the paper's main intellectual contribution. The SVD factorization approach for explicit content-speaker separation is novel in the context of SSL-based VC, and the constrained-transformation ablation study (translation, orthogonal, unconstrained) provides unusually clean evidence for the orthogonal-subspace structure hypothesis.

The limitation is that the analysis is confined to WavLM-Large layer 6 and English LibriSpeech. Whether the orthogonal subspace structure holds across SSL models, layers, or languages remains untested. The HiFi-GAN vocoder is also pretrained and shared across all systems, meaning naturalness scores partly reflect vocoder quality rather than the conversion method alone.

## Field Significance

Moderate — LinearVC demonstrates that competitive voice conversion can be achieved with a purely linear transformation of SSL features, providing clean empirical evidence that content and speaker information occupy orthogonal low-dimensional subspaces in WavLM representations. The constrained-transformation experiments and SVD factorization offer a principled analytical lens that complements prior orthogonality studies. The contribution is primarily investigative rather than advancing the frontier of VC performance, and the analysis is restricted to a single SSL model and language.

## Claims

- **supports:** Linear transformations of self-supervised speech features are sufficient for competitive voice conversion, without complex nonlinear decoders or model fine-tuning.
  > *Evidence:* LinearVC's single learned projection matrix W on WavLM-Large layer 6 achieves WER 4.9%, EER 33.6%, and MUSHRA naturalness 62.5 on LibriSpeech test-clean, statistically indistinguishable from kNN-VC and SoundStorm in both naturalness and speaker similarity. *(§3.2, Table 1)*

- **supports:** Content and speaker identity information reside in orthogonal low-dimensional subspaces within the same SSL layer, enabling voice style transfer through geometric manipulation.
  > *Evidence:* Constraining the linear transformation to rotation and reflection only achieves EER 27.7% vs. 31.8% for unconstrained; adding translation alone gives only EER 7.7% with intelligibility maintained (CER 2.9%), confirming a shared phonetic subspace across speakers. SVD factorization at rank 16 achieves CER < 4%, while speaker similarity requires rank ~100. *(§4, Table 2; §5.2, Figure 4)*

- **complicates:** High naturalness in voice conversion does not imply high speaker similarity — these objectives can trade off sharply depending on the system design.
  > *Evidence:* FreeVC achieves the highest naturalness (71.1 MUSHRA) among all systems but the lowest speaker similarity (EER 10.5% vs. 33.6% for LinearVC and 38.9% for kNN-VC), indicating that perceptual smoothness and target-speaker fidelity are partly in tension. *(§3.2, Table 1)*

- **refines:** EER as an objective speaker similarity metric provides only coarse correspondence with perceived speaker similarity in voice conversion evaluation.
  > *Evidence:* The paper notes that small EER differences do not reliably track subjective similarity ratings, and that EER gives a coarse correspondence with perceptual judgements. LinearVC and kNN-VC have comparable subjective similarity (67.5 vs. 67.2) but different EERs (33.6 vs. 38.9). *(§3.2)*

## Limitations and Open Questions

The analysis is restricted to WavLM-Large (layer 6) on English LibriSpeech. Whether the orthogonal subspace structure holds for other SSL models (HuBERT, wav2vec 2.0), other layers, other languages, or noisy/spontaneous speech settings is left as future work. The vocoder is shared across all systems, making it difficult to isolate conversion quality from synthesis quality. The training data requirement (2.7 minutes per target speaker) means LinearVC is not a zero-shot system in the strictest sense; it requires a small per-speaker adaptation step. Performance at very low reference amounts is untested. Future applications the authors suggest include speech anonymization and phonetic content extraction.

## Wiki Connections

- [[voice-conversion|Voice Conversion]] — LinearVC directly addresses voice conversion, achieving competitive quality with a simple linear projection of SSL features as an alternative to nonlinear nearest-neighbour or neural decoder approaches.
- [[self-supervised-speech|Self-Supervised Speech]] — the paper's core method depends on WavLM-Large features and uses the geometric structure of these representations as the basis for conversion and analysis.
- [[disentanglement|Disentanglement]] — the SVD factorization explicitly separates content and speaker information into orthogonal subspaces, and the constrained transformation experiments validate this disentanglement hypothesis.
- [[evaluation-metrics|Evaluation Metrics]] — the paper uses a multi-metric evaluation framework combining WER/CER for intelligibility, EER for objective speaker similarity, and MUSHRA-like tests for subjective naturalness and similarity, noting the coarse correspondence between EER and perceived similarity.
- [[subjective-evaluation|Subjective Evaluation]] — MUSHRA-like listening tests with at least 15 raters per conversion, Friedman tests, and Wilcoxon signed-rank tests with Bonferroni correction are used to establish statistical significance of system differences.
