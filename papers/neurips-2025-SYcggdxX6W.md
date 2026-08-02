---
id: "neurips-2025-SYcggdxX6W"
title: "Word-Level Emotional Expression Control in Zero-Shot Text-to-Speech Synthesis"
authors: ["Tianrui Wang", "Haoyu Wang", "Meng Ge", "Cheng Gong", "Chunyu Qiang", "Ziyang Ma", "Zikang Huang", "Guanrou Yang", "Xiaobao Wang", "EngSiong Chng", "Xie Chen", "Longbiao Wang", "Jianwu Dang"]
organization: "Tianjin University / Nanyang Technological University / Shanghai Jiao Tong University"
venue: NeurIPS
venue_type: conference
year: 2025
month: 12
published_date: "2025-12-10"
ingested_date: "2026-08-02"
task: [TTS]
architecture: [autoregressive-LM, flow-matching]
conditioning: [text-conditioned, speaker-conditioned, zero-shot, prompt-conditioned, emotion-conditioned, prosody-conditioned]
training: [supervised, fine-tuning, distillation]
model_size: "not reported (backbone: CosyVoice2; added modules: 5-layer content aligner + 2-layer emotion aligner + lightweight attention-bias MLP, all comparatively small)"
codec_used: "CosyVoice2 supervised semantic speech tokens (inherited from backbone)"
datasets_train: ["LibriSpeech-100-Clean", "AISHELL-1", "ESD"]
datasets_eval: ["ESD test set", "SEED test-zh"]
metrics:
  - name: WER
    value: 3.192
    system: "WeSCon (2nd stage)"
    testset: "ESD-based English test set, word-level emotion/speed control"
  - name: CER
    value: 2.122
    system: "WeSCon (2nd stage)"
    testset: "ESD-based Chinese test set, word-level emotion/speed control"
  - name: SPK-SIM
    value: 0.532
    system: "WeSCon (2nd stage)"
    testset: "ESD-based English test set"
  - name: SMOS
    value: "3.96 ± 0.19"
    system: "WeSCon"
    testset: "15-listener subjective evaluation"
code_available: null
demo_available: true
url: "https://openreview.net/forum?id=SYcggdxX6W"
related_concepts: [emotion-synthesis, prosody-control, zero-shot-tts, subjective-evaluation]
related_papers: ["2412.10117", "2025.acl-long.313"]
field_significance:
  level: "high"
  type: [architectural-novelty]
generation:
  schema_version: 2
  date: "2026-08-02"
  runtime: "claude-code"
  provider: "anthropic"
  agent: speech-generation-ingest-agent
  model: "claude-sonnet-5"
  commit: "b6e27f4"
---

