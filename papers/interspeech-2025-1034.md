---
id: "interspeech-2025-1034"
title: "Non-Standard Accent TTS Support via Large Multi-Accent Frontend Pronunciation Knowledge Transfer"
authors: ["Noe Berger", "Siqi Sun", "Korin Richmond"]
organization: "University of Edinburgh"
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: "2025-08-17"
ingested_date: "2026-05-30"
task: [TTS]
architecture: [transformer-enc-dec]
conditioning: [text-conditioned]
training: [supervised, fine-tuning]
model_size: "not reported (LSTM encoder-decoder frontend)"
codec_used: none
datasets_train: ["LibriTTS (pronunciation frontend training, 199,612 sentences)", "HiFi-TTS (test set)"]
datasets_eval: ["HiFi-TTS (4,431 sentences per accent)"]
metrics:
  - name: WER
    value: 89.21
    system: Large multi-accent model, 1k training sentences (unseen word accuracy %)
    testset: HiFi-TTS EDI accent (high-similarity pair ABD1/EDI)
  - name: WER
    value: 88.32
    system: Ceiling-baseline (full 199,612 sentences)
    testset: HiFi-TTS EDI accent
  - name: WER
    value: 87.28
    system: Large multi-accent model, 1k training sentences (moderate similarity GAM/EDI)
    testset: HiFi-TTS EDI accent
code_available: null
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/berger25_interspeech.html"
related_concepts: [multilingual-tts, speaker-adaptation]
related_papers: []
field_significance:
  level: moderate
  type: [engineering-integration, empirical-benchmark]
generation:
  date: 2026-06-22
  agent: speech-generation-review-agent
  model: claude-sonnet-4-6
  commit: "24ea64d"
