---
id: "interspeech-2025-2189"
title: "ProMode: A Speech Prosody Model Conditioned on Acoustic and Textual Inputs"
authors: ["Eray Eren", "Qingju Liu", "Hyeongwoo Kim", "Pablo Garrido", "Abeer Alwan"]
organization: null
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-07-04
task: ["TTS"]
architecture: ["transformer-enc-dec"]
conditioning: ["text-conditioned", "speaker-conditioned", "zero-shot", "prompt-conditioned"]
training: ["supervised"]
model_size: "not reported"
codec_used: "none"
datasets_train: ["GigaSpeech"]
datasets_eval: ["GigaSpeech"]
metrics:
  - name: UTMOS
    value: 3.1
    system: ProMode
    testset: GigaSpeech Test
  - name: WER
    value: 3.99
    system: ProMode
    testset: GigaSpeech Test
  - name: SPK-SIM
    value: 0.83
    system: ProMode
    testset: GigaSpeech Test
  - name: F0-RMSE
    value: 21.5
    system: ProMode
    testset: GigaSpeech Test
code_available: null
demo_available: true
url: "https://www.isca-archive.org/interspeech_2025/eren25_interspeech.html"
related_concepts: [prosody-control, transformer-enc-dec-tts, zero-shot-tts, evaluation-metrics, subjective-evaluation]
related_papers: ["2025.acl-long.313"]
field_significance:
  level: "moderate"
  type: ["architectural-novelty"]
generation:
  date: 2026-07-04
  agent: speech-generation-ingest-agent
  model: claude-sonnet-4-6
  commit: "318f1b4"
---

