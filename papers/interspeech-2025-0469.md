---
id: "interspeech-2025-0469"
title: "Developing High-Quality TTS for Punjabi and Urdu: Benchmarking against MMS Models"
authors: ["Fatima Naseem", "Maham Sajid", "Farah Adeeba", "Sahar Rauf", "Asad Mustafa", "Sarmad Hussain", "Faisal Kamiran"]
organization: "University of Engineering and Technology, Lahore (KICS)"
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: "2025-08-17"
ingested_date: "2026-05-30"
task: [TTS, evaluation]
architecture: [transformer-enc-dec]
conditioning: [text-conditioned, speaker-conditioned]
training: [supervised]
model_size: "Tacotron 1: 6.9M params; Tacotron 2 + WaveGlow: 28M + 268M params"
codec_used: none
datasets_train: ["custom Punjabi Shahmukhi corpus (20 hrs, 14,450 utterances)", "custom Urdu corpus (10 hrs/gender, 8,081 utterances)"]
datasets_eval: ["custom Punjabi Shahmukhi test set", "custom Urdu test set"]
metrics:
  - name: WER
    value: 16.1
    system: Tacotron2+WaveGlow
    testset: Punjabi Male
  - name: WER
    value: 31.4
    system: MMS
    testset: Punjabi Male
  - name: WER
    value: 13.1
    system: Tacotron1
    testset: Urdu Male
  - name: WER
    value: 25.6
    system: MMS
    testset: Urdu Male
  - name: CER
    value: 9.1
    system: Tacotron2+WaveGlow
    testset: Punjabi Male
  - name: CER
    value: 16.1
    system: MMS
    testset: Punjabi Male
code_available: null
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/naseem25_interspeech.html"
related_concepts: [multilingual-tts, evaluation-metrics, subjective-evaluation]
related_papers: []
---
> [!abstract] Interspeech · 2025 · Conference
> **Fatima Naseem et al.** (University of Engineering and Technology, Lahore (KICS)) · [→ Paper](https://www.isca-archive.org/interspeech_2025/naseem25_interspeech.html) · Demo: ? · Code: ?
>
> This paper develops dedicated Tacotron-based TTS systems for Punjabi Shahmukhi and Urdu, including new speech corpora and phonetic lexicons, and demonstrates they substantially outperform Meta's MMS models in intelligibility, naturalness, and phonetic accuracy.

## Problem

Existing TTS solutions for Punjabi focus exclusively on the Gurumukhi script; serving the Shahmukhi script requires transliteration, which introduces letter substitutions and omissions that degrade synthesis quality. Urdu TTS systems similarly struggle with language-specific numbers, symbols, and special characters, causing omissions in synthesized speech. Meta's Massively Multilingual Speech (MMS) system, despite covering both languages, relies on character-to-sound mappings that are insufficient for accurate phoneme generation, and does not natively support Punjabi Shahmukhi at all. There was no dedicated, phoneme-based, script-aware TTS pipeline for either language.

## Method

The authors developed three components for each language: (1) a text corpus, (2) a phonetic lexicon, and (3) a text analysis module handling numbers, special characters, and out-of-vocabulary words via letter-to-sound (LTS) rules.

**Data:** For Punjabi, text was sourced from news sites, books, and journals across seven domains; professional male speakers recorded ~20 hours (14,450 utterances, Majhi accent) in a soundproof environment at 48 kHz. For Urdu, both male and female corpora of ~10 hours each (8,081 utterances) were constructed from an existing Urdu news corpus. Phonetic lexicons were produced by expert linguists: 184k entries for Urdu (CISAMPA format) and 16k for Punjabi Shahmukhi.

**Models:** Two architectures were trained on each dataset: (a) Tacotron 1 (encoder-decoder attention TTS, ~6.9M params) with Griffin-Lim vocoder, and (b) Tacotron 2 (~28M params) with WaveGlow (~268M params). Both were trained for 1,500 epochs with batch size 32 on dual RTX 3060 GPUs. All Tacotron models used phoneme (CISAMPA) input, contrasting with MMS's character-based input.

**Baseline:** Meta MMS Urdu and Punjabi Gurumukhi TTS models were used as baselines. For Punjabi, MMS output required an additional Shahmukhi-to-Gurumukhi transliteration step via ChatGPT before synthesis.

**Evaluation** used both objective metrics (WER/CER via Whisper, MCD-DTW, SpeechBERTScore) and subjective tests (MOS for intelligibility and naturalness, SUS intelligibility test, comprehension test, DRT/MRT phoneme-level tests) with 20 native-speaker participants.

## Key Results

Tacotron 2 + WaveGlow outperformed all other systems for Punjabi male: WER 16.1% vs. MMS's 31.4%, CER 9.1% vs. 16.1%, MCD-DTW 6.6 dB vs. not computable for MMS. For Urdu male, Tacotron 1 achieved the best transcription accuracy (WER 13.1%, CER 4.0%) vs. MMS WER 25.6%, CER 10.3%. MOS scores consistently favored the Tacotron-based models over MMS in both intelligibility and naturalness. SUS scores exceeded 97% for Tacotron models compared to 90.8% for MMS on Punjabi. MMS failed completely to synthesize numbers and symbols in both languages, which severely degraded intelligibility. For Urdu female, MMS has no available model.

Phoneme-feature accuracy (DRT/MRT) showed MMS weakest at word-final positions across voicing, nasality, aspiration, and sibilation in Punjabi, while Tacotron 2+WG was most balanced. In Urdu, MMS performed better on some word-initial features but remained weak at word-final aspiration and voicing.

Real-time factor: Tacotron 1 (RTF = 0.17, 6.9M params) is suitable for real-time deployment, while Tacotron 2+WaveGlow (RTF = 3.54, 296M params combined) delivers higher quality at the cost of inference speed.

## Novelty Assessment

The primary contribution is practical and linguistic rather than architectural: the creation of the first dedicated Punjabi Shahmukhi phonetic lexicon and TTS training corpus, along with script-specific text-analysis modules. The paper does not propose new neural architectures; it applies well-established Tacotron models in a rigorous multilingual low-resource setting. The comprehensive phoneme-level evaluation methodology (DRT, MRT, SUS) is more thorough than typical TTS papers in this space. The finding that phoneme-based training is necessary (vs. character-based MMS) for these languages is a meaningful empirical result.

## Limitations and Open Questions

The Punjabi corpus covers only the Majhi accent; other dialects (Shahpuri, etc.) are not addressed. The Tacotron 2 + WaveGlow system has an RTF of 3.54, making it unsuitable for real-time applications. The MMS comparison is limited since MMS training data details are unavailable, preventing MCD-DTW and SpeechBERTScore computation for MMS. The high MCD-DTW values even for Tacotron models on Punjabi indicate remaining difficulty with position-dependent phonetic transformations (e.g., the aspiration-to-stop neutralization in Majhi accent word-initial position). Subjective tests involved only 20 participants, which is a relatively small panel.

## Wiki Connections

This paper advances [[multilingual-tts]] by demonstrating that script-specific phoneme-based pipelines are essential for low-resource South Asian languages not well served by character-level massively multilingual approaches. The evaluation methodology draws on [[subjective-evaluation]] (MOS, SUS, DRT/MRT, comprehension tests) and [[evaluation-metrics]] (WER, CER, MCD-DTW). It implicitly supports the case for [[self-supervised-speech]] and phoneme-based intermediate representations when building TTS for low-resource languages. The comparison to MMS (Meta's large multilingual model) highlights the trade-off between scale and language-specific quality, a recurring theme in [[multilingual-tts]] literature.
