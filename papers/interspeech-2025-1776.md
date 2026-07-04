---
id: "interspeech-2025-1776"
title: "SpeechSEC: A Unified Multi-Task Framework for Speech Synthesis, Editing, and Continuation"
authors: ["Liming Liang", "Dongchao Yang", "Xianwei Zhuang", "Yuxin Xie", "Luo Chen", "Yuehan Jin", "Yuexian Zou"]
organization: null
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-07-04
task: ["TTS"]
architecture: ["hybrid"]
conditioning: ["text-conditioned", "speaker-conditioned", "prompt-conditioned"]
training: ["supervised"]
model_size: "not reported"
codec_used: "SpeechTokenizer / Descript-Audio-Codec (multiple configurations)"
datasets_train: ["LibriTTS-R"]
datasets_eval: ["LibriTTS-R"]
metrics:
  - name: MOS
    value: 4.20
    system: SpeechSEC (ST config)
    testset: LibriTTS-R test-clean
  - name: SPK-SIM
    value: 0.72
    system: SpeechSEC (ST config)
    testset: LibriTTS-R test-clean
  - name: WER
    value: 8.7
    system: SpeechSEC (ST config)
    testset: LibriTTS-R test-clean
  - name: CER
    value: 3.6
    system: SpeechSEC (ST config)
    testset: LibriTTS-R test-clean
code_available: null
demo_available: true
url: "https://www.isca-archive.org/interspeech_2025/liang25e_interspeech.html"
related_concepts: ["neural-codec", "self-supervised-speech", "autoregressive-codec-tts", "evaluation-metrics", "subjective-evaluation"]
related_papers: []
field_significance:
  level: "moderate"
  type: ["architectural-novelty", "engineering-integration"]
generation:
  date: 2026-07-04
  agent: speech-generation-ingest-agent
  model: claude-sonnet-4-6
  commit: "318f1b4"
---

