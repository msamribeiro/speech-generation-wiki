---
id: "interspeech-2025-2679"
title: "Can We Reconstruct a Dysarthric Voice with the Large Speech Model Parler TTS?"
authors: ["Ariadna Sanchez", "Simon King"]
organization: "University of Edinburgh"
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-07-05
task: ["TTS"]
architecture: ["autoregressive-LM"]
conditioning: ["instruction-conditioned"]
training: ["fine-tuning"]
model_size: "not reported"
codec_used: "not reported"
datasets_train: ["Speech Accessibility Project (SAP)", "Multilingual LibriSpeech (MLS)"]
datasets_eval: ["Clarity 2021 challenge devset"]
metrics:
  - name: WER
    value: "20-25"
    system: proposed
    testset: Clarity 2021 devset
code_available: null
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/sanchez25_interspeech.html"
related_concepts: ["instruction-conditioned-tts", "speaker-adaptation", "evaluation-metrics"]
related_papers: []
field_significance:
  level: "moderate"
  type: ["engineering-integration", "evaluation-contribution"]
generation:
  date: 2026-07-05
  agent: speech-generation-ingest-agent
  model: claude-sonnet-4-6
  commit: "2829243"
---

> [!abstract] Interspeech · 2025 · Conference
> **Sanchez et al.** (University of Edinburgh) · [→ Paper](https://www.isca-archive.org/interspeech_2025/sanchez25_interspeech.html) · Demo: ? · Code: ?
>
> Fine-tunes Parler TTS on a curated dysarthric speech dataset to investigate voice reconstruction, finding that the model can sample from dysarthric speech distributions but cannot reliably control intelligibility or speaker identity through natural language prompts.

## Problem

People who develop degenerative conditions such as ALS may eventually lose the ability to speak and turn to voice output communication aids (VOCAs). Voice reconstruction, unlike voice banking, must work from recordings that already exhibit disordered characteristics, synthesising speech that approximates the speaker's pre-disorder voice. Prior approaches relied on HMM-based interpolation of average and user voice models, or on voice conversion to transform healthy-speaker speech toward a target speaker's identity without their dysarthric traits. No prior work had attempted voice reconstruction using a large, instruction-conditioned speech model, and it was unknown whether prompt-based control over intelligibility would be sufficient for this task.

## Method

The paper fine-tunes Parler TTS Mini v0.1, an autoregressive language model-based TTS system conditioned on natural language text descriptions, on a combined dataset of dysarthric and healthy speech. Dysarthric data comes from the Speech Accessibility Project (SAP), restricted to ALS and Cerebral Palsy speakers, totalling 22.97 hours across 22 speakers. A matched healthy control set of equal size is drawn from the Multilingual LibriSpeech (MLS) training partition, pairing each SAP speaker with an MLS speaker of the same predicted gender and similar total data length.

Fine-tuning data is annotated using the DataSpeech pipeline, which generates descriptive text prompts covering speaker gender (via an XLSR-53 gender classifier), speaking style, and intelligibility. Intelligibility labels are derived from speech therapist ratings of "imprecise consonants" on a 1-7 Likert scale from the SAP metadata; healthy controls receive a rating of 0. Fine-tuning runs for 2 epochs on a single NVIDIA A100 GPU, taking approximately 2 hours.

At inference, six test speakers (three dysarthric, three healthy) are each synthesised at multiple intelligibility levels by varying the intelligibility descriptor in the prompt, from the speaker's true rating down to the "extremely good intelligibility" target. Test sentences are drawn from the Clarity 2021 challenge devset, a set not present in either Parler's original training data or the fine-tuning set.

Evaluation uses WER (Whisper large), speaker similarity (Resemblyzer), and UTMOS for naturalness, supplemented by expert listening. Crucially, the paper first validates each metric on the natural speech in the fine-tuning dataset before applying it to synthetic output.

## Key Results

WER for healthy controls in the fine-tuning data is near zero, while SAP speakers show substantially elevated WER consistent with their intelligibility ratings, confirming the labels are informative. For synthesised speech, however, WER remains roughly constant at 20-25% across all prompted intelligibility levels and across all six test speakers, including the healthy controls (Figure 5). Speaker similarity scores show similarly flat profiles across intelligibility conditions (Figure 6). Expert listening confirms the model samples across a wide range of intelligibility and speaker identity values for any given prompt, explaining the flat objective metrics.

UTMOS scores for natural dysarthric speech fall consistently below 3, even for the most intelligible SAP speakers, rendering the metric unusable for this domain. Resemblyzer speaker similarity, by contrast, shows comparable score ranges for both healthy and dysarthric recorded speech, indicating it is robust to speech disorder type and suitable for this evaluation context.

## Novelty Assessment

The primary contribution is engineering integration: Parler TTS, an existing instruction-conditioned model, is applied to a new domain (dysarthric voice reconstruction) without architectural change. The dataset curation and annotation pipeline, which pairs SAP dysarthric speakers with MLS controls and annotates both with DataSpeech prompts, involves meaningful effort but is not technically novel. The paper's most reusable finding is the evaluation contribution: the demonstration that UTMOS fails on dysarthric speech, while Resemblyzer remains robust, directly informs future work in atypical speech synthesis. The negative result regarding prompt controllability, while confirmatory of the authors' own hypothesis, is clearly articulated and accompanied by plausible architectural explanations.

## Field Significance

Moderate — This paper provides the first examination of large instruction-conditioned TTS for dysarthric voice reconstruction, a task that lies outside the typical distribution of current TTS research. Its primary value is diagnostic: it shows that prompt-based intelligibility control, as implemented in Parler TTS, is not granular enough for consistent reconstruction, and it establishes that standard automatic evaluation metrics require validation before use in atypical speech domains. These findings can inform future architectural and evaluation choices for personalised assistive TTS.

## Claims

- **complicates:** Natural language prompt conditioning provides insufficient fine-grained control for voice reconstruction tasks requiring consistent intelligibility and speaker identity.
  > *Evidence:* WER for synthesised speech remains approximately 20-25% across all prompted intelligibility levels (from "extremely poor" to "extremely good") and across all six test speakers; speaker similarity scores are similarly insensitive to prompt variation; expert listening confirms stochastic sampling across traits despite fixed prompts. *(§5.2, §5.3, §6)*

- **supports:** Large instruction-conditioned TTS models can be adapted to dysarthric speech through fine-tuning on small curated datasets.
  > *Evidence:* Parler TTS Mini fine-tuned on ~23 hours of combined SAP dysarthric and MLS healthy speech learns to generate from the combined distribution; WER for fine-tuning data correlates with intelligibility ratings, confirming the model internalises the distribution. *(§4.1, §5.1)*

- **complicates:** Automatic evaluation metrics calibrated on healthy synthetic speech do not generalise reliably to dysarthric speech and require domain validation before use.
  > *Evidence:* UTMOS scores fall consistently below 3 for all SAP dysarthric speakers, even those with mild impairment (rating 1-2), indicating a domain mismatch that makes UTMOS unusable in this context; Resemblyzer speaker similarity shows comparable ranges across healthy and dysarthric conditions, demonstrating metric-level robustness varies substantially. *(§5.1)*

## Limitations and Open Questions

> [!warning]
> UTMOS is found unusable for evaluating dysarthric speech synthesis in this work. Papers that report UTMOS for atypical speech without a domain validation step should be interpreted with caution.

The dataset is limited to 22 speakers across two etiologies (ALS and Cerebral Palsy), with no speakers at the most severe intelligibility levels (ratings 6-7). Generalisation to other dysarthric conditions or more severe impairment is undemonstrated. Subjective evaluation was restricted to expert listening rather than a formal MOS study, which makes comparison to other voice reconstruction and personalised TTS systems difficult.

The paper attributes the controllability failure to Parler's architecture: the entire prompt is embedded by a pre-trained NLP model and attended to by the speech LM, which may be too indirect for categorical control signals like speaker identity. The authors suggest that fine-grained, direct conditioning mechanisms for speaker identity and intelligibility are needed. They also propose data augmentation via acoustic-to-articulatory inversion as a path to larger training sets, and note that proper evaluation will require involvement of VOCA users and speech therapists, for which no established protocol currently exists.

## Wiki Connections

- [[instruction-conditioned-tts|Instruction-Conditioned TTS]] — this paper applies Parler TTS, a prototype of instruction-conditioned synthesis, to dysarthric voice reconstruction, exposing the limits of prompt-based control for fine-grained speech characteristic targeting.
- [[speaker-adaptation|Speaker Adaptation]] — fine-tuning on a per-speaker dysarthric dataset represents a supervised speaker adaptation approach, but the paper finds the prompt-based speaker identity signal remains inconsistent.
- [[evaluation-metrics|Evaluation Metrics]] — the paper contributes evidence that UTMOS and Resemblyzer behave very differently on dysarthric speech, with practical implications for metric selection in atypical TTS evaluation.
