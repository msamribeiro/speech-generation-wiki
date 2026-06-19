---
id: "interspeech-2025-0874"
title: "Efficient and Direct Duplex Modeling for Speech-to-Speech Language Model"
authors: ["Ke Hu", "Ehsan Hosseini-Asl", "Chen Chen", "Edresson Casanova", "Subhankar Ghosh", "Piotr Zelasko", "Zhehuai Chen", "Jason Li", "Jagadeesh Balam", "Boris Ginsburg"]
organization: NVIDIA
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-06-03
task: [SCA]
architecture: [autoregressive-LM, hybrid]
conditioning: [text-conditioned, prompt-conditioned]
training: [supervised, fine-tuning]
model_size: "~1.2B params (100M encoder + 1.1B TinyLlama)"
codec_used: "NanoCodec 0.6 kbps, 12.5 Hz"
datasets_train: ["MS MARCO (ASR-QA, 20k hours)", "Alpaca (0.2k hours)", "Internal SFT (3k hours)", "UltraChat (3k hours)", "Topic (0.3k hours)"]
datasets_eval: ["UltraChat", "Roleplay", "Topic", "ASR-QA", "Alpaca", "Impatient"]
metrics:
  - name: UTMOS
    value: 4.3
    system: proposed
    testset: UltraChat
  - name: UTMOS
    value: 3.9
    system: Moshi
    testset: UltraChat
  - name: UTMOS
    value: 4.0
    system: proposed
    testset: Impatient
  - name: UTMOS
    value: 3.8
    system: Moshi
    testset: Impatient
  - name: MOS
    value: 4.75
    system: proposed (personalized NanoCodec 0.6 kbps)
    testset: codec reconstruction
code_available: true
demo_available: true
url: "https://www.isca-archive.org/interspeech_2025/hu25f_interspeech.html"
related_concepts: ["spoken-language-model", "speech-to-speech", "neural-codec", "streaming-tts", "autoregressive-codec-tts"]
related_papers: []
field_significance:
  level: high
  type: [architectural-novelty, engineering-integration]
generation:
  date: 2026-06-03
  model: claude-sonnet-4-6
  commit: "55d0339"
---

