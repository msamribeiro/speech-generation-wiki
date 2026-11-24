---
id: "interspeech-2017-1386"
title: "Montreal Forced Aligner: Trainable Text-Speech Alignment Using Kaldi"
authors: ["Michael McAuliffe", "Michaela Socolof", "Sarah Mihuc", "Michael Wagner", "Morgan Sonderegger"]
organization: "McGill University / University of Maryland"
venue: Interspeech
venue_type: conference
year: 2017
month: 8
published_date: "2017-08-20"
ingested_date: "2026-11-24"
corpus_role: evaluation
ingest_tier: 2
task: []
architecture: []
conditioning: []
training: []
model_size: "not reported"
datasets_train: [LibriSpeech, Buckeye, Phonsay]
datasets_eval: [Buckeye, Phonsay]
metrics: []
code_available: true
demo_available: null
url: "https://www.isca-archive.org/interspeech_2017/mcauliffe17_interspeech.html"
related_concepts: []
related_papers: []
field_significance:
  level: "moderate"
  type: [engineering-integration, empirical-benchmark]
generation:
  schema_version: 2
  date: "2026-11-24"
  runtime: "claude-code"
  provider: "anthropic"
  agent: speech-generation-lightweight-ingest-agent
  model: "claude-opus-5-5"
  commit: "8d6e2db"
---

> [!abstract] Interspeech · 2017 · Conference
> **McAuliffe et al.** (McGill University / University of Maryland) · [→ Paper](https://www.isca-archive.org/interspeech_2017/mcauliffe17_interspeech.html) · Demo: ? · Code: ✓
>
> Introduces the Montreal Forced Aligner, an open-source, trainable Kaldi-based tool that aligns speech with its transcript at the word and phone level using speaker-adapted triphone GMM-HMM acoustic models.

> [!info] Citation Stub
> This paper is not a speech generation paper but is cited by the corpus. See Context in Speech Generation below for why it is relevant.

## Context in Speech Generation

MFA takes audio plus an orthographic transcript and a pronunciation lexicon and outputs word and phone boundaries as Praat TextGrids, using a GMM-HMM pipeline (monophone, then triphone, then speaker-adapted triphone training with fMLLR) over 39-dimensional MFCC features (§2). It ships with an English acoustic model trained on LibriSpeech, can be retrained from scratch on the corpus to be aligned, and models out-of-dictionary words with a dedicated unknown-word phone (§2). On conversational English (Buckeye), the LibriSpeech-trained model gives a mean word-boundary error of 24.1 ms and a mean phone-boundary error of 17 ms against manual annotation, with lower mean errors than the LibriSpeech-trained monophone Prosodylab-Aligner on every boundary type and dataset (Table 2). For speech generation, these phone-level boundaries can supply per-phoneme duration targets for training duration predictors, as well as phoneme-to-frame alignments for text-speech interleaving or alignment-constrained decoding.

## Wiki Connections

- [[2006.04558|FastSpeech 2]] — uses MFA phoneme durations in place of teacher-model durations to train its duration predictor, reporting a CMOS gain from the more accurate alignments.
- [[2401.07333|ELLA-V]] — uses MFA alignments between transcripts and EnCodec tokens to interleave phonemes with their corresponding acoustic tokens.
- [[2406.07855|VALL-E R]] — uses MFA phoneme-to-frame alignments as teacher-forcing targets for its jointly predicted phoneme stream.
- [[2502.18924|MegaTTS 3]] — depends on MFA at training time to derive the sparse alignment anchors for its latent diffusion transformer.
- [[2109.13821|Diffusion-Based VC]] — aligns LibriTTS with MFA to build per-phoneme "average voice" mel targets for its encoder.
