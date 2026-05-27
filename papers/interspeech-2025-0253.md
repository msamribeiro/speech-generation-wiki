---
id: interspeech-2025-0253
title: "Long-Context Speech Synthesis with Context-Aware Memory"
authors: [Zhipeng Li, Xiaofen Xing, Jingyuan Xing, Hangrui Hu, Heng Lu, Xiangmin Xu]
organization: South China University of Technology / Alibaba Group
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-05-27
task: [TTS]
architecture: [autoregressive-LM, hybrid]
conditioning: [speaker-conditioned, prompt-conditioned]
training: [supervised]
model_size: "not reported"
codec_used: "50Hz speech tokenizer (CosyVoice-LM backbone)"
datasets_train: ["15,000+ hours Chinese Mandarin audiobooks (internal), ~75,000 novel chapters"]
datasets_eval: ["internal Chinese Mandarin audiobook test set (100 chapters, 20 sentence-level + 10 long-form samples)"]
metrics:
  - name: MOS
    value: 3.796
    system: proposed (CAM-based Long-Context LM)
    testset: internal Chinese Mandarin audiobook
  - name: MOS
    value: 3.557
    system: MMCE-Qformer
    testset: internal Chinese Mandarin audiobook
  - name: MOS
    value: 3.489
    system: CLAP-RAG
    testset: internal Chinese Mandarin audiobook
  - name: MOS
    value: 3.468
    system: Baseline
    testset: internal Chinese Mandarin audiobook
  - name: CER
    value: 4.140
    system: proposed
    testset: internal Chinese Mandarin audiobook
  - name: SPK-SIM
    value: 85.685
    system: proposed (mean)
    testset: internal Chinese Mandarin audiobook
code_available: false
demo_available: true
url: https://www.isca-archive.org/interspeech_2025/li25b_interspeech.html
related_concepts: [autoregressive-codec-tts, prosody-control, neural-codec]
related_papers: []
---

# Long-Context Speech Synthesis with Context-Aware Memory

**Paper:** [https://www.isca-archive.org/interspeech_2025/li25b_interspeech.html](https://www.isca-archive.org/interspeech_2025/li25b_interspeech.html)

**One-sentence contribution:** A Context-Aware Memory (CAM) block for autoregressive LM-based TTS that compresses, retrieves, and dynamically updates both speech and text memory across paragraph-length sequences, enabling more coherent and expressive long-form synthesis at lower inference cost than prior methods.

## Problem

Current TTS systems generate speech sentence by sentence and concatenate the results, ignoring the cross-sentence contextual dependencies that make paragraph-level speech coherent. This produces inconsistencies in style, timbre, and prosody across a paragraph. Prior long-context TTS methods either feed excessively long speech prompts (increasing KV-cache cost) or compress too little context (losing distal information). There is no efficient mechanism that maintains a concise, dynamically updated summary of the preceding paragraph.

## Method

The system builds on the CosyVoice-LM backbone (50Hz speech tokenizer) and extends it with a Context-Aware Memory (CAM) block and a prefix mask.

**CAM block** operates in three stages per sentence:
- **Compress:** A Perceiver Resampler compresses the target text into a fixed-length (32-token) latent query.
- **Retrieve:** The compressed query cross-attends over a fused representation of the long-term memory (Mem_{n-1}) and the immediately preceding sentence's speech/text history (History_{n-1}), extracting the most contextually relevant information as Mem*_n.
- **Update:** The updated memory Mem_n is a learned scalar blend of Mem*_n and Mem_{n-1}: `Mem_n = sigmoid(α) * Mem*_n + (1-sigmoid(α)) * Mem_{n-1}`. This allows the model to balance recent context with long-term paragraph structure.

Separate CAM-Speech and CAM-Text blocks (shared structure, independent weights) process acoustic and textual modalities, producing Mem-S_n and Mem-T_n. These memory tokens, together with paragraph-level X-vectors from Cam++, are prepended to the LM input for sentence-level speech token generation. Flow matching and a vocoder convert speech tokens to waveforms.

**Prefix mask** replaces the standard causal mask for the memory/text prefix tokens, applying bidirectional attention to them while maintaining unidirectional attention over generated speech tokens. This improves in-context learning without breaking autoregressive generation.

Training used ~15,000 hours of Chinese Mandarin audiobooks for 100M steps with 10,000 tokens/batch dynamic batching. Inference requires only one preceding context plus fixed-length 64-token memory, compared to 5 full preceding sentences (MMCE-Qformer) or variable-length prompts up to ~900 tokens (CLAP-RAG).

## Key Results

On the internal Chinese Mandarin audiobook test set (Table 1):

| System | MOS | CoMOS | SpeechBERT | CER | SPK-SIM (mean, var) |
|---|---|---|---|---|---|
| Proposed | **3.796** | **3.992** | **80.448** | **4.140%** | 85.685 (0.019) |
| MMCE-Qformer | 3.557 | 3.885 | 79.031 | 5.075% | 85.110 (0.021) |
| CLAP-RAG | 3.489 | 3.717 | 78.892 | 6.234% | 84.920 (0.037) |
| Baseline | 3.468 | 3.460 | 77.776 | 5.850% | 85.051 (0.035) |

The proposed method achieves the best naturalness (MOS), coherence (CoMOS), content accuracy (CER), and speaker consistency (lowest SPK-SIM variance) while using only fixed 64-token context — the same cost as MMCE-Qformer but without requiring 5 full preceding sentences.

Ablation confirms that both speech memory (Mem-S) and text memory (Mem-T) contribute, with speech memory having the larger effect (likely because speech captures richer prosodic information than text). The prefix mask provides a significant boost to both naturalness and coherence.

## Novelty Assessment

The main contribution is the CAM block design: a perceiver-resampler-based compression fed into a cross-attention retrieval over a blended long-term/local-context memory, updated dynamically per sentence. This is architecturally inspired by Infini-Attention (Munkhdalai et al., 2024) but applied to TTS. The idea of maintaining separate speech and text memory streams is genuinely new. The prefix mask for LM-based TTS is an independent but useful contribution. Overall, the contribution is primarily architectural — the training data, backbone, and evaluation are fairly standard. The context budget reduction (from variable to fixed 64 tokens) is practically valuable.

## Limitations and Open Questions

- Evaluation is monolingual (Chinese Mandarin only); generalizability to other languages is unconfirmed.
- The gap to Ground Truth (MOS 4.406 vs 3.796) remains substantial; paragraph-level naturalness is still an open problem.
- No code release; replication requires re-implementing CAM on a CosyVoice backbone.
- The model is evaluated only on single-speaker audiobooks; how it handles multi-speaker paragraphs is unknown.
- The fixed 32-embedding Perceiver Resampler bottleneck may lose fine-grained phonetic detail.

## Wiki Connections

This paper directly advances [[autoregressive-codec-tts]] by introducing a memory mechanism for paragraph-level coherence in LM-based TTS. It is closely related to work on [[prosody-control]] since coherence across sentences is fundamentally a prosody continuity problem. The use of a fixed-size compressed memory is analogous to approaches in efficient neural architectures. The speech tokenizer and flow-matching vocoder tie into [[neural-codec]].
