---
id: "interspeech-2025-raju25_interspeech"
title: "End-to-End Indian Language Dubbing with Zero-Shot Speaker Preservation"
authors: ["Giri Raju", "Sandeep Konam"]
organization: "Hitloop"
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-07-13
task: ["TTS"]
architecture: ["flow-matching"]
conditioning: ["text-conditioned", "speaker-conditioned", "zero-shot", "multilingual"]
training: ["fine-tuning", "supervised"]
model_size: "not reported"
codec_used: "none"
datasets_train: ["IndicTTS", "LIMMITS", "SnowMountain", "Rasa", "IndicVoices-R", "in-house dataset (79.2h, 70,767 utterances)"]
datasets_eval: ["not reported"]
metrics: []
code_available: null
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/raju25_interspeech.html"
related_concepts: ["zero-shot-tts", "multilingual-tts", "flow-matching", "speaker-adaptation"]
related_papers: ["2025.acl-long.313"]
field_significance:
  level: "low"
  type: ["engineering-integration"]
generation:
  date: 2026-07-13
  agent: speech-generation-ingest-agent
  model: claude-sonnet-4-6
  commit: "b035ebb"
---

> [!abstract] Interspeech · 2025 · Conference
> **Raju and Konam** (Hitloop) · [→ Paper](https://www.isca-archive.org/interspeech_2025/raju25_interspeech.html) · Demo: ? · Code: ?
>
> An end-to-end dubbing platform for Indian languages that integrates ASR, machine translation, and a fine-tuned F5-TTS model to automate cross-lingual content localisation with zero-shot speaker identity preservation.

## Problem

Dubbing content into India's many regional languages is expensive, slow, and dependent on specialist voice talent. Existing TTS solutions for Indic languages are limited in speaker diversity and cannot preserve the original speaker's vocal characteristics at inference time, breaking the identity coherence that is important for both entertainment and educational content.

## Method

The platform chains six sequential steps: audio extraction with source separation, transcription (Whisper-style ASR), speaker diarisation, machine translation (IndicTrans2 for Indic language groups), TTS synthesis, and output re-encoding that merges synthesized vocals with the original non-vocal audio tracks.

The synthesis component is a fine-tuned version of F5-TTS, a flow-matching model that performs zero-shot speaker-conditioned generation from a short reference clip. To support Indic languages, the base F5-TTS vocabulary was expanded to cover Indian language scripts and character-based tokenisation was replaced with phoneme-based tokenisation to improve pronunciation accuracy. A Vocos vocoder was trained from scratch on Indic-language data for 2 million steps to convert mel spectrograms to audio.

Training data totals approximately 1,626 hours (737,632 utterances) across English, Bengali, Gujarati, Hindi, Kannada, Malayalam, Marathi, Punjabi, Tamil, and Telugu, drawn from five open-source corpora (IndicTTS, LIMMITS, SnowMountain, Rasa, IndicVoices-R) plus an in-house set of 79.2 hours from native speakers of six languages. The F5-TTS model was fine-tuned for 1.84 million steps on two NVIDIA L40 GPUs.

At inference, the system can use one of three speaker reference sources: the separated source vocal track, an externally uploaded reference clip, or a preset voice. Users can toggle duration alignment (constraining output to match source timestamps for lip-sync), adjust speech rate between 0.5x and 1.5x, and apply loudness and pitch controls per utterance.

## Key Results

No formal evaluation against quantitative baselines is reported. The paper characterises system quality through deployment evidence: the platform is described as in active use by content creators and educators. No MOS, WER, or speaker similarity scores are provided.

## Novelty Assessment

The contribution is engineering integration. F5-TTS, Vocos, Whisper-class ASR, IndicTrans2 MT, and pyannote-based speaker diarisation are all established components assembled into a production dubbing pipeline. The adaptation of F5-TTS to Indic languages through vocabulary expansion and phoneme-based tokenisation represents a practical extension rather than an architectural innovation. The absence of formal evaluation makes it impossible to assess quality gains against prior multilingual TTS baselines or to understand which of the ten supported languages perform reliably.

## Field Significance

Low — a system description paper demonstrating practical deployment of a multilingual dubbing pipeline for Indian languages. It provides evidence that flow-matching TTS can be adapted to low-resource language families through fine-tuning, but without quantitative evaluation the contribution is primarily an existence proof that such a system can be built and deployed, rather than a characterisation of where it succeeds or fails.

## Claims

- **supports:** Multilingual adaptation of flow-matching TTS through vocabulary expansion and phoneme-based tokenisation can extend zero-shot speaker preservation to typologically diverse language families not covered by the base model.
  > *Evidence:* F5-TTS fine-tuned on 1,626 hours across English and 9 Indian languages with an expanded vocabulary and phoneme-based tokenisation achieves zero-shot speaker-conditioned synthesis across all supported languages. *(§2.2.2)*

- **supports:** End-to-end automated dubbing pipelines integrating ASR, machine translation, and zero-shot TTS can substantially reduce manual effort in cross-lingual content localisation.
  > *Evidence:* The platform automates six sequential stages from audio extraction to output re-encoding, supports 9 Indian languages, and has been adopted by content creators and educators without requiring specialist voice talent. *(§2.1, §3)*

- **complicates:** Duration alignment for lip-synchronisation in automated dubbing requires a user-controlled trade-off between temporal precision and naturalness that cannot be resolved automatically.
  > *Evidence:* Duration alignment is a toggleable feature; when active it constrains synthesis duration to source timestamps, potentially affecting naturalness; when disabled, utterances are produced sequentially with a fixed 1-second pause between them. *(§2.3)*

## Limitations and Open Questions

No quantitative evaluation is reported, making it impossible to verify speaker similarity, naturalness, or ASR accuracy in the dubbed output. The paper does not compare against any TTS or dubbing baseline. Coverage is uneven: the in-house training data covers only six of the ten supported languages, and the paper does not break out performance by language. Cross-lingual expressivity is flagged as a limitation by the authors, with the acknowledgement that nuanced prosodic transfer across typologically distant language pairs remains unsolved.

## Wiki Connections

- [[zero-shot-tts|Zero-Shot TTS]] — the platform's speaker preservation capability relies on zero-shot conditioning, using a short reference clip at inference time to transfer speaker identity without per-speaker fine-tuning.
- [[multilingual-tts|Multilingual TTS]] — the system targets 10 languages (English plus 9 Indian languages) and is the primary design constraint driving the training data composition and tokenisation choices.
- [[flow-matching|Flow Matching]] — the synthesis backbone is a fine-tuned F5-TTS model, which uses flow matching as its generative framework.
- [[speaker-adaptation|Speaker Adaptation]] — zero-shot speaker preservation through reference audio conditioning is the core TTS feature enabling identity transfer in the dubbing pipeline.
- [[2025.acl-long.313|F5-TTS]] — the synthesis component is built directly on F5-TTS, which is fine-tuned on Indic language data with vocabulary expansion and phoneme-based tokenisation.
