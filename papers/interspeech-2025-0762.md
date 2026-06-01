---
id: interspeech-2025-0762
title: "Intrasentential English in Swedish TTS: perceived English-accentedness"
authors: ["Christina Tannander", "David House", "Jonas Beskow", "Jens Edlund"]
organization: KTH Royal Institute of Technology / Swedish Agency for Accessible Media (MTM)
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-05-30
task: [TTS]
architecture: [flow-matching]
conditioning: [text-conditioned, multilingual, prosody-conditioned]
training: [supervised]
model_size: "not reported"
codec_used: none
datasets_train: ["20,133 Swedish sentences + 8,243 English sentences (single professional Swedish speaker)"]
datasets_eval: ["Custom stimuli: 20–40 Swedish sentences with English insertions"]
metrics:
  - name: MOS
    value: "not reported"
    system: "not reported"
    testset: "not reported"
code_available: null
demo_available: null
url: https://www.isca-archive.org/interspeech_2025/tannander25_interspeech.html
related_concepts: [multilingual-tts, prosody-control, evaluation-metrics]
related_papers: []
---
> [!abstract] Interspeech · 2025 · Conference
> **Christina Tannander et al.** (KTH Royal Institute of Technology / Swedish Agency for Accessible Media (MTM)) · [→ Paper](https://www.isca-archive.org/interspeech_2025/tannander25_interspeech.html) · Demo: ? · Code: ?
>
> A Swedish TTS voice built on Matcha-TTS with a continuous English-accentedness conditioning parameter (EA) is mapped to a perceptually calibrated scale (PEA) via psychometric function estimation, and listener preference tests show that preferred accentedness for intrasentential English insertions varies with the type of insertion.

## Problem

Swedish text frequently contains intrasentential English words — proper names, specialized terms, loanwords — that human readers pronounce with varying degrees of English accent depending on context. Existing Swedish TTS voices either ignore this variation (producing fully Swedish pronunciations) or must switch to an English voice for longer passages. No prior system provides perceptually calibrated, continuous control over the degree of English-accentedness for such insertions.

## Method

The work proceeds in four stages:

**1. TTS training**: Matcha-TTS (a flow-matching TTS model) is extended with a per-phoneme English-accentedness parameter EA ∈ [0,1], mapped through a linear layer to the same 192-dimensional embedding space as phonetic input. Training uses 20,133 Swedish sentences (EA=0) and 8,243 English sentences (EA=1) from a single professional female Swedish speaker with a mild English accent (~1,599 epochs, HiFi-GAN vocoder). Swedish sentences containing xenophones (foreign-specific phonemes) are excluded to prevent unintended accent triggers.

**2. Psychometric mapping (MAP-SV, MAP-EN)**: A recursive binary-search algorithm generates stimulus pairs differing only in EA value, asks listeners to rank them by perceived accentedness, and fits a sigmoid to the cumulative perception data. This yields a mapping from EA to Perceived English-Accentedness (PEA). For Swedish-accented English, only 3 distinguishable levels exist; for English-accented Swedish, at least 9 significant distinctions were found (step size ~0.11), with R² = 0.983 after normalization.

**3. Transcription/PEA alignment (TEA-ALIGN)**: A Best-Worst listening test with 16 Swedish native speakers tests 20 Swedish carrier phrases with English insertions across four conditions (Swedish or English transcription × PEA=0 or PEA=1). Aligned conditions (sv-0, en-1) receive significantly more "best" votes than misaligned (sv-1, en-0), confirming that transcription style must match accentedness.

**4. Preference variation (PREF-VAR)**: 32 listeners rate 40 sentences at four PEA settings (sv-0.00, sv-0.33, en-0.67, en-1.00) using Best-Worst design. Results show no single PEA setting is uniformly preferred: well-known names (e.g., Dolly Parton) are preferred in Swedish; English technical terms (e.g., backpropagation, theory of mind) in English. More fine-grained intermediate values also attract substantial preference.

## Key Results

- The sigmoid psychometric function for English-accented Swedish achieves R² = 0.983, validating the PEA scale.
- For MAP-SV: at least 9 perceptually distinguishable EA levels within [0,1]; for MAP-EN: only 3.
- Aligned transcription+PEA conditions receive significantly more "best" votes (χ²(2, N=630) = 50.2, p < .05).
- No single PEA setting dominates across all insertion types in PREF-VAR; both fully Swedish (sv-0.00) and intermediate English (en-0.67) receive notable best-vote counts.

## Novelty Assessment

The psychometric calibration approach — using recursive binary search to fit a sigmoid from perceptual data — is methodologically careful and produces a principled mapping from an engineering parameter to perception, which is relatively rare in TTS controllability work. The finding that listener preferences are insertion-type-dependent (rather than uniformly preferring one accentedness level) is a useful empirical result for practical Swedish TTS deployment. The architecture contribution (adding a per-phoneme EA scalar to Matcha-TTS) is straightforward. The work is narrow in scope: one speaker, one language pair, small listener panels (16–32 participants per experiment).

## Limitations and Open Questions

- Single speaker; generalization to other Swedish voices or to multilingual models with many speakers is unclear.
- Listener panels are small (16–32 per experiment), and the paper acknowledges insufficient power for detailed analysis of insertion-type effects.
- Only one language pair (Swedish/English); the methodology could generalize to other code-switching scenarios but was not tested.
- The recursive psychometric fitting procedure requires significant respondent time per EA comparison set.
- The matrix result is missing one sentence due to a technical failure.

## Wiki Connections

This paper informs [[multilingual-tts]] by providing an empirically validated approach to controlling accentedness in code-switching TTS scenarios, with a novel psychometric calibration methodology. It relates to [[prosody-control]] through the perceptual mapping of a continuous conditioning parameter to listener perception. The Best-Worst evaluation design and psychometric function methodology contribute to [[subjective-evaluation]]. The base architecture (Matcha-TTS) connects this work to [[flow-matching]].