> [!abstract] Interspeech · 2025 · Conference
> **Eren et al.** · [→ Paper](https://www.isca-archive.org/interspeech_2025/eren25_interspeech.html) · Demo: ✓ · Code: ?
>
> ProMode is a task-agnostic, zero-shot prosody encoder-decoder that jointly conditions on masked acoustic and textual inputs to predict F0 and energy, outperforming style encoder baselines and improving prosody quality when integrated into a downstream TTS system.

## Problem

Most prosody models are tightly coupled with a specific downstream task: style encoders in TTS systems model prosody only as a conditioning signal for synthesis, while representations trained for speech emotion recognition (SER) are optimised for classification rather than prosody prediction. The one exception, ProsodyBERT, produces discrete prosody codes from acoustic features alone, omitting textual information that carries prosodic meaning (lexical stress, phrase structure, emphasis). All existing approaches require a paired TTS system to produce speech from predicted prosody, leaving a gap for a general-purpose, task-agnostic prosody predictor that can be plugged into arbitrary downstream systems.

## Method

ProMode is a stand-alone, zero-shot prosody model built on the Perceiver IO architecture. The input to the encoder is a concatenation of acoustic features extracted at 11.6 ms per frame (10-dimensional low-frequency Mel-spectrogram spanning under 380 Hz, F0, energy, phoneme durations, and a voiced/unvoiced binary flag) alongside length-regulated phoneme embeddings that are aligned to the acoustic frame rate via a forced aligner. Sixty percent of frames are masked at phoneme boundaries, so each phoneme is either fully available or fully masked. The Perceiver IO encoder (18 self-attention layers, 8 heads) compresses the variable-length sequence into a fixed-length latent prosody embedding via cross-attention.

Two decoders operate on this embedding. An unconditional decoder (PD1) attends only to acoustic context; a conditional decoder (PD2) additionally attends to unmasked text embeddings processed through ConvNeXt V2 layers. PD1 contributes an auxiliary acoustic-only loss (AOL) during training: without it, the model ignores acoustic inputs and collapses onto the text-only pathway. At inference, only PD2 is retained. PD2 employs a modified adaLN-zero conditioning adapted from [[2025.acl-long.313|F5-TTS]], replacing the standard global scale/shift parameters with temporally dependent per-frame parameters generated via cross-attention over the prosody embeddings.

Speaker identity is provided through frozen ECAPA2 embeddings, making the system zero-shot for both unseen speakers and prosody styles. The model is trained on the M subset of GigaSpeech (~750 hours filtered from a 1,000-hour set) using L1 loss for F0, MSE for energy and Mel-spectrogram, and binary cross-entropy for voiced/unvoiced.

![The proposed ProMode features the acoustic and textual feature extraction (left to the red dashed line) and the Perceiver IO architecture (right). Different features are projected to embeddings and stacked together (plus-circle). The modified Perceiver has two decoders, with and without textual conditions. Note, Decoder 1 is an axillary decoder to boost convergence of Decoder 2, and only the latter will be kept in the inference stage. Speaker embeddings are obtained from ECAPA2 model.](assets/interspeech-2025-2189/figure-1.png)

## Key Results

On GigaSpeech Test, ProMode achieves frame-level F0 RPA of 43.9% and RMSE of 21.5 Hz, outperforming the best baseline (StyleTTS2*: RPA 40.2%, RMSE 22.3 Hz; Wav2Vec2-SER*: RPA 39.9%, RMSE 22.0 Hz) across all frame-level and phoneme-level F0 and energy metrics (Table 1). The two SER-based encoders trail substantially despite fine-tuning on the same training data and decoder. Ablations confirm that AOL is the most critical single component: removing it collapses F0 RPA from 43.9% to 25.8% and raises RMSE from 21.5 to 30.1 Hz, while removing any other input feature causes more modest degradation (Table 1).

When ProMode-predicted pitch replaces the built-in pitch predictor in a retrained FluentSpeech TTS system, objective scores on GigaSpeech Test improve across all dimensions relative to FluentSpeech baseline: UTMOS 3.10 vs. 3.00, WER 3.99% vs. 4.41%, speaker similarity (SECS) 0.83 vs. 0.81, and AutoPCP prosody similarity 2.64 vs. 2.47 (Table 2). In ABX preference tests with 39 native English listeners, ProMode is preferred over all three baselines with statistical significance (p < 0.05).

## Novelty Assessment

The core contribution is the dual-decoder training strategy with an acoustic-only loss to prevent encoder collapse, applied within a Perceiver IO framework for prosody prediction. The AOL is inspired by classifier-free guidance in diffusion models but repurposed as a training stabiliser rather than an inference mixer. The modified adaLN-zero is an incremental adaptation of F5-TTS's conditioning mechanism, adding temporal dependence via cross-attention. Applying Perceiver IO to prosody is novel but not surprising: the architecture was designed for variable-length inputs, and prosody sequences naturally vary in length. Evaluations are thorough at the objective level across multiple granularities, and the subjective test is well-designed (39 listeners, ABX methodology). The main limitation is that all experiments are confined to English GigaSpeech, leaving generalisation to other languages or speaking conditions unevaluated.

## Field Significance

Moderate - ProMode establishes that joint acoustic-textual conditioning with a dual-decoder training stabiliser outperforms style encoders and emotion representations for prosody prediction, and demonstrates that a task-agnostic prosody module can directly improve downstream TTS quality. The contribution is primarily architectural and scoped to English, but provides a clear design template for separable prosody modelling as a pre-processing step in TTS pipelines.

## Claims

- **supports:** Combining acoustic context with text conditioning in a masked prediction framework produces more accurate prosody prediction than style encoders or emotion representations adapted to the task.
  > *Evidence:* ProMode outperforms StyleTTS2*, Wav2Vec2-SER*, and Emotion2Vec* across all F0 and energy metrics at frame and phoneme level on GigaSpeech Test; since all models share the same decoder, performance differences isolate the encoder contribution. *(§4.1, Table 1)*

- **complicates:** Self-supervised representations trained for emotion recognition do not transfer effectively to prosody prediction, even after full fine-tuning on prosody-labelled data.
  > *Evidence:* Emotion2Vec* and Wav2Vec2-SER* show markedly lower F0 RPA and higher RMSE than StyleTTS2* and ProMode; Emotion2Vec* underperforms Wav2Vec2-SER* despite training on more labeled emotion data, suggesting the SER pre-training introduces a representation bias that hinders prosody regression. *(§4.1, Table 1)*

- **supports:** A dual-decoder architecture with an auxiliary acoustic-only loss prevents the prosody encoder from collapsing onto text context in masked prediction, where removing the loss causes catastrophic degradation.
  > *Evidence:* Ablation removing AOL drops F0 RPA from 43.9% to 25.8% (the largest degradation of any ablation), confirming that without AOL the decoder attends exclusively to text and the prosody encoder's contribution is suppressed. *(§4.3, Table 1)*

- **supports:** Task-agnostic prosody models that predict F0 independently can improve both naturalness and prosody preference in downstream TTS systems without requiring joint end-to-end training.
  > *Evidence:* Replacing FluentSpeech's built-in pitch predictor with ProMode-predicted pitch improves UTMOS from 3.00 to 3.10, WER from 4.41% to 3.99%, and AutoPCP from 2.47 to 2.64; ABX prosody preference is statistically significant over all three baselines. *(§4.2, Table 2, §4.2.2)*

## Limitations and Open Questions

All experiments are conducted on English GigaSpeech (read and spontaneous speech); the authors identify multilingual extension as future work but provide no cross-lingual evidence. ProMode requires ground-truth phoneme durations from a forced aligner during evaluation, which creates a dependency on accurate alignment at inference time. The downstream integration only tests replacement of the pitch predictor within FluentSpeech; whether the benefit holds for other TTS backbones or for energy-conditioned synthesis is unevaluated. The model size is not reported, making compute cost comparisons to baselines unclear.

## Wiki Connections

- [[prosody-control|Prosody Control]] - ProMode introduces a stand-alone masked prediction model that explicitly predicts F0 and energy as separable prosody outputs, advancing prosody prediction as an independent module rather than a baked-in TTS component.
- [[transformer-enc-dec-tts|Transformer Encoder-Decoder TTS]] - The Perceiver IO encoder and dual transformer decoder structure follows the encoder-decoder paradigm, with cross-attention as the coupling mechanism between prosody embeddings and output predictions.
- [[zero-shot-tts|Zero-Shot TTS]] - ProMode operates zero-shot for unseen speakers and prosody styles, using an audio reference prompt at inference to condition the prosody encoder without speaker-specific training.
- [[evaluation-metrics|Evaluation Metrics]] - The paper employs a comprehensive set of prosody-specific objective metrics including F0 RPA, RCA, RMSE, and MAE at frame and phoneme levels, alongside AutoPCP for downstream TTS prosody assessment.
- [[subjective-evaluation|Subjective Evaluation]] - An ABX preference test with 39 native English listeners recruited via Prolific provides perceptual validation of ProMode's prosody improvement over baselines.
- [[2025.acl-long.313|F5-TTS]] - ProMode directly adapts the adaLN-zero conditioning mechanism from F5-TTS, extending it to produce temporally dependent per-frame scale/shift parameters via cross-attention over prosody embeddings.
