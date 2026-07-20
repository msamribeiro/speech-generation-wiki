---
id: "interspeech-2025-2159"
title: "Generating Consistent Prosodic Patterns from Open-Source TTS Systems"
authors: ["Ha Eun Shim", "Olivia Yung", "Paige Tuttösi", "Boey Kwan", "Angelica Lim", "Yue Wang", "H. Henny Yeung"]
organization: Simon Fraser University
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-07-04
task: ["TTS", "evaluation"]
architecture: ["flow-matching"]
conditioning: ["text-conditioned"]
training: ["fine-tuning", "supervised"]
model_size: "not reported"
codec_used: "none"
datasets_train: ["custom prosodic boundary dataset (100 sentences, human-recorded)"]
datasets_eval: ["custom prosodic boundary test set (40 unseen sentences)"]
metrics:
  - name: pause duration (ms)
    value: "101.55 (median, List-first boundary post-fine-tuning, seen set)"
    system: fine-tuned Matcha-TTS
    testset: custom List-type stimuli
code_available: null
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/shim25_interspeech.html"
related_concepts: [prosody-control, evaluation-metrics, flow-matching]
related_papers: []
field_significance:
  level: "low"
  type: ["evaluation-contribution", "engineering-integration"]
generation:
  date: 2026-07-04
  agent: speech-generation-ingest-agent
  model: claude-sonnet-4-6
  commit: "318f1b4"
---