> [!abstract] NeurIPS · 2025 · Conference
> **Wang et al.** (Tianjin University / NTU / Shanghai Jiao Tong University) · [→ Paper](https://openreview.net/forum?id=SYcggdxX6W) · Demo: ✓ · Code: ?
>
> Introduces WeSCon, the first self-training framework that gives a pretrained zero-shot TTS model word-level control over both emotion and speaking rate within a single utterance, without requiring any training data that itself contains intra-sentence emotion or speed transitions.

## Problem

Emotional TTS has largely been limited to utterance-level control, where one emotion label or reference governs an entire sentence, which does not match how humans actually modulate emotion and speaking rate dynamically within a sentence. Prior attempts at finer-grained control either predict emotion from text alone (missing acoustic cues like intensity and prosody that carry much of the emotional signal) or use reference speech with time-varying expressive patterns (as in ELaTE and EmoCtrl-TTS), which requires large-scale emotional datasets with time-aligned intra-sentence transitions, data that is scarce, expensive to annotate, and rarely public. This raises the question the paper addresses directly: can word-level control of emotion and speaking rate be achieved using only ordinary, utterance-level emotional speech data, without any transition-labeled training examples at all?

## Method

WeSCon is a two-stage self-training framework built on a frozen CosyVoice2 backbone. In the first stage, a teacher pipeline achieves word-level control purely through inference-time engineering, with no emotional fine-tuning of the TTS model itself: the input text is segmented according to a user-specified emotion plan, and each segment is synthesized in a separate round using a different emotional prompt, with the previous round's tail text and speech tokens appended to the next round's input to create explicit continuity (a "tail-to-head" linkage that fits CosyVoice2's native continuation-style generation). A lightweight, ASR-trained content aligner (added without modifying CosyVoice2) predicts per-token text alignment to support this smoothing, requiring no emotional supervision itself. Word-level speaking-rate control is achieved by interpolating or downsampling the prompt speech tokens (slowing or speeding up the generated segment respectively), and speaker consistency across emotional prompts is maintained by sourcing prompts from the same speaker and letting CosyVoice2's flow-matching stage, rather than its language-modeling stage, carry the final target-speaker identity.

Because this multi-round teacher pipeline is complex at inference time, the second stage distills its capability into a simplified, single-pass student model via self-training. GPT-4o generates emotion-transition text plans, which are paired with ESD's ordinary (non-transition) emotional prompts and synthesized by the teacher into pseudo-labeled training examples, filtered by character accuracy and expressive similarity. The student (initialized from the same CosyVoice2 backbone) is fine-tuned on this filtered data with two losses: next-speech-token prediction and a token-level emotion-classification loss. To let the student track which of several emotional prompts a given output token should follow, the input sequence inserts explicit emotion-boundary tokens between prompt segments, and a dynamic emotional attention bias mechanism, a lightweight causal Transformer predicting token-level emotion labels, feeding a learned mixture of seven attention-bias templates, constrains each layer's self-attention toward the emotionally-relevant prompt region rather than letting attention drift across emotion boundaries.

![Overview of WeSCon. The 1st-stage teacher extends a zero-shot TTS model with dynamic speed control, transition smoothing, and multi-round inference to enable word-level emotion and speaking rate control. In the 2nd stage, it supervises a student model with a dynamic emotion attention bias (DEAB) to achieve the same control in an end-to-end manner with reduced inference complexity.](assets/neurips-2025-SYcggdxX6W/figure-2.png)

![The proposed self-training strategy. A teacher model under a complex multi-round inference manner supervises a student TTS model to enable word-level emotion and speaking rate control. The dynamic emotional attention bias mechanism further enhances expressive generation in a simplified end-to-end single-pass inference manner.](assets/neurips-2025-SYcggdxX6W/figure-4.png)

## Key Results

On both English and Chinese word-level emotion/speed control test sets built from ESD, both WeSCon stages outperform strong zero-shot TTS baselines (Index-TTS, F5-TTS, Spark-TTS, CosyVoice2) on expressiveness metrics, with the second-stage (self-trained, end-to-end) model achieving the highest emotion-similarity scores in both languages and cutting transition discontinuity (DNSV) roughly in half relative to the CosyVoice2 backbone (7.894 to 4.361 in English, 7.612 to 4.210 in Chinese). Character/word error rates remain close to the CosyVoice2 backbone's own level, and in a 15-listener subjective evaluation WeSCon outperforms all baselines on emotional expressiveness (EMOS), speaking-rate match (SPMOS), speaker similarity (SMOS), and transition naturalness (NMOS). On the standard (non-word-level) zero-shot TTS benchmark SEED test-zh, WeSCon's fine-tuned model performs essentially identically to unmodified CosyVoice2 (CER 1.45 vs. 1.47, speaker similarity 0.748 vs. 0.744), indicating the added controllability does not measurably degrade the backbone's original capability. Ablations show each mechanism matters: removing transition smoothing nearly doubles DNSV; removing the dynamic emotional attention bias degrades emotion similarity substantially; and replacing the paper's pretraining-consistent input data format with a naive concatenation sharply raises CER (2.166 to 4.141).

## Novelty Assessment

The paper's central claim, that word-level control over both emotion and speaking rate can be learned without any intra-sentence-transition training data, is the genuinely novel piece: prior fine-grained expressive TTS work (ELaTE, EmoCtrl-TTS) relied on exactly the kind of time-aligned transition data this method avoids needing. The two-stage teacher-student design is a reasonable if incremental combination of existing ideas (multi-round concatenative inference, self-training, attention biasing), but the dynamic emotional attention bias mechanism, using a predicted emotion trajectory to construct a learned mixture of attention-bias templates that steer attention toward the currently-relevant emotional prompt, is a concrete, ablated architectural contribution specific to this multi-prompt setting.

## Field Significance

high — WeSCon addresses a genuine and previously under-solved data-scarcity problem in fine-grained expressive TTS: prior approaches to word-level or transition-based emotional control required exactly the annotated, time-aligned data this method sidesteps entirely. The demonstrated preservation of the backbone's original zero-shot quality alongside the new controllability, combined with thorough ablations isolating each design component's contribution, makes this a well-validated and likely broadly applicable technique for adding fine-grained control to other pretrained autoregressive zero-shot TTS backbones beyond CosyVoice2 specifically.

## Claims

- **supports:** Word-level control of both emotion and speaking rate in a pretrained zero-shot TTS model can be achieved without any training data containing intra-sentence emotion or speed transitions, by using multi-round inference over multiple utterance-level emotional prompts with explicit transition smoothing.
  > *Evidence:* Using only ESD's non-transition emotional prompts, WeSCon's first-stage teacher achieves state-of-the-art word-level emotion and speaking-rate control metrics with no transition-labeled training data, reducing transition discontinuity (DNSV) from 7.612-7.894 (CosyVoice2 baseline) to 4.21-4.58 across English and Chinese. *(§4.2.1, Table 1)*
- **supports:** The fine-grained expressive control ability of a complex multi-round teacher inference pipeline can be distilled into a single end-to-end student model via self-training on teacher-generated pseudo-labels, eliminating the need for multi-round inference at deployment.
  > *Evidence:* The self-trained student model matches or slightly exceeds the multi-round teacher on nearly all objective and subjective metrics (e.g., DNSV improves from 4.980 to 4.210 on Chinese) while replacing multi-round concatenative inference with a single end-to-end forward pass. *(§4.2.1, Tables 1-2)*
- **supports:** Adding fine-grained expressive controllability to a pretrained zero-shot TTS model does not have to degrade its original zero-shot synthesis quality, if the control mechanism preserves the model's original input format.
  > *Evidence:* On the standard zero-shot TTS benchmark SEED test-zh, WeSCon's fine-tuned model achieves CER and speaker similarity nearly identical to the unmodified CosyVoice2 backbone (CER 1.45 vs. 1.47, S-SIM 0.748 vs. 0.744). *(§4.2.1, Table 3)*
- **complicates:** Explicit word-level speaking-rate control is not an independent prosodic dimension from emotional expressiveness; removing it measurably degrades emotion-similarity metrics, not just rate accuracy.
  > *Evidence:* Ablating the dynamic speed control mechanism degrades AutoPCP from 2.650 to 2.499 and emotion2vec similarity from 0.866 to 0.844, alongside the expected prosody-metric changes. *(§4.2.2, Table 4)*
- **complicates:** Self-training data scale for fine-grained expressive control has a diminishing and eventually reversing return when the underlying source dataset has limited emotional and speaker diversity, rather than benefiting monotonically from more synthetic training data.
  > *Evidence:* Performance of the self-trained student model peaks at 500 hours of teacher-generated data and declines beyond that point, which the authors attribute to overfitting given ESD's limited emotion categories and speaker identities. *(§4.2.2, Figure 5)*

## Limitations and Open Questions

The authors identify three concrete limitations: WeSCon achieves smooth signal-level transitions but does not semantically model gradual emotional evolution (human emotional shifts often pass through intermediate states); the model supports only a fixed set of discrete emotions with no compositional or blended expression (e.g., anger blended with sadness to convey despair); and emotional transition plans are currently predefined by GPT-4o rather than generated dynamically in response to context, limiting interactive flexibility. The self-training data is also sourced entirely from ESD, whose limited speaker and emotion diversity is directly implicated in the observed performance decline beyond 500 hours of synthetic training data.

## Wiki Connections

- [[emotion-synthesis|Emotion Synthesis]] — introduces the first self-training framework for word-level (rather than utterance-level) emotional expression control in a pretrained zero-shot TTS model.
- [[prosody-control|Prosody Control]] — implements an explicit dynamic speaking-rate control mechanism that operates at the word level, independent of the emotion control pathway, validated by ablation.
- [[zero-shot-tts|Zero-Shot TTS]] — builds on and explicitly validates preservation of a pretrained zero-shot TTS backbone's original synthesis quality after adding fine-grained controllability.
- [[subjective-evaluation|Subjective Evaluation]] — reports four separate human MOS ratings (expressiveness, speed match, speaker similarity, transition naturalness) from a 15-listener study.
- [[2412.10117|CosyVoice 2]] — used directly as the frozen backbone TTS model that both the teacher and student stages build on and fine-tune.
- [[2025.acl-long.313|F5-TTS]] — used as a zero-shot TTS baseline in the word-level emotion/speed control comparison (Table 1).