> [!abstract] Interspeech · 2025 · Conference
> **Liang et al.** · [→ Paper](https://www.isca-archive.org/interspeech_2025/liang25e_interspeech.html) · Demo: ✓ · Code: ?
>
> SpeechSEC proposes a unified non-autoregressive framework that handles speech synthesis, editing, and continuation within a single Conformer model, using task-specific input conditioning and multi-task joint training to improve performance on each task over single-task baselines.

## Problem

Codec-based non-autoregressive speech synthesis systems are typically trained on a single task, mapping semantic tokens to acoustic tokens for synthesis alone. This design overlooks the complementary knowledge available across related audio tasks: acoustic continuation requires modeling internal relationships between audio tokens, and speech editing demands knowledge of seamless segment transitions. Systems like SoundStorm improve generation speed over autoregressive approaches but are scoped to synthesis only, leaving editing and continuation as separate problems requiring separate pipelines. There is no established framework that jointly trains on all three tasks using non-autoregressive techniques and exploits their shared structure.

## Method

SpeechSEC extends the SoundStorm non-autoregressive decoding paradigm to a unified multi-task setting. A single Conformer network with bidirectional self-attention and rotary positional embeddings is trained to predict masked acoustic tokens across three tasks, distinguished at inference and training time by a learned Task Register embedding (z_i).

Semantic tokens are obtained from text via a T5-based text-to-semantic model for synthesis and editing; for continuation, no text input is used. Acoustic tokens are extracted from raw audio using one of three supported configurations: ST (SpeechTokenizer for both semantic q₁ and acoustic q₂–q₈ layers), STDAC (SpeechTokenizer for semantics, Descript-Audio-Codec for acoustics), and HuDAC (HuBERT for semantics, DAC for acoustics). Each configuration is evaluated independently to demonstrate tokenizer-agnostic applicability.

Input construction differs per task using a cosine masking schedule. For synthesis, tokens are masked at random positions. For continuation, future tokens from a start position onward are masked. For editing, a variable-length span between start and end positions is masked. A speaker embedding (s_e) extracted from the reference audio is added to the model input alongside the Task Register, enabling speaker-conditioned generation. The training objective is cross-entropy loss over masked token positions, computed only at positions selected by the task-specific mask. Decoding follows SoundStorm's iterative RVQ-layer-by-layer confidence-based sampling. Training uses LibriTTS-R (train-clean-100, -360, and -500 subsets; approximately 585 hours at 24 kHz, resampled to 16 kHz for token extraction), with audio files under one second filtered out.

![Illustration of the proposed SpeechSEC training Framework. We sample one of the tasks at each training step and build its condition inputs x_i by padding the raw audio and processing differently, where red, blue and yellow denotes valid tokens and white is padding.](assets/interspeech-2025-1776/figure-2.png)

## Key Results

On the synthesis task in the ST configuration, multi-task SpeechSEC achieves MOS 4.20 versus 3.98 for the single-task ST baseline, and speaker cosine similarity of 0.72 versus 0.68 (Table 1). WER improves from 9.2 to 8.7 and CER from 3.8 to 3.6. The STDAC and HuDAC configurations show smaller but consistent improvements over their respective single-task baselines. The paper also claims SpeechSEC outperforms SoundStorm (4.20 MOS versus 4.00 and 0.72 speaker similarity versus 0.58), though this comparison is informal — the two evaluations use independent test sets and listener pools rather than a controlled head-to-head experiment.

For speech editing (ST configuration), the model achieves WER 5.1, CER 1.8, MOS 3.93, and speaker similarity 0.82 (Table 2). For speech continuation (ST), MOS 3.63 and speaker similarity 0.66 are reported; WER and CER are not applicable because generated continuations have no ground-truth transcript.

Ablation results (Table 3) confirm that removing either auxiliary task degrades synthesis performance. Removing the editing task increases WER by up to 4.4 points and CER by 4.5 points; removing the continuation task primarily reduces audio quality (up to 0.18 MOS) and voice preservation (up to 0.06). The two tasks contribute complementary knowledge.

## Novelty Assessment

The primary novelty is the multi-task formulation: demonstrating that joint training of a single non-autoregressive codec model on synthesis, editing, and continuation improves synthesis quality over single-task training. The ablation evidence for task complementarity is concrete and controlled. The component choices are derived entirely from prior work: the Conformer backbone, SoundStorm-style iterative RVQ decoding, MAGVIT-inspired cosine masking, and the supported codecs (SpeechTokenizer, DAC, HuBERT) are all existing tools. The framework design and the empirical finding about multi-task benefits constitute the contribution, not any structural innovation in architecture or training objective. Reproducibility is supported by the use of public datasets and standard codecs, though model size is not reported.

## Field Significance

Moderate — SpeechSEC contributes empirical evidence that multi-task joint training across synthesis, editing, and continuation tasks benefits each task through shared acoustic knowledge, demonstrated across three distinct tokenizer configurations on LibriTTS-R. The result is a useful data point for practitioners considering multi-task codec-based speech systems, though the evaluation is limited to one clean-speech corpus and the cross-system comparisons with prior work lack controlled experimental conditions.

## Claims

- **supports:** Multi-task joint training on synthesis, editing, and continuation tasks improves speech synthesis quality over single-task training in non-autoregressive codec models.
  > *Evidence:* In all three tokenizer configurations (ST, STDAC, HuDAC), multi-task SpeechSEC consistently outperforms the corresponding single-task baseline on MOS, voice preservation, WER, and CER, with gains confirmed by ablation. *(§3.2, §3.3, Tables 1, 3)*

- **refines:** In multi-task speech generation training, editing tasks primarily contribute intelligibility improvements while continuation tasks primarily contribute acoustic quality and voice preservation.
  > *Evidence:* Ablation removing the editing task increases WER by up to 4.4 points with minimal audio quality change; removing continuation degrades MOS by up to 0.18 and voice preservation by up to 0.06 with smaller intelligibility effects. *(§3.3, Table 3)*

- **supports:** Non-autoregressive masked token prediction frameworks can unify speech synthesis, editing, and continuation tasks through task-specific input conditioning within a single model.
  > *Evidence:* SpeechSEC handles all three tasks with a shared Conformer backbone, differentiating tasks via a Task Register embedding and per-task masking strategies, achieving competitive quality on editing (MOS 3.93) and continuation (MOS 3.63) alongside synthesis. *(§2, §3.2, Table 2)*

- **complicates:** The choice of semantic and acoustic token extractor significantly affects absolute synthesis quality in codec-based TTS, even when model architecture and training are held constant.
  > *Evidence:* With the same SpeechSEC architecture and training scheme, MOS ranges from 3.65 (STDAC) to 4.20 (ST) across the three tokenizer configurations, and voice preservation from 0.61 to 0.72, indicating that tokenizer quality is a dominant factor. *(§3.2, Table 1)*

## Limitations and Open Questions

The evaluation is restricted to LibriTTS-R, a clean studio-quality English corpus, leaving generalization to noisy, spontaneous, or multilingual speech untested. Cross-system comparisons with SoundStorm use independently reported numbers from separate evaluations, weakening the claim of surpassing prior state of the art. Model parameter count is not reported, preventing meaningful comparisons of capacity-normalised performance. Speech continuation lacks intelligibility metrics (WER, CER) by design, limiting interpretability of those results. The paper does not evaluate the editing task on real-world editing scenarios beyond random masking.

## Wiki Connections

- [[neural-codec|Neural Audio Codec]] — SpeechSEC uses SpeechTokenizer and Descript-Audio-Codec as acoustic token extractors, and all generation operates within the RVQ token space of these codecs.
- [[self-supervised-speech|Self-Supervised Speech]] — HuBERT serves as the semantic token extractor in the HuDAC configuration, making self-supervised speech representations a core architectural component in that pipeline variant.
- [[autoregressive-codec-tts|Autoregressive Codec TTS]] — SpeechSEC is explicitly positioned as a non-autoregressive alternative in the same codec token space, contrasting with autoregressive approaches such as AudioLM while inheriting the semantic-to-acoustic token generation paradigm.
- [[evaluation-metrics|Evaluation Metrics]] — the paper evaluates synthesis, editing, and continuation across WER, CER, MOS, and speaker cosine similarity, reporting task-specific baselines and multi-task deltas via ablation.
- [[subjective-evaluation|Subjective Evaluation]] — MOS scores for audio quality are reported following the evaluation protocol used in SoundStorm, enabling informal comparison with prior perceptual evaluation results.
