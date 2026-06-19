---
id: "interspeech-2025-1344"
title: "Parameter-Efficient Fine-Tuning for Low-Resource Text-to-Speech via Cross-Lingual Continual Learning"
authors: ["Ki-Joong Kwon", "Jun-Ho So", "Sang-Hoon Lee"]
organization: Ajou University
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-06-03
task: [TTS]
architecture: [flow-matching]
conditioning: [text-conditioned, zero-shot, multilingual]
training: [fine-tuning, continual-learning]
model_size: "335.8M params (base F5-TTS); 5.81M trainable (1.72%)"
codec_used: "not reported"
datasets_train: ["KSS (Korean Single Speaker, 12.65h)"]
datasets_eval: ["KSS test (200 utterances)", "AI Hub Korean Multi-Speaker Speech Synthesis Dataset (164 utterances)"]
metrics:
  - name: CER
    value: 4.906
    system: PEFT-TTS
    testset: KSS single-speaker
  - name: WER
    value: 12.625
    system: PEFT-TTS
    testset: KSS single-speaker
  - name: MOS
    value: 3.88
    system: PEFT-TTS
    testset: KSS single-speaker
  - name: SMOS
    value: 2.81
    system: PEFT-TTS
    testset: AI Hub Korean multi-speaker
code_available: true
demo_available: true
url: "https://www.isca-archive.org/interspeech_2025/kwon25_interspeech.html"
related_concepts: ["flow-matching", "multilingual-tts", "speaker-adaptation", "zero-shot-tts", "self-supervised-speech"]
related_papers: ["2410.06885", "2412.10117"]
field_significance:
  level: moderate
  type: [engineering-integration]
generation:
  date: 2026-06-03
  model: claude-sonnet-4-6
  commit: "55d0339"
---