> [!abstract] Interspeech · 2025 · Conference
> **Ke Hu et al.** (NVIDIA) · [→ Paper](https://www.isca-archive.org/interspeech_2025/hu25f_interspeech.html) · Demo: ✓ · Code: ✓
>
> A novel full-duplex speech-to-speech architecture that models simultaneous user and agent streams via a continuous encoder for user input and a codec for agent output, enabling turn-taking and barge-in without speech pretraining — the first open-source duplex S2S model with training and inference code.

## Problem

Spoken dialogue systems capable of real-time interaction — where users can interrupt the agent mid-response (barge-in) without waiting for a turn boundary — remain challenging to build. Prior half-duplex models operate in strict turn-based mode, and earlier full-duplex efforts either require extensive speech-text pretraining on top of the LLM backbone, add specialised turn-taking submodules that break end-to-end design, or use codec representations for both user and agent that complicate codec personalisation and inflate bitrate requirements. The result is a class of systems that are either computationally expensive to train, closed-source, or architecturally fragile.

## Method

The proposed duplex S2S model takes two simultaneous input streams: a continuous user speech stream and a combined agent speech-and-text stream. User speech is encoded by a 100M-parameter causal streaming CTC encoder (80ms right context) at an 80ms frame rate; the encoder output is projected via a modality adapter before being fed to the LLM. Agent speech is tokenised by a partially causal NanoCodec producing 4 parallel codebooks at 12.5 frames per second and 0.6 kbps. The backbone LLM is TinyLlama-1.1B-chat, initialised from a pretrained checkpoint and extended with codec vocabulary tokens. At each timestep the two input embeddings are time-aligned and summed (channel fusion) before being processed by the LLM.

![The proposed duplex S2S model without requiring speech-text pretraining. Our model includes a streaming speech encoder, a personalized codec, and an LLM. The model is trained to predict both text and audio channels in parallel with turn-level alignments.](assets/interspeech-2025-0874/figure-1.png)

The model is trained with multi-channel next-token prediction: text and speech targets are maintained in separate channels, aligned at the turn level (not word level), with text weighted 3× relative to speech in the loss. A small one-token delay is introduced on the speech channel to let the model condition on the text channel first. Because the user stream uses a pretrained encoder rather than codec tokens, no speech-text pretraining of the LLM backbone is required — a sharp contrast with prior duplex systems such as Moshi. Codec personalisation for the agent voice is enabled by fine-tuning NanoCodec on 21k utterances from the target speaker; this is practical precisely because agent and user modalities are handled by separate components.

Training data totals approximately 26.5k hours, constructed synthetically from MS MARCO, Alpaca, UltraChat, internal SFT, and topic conversation datasets. Multi-turn and barge-in behaviours are injected by splitting QA pairs into dual streams and cutting off agent speech at simulated interruption points. The model is trained on 32 A100 GPUs using the NVIDIA NeMo Toolkit.

## Key Results

On turn-taking and speech generation quality (Table 2), the model substantially outperforms Moshi on barge-in success rate (83.0% vs. 56.0% on UltraChat; 94.5% vs. 55.1% on the adversarial Impatient set), with lower barge-in latency (0.52s vs. 0.63s on UltraChat) and zero false alarms. UTMOS scores show better speech quality: 4.3 vs. 3.9 on UltraChat and 4.0 vs. 3.8 on Impatient. On reasoning quality (Table 3, GPT-4o-mini scores), the proposed model outperforms Moshi across all five evaluation sets despite using a far smaller backbone; it is competitive with an oracle cascaded system on two of five sets.

Codec personalisation (Table 4) is a notable sub-result: the personalised 0.6 kbps NanoCodec achieves MOS 4.75, CER 1.36%, and SECS 0.94 — outperforming both 1.2 kbps NanoCodec and 1.1 kbps Mimi on all audio reconstruction metrics while operating at roughly half the bitrate.

> [!note]
> Comparisons with Moshi are on shared test sets but Moshi uses a larger backbone and different training data, so the comparisons are not fully controlled. The "Impatient" set is a synthetic stress-test, not a natural user distribution.

## Novelty Assessment

The core architectural contribution — separating user and agent streams into a continuous encoder path and a codec path, summed via channel fusion at the LLM input — is a genuine structural choice that resolves the tension between speech perception and generation that plagues unified codec-only duplex models. Eliminating speech pretraining is a direct practical consequence of this design and meaningfully lowers the barrier to entry. The codec personalisation method (fine-tuning NanoCodec on target speaker data) is straightforward but the evaluation is thorough. The turn-taking evaluation metrics (barge-in success rate, barge-in latency, false alarm rate) are a useful addition to the measurement toolkit for SCA systems, though not deeply novel. The open-source release with both training and inference code is a significant contribution to reproducibility in a subfield that has been largely dominated by closed systems.

## Field Significance

> [!tip]
> High — This paper advances the practical frontier of full-duplex spoken dialogue systems by demonstrating that speech pretraining can be eliminated through architectural separation of encoder and codec roles, and by providing the first open-source duplex S2S model with training code. The barge-in evaluation framework and the codec personalisation results are directly useful to practitioners building real-time voice agents.

## Claims

- Separating user and agent stream representations — using a pretrained speech encoder for input and a neural codec for generation — allows full-duplex S2S models to bypass LLM speech pretraining without sacrificing conversation quality.
- Codec personalisation through fine-tuning on target-speaker data can recover audio quality at half the bitrate of an untuned codec, as measured by MOS, CER, and speaker similarity.
- Turn-level alignment between text and speech tokens in duplex training is sufficient to learn barge-in behaviour; word-level alignment provides no measurable improvement.
- Full-duplex end-to-end models remain at a reasoning disadvantage compared to cascaded oracle systems, though the gap narrows as backbone LLM quality increases.
- Open-source availability of training code and model weights is a critical bottleneck for research progress in full-duplex spoken dialogue, given the historical concentration of such systems in closed industrial labs.

## Limitations and Open Questions

> [!warning]
> The backbone is TinyLlama-1.1B, a relatively small LLM. The reasoning gap between the end-to-end model and the GT+LLM cascaded oracle is real and acknowledged, particularly on QA tasks. Scaling to a larger backbone remains untested and its interaction with the duplex architecture is an open question.

Training data is entirely synthetic (TTS-generated user and agent speech) except for the ASR-QA portion, which introduces a domain mismatch with natural conversation. The fixed 0.64-second silence inserted before agent turns is a hard-coded heuristic that will affect latency in practice and may not generalise to more varied conversational pacing. The evaluation does not include a listening test (MOS via human raters) for duplex conversation quality — UTMOS and GPT score are proxies. The first-response latency metric is not comparable to Moshi because Moshi's proactive interruption behaviour makes the metric inapplicable, which limits direct system comparison.

## Wiki Connections

This paper primarily informs [[spoken-language-model]], [[speech-to-speech]], [[neural-codec]], and [[streaming-tts]]. The duplex architecture's channel fusion approach connects to [[autoregressive-codec-tts]] and the codec personalisation work relates to [[speaker-adaptation]]. The turn-taking evaluation framework is relevant to [[evaluation-metrics]].
