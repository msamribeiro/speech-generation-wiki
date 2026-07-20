---
id: "interspeech-2025-2031"
title: "Kinship in Speech: Leveraging Linguistic Relatedness for Zero-Shot TTS in Indian Languages"
authors: ["Utkarsh Pathak", "Chandra Sai Krishna Gunda", "Anusha Prakash", "Keshav Agarwal", "Hema A. Murthy"]
organization: "Indian Institute of Technology, Madras"
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-07-04
task: ["TTS"]
architecture: ["transformer-enc-dec"]
conditioning: ["text-conditioned", "zero-shot", "multilingual"]
training: ["supervised"]
model_size: "not reported"
codec_used: "none"
datasets_train: ["IIIT-H TTS (Indian Languages)"]
datasets_eval: ["Sanskrit test set", "Konkani test set (Maharashtrian and Canara)", "Maithili test set", "Kurukh test set"]
metrics:
  - name: MOS
    value: 4.12
    system: Kannada TTS (DR rules)
    testset: Sanskrit zero-shot
  - name: MOS
    value: 4.02
    system: Hindi TTS (IA rules)
    testset: Kurukh zero-shot
  - name: WER
    value: 4.6
    system: Kannada TTS (DR rules)
    testset: Sanskrit SUS
code_available: null
demo_available: true
url: "https://www.isca-archive.org/interspeech_2025/pathak25_interspeech.html"
related_concepts: [multilingual-tts, zero-shot-tts, evaluation-metrics, subjective-evaluation, gan-vocoder]
related_papers: []
field_significance:
  level: "moderate"
  type: ["engineering-integration"]
generation:
  date: 2026-07-04
  agent: speech-generation-ingest-agent
  model: claude-sonnet-4-6
  commit: "318f1b4"
---