> [!abstract] Interspeech · 2025 · Conference
> **Kwon et al.** (Ajou University) · [→ Paper](https://www.isca-archive.org/interspeech_2025/kwon25_interspeech.html) · Demo: ✓ · Code: ✓
>
> A parameter-efficient fine-tuning framework that adapts F5-TTS to Korean using only 1.72% of model parameters, a 12-hour single-speaker dataset, and a single GPU, while preserving zero-shot TTS capability for previously learned languages.

## Problem

Adapting a multilingual TTS model to a new language typically requires large multi-speaker datasets and full model fine-tuning, both of which are prohibitively expensive in low-resource scenarios. Full fine-tuning also risks catastrophic forgetting: the model may lose its previously acquired language capabilities. Prior PEFT work on speech has concentrated on ASR and speaker adaptation, leaving cross-lingual TTS adaptation in low-data regimes largely unexplored.

## Method

The system builds on F5-TTS (arXiv:2410.06885), a fully non-autoregressive flow-matching TTS model based on a Diffusion Transformer (DiT) with ConvNeXt V2 text embeddings. Rather than fine-tuning the 335.8M-parameter base model, three lightweight adapter modules are inserted while keeping all pre-trained weights frozen:

**Conditioning Adapter** attaches a Conv-Adapter to the depth-wise convolution layers of the ConvNeXt V2 text encoder. It uses a compression factor of 0.25 with a kernel size of 3, modulating feature responses via squeeze-and-excitation parameters to adapt the text encoder to new phonemic patterns. A key empirical finding is that adapting the full text encoder (rather than using a compressed adapter) is necessary for reliable Korean pronunciation — the compression factor that works for English adaptation is too restrictive for a typologically distinct language.

**Prompt Adapter** applies LoRA to the linear projection layer that combines text and audio features (the joint prompt embedding). This adapter mediates a fundamental trade-off in single-speaker fine-tuning: as the model learns a specific speaker's acoustic characteristics, it risks degrading generalisation across speakers. DropPath regularisation (rate 0.3) is applied here to balance pronunciation accuracy against speaker similarity, preventing overfitting to the target speaker's voice during adaptation.

**DiT LoRA Adapter** injects LoRA (rank 16) into the query and value projection layers of the DiT blocks, which are most strongly tied to speaker-acoustic characteristics. LoRA rank 16 is chosen deliberately over larger ranks — experiments show that increasing rank improves pronunciation but reduces zero-shot speaker adaptation capability, and a small rank preserves the pre-trained model's generative flexibility.

Training runs for 355,000 steps on the 12.65-hour KSS Korean single-speaker dataset with a batch size of 3,200 Mel-spectrogram frames, AdamW at 1e-5 learning rate with linear warm-up, on a single NVIDIA TITAN RTX GPU. Only 5.81M parameters (1.72% of total) are updated, yielding approximately 2× training speed over full fine-tuning.

## Key Results

On the single-speaker Korean test set, PEFT-TTS achieves CER 4.906 and WER 12.625, improving substantially over fully fine-tuned F5-TTS (CER 5.671, WER not clearly isolatable from the table due to formatting artefacts) and dramatically over F5-TTS trained from scratch on the same data (CER 17.345, WER 45.883). Speaker similarity (SIM-O: 0.641) is comparable to full fine-tuning (0.659). Naturalness MOS reaches 3.88 ± 0.16, versus 3.58 ± 0.32 for the fully fine-tuned baseline and 2.64 ± 0.41 for training from scratch.

On the multi-speaker evaluation (AI Hub Korean dataset), full fine-tuning loses zero-shot capability entirely (NMOS 2.24 ± 0.21), while PEFT-TTS retains it (NMOS 3.06 ± 0.28). CosyVoice 2, trained on large-scale multilingual data including Korean, achieves better WER (21.6 vs. 39.4 for PEFT-TTS multi-speaker) but PEFT-TTS with its single-GPU constraint approaches reasonable speaker similarity scores.

The ablation confirms that all three adapter modules contribute: freezing the prompt embedding improves audio quality but hurts speaker similarity; larger DiT LoRA ranks improve pronunciation at the cost of generalisation; and the Conditioning Adapter compression factor must be low (0.25) for a morphologically rich language like Korean.

> [!note]
> Comparisons between PEFT-TTS and CosyVoice 2 are not on equivalent footing — CosyVoice 2 is trained on a large-scale multilingual corpus that includes Korean, while PEFT-TTS adapts from an English/Chinese pre-trained model with 12h Korean data. The comparison is illustrative rather than a fair benchmark.

## Novelty Assessment

The contribution is primarily engineering integration: LoRA and adapter techniques already established in NLP (and appearing in concurrent TTS work) are applied to the three distinct functional modules of F5-TTS, each chosen for their architectural role in language conditioning, prompt fusion, and acoustic generation. The choice of which layer to adapt and the DropPath-mediated trade-off analysis constitute genuine design insight, but no new training objective or architecture is introduced. The value of the paper is in the practical demonstration — 12 hours, one GPU, 1.72% of parameters — and the systematic ablation quantifying adapter interactions, rather than in conceptual novelty.

## Field Significance

Moderate — This paper advances the practitioner's toolkit for low-resource multilingual TTS by establishing that F5-TTS-style flow-matching models can be cross-lingually adapted under extreme resource constraints through targeted PEFT. The empirical finding that full fine-tuning destroys zero-shot capability while PEFT preserves it is a useful data point for the field, reinforcing that parameter-efficient methods are not just computationally convenient but functionally necessary when preserving pre-trained behaviour matters.

## Claims

- Adapter-based fine-tuning preserves zero-shot TTS capability in multilingual models more reliably than full parameter fine-tuning on single-speaker low-resource data.
- Cross-lingual adaptation of TTS models is feasible with fewer than 15 hours of single-speaker data when a multilingual pre-trained foundation is available.
- In flow-matching TTS fine-tuning, the text conditioning module requires more parameters to adapt to a typologically distant language than the acoustic generation module.
- Low-rank adaptation in DiT-based TTS models exhibits a rank-dependent trade-off between linguistic accuracy and speaker generalisation, with optimal rank dependent on dataset scale.

## Limitations and Open Questions

> [!warning]
> Multi-speaker evaluation shows substantially higher WER (21.6%) compared to a large-scale trained system (10.9%), and the system shows hallucination artefacts (word repeating and skipping) not fully resolved by the proposed adapters. Reproducibility is limited to the Korean-specific KSS dataset; generalisation to other low-resource languages is assumed but not tested.

The paper does not address languages with non-Latin scripts that differ even more from F5-TTS's training data (e.g., Arabic, Swahili). The speaker similarity ceiling for multi-speaker scenarios remains low given that adaptation data comes from a single female speaker only. The interaction between DropPath rate and language-pair linguistic distance is not explored, leaving hyperparameter transfer uncertain. Future work identified by the authors includes more robust text conditioning and improved alignment methods to reduce hallucinations.

## Wiki Connections

Concept pages most relevant to this paper: [[flow-matching]], [[multilingual-tts]], [[zero-shot-tts]], [[speaker-adaptation]], [[self-supervised-speech]].

In-corpus papers this work builds on: [[2410.06885]] (F5-TTS, the pre-trained base model), [[2412.10117]] (CosyVoice 2, used as a large-scale multilingual baseline).
