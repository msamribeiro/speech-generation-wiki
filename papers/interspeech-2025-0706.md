---
id: "interspeech-2025-0706"
title: "Contextual Paralinguistic Data Creation for Multi-Modal Speech-LLM: Data Condensation and Spoken QA Generation"
authors: ["Qiongqiong Wang", "Hardik B. Sailor", "Tianchi Liu", "Ai Ti Aw"]
organization: "Institute for Infocomm Research (I2R), A*STAR, Singapore"
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: "2025-08-17"
ingested_date: "2026-07-02"
task: ["SCA"]
architecture: []
conditioning: []
training: []
model_size: "not reported"
codec_used: "none"
datasets_train: ["not applicable (survey)"]
datasets_eval: ["SG TV/Movie dataset (internal, 117k segments / 120h)", "CPQA evaluation set (480 samples, ~6.5h, released)"]
metrics:
  - name: other
    value: 60.28
    system: Qwen2-Audio-7B-Instruct (LLM QA, ChatGPT judge, Prompt 2)
    testset: CPQA evaluation set
  - name: other
    value: 59.46
    system: Qwen2-Audio-7B-Instruct (Human QA, ChatGPT judge, Prompt 2)
    testset: CPQA evaluation set
code_available: null
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/wang25e_interspeech.html"
related_concepts: [emotion-synthesis, evaluation-metrics, subjective-evaluation, self-supervised-speech]
related_papers: ["interspeech-2025-0166"]
field_significance:
  level: moderate
  type: ["dataset-contribution", "evaluation-contribution"]
generation:
  date: "2026-07-02"
  agent: speech-generation-ingest-agent
  model: claude-sonnet-4-6
  commit: "d8ecfd4"
---

