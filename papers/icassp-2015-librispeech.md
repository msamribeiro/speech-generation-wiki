---
id: "icassp-2015-librispeech"
title: "Librispeech: An ASR corpus based on public domain audio books"
authors: ["Vassil Panayotov", "Guoguo Chen", "Daniel Povey", "Sanjeev Khudanpur"]
organization: "Johns Hopkins University"
venue: ICASSP
venue_type: conference
year: 2015
month: 4
published_date: "2015-04-19"
ingested_date: "2026-11-24"
corpus_role: dataset
ingest_tier: 2
task: []
architecture: []
conditioning: []
training: []
model_size: "not reported"
datasets_train: [LibriSpeech]
datasets_eval: [LibriSpeech, WSJ]
metrics: [WER]
code_available: true
demo_available: null
url: "https://doi.org/10.1109/ICASSP.2015.7178964"
related_concepts: []
related_papers: []
field_significance:
  level: "high"
  type: [dataset-contribution]
generation:
  schema_version: 2
  date: "2026-11-24"
  runtime: "claude-code"
  provider: "anthropic"
  agent: speech-generation-lightweight-ingest-agent
  model: "claude-opus-5-5"
  commit: "8d6e2db"
---

> [!abstract] ICASSP · 2015 · Conference
> **Panayotov et al.** (Johns Hopkins University) · [→ Paper](https://doi.org/10.1109/ICASSP.2015.7178964) · Demo: ? · Code: ✓
>
> LibriSpeech releases about 1000 hours of read English speech at 16 kHz, aligned and segmented from LibriVox public domain audiobooks, with matched language-model resources and Kaldi recipes.

> [!info] Citation Stub
> This paper is not a speech generation paper but is cited by the corpus. See Context in Speech Generation below for why it is relevant.

## Context in Speech Generation

LibriSpeech is built by decoding LibriVox audiobook chapters with a Kaldi acoustic model, locating the matching Project Gutenberg text with Smith-Waterman alignment, and rejecting any segment whose second-pass decoding (a per-segment graph with a phone-level bigram filler) deviates from the book text. The released corpus is split by an ASR-difficulty ranking into "clean" and "other" pools, with training subsets of roughly 100, 360 and 500 hours and speaker-disjoint dev and test sets of 40 (clean) and 33 (other) speakers each (Table 1). It provides a large, multi-speaker, CC BY 4.0 licensed read-speech resource and fixed speaker-disjoint test splits, which can serve speech generation work as a source of training audio and as held-out speakers for intelligibility and speaker-similarity evaluation. Its design targets recognition rather than synthesis: audio is 16 kHz MP3-derived, training data is split at silences longer than 0.3 seconds rather than at sentence boundaries, and transcripts are upper-cased with punctuation removed (§2.1, §2.4).

## Wiki Connections

- [[1904.02882|LibriTTS]] — Reprocesses the same LibriVox and Project Gutenberg source material into a 24 kHz, sentence-segmented TTS corpus, directly addressing LibriSpeech's sampling-rate, segmentation and text-normalisation limitations for synthesis.
- [[2305.18802|LibriTTS-R]] — Applies speech restoration to LibriTTS, extending the LibriSpeech-derived lineage of audiobook corpora toward higher-quality TTS training data.
- [[2301.02111|VALL-E]] — Uses LibriSpeech test-clean as its zero-shot TTS evaluation set for WER, speaker similarity and subjective scores.