---
> [!abstract] Interspeech · 2025 · Conference
> **Noe Berger et al.** (University of Edinburgh) · [→ Paper](https://www.isca-archive.org/interspeech_2025/berger25_interspeech.html) · Demo: ? · Code: ?
>
> A large (14-accent) multi-accent neural frontend model reduces the pronunciation transcription data needed to support a new non-standard English accent by 95% — from 20k to just 1k sentences — by enabling cross-accent pronunciation knowledge transfer, with the transfer strength weakly mediated by accent pair similarity.

## Problem

Commercial TTS applications overwhelmingly serve a small set of standard accents, causing material representational harm to speakers of non-standard varieties who receive unnatural synthesis experiences. Building TTS for non-standard accents requires extensive phonetically transcribed pronunciation data, which is prohibitively costly to produce for low-resource accent varieties where expert annotators are scarce. Prior work (Sun and Richmond 2024) showed a tri-accent neural frontend model could reduce pronunciation data requirements from 206k to 20k sentences — still too many for practical low-resource voice-building. Two open questions remained: (1) can data requirements be pushed dramatically lower still, and (2) is accent similarity a relevant factor in the knowledge transfer effect?

## Method

The paper builds on the neural sequence-to-sequence (Seq2Seq) frontend framework from Sun and Richmond (2024). The frontend model translates input text (character sequence) to a linguistic specification in the Unilex phoneset (phones, lexical stress, syllable breaks, word boundaries, prosodic boundary labels), using a bidirectional LSTM encoder and unidirectional LSTM decoder with monotonic Gaussian mixture model (GMM) attention.

**Accent coverage:** 14 English accents from 5 countries (Australia, Ireland, New Zealand, UK regions, USA regions), all using the Unilex phoneset. Pronunciation data is bootstrapped from the Festival TTS frontend using Unilex accent translation rules applied to LibriTTS (training, 199,612 sentences) and HiFi-TTS (test, 4,431 sentences). Only in-vocabulary words (present in Festival's lexicon) are included to minimize ground-truth errors.

**Accent similarity metric:** Levenshtein distance between accent-specific phonetizations of every unique LibriTTS word is computed for all 91 pairwise accent combinations. A normalized Levenshtein ratio per word is averaged across all unique words to yield a similarity score in [0, 1].

**Experiments:**

*Experiment 1 — Multi-accent baseline validity:* A 14-accent model is compared to two uni-accent baselines on Edinburgh (EDI) and LDS1 accents. Multi-accent models perform comparably to or better than uni-accent models on unseen word accuracy (~78–84%), confirming 14-accent joint modeling does not degrade performance.

*Experiment 2 — Scaling of knowledge transfer:* EDI is designated as the target accent. The source accent with highest similarity to EDI is ABD1 (Aberdeen, score = 0.99). A series of multi-accent models is trained with progressively reduced target accent training data (from 199,612 down to 1k and 500 sentences), while augmenting the source accent (ABD1) with the target's test sentences. Both target and non-target accent data are scaled down jointly to prevent data imbalance overfitting. Key result: 1k sentences (5,870 unique word types) achieves 89.21% unseen word accuracy — matching the 199,612-sentence ceiling baseline (88.32%). 500 sentences causes complete model failure.

*Experiment 3 — Effect of accent similarity:* The 1k-sentence experiment is repeated with a moderate-similarity pair (GAM/EDI, score = 0.94) and a low-similarity pair (GNZ/EDI, score = 0.91). High and moderate similarity pairs both succeed at 1k sentences. The low-similarity pair (GNZ/EDI) fails: a statistically significant accuracy drop (p < 0.05) occurs at 1k sentences, confirming that accent similarity weakly mediates knowledge transfer.

## Key Results

The headline result is a 95% reduction in required pronunciation training data relative to the prior SOTA of 20k sentences (Sun and Richmond 2024): 1k sentences achieves ceiling-level unseen word accuracy for high-similarity and moderate-similarity source-target accent pairs. This result holds for a 14-accent model — substantially larger than the tri-accent model from prior work, suggesting that additional intermediate accent examples strengthen knowledge transfer by providing a broader phonological bridge.

Accent similarity scores range from 0.91 (GNZ/EDI) to 0.99 (ABD1/EDI); the range is narrow but the effect of similarity on knowledge transfer is detectable. The recommendation is to select the most similar source accent when using this paradigm for new accent development.

## Novelty Assessment

The work is a direct and well-executed extension of Sun and Richmond (2024) with two novel contributions: (1) demonstrating that scaling from 3 to 14 accents strengthens knowledge transfer enough to reduce data requirements by another 95%, and (2) quantifying the accent similarity effect using a principled Levenshtein-based metric applied across all 91 pairwise accent combinations. The scale of the multi-accent experiment (14 accents, 91 similarity pairs) and the careful experimental design (joint scaling of target and non-target data) are methodological strengths. The result is empirically significant for practical low-resource TTS development.

The paper is appropriately cautious: the similarity effect is supported by only one example per similarity tier, and the bootstrapped pronunciation data may not fully model real-world annotation noise.

## Field Significance

Moderate — This paper extends a known multi-accent frontend strategy to 14 accents and demonstrates that the additional intermediate accents substantially strengthen pronunciation knowledge transfer, enabling ceiling-level performance from just 1k training sentences. The contribution is primarily empirical: the LSTM Seq2Seq architecture is unchanged from prior work, and the value lies in the systematic similarity analysis across 91 accent pairs and the practical 95% data reduction result. It provides a usable pathway for low-resource accent TTS frontend development that can serve as a reference point for future work on data-efficient non-standard accent coverage.

## Claims

- Scaling a multi-accent pronunciation frontend from three to fourteen accents substantially reduces the minimum training data required to achieve ceiling-level performance in a new target accent. *(§4.2, Table 4)*
- Accent similarity, quantified via Levenshtein distance over phoneme sequences, weakly predicts the success of cross-accent pronunciation knowledge transfer, with low-similarity pairs failing at the data threshold where high- and moderate-similarity pairs succeed. *(§4.3, Table 5)*
- Joint scaling of target and non-target accent training data is necessary to prevent overfitting when one accent is severely underrepresented in a multi-accent model. *(§4.2)*
- Bootstrapped pronunciation data derived from a rule-based TTS frontend can substitute for hand-transcribed annotations in multi-accent frontend training while maintaining high unseen word accuracy. *(§3.2, §4.1)*

## Limitations and Open Questions

Accent similarity effect is assessed with only one example per tier (low/moderate/high); replication with additional pairs would strengthen the conclusion. The experiment uses a single target accent (EDI); generalization to non-British accents or non-English languages is not tested. Pronunciation data is bootstrapped (not real human annotation), which may understate difficulties in real-world deployment where annotations are noisy. The paper addresses only the frontend (text-to-phoneme) component of TTS; it does not evaluate full end-to-end accent TTS quality in listening tests. The ethical caveat — that easy accent building could enable appropriation or mockery — is raised but not operationalized into concrete safeguards.

## Wiki Connections

- [[multilingual-tts]] — demonstrates a data-efficient pathway for building non-standard accent TTS frontends, directly relevant to the challenge of extending TTS coverage beyond standard varieties
- [[speaker-adaptation]] — the multi-accent knowledge transfer framework enables a model to adapt to new accent phonology with minimal new data, analogous to low-resource speaker adaptation