> [!abstract] Interspeech · 2025 · Conference
> **Wang et al.** (Institute for Infocomm Research (I2R), A*STAR, Singapore) · [→ Paper](https://www.isca-archive.org/interspeech_2025/wang25e_interspeech.html) · Demo: ? · Code: ?
>
> A two-stage pipeline for generating contextual paralinguistic question-answer (QA) pairs from in-the-wild speech: pseudo emotion label-based data condensation followed by LLM-driven CPQA generation, validated by comparing LLM-generated and human-generated QA sets in speech-LLM evaluations.

## Problem

Existing QA datasets for speech LLMs cover either linguistic content or paralinguistic attributes (emotion, gender) in isolation, not their combination. Benchmarks such as AudioBench, Dynamic-Superb, and MMAU draw from small, task-specific emotion corpora (IEMOCAP, MELD) and frame questions as direct label lookups rather than contextual reasoning tasks. This leaves speech LLMs with no training signal for empathetic reasoning, the ability to infer why a speaker feels a certain way from conversational context.

## Method

The framework operates in two stages applied to in-the-wild speech data. In the data condensation stage, speech segments are pre-processed with voice activity detection and split into overlapping windows (2 sec target, 1 sec context on each side). An ensemble of three Emotion2Vec+ models provides categorical emotion labels across nine classes, while a wav2vec2-large-robust model fine-tuned on MSP-Podcast supplies continuous valence scores. A consistency filter discards sub-segments where the categorical sentiment class and valence score disagree; an occurrence filter then retains only segments where a given emotion label appears enough times to be attributed reliably to the full utterance. Gender labels are added using a WavLM-ECAPA model fine-tuned on VoxCeleb2. The result is a condensed subset with high-confidence, balanced paralinguistic annotations. In the QA generation stage, WhisperX produces word-level transcriptions with timestamps; each word is assigned the emotion and gender label of the paralinguistic segment it falls within. GPT-4o then generates diverse question-answer pairs from a structured prompt that directs it to cover speaker attributes, content reasoning, and contextual paralinguistic reasoning without relying on metadata availability at test time.

Parameters are calibrated on an internal Singapore TV/movie dataset (117k segments, 120 hours). The published evaluation benchmark contains 480 audio samples (30 to 60 seconds each, approximately 6.5 hours) with 2,647 LLM-generated CPQA pairs and a parallel human-annotated CPQA set across six non-neutral emotion categories.

## Key Results

The Qwen2-Audio-7B-Instruct model achieves comparable LLM-judge scores on both the ChatGPT-generated and human-generated CPQA sets: 60.28 vs. 59.46 (ChatGPT judge, Prompt 2) and 56.82 vs. 54.33 (Llama-70B judge, Prompt 2). This near-parity between generated and human-authored QA, across two independent judges, is the primary validation of the framework's output quality. A refined evaluation prompt that incorporates paralinguistic context and removes the brevity penalty (Prompt 2) consistently yields higher scores than the AudioBench-derived Prompt 1 for both QA sets and both judges. The study also surfaces a limitation of current speech LLMs: despite comparable scores overall, the results point to weakness in empathetic reasoning tasks specifically (§5).

## Novelty Assessment

The primary contribution is a practical data pipeline rather than a modelling advance. Data condensation using dual SER paradigms (categorical + dimensional) with consistency filtering is a sensible engineering choice, but each component (Emotion2Vec, wav2vec2 valence, WavLM gender) is an existing system. The CPQA generation step is a structured prompting recipe applied to GPT-4o. The novelty lies in the end-to-end framework design and its application to in-the-wild speech with no manual annotation. The open-source 480-sample benchmark is a concrete and useful deliverable. The claim that LLM-generated QA correlates with human QA in speech-LLM evaluations is a practical finding with direct implications for scalable benchmark creation, though the evidence rests on a single model (Qwen2-Audio-7B-Instruct) evaluated on a small and domain-specific dataset.

## Field Significance

Moderate — This paper addresses a real gap in training data for empathetic speech LLMs and demonstrates that automated QA generation can produce evaluation sets comparable to human-authored ones. The dataset is small (480 samples) and the evaluation is single-model, which limits the scope of the conclusions. The framework can serve as a template for larger-scale paralinguistic dataset creation, but the generality of the approach beyond Singapore English in-the-wild speech remains to be demonstrated.

## Claims

- **supports:** Automated LLM-based QA generation can produce paralinguistic evaluation sets that correlate closely with human-authored ones for assessing speech LLMs.
  > *Evidence:* Qwen2-Audio-7B-Instruct achieves 60.28 (LLM QA) vs. 59.46 (Human QA) under the ChatGPT judge with Prompt 2, and 56.82 vs. 54.33 under the Llama-70B judge; both differences are small and directionally consistent. *(§4, Table 2)*

- **supports:** Combining categorical and dimensional emotion recognition with consistency filtering reduces annotation noise in in-the-wild speech data condensation.
  > *Evidence:* The dual SER consistency condition (categorical sentiment class aligned with valence score) applied on SG TV/Movie data achieved the highest UWA of 33.65% at valence thresholds x=0.5, y=0.4, outperforming single-paradigm labelling and improving class balance. *(§3.1, Figure 3)*

- **complicates:** Current speech LLMs exhibit weak performance on contextual empathetic reasoning even when evaluated against well-formed paralinguistic QA.
  > *Evidence:* Evaluation of Qwen2-Audio-7B-Instruct on the CPQA set reveals limitations in handling empathetic reasoning tasks; the paper identifies this as a motivating gap requiring both better data and more robust models. *(§4, §5)*

- **complicates:** Scalable automatic QA generation from speech introduces systematic biases that require post-filtering to produce usable evaluation data.
  > *Evidence:* ChatGPT-generated QA contained repetitive question variants (multiple paraphrases asking about reasons behind emotion) and irrelevant questions assuming a text transcript was available; keyword-based post-filtering was required to remove these. *(§3.2)*

## Limitations and Open Questions

The evaluation benchmark is small (480 samples, 6.5 hours) and drawn from a single domain (Singaporean YouTube channels) in a mostly English-Mandarin context, so generalisation to other languages and speaking styles is unknown. The entire framework is validated against a single speech LLM (Qwen2-Audio-7B-Instruct), and the LLM judge scores are relatively similar across generated and human QA, which leaves open the question of whether the difference would matter more with a weaker model or a harder task. Speaker diarization is absent, causing the LLM to under-generate questions about multi-speaker interactions when multiple speakers share a gender. The internal SG TV/Movie dataset used for parameter tuning is not publicly available, making exact replication of the condensation configuration difficult.

## Wiki Connections

- [[emotion-synthesis|Emotion Synthesis]] — The data condensation pipeline focuses on emotion as the primary paralinguistic attribute, using dual SER models to generate reliable emotion labels from in-the-wild speech.
- [[evaluation-metrics|Evaluation Metrics]] — The paper introduces and compares two LLM-judge prompts for assessing speech-LLM performance on paralinguistic QA, with UWA and LLM-judge accuracy as primary metrics.
- [[subjective-evaluation|Subjective Evaluation]] — Human annotators created a parallel CPQA set used as a reference to validate the LLM-generated QA through side-by-side comparison in speech-LLM evaluations.
- [[self-supervised-speech|Self-Supervised Speech]] — Gender labelling uses WavLM-ECAPA (a self-supervised pre-trained model) fine-tuned on VoxCeleb2 as a core annotation component in the data condensation stage.
- [[interspeech-2025-0166|Frozen LLMs Can Perceive Paralinguistic Aspects of Speech]] — Related prior work on enabling LLMs to process paralinguistic speech cues, cited as part of the context motivating better paralinguistic QA data.