> [!abstract] Interspeech · 2025 · Conference
> **Shim et al.** (Simon Fraser University) · [→ Paper](https://www.isca-archive.org/interspeech_2025/shim25_interspeech.html) · Demo: ? · Code: ?
>
> A two-study investigation revealing that five open-source TTS systems fail to produce reliable prosodic phrase-boundary contrasts for syntactically ambiguous sentences, and demonstrating that fine-tuning Matcha-TTS on a small custom prosodic dataset partially resolves the gap for list-type sentences.

## Problem

Standard neural TTS systems are trained primarily on read-speech corpora that lack systematic prosodic variation for syntactically ambiguous constructions. As a result, they cannot reliably generate the acoustic distinctions (particularly pause placement) needed to signal different syntactic parsings of identical word sequences. This gap is practically significant: incorrect prosody can change the meaning of an utterance, with potential consequences in high-stakes communication contexts. The paper asks whether off-the-shelf open-source TTS systems handle punctuation-induced prosodic contrasts (comma vs. no comma), and whether fine-tuning on a targeted small dataset can remedy the deficiency.

## Method

The study is divided into two phases. In Study 1, five open-source TTS systems are evaluated against two sentence types designed to test prosodic disambiguation: "Direct Address" (DA) sentences, where a comma signals vocative use of a proper noun, and "List" sentences, where commas mark the boundary between listed items. Each sentence type is contrasted in two conditions: Condition A (with commas) and Condition B (without commas). Four stimuli are synthesised per system using each system's publicly accessible Gradio demo, and the outputs are assessed qualitatively using the Consensual Qualitative Research (CQR) protocol, in which the authors collectively evaluate synthesis quality, consistency, real-time factor, disfluencies, and perceived meaning contrast between conditions.

The five systems span autoregressive approaches (Parler-TTS, Coqui-XTTS, VoiceCraft) and non-autoregressive flow-matching approaches (ToucanTTS, Matcha-TTS). Matcha-TTS is identified as the best candidate: it is the only system with RTF below 1.0, produces consistent voice quality, and provides sufficient user control over speaking rate for the task.

In Study 2, Matcha-TTS is fine-tuned on a custom human-recorded dataset of 100 sentences (80 for training, 20 held out for validation), including 10 contrastive sentence pairs per type plus controlled filler sentences. Recording was performed by a single native English female speaker who was aware of the A/B meaning distinctions, and validated by four research assistants. Fine-tuning uses the VCTK speaker-15 checkpoint, an Adam optimiser with learning rate 1e-4, batch size 20, and runs for 85 epochs on an NVIDIA GeForce RTX 4070. The evaluation metric is pause duration (in milliseconds) at comma-marked boundaries, measured pre- and post-fine-tuning, compared using Mann-Whitney U tests.

## Key Results

In Study 1, no system reliably produced prosodic contrasts across both sentence types. VoiceCraft generated male voices by default and introduced speech disfluencies; Parler-TTS returned inconsistent voices across repeated generations; ToucanTTS showed robotic cadence when given reference audio; Coqui-XTTS output contained background noise and audible vocal fry. Only Coqui-XTTS and Matcha-TTS demonstrated any prosodic distinction between Conditions A and B.

In Study 2, fine-tuning Matcha-TTS produced markedly different outcomes depending on sentence type. For List sentences, pause duration contrasts improved significantly at both comma boundaries. At the first boundary (seen set), median pause duration increased from 11.26 ms to 101.55 ms (U = 4, p < 0.01); at the unseen set, from 1.69 ms to 107.81 ms (U = 0, p < 0.001). The second boundary showed similar significant improvements (p < 0.01 and p < 0.05 for seen and unseen sets respectively). For Direct Address sentences, fine-tuning produced no statistically significant improvement in either the seen or unseen sets (p = 0.89 and p = 0.59), with pre- and post-fine-tuning median pauses remaining comparable.

## Novelty Assessment

The contribution is engineering integration combined with a narrow evaluation exercise rather than a novel architectural or methodological advance. Fine-tuning a flow-matching TTS system on custom data is a well-established practice; the novelty lies in applying it specifically to prosodic boundary generation for syntactically ambiguous sentences. The evaluation in Study 1 is small in scale (four stimuli per system, five systems, qualitative CQR assessment by the authors) and does not include external listener ratings, which limits the generalisability of its conclusions. The fine-tuning experiment in Study 2 demonstrates proof-of-concept for one prosodic cue (pause duration) on one system, with only 100 training sentences and a single speaker. The finding that the approach works for List types but not Direct Address types is honest and useful, but the reasons are speculative (possible systematic stress placement constraints in the base model). The paper is best read as a preliminary investigation rather than a deployable framework.

## Field Significance

Low — This paper contributes a preliminary qualitative audit of five open-source TTS systems on prosodic boundary accuracy and a small-scale fine-tuning demonstration. It identifies a real gap in how standard training corpora represent prosodic variation, and provides suggestive evidence that targeted custom datasets can partially correct it. The scope is narrow and the evaluation scale is limited, so the results are informative for researchers interested in TTS prosody and syntactic disambiguation rather than constituting a broadly reproducible finding.

## Claims

- **complicates:** Neural TTS systems trained on standard read-speech corpora do not reliably generate fine-grained prosodic distinctions required for syntactic disambiguation.
  > *Evidence:* Qualitative CQR assessment of five open-source systems (Parler-TTS, Coqui-XTTS, VoiceCraft, ToucanTTS, Matcha-TTS) found that none reliably produced distinct acoustic cues distinguishing comma vs. no-comma conditions; issues included voice instability, disfluencies, and failure to convey meaning differences between prosodic conditions. *(§2.4, Table 3)*

- **supports:** Fine-tuning a TTS system on a small targeted dataset with explicit prosodic contrasts can improve measurable prosodic distinctions for some sentence types.
  > *Evidence:* Fine-tuning Matcha-TTS on 100 human-recorded sentences with controlled pause contrasts yielded statistically significant pause duration improvements for List-type sentences (unseen set first boundary: U = 0, p < 0.001, median 1.69 ms before vs. 107.81 ms after); improvement did not generalise to Direct Address types (p = 0.59). *(§3.4)*

- **complicates:** Targeted fine-tuning for prosodic control does not generalise uniformly across syntactically distinct prosodic boundary types.
  > *Evidence:* The same fine-tuning procedure that significantly improved List-type pause contrasts had no measurable effect on Direct Address types, suggesting the base model has systematic prosodic constraints (such as fixed stress placement on proper names) that a small dataset cannot override. *(§3.4, §4)*

- **supports:** Prosodic training data diversity, rather than model architecture alone, is a limiting factor in TTS prosody accuracy.
  > *Evidence:* The authors attribute systematic prosodic failures across all five evaluated systems to the absence of diverse prosodic patterns in standard training corpora (e.g., LJ Speech), and demonstrate that adding a 100-sentence targeted dataset produces measurable improvement in at least one prosodic boundary type. *(§4, §5)*

## Limitations and Open Questions

> [!warning]
> The Study 1 evaluation is based on four stimuli per system assessed qualitatively by the authors alone, without external listeners or statistical analysis. Conclusions about which systems "fail" should be treated as suggestive observations rather than systematic evidence.

The fine-tuning experiment uses a single speaker for both training data collection and a small stimulus size (100 training sentences), with no multi-speaker generalisation tested. The acoustic analysis focuses solely on pause duration and does not account for F0, intensity, or lengthening cues that also mark prosodic boundaries. Matcha-TTS's continued failure on Direct Address types despite fine-tuning points to possible inductive biases in the base model's duration predictor that are not well understood. The authors propose extending the approach to combinations of pause, pitch, and lengthening in future work.

## Wiki Connections

- [[prosody-control|Prosody Control]] — this paper targets prosodic boundary generation directly, measuring pause duration as the primary control signal and using fine-tuning to improve the model's response to punctuation-induced prosodic cues.
- [[evaluation-metrics|Evaluation Metrics]] — the paper develops a qualitative CQR methodology for assessing prosodic accuracy in TTS output, complementing quantitative acoustic measurement of pause duration.
- [[flow-matching|Flow Matching]] — Matcha-TTS, the system selected and fine-tuned in Study 2, uses conditional flow matching as its generative backbone, and is favoured precisely because of its stable, controllable inference.
