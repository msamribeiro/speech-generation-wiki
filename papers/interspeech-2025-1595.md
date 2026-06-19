---
id: "interspeech-2025-1595"
title: "Scheduled Interleaved Speech-Text Training for Speech-to-Speech Translation with LLMs"
authors: ["Hayato Futami", "Emiru Tsunoo", "Yosuke Kashiwagi", "Yuki Ito", "Hassan Shahmohammadi", "Siddhant Arora", "Shinji Watanabe"]
organization: Sony Group Corporation
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-06-03
task: [SCA]
architecture: [autoregressive-LM]
conditioning: [multilingual]
training: [fine-tuning, supervised]
model_size: "1B params (LLaMA 3.2-1B)"
codec_used: "none (semantic units from w2v-BERT)"
datasets_train: ["CVSS-C", "CoVoST2"]
datasets_eval: ["CVSS-C"]
metrics:
  - name: UTMOS
    value: 4.23
    system: proposed (Scheduled ILT)
    testset: CVSS-C Fr-en
  - name: UTMOS
    value: 4.21
    system: baseline
    testset: CVSS-C Fr-en
code_available: null
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/futami25_interspeech.html"
related_concepts: ["spoken-language-model", "self-supervised-speech", "multilingual-tts", "speech-to-speech", "neural-codec"]
related_papers: []
field_significance:
  level: moderate
  type: [engineering-integration]
generation:
  date: 2026-06-03
  model: claude-sonnet-4-6
  commit: "55d0339"
---

> [!abstract] Interspeech · 2025 · Conference
> **Futami et al.** (Sony Group Corporation) · [→ Paper](https://www.isca-archive.org/interspeech_2025/futami25_interspeech.html) · Demo: ? · Code: ?
>
> A scheduled interleaved speech-text training strategy for fine-tuning text LLMs on speech-to-speech translation, in which word-aligned text tokens are gradually replaced by speech units over training to ease the modality transition.

## Problem

Fine-tuning text-only LLMs for speech generation tasks is hampered by a modality gap: text and speech units differ substantially in sequence length (speech units can be 13–21 times longer than the corresponding text) and in learned representation space. When supervised S2ST data is limited, this gap makes cross-modal adaptation particularly difficult and can leave LLMs unable to adequately model speech unit sequences. Prior work (AudioPaLM) partially addressed this by fine-tuning the SSL encoder for ASR to bring speech representations closer to text, but the modality gap at the LLM level remained unaddressed during fine-tuning itself.

## Method

The system builds on the AudioPaLM framework: a pre-trained LLaMA 3.2-1B LLM is fine-tuned end-to-end on semantic speech units for S2ST, with chain-of-thought (CoT) prompting that includes ASR transcription and text translation as intermediate steps. Semantic units are derived from a w2v-BERT encoder fine-tuned for CTC-based ASR, clustered into 2048 categories via k-means on 21-language CoVoST+CVSS data. Waveform output is produced by a unit-based HiFi-GAN vocoder conditioned on those same semantic units.

The key contribution is scheduled interleaved speech-text training (Scheduled ILT). At training start, a high proportion (p = 0.9) of speech units in both the source and target sequences are replaced with their word-aligned BPE text equivalents. Word alignments are extracted from the CTC decoder of the fine-tuned w2v-BERT without any external aligner. Replacement spans are selected randomly with Poisson-distributed lengths. The text ratio p is decayed by 0.1 every 300 steps, so training begins in an almost text-native regime and progressively transitions to pure speech units by the end. Interleaving is applied to both the input and output sides simultaneously; ablations show that one-sided interleaving is substantially less effective.

## Key Results

On seven language pairs from CVSS-C, Scheduled ILT consistently improves ASR-BLEU over the ASR-fine-tuned baseline without degrading UTMOS audio quality. The largest gains appear for low-resource language pairs: Portuguese-to-English improves from 10.3 to 19.5 ASR-BLEU (+89%), Russian-to-English from 6.0 to 14.1 (+135%), and Italian-to-English from 12.8 to 19.5 (+52%). High-resource languages show smaller but consistent gains (French-to-English: 28.8 → 29.5, Spanish-to-English: 33.5 → 33.9). UTMOS scores remain in the 4.17–4.28 range throughout, indicating that the gains are in translation accuracy rather than perceptual speech quality.

Ablation studies on Portuguese-to-English confirm that interleaving both input and output sides is necessary (one-sided variants score 11.1–12.6 ASR-BLEU vs. 19.5); that replacing units with a mask token rather than aligned text yields only 6.2 ASR-BLEU; and that alignment-free uniform-interval interleaving reduces performance to 11.5 ASR-BLEU. ILT without scheduling underperforms the baseline in several languages (French-to-English: 25.8 vs. 28.8), indicating that a constant text ratio is insufficient or harmful.

## Novelty Assessment

The contribution is a training procedure, not a model architecture. Interleaved speech-text representations were established by SpiRitLM for large-scale pre-training; this paper adapts the concept to supervised fine-tuning of a modest-size LLM on limited data and adds a decay schedule over the text ratio. The innovation is incremental but well-motivated and practically important: the scheduling mechanism addresses the specific challenge of adapting a text LLM with a small dataset, and the empirical gains — especially for low-resource languages — are substantial. The work does not introduce new model components, new codecs, or new evaluation methodologies. Reproducibility is reasonable, using publicly available CVSS data and LLaMA 3.2-1B, though code is not released.

## Field Significance

Moderate — This paper extends the idea of interleaved speech-text training from large-scale pre-training to low-data fine-tuning, with a scheduling mechanism that meaningfully narrows the modality gap in LLM-based S2ST. The practical value — consistent improvements across seven languages, largest for low-resource settings — is clear, but the contribution is an engineering adaptation of an existing technique rather than a new direction. The study corroborates the growing view that gradual modality transition is a reliable strategy when adapting text LLMs to speech generation tasks.

## Claims

- Gradual modality transition during fine-tuning, rather than abrupt full-speech training, improves LLM adaptation from text to speech units.
- The benefit of interleaved speech-text training is amplified in low-resource language settings where speech-domain supervision is scarce.
- Applying word-aligned text interleaving to both source and target sequences simultaneously is necessary; one-sided interleaving provides substantially weaker adaptation.
- The modality gap between speech and text in LLM-based S2ST manifests as both a length disparity and a representation distance, and scheduled training addresses both.

## Limitations and Open Questions

> [!warning]
> The system uses a unit-based HiFi-GAN vocoder that maps semantic units to single-speaker synthesized English speech (from CVSS-C, which uses a canonical TTS voice). Speaker identity is not preserved, so the evaluation reflects translation accuracy and audio naturalness but not voice characteristics. Results may not generalise to multi-speaker or voice-preserving S2ST settings.

The evaluation is limited to seven language pairs from a single corpus (CVSS-C) and uses a 1B-parameter LLM. It is unclear whether the scheduling strategy scales to larger models or to more diverse multilingual corpora. The method relies on CTC-based word alignments, which requires a separately fine-tuned SSL encoder; its applicability when high-quality alignments are unavailable is not explored. The paper also does not compare against systems using acoustic (codec) units for speech generation, which would require a different vocoder setup. Future work suggested by the authors includes extension to spoken dialogue systems.

## Wiki Connections

This paper applies [[spoken-language-model]] fine-tuning to end-to-end [[speech-to-speech]] translation. It relies on [[self-supervised-speech]] representations (w2v-BERT semantic units) and a unit HiFi-GAN vocoder. The multilingual coverage across seven language pairs links to [[multilingual-tts]] practice.