> [!abstract] Interspeech · 2025 · Conference
> **Pathak et al.** (Indian Institute of Technology, Madras) · [→ Paper](https://www.isca-archive.org/interspeech_2025/pathak25_interspeech.html) · Demo: ✓ · Code: ?
>
> Zero-shot TTS for five low-resource Indian languages by selecting G2P parsing rules according to the phonotactic family of the target language rather than its script family, enabling cross-family synthesis without any adaptation data.

## Problem

India has 22 official languages using 13 scripts, and most of its 1,369 documented languages lack the digital resources needed to train dedicated TTS systems. Existing multilingual TTS frameworks depend on large-scale datasets that under-represent most Indian languages, and even the official languages are largely absent from large multilingual models. Beyond data scarcity, a structural problem compounds the challenge: some Indian languages use a script from one language family while their phonotactics belong to another. Direct application of script-family-based G2P rules to such languages produces systematic errors, most notably incorrect schwa deletion in languages that preserve the schwa despite using an Indo-Aryan script.

## Method

The approach builds on two pre-existing components from the IIT Madras speech synthesis group: the Common Label Set (CLS), a 72-token shared phoneme inventory mapping acoustically similar phones across 13 Indian languages, and a unified G2P parser with separate rule sets for Indo-Aryan (IA) and Dravidian (DR) scripts. Monolingual FastSpeech2 acoustic models and HiFiGAN vocoders are trained on 10 hours each of studio data for Hindi, Kannada, Marathi, and Telugu.

The key contribution is a three-stage methodology for cross-family zero-shot synthesis. First, the CLS is extended to cover sounds absent from DR scripts, such as nukta-marked consonants, by mapping them to the nearest phonetically equivalent base character. Second, G2P rules are selected based on the phonotactic family of the target language rather than its script family. For example, Sanskrit is written in Devanagari (IA script) but retains the schwa and lacks schwa deletion, so DR rules are applied. Similarly, Canara Konkani uses Kannada script but its rule assignment is guided by the dialect's phonotactic profile. Third, the source-language synthesiser is chosen by combining the rule-family assignment with a broader assessment of phonological and lexical overlap.

Five target languages are evaluated in this way: Sanskrit (classical, IA family but DR-like phonotactics), Maharashtrian Konkani and Canara Konkani (two dialects with different phonotactic affiliations), Maithili (IA, Devanagari), and Kurukh (Dravidian, Devanagari script but with borrowed IA features including schwa deletion).

## Key Results

MOS evaluations by native speakers (18-50 age range) on a five-point scale demonstrate that phonotactic-aware rule selection consistently outperforms script-based rule assignment. For Sanskrit, the Kannada system with DR rules achieves MOS 4.12 against the Hindi system with IA rules at 3.25 (ground truth: 4.48). For Canara Konkani, the Kannada system (DR rules) scores 3.34 versus the Marathi system at 2.31; the pattern reverses for Maharashtrian Konkani, where the Marathi system reaches 3.64 and Kannada falls to 2.54. Maithili with Hindi (IA rules) reaches 3.72, and Kurukh with Hindi achieves 4.02 versus Kannada at 2.96 (GT: 4.28).

SUS intelligibility tests (WER on semantically unpredictable sentences) confirm the same ordering. Sanskrit with the Kannada system achieves 4.6% WER, compared to 28.0% for the Hindi system, with the large gap traceable to systematic schwa deletion errors. Mel-cepstral distortion scores independently corroborate the trend: for Sanskrit, Telugu and Kannada systems reach MCD 6.94 and 7.15 respectively, versus 8.48 for Hindi. Notably, for Maharashtrian Konkani, the Marathi system yields a higher MCD (10.74) than Kannada (8.28) despite better MOS, an anomaly the authors flag as warranting further investigation.

All evaluations are subjective or signal-level; there is no comparison against other zero-shot multilingual TTS systems such as VALL-E X or MMS on the same target languages.

## Novelty Assessment

The contribution is primarily methodological engineering. The CLS representation, the unified G2P parser, the FastSpeech2 acoustic model, and HiFiGAN vocoder are all pre-existing components from prior work. What is genuinely new is the insight that phonotactic family, not script family, should govern rule selection for languages where these two properties diverge, and the systematic application of this principle to five Indian languages spanning two language families and two scripts. The CLS extension (adding missing phones via phonetic substitution) is incremental but necessary for the methodology to function on Dravidian-script targets.

The evaluation is credible for what it measures: native speakers evaluate synthesised audio for naturalness (MOS) and intelligibility (SUS), and MCD provides an objective cross-check. The main limitation is the absence of a competitive baseline system such as a large multilingual model applied without phonotactic-aware adaptation, which would more directly quantify the gains from the proposed rule-selection strategy.

## Field Significance

Moderate - This paper addresses a clearly defined and underserved sub-problem in multilingual TTS: cross-family zero-shot synthesis for languages whose script and phonotactic family diverge due to historical borrowing. The language-centric methodology is practically replicable and the results are compelling for all five target languages. The approach is most directly relevant to the Indian language TTS community, but the general principle of phonotactic-over-script rule assignment extends to similar situations elsewhere (African and Southeast Asian language scripts with analogous mismatches).

## Claims

- **supports:** For zero-shot TTS across language families, assigning G2P rules by phonotactic family rather than script family produces substantially more natural and intelligible synthesis.
  > *Evidence:* Sanskrit synthesis with Dravidian rules (Kannada system, MOS 4.12, WER 4.6%) substantially outperforms synthesis with Indo-Aryan rules (Hindi system, MOS 3.25, WER 28.0%), because schwa deletion applied via IA rules violates Sanskrit's phonotactics despite its Devanagari script. *(§5.3, Tables 1, 2)*

- **supports:** A shared phoneme inventory extended via phonetic substitution can enable zero-shot synthesis for target languages with sounds absent from all source training sets.
  > *Evidence:* The CLS was extended to represent nukta-diacritic consonants and borrowed aspiration in Kurukh by mapping them to nearest-neighbour base characters, enabling intelligible synthesis of Kurukh without any Kurukh training data. *(§4.2)*

- **supports:** Dialect-specific zero-shot TTS requires matching the source synthesiser to the dialect's phonotactic affiliation, not its surface script or nominal language grouping.
  > *Evidence:* Two Konkani dialects yield opposite synthesiser preferences: Maharashtrian Konkani (IA phonotactics) is best with Marathi at MOS 3.64, while Canara Konkani (DR phonotactics) is best with Kannada at MOS 3.34, despite both dialects being classified under the same language label. *(§5.3, Table 1)*

- **complicates:** Cross-family zero-shot synthesis degrades for target languages with phonological features structurally absent from all available source synthesisers.
  > *Evidence:* Kurukh synthesis struggles specifically with glottal stops across both Hindi and Kannada systems, and question intonation contours were misrendered as assertions in some utterances, because neither source language encodes these features. *(§5.3)*

- **refines:** Linguistic proximity for zero-shot TTS transfer is captured more precisely by phonotactic family than by script family or geographic proximity.
  > *Evidence:* Sanskrit belongs to the IA family but achieves lower MOS with IA synthesisers (3.25) than with DR synthesisers (4.12), and MCD scores independently confirm the same pattern (IA: 8.48 vs. DR: 7.15/6.94). Phonotactic family membership predicts transfer quality more reliably than script or family taxonomy. *(§5.3, Tables 1, 3)*

## Limitations and Open Questions

The evaluation relies exclusively on four monolingual source synthesisers (Hindi, Kannada, Marathi, Telugu), and the method is not tested against any large multilingual TTS baseline that might implicitly have learned cross-family phonotactics from scale. For Kurukh, the ground truth audio is Bible recordings with faint background music (cleaned with VoiceFixer), which may depress the GT MOS and make the synthesised system comparisons harder to interpret. The approach requires linguistic expertise to correctly classify the phonotactic family of a new target language, which limits its applicability as an automated pipeline for new low-resource languages.

The anomalous MCD result for Maharashtrian Konkani (higher distortion with Marathi despite better naturalness MOS) is unresolved and may indicate that MCD is less reliable for measuring synthesis quality under phonotactic family mismatch conditions.

## Wiki Connections

- [[multilingual-tts|Multilingual TTS]] - contributes a phonotactic-family methodology for extending zero-shot synthesis to low-resource Indian languages across script-phonotactic mismatches.
- [[zero-shot-tts|Zero-Shot TTS]] - demonstrates zero-shot synthesis without any target-language adaptation data by exploiting linguistic relatedness between source and target language families.
- [[evaluation-metrics|Evaluation Metrics]] - evaluates using MOS (naturalness), WER via SUS intelligibility tests, and MCD as an objective distortion measure across five zero-shot target languages.
- [[subjective-evaluation|Subjective Evaluation]] - employs native speaker panels (6-25 raters per language) for both MOS naturalness ratings and SUS intelligibility tests, covering five distinct low-resource Indian languages.
- [[gan-vocoder|GAN Vocoder]] - uses HiFiGAN as the vocoder for all four monolingual source TTS systems on which zero-shot cross-family synthesis depends.
