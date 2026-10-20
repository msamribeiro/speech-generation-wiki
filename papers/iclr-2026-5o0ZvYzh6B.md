---
id: "iclr-2026-5o0ZvYzh6B"
title: "UniSS: Unified Expressive Speech-to-Speech Translation with Your Voice"
authors: ["Sitong Cheng", "Weizhen Bian", "Xinsheng Wang", "Ruibin Yuan", "Jianyi Chen", "Shunshun Yin", "Yike Guo", "Wei Xue"]
organization: "HKUST / Soul AI Lab"
venue: ICLR
venue_type: conference
year: 2026
month: 4
published_date: "2026-04-27"
ingested_date: "2026-10-20"
task: [TTS]
architecture: [autoregressive-LM]
conditioning: [speaker-conditioned, zero-shot, multilingual]
training: [supervised, fine-tuning]
model_size: "1.5B params (Qwen2.5-1.5B-Instruct backbone); 0.5B UniSS-Small variant"
codec_used: "BiCodec (32 global speaker tokens + 50 Hz semantic tokens, 16 kHz) for generation; GLM-4 speech tokenizer (12.5 Hz) for input content"
datasets_train: ["UniST General (44.8k h, synthesized EN-ZH S2ST)", "UniST High-Quality (19.8k h)", "WMT17 (MT text)", "AISHELL-3", "CoVoST2", "Common Voice 4.0 (EN, ZH)", "CVSS-T", "DailyTalk", "Emilia", "FLEURS", "GigaSpeech", "Hi-Fi TTS", "HQ-Conversations", "LibriSpeech", "LibriTTS-R", "MAGICDATA", "NCSSD-C", "NCSSD-R", "VCTK", "WenetSpeech4TTS"]
datasets_eval: ["CVSS-T (EN-ZH, ZH-EN test)", "ESD (300 utt. sample)", "CREMA-D (300 utt. sample)", "FLEURS (EN, ZH test)", "LibriSpeech test-clean", "LibriSpeech test-other"]
metrics:
  - name: UTMOS
    value: 3.76
    system: UniSS (Q)
    testset: CVSS-T EN-ZH (Table 1)
  - name: UTMOS
    value: 3.86
    system: UniSS (Q)
    testset: CVSS-T ZH-EN (Table 1)
  - name: UTMOS
    value: 3.77
    system: UniSS (P)
    testset: CVSS-T EN-ZH (Table 1)
  - name: UTMOS
    value: 3.41
    system: UniSS (Q)
    testset: FLEURS EN-ZH (Table D3)
  - name: UTMOS
    value: 4.16
    system: UniSS (Q)
    testset: FLEURS ZH-EN (Table D3)
  - name: MOS
    value: 4.45
    system: UniSS (Q), naturalness
    testset: ESD + CREMA-D emotion sample (Table 2)
  - name: SMOS
    value: 4.42
    system: UniSS (Q), speaker similarity to source
    testset: ESD + CREMA-D emotion sample (Table 2)
  - name: MOS
    value: 4.51
    system: UniSS (Q), emotion similarity to source
    testset: ESD + CREMA-D emotion sample (Table 2)
  - name: WER
    value: 4.75
    system: UniSS, TTS task
    testset: LibriSpeech test-clean (Table D4)
  - name: SPK-SIM
    value: 0.568
    system: UniSS, TTS task
    testset: LibriSpeech test-clean (Table D4)
  - name: WER
    value: 2.4
    system: UniSS, ASR task
    testset: LibriSpeech test-clean (Table D4)
code_available: null
demo_available: true
url: "https://openreview.net/forum?id=5o0ZvYzh6B"
related_concepts: [speech-to-speech, spoken-language-model, autoregressive-codec-tts, multilingual-tts, subjective-evaluation]
related_papers: ["2503.01710", "2412.10117", "2412.02612", "2503.20215", "2407.10759", "2410.21276", "2412.15115", "2204.02152", "2312.05187", "2507.17527", "2212.04356", "2407.05361", "2206.08317", "2306.12925"]
field_significance:
  level: "moderate"
  type: [engineering-integration, dataset-contribution]
generation:
  schema_version: 2
  date: "2026-10-20"
  runtime: "claude-code"
  provider: "anthropic"
  agent: speech-generation-ingest-agent
  model: "claude-opus-5-5"
  commit: "fb8dafe"
---

> [!abstract] ICLR · 2026 · Conference
> **Sitong Cheng et al.** (HKUST / Soul AI Lab) · [→ Paper](https://openreview.net/forum?id=5o0ZvYzh6B) · Demo: ✓ · Code: ?
>
> UniSS turns an unmodified text LLM (Qwen2.5-1.5B-Instruct) into a single-stage, voice-preserving English-Chinese speech-to-speech translator by feeding it separate speaker and content tokens, having it write the transcript and translation as text before emitting target speech tokens, and training it on a 44.8k-hour synthetic expressive S2ST corpus.

## Problem

Expressive speech-to-speech translation (S2ST) must translate content while keeping the source speaker's timbre, emotion, and timing. Cascaded ASR, MT, and TTS pipelines accumulate errors and discard paralinguistic information. Recent LLM-based S2ST systems either predict multi-stream acoustic tokens with heavily modified architectures trained from scratch (Hibiki) or split generation into an autoregressive semantic stage plus a second autoregressive or non-autoregressive acoustic stage (PolyVoice, AudioPaLM, SeamlessExpressiveLM, MSLM-S2ST). The authors argue that these designs add architectural complexity and treat the LLM as a generic sequence converter, leaving its pre-trained text translation ability unused. A second obstacle is data: parallel S2ST corpora that keep the same voice and emotion on both sides are either small (CVSS) or noisy web mining (Seamless).

## Method

UniSS represents each utterance with three single-stream token types from two existing tokenizers (§3.1.2). From BiCodec (the Spark-TTS codec), a global encoder yields a fixed sequence of 32 speaker tokens capturing timbre, emotion, and prosody, and a semantic tokenizer yields 50 Hz semantic tokens used for generation. Source content is encoded separately with the GLM-4 speech tokenizer, a quantized Whisper encoder at 12.5 Hz. The authors chose this split after finding that BiCodec semantic tokens reconstruct well but are weak for content understanding, a finding the ablation later quantifies. The source is therefore represented as (speaker tokens, linguistic tokens) and the target as BiCodec semantic tokens. The BiCodec decoder reconstructs 16 kHz audio from the source speaker tokens concatenated with the generated target semantic tokens, so voice transfer happens at decoding time without a second acoustic model (§3.1.3).

The generator is Qwen2.5-1.5B-Instruct with its vocabulary expanded to 180,407 entries to cover speech and control tokens. There are no other architectural changes, and training uses standard next-token prediction (§3.1.1, §5.1). The prompt holds a task token, a target-language token, a discretized speed token (source-to-target duration ratio in 0.1 steps), and then the source speaker and linguistic tokens. Cross-modal chain-of-thought (CoT) prompting breaks translation into steps (§3.2). In Quality mode the model emits the source transcript, then the target-language text, then target semantic tokens ("listen, translate, speak"). Performance mode skips the transcript. A direct mode with no intermediate text is also trained.

Training runs in three phases (§3.3, Appendix B.1). Phase 1 aligns speech and text through ASR, TTS, speech-to-text translation, and text MT tasks, using 77.1k hours of speech plus 2.3B WMT17 MT tokens (about 32B tokens per epoch, 3 epochs). Phase 2 introduces the three S2ST output formats on UniST General, mixed 2:1 with Phase 1 data (about 55B tokens). Phase 3 anneals the learning rate on UniST High-Quality (about 10B tokens). Training took about 6 days on 16 H800 GPUs. A 0.5B variant (UniSS-Small) is also trained.

UniST, the training corpus, is built synthetically (§4). Public EN and ZH TTS/ASR corpora are cleaned with Paraformer (WER > 0.05 discarded), translated with Qwen2.5-72B-Instruct, and voiced in the target language by Spark-TTS conditioned on the source audio. Samples are kept only if the target WER is at most 0.01 and the duration ratio falls within [0.5, 2.0]. That gives UniST General at 44.8k hours. UniST High-Quality (19.8k hours) adds VAD trimming and a tighter [0.7, 1.5] duration ratio.

## Key Results

On CVSS-T (Table 1, EN-ZH | ZH-EN), UniSS (Q) reaches Speech-BLEU 32.20 | 24.28. That beats every cascaded baseline (best 26.94 | 20.86 from SeamlessM4T-v2 + CosyVoice 2), SeamlessExpressive (24.45 | 15.84), and GPT-4o (31.64 | 19.27). Qwen2.5-Omni scores higher on ZH-EN Speech-BLEU (22.66) than the cascades but fails badly on EN-ZH (7.10). Duration compliance is where UniSS stands out most: SLC-0.2 is 0.98 | 0.87 against at most 0.68 | 0.54 for any baseline. Note that UniSS conditions on a speed token and its training targets were filtered by duration ratio, so this is partly a designed-in advantage. On prosody (A.PCP), UniSS (2.71–2.75) falls slightly below SeamlessExpressive (2.83 | 2.87) and the cascades (2.80–2.87). UTMOS (3.76–3.77 | 3.86) matches the cascades on EN-ZH and exceeds every end-to-end S2ST system. GPT-4o and Qwen2.5-Omni still have higher ZH-EN UTMOS (4.18, 4.30), but neither preserves the source voice (Appendix A). FLEURS results (Table D3) follow the same pattern. There, UniSS (Q) leads EN-ZH Speech-BLEU among open systems (36.16) but trails GPT-4o (38.58), and on ZH-EN it is roughly level with the larger systems (23.06 against 23.84 for SeamlessM4T-v2-Large and 23.21 for GPT-4o).

In the subjective test (Table 2, six bilingual raters, 5-point scales, emotional speech sampled from ESD and CREMA-D), UniSS (Q) gets 4.51 for emotion similarity, 4.42 for speaker similarity, and 4.45 for naturalness. SeamlessExpressive gets 3.56 / 2.94 / 3.10, and the closed Seed LiveInterpret 2.0 gets 4.56 / 4.19 / 4.69. The speaker-similarity margin over the CosyVoice 2 cascades is small (4.42 against 4.33–4.35), and no confidence intervals are reported.

The ablations (Table 4, CVSS-T Speech-BLEU, Performance mode) show which components matter most. Generating speech directly without intermediate text drops Speech-BLEU by 14.94 | 14.40. Swapping the GLM-4 linguistic tokens for BiCodec semantic tokens on the input side costs 15.01 | 8.73. Skipping Phase 1 alignment and training only on UniST costs 7.18 | 10.15, and Phase 3 refinement adds 0.90 | 2.06. Performance mode saves little time: it is only 1.07× faster than Quality mode for the LM component and loses 1.84 average Speech-BLEU (Table 3). UniSS-Small (P) reaches 1.25× at 25.68. Used as a plain TTS model, UniSS is clearly behind Spark-TTS on LibriSpeech test-clean (WER 4.75 vs 1.98, SIM 0.568 vs 0.584, Table D4).

## Novelty Assessment

The contribution is mainly integration plus data. Every model component already exists: the Qwen2.5 backbone, BiCodec speaker and semantic tokens, and the GLM-4 speech tokenizer. Chain-of-thought text intermediates for S2ST also appear in earlier work (SeamlessExpressiveLM). What is new is the arrangement. Separate understanding-side (Whisper-derived) and generation-side (BiCodec) tokens, with a global speaker-token stream carried straight to the decoder, let a stock text LLM do expressive S2ST in one autoregressive pass with no second acoustic stage. The ablations give real evidence for both the tokenizer split and the text-intermediate CoT. UniST itself is a distillation of Spark-TTS, so voice and emotion "preservation" in the training targets is bounded by that TTS model. The model largely learns to translate text and then re-synthesize with a Spark-TTS-like voice transfer. The baselines are broad (cascades, SeamlessExpressive, GPT-4o, Qwen2.5-Omni, Seed LiveInterpret 2.0). However, evaluation covers only English-Chinese, and the voice-preservation advantage rests on a six-rater MOS study without objective speaker-similarity scores for S2ST.

## Field Significance

Moderate — UniSS shows that an off-the-shelf text LLM can be adapted to single-stage, voice-preserving S2ST through token design and CoT output formatting alone, without architectural changes. It also gives controlled ablations showing that intermediate text generation and a content-oriented input tokenizer each account for large translation-fidelity gains. The 44.8k-hour UniST corpus is a sizable synthetic EN-ZH expressive S2ST resource. Its expressiveness is inherited from the TTS model used to build it.

## Claims

- **supports:** Generating intermediate source and target text before target speech tokens substantially improves translation fidelity in LLM-based speech-to-speech translation compared with direct speech-token generation.
  > *Evidence:* Running the same trained model in direct S2ST mode without intermediate text lowers CVSS-T Speech-BLEU by 14.94 (EN-ZH) and 14.40 (ZH-EN) relative to the Performance-mode base. Quality mode, which also transcribes the source, is a further 1.84 points better on average than Performance mode. *(§5.6.3, Table 4; §5.5, Table 3)*
- **supports:** Discrete speech tokens optimized for waveform reconstruction are poorer inputs for content understanding than tokens derived from a supervised ASR encoder, which motivates separate tokenizers for the understanding and generation sides of a speech LM.
  > *Evidence:* Replacing the GLM-4 (quantized Whisper encoder) linguistic tokens with BiCodec semantic tokens as the source representation drops Speech-BLEU by 15.01 (EN-ZH) and 8.73 (ZH-EN), while BiCodec semantic tokens remain the generation target. *(§3.1.2, §5.6.2, Table 4)*
- **supports:** Explicit speech-text alignment pre-training of a text LLM is a prerequisite for learning speech-to-speech translation from parallel speech data.
  > *Evidence:* Training only on UniST S2ST data without the Phase 1 ASR/TTS/S2TT/MT alignment stage costs 7.18 (EN-ZH) and 10.15 (ZH-EN) Speech-BLEU, while Phase 3 high-quality annealing adds 0.90 and 2.06. *(§5.6.1, Table 4)*
- **supports:** Carrying a fixed-length global speaker-token sequence from source to decoder can preserve voice and emotion across languages in single-stage speech translation without a separate acoustic-generation stage.
  > *Evidence:* UniSS (Q) gets subjective emotion-similarity 4.51 and speaker-similarity 4.42, against 3.56 and 2.94 for SeamlessExpressive and 4.33–4.35 speaker similarity for CosyVoice 2 cascades, from six bilingual raters on ESD/CREMA-D samples. *(§3.1.3, §5.4, Table 2, Appendix C.2)*
- **complicates:** General-purpose speech LLMs can lead on output speech quality and translation in some directions while failing at expressive S2ST, because they do not preserve the source voice and can translate very unevenly across directions.
  > *Evidence:* GPT-4o and Qwen2.5-Omni have the highest ZH-EN UTMOS on CVSS-T (4.18, 4.30) but do not preserve source voice. Qwen2.5-Omni gets only 7.10 EN-ZH Speech-BLEU despite 34.85 Text-BLEU, and both trail on duration compliance (SLC-0.2 ≤ 0.47). *(§5.3, Table 1; Appendix A)*

## Limitations and Open Questions

> [!warning]
> All S2ST training targets are Spark-TTS syntheses of LLM translations. Voice and emotion preservation is therefore distilled from one TTS model rather than learned from natural cross-lingual speech, and evaluation covers only English-Chinese.

- Language coverage is limited to English and Chinese. The authors name multilingual extension as their immediate next step (§6).
- Two tokenizer families (BiCodec and GLM-4) inflate the vocabulary. The authors propose a unified tokenizer as future work (§6).
- Voice preservation is evaluated only subjectively, with six raters and no confidence intervals or significance tests. No objective speaker-similarity score is reported for S2ST outputs.
- Prosody preservation (A.PCP) is slightly below SeamlessExpressive and the cascades. The strong SLC duration scores benefit from speed-token conditioning and from duration-ratio filtering of the training targets.
- Performance mode gives only a 1.07× LM speedup. End-to-end latency, streaming, and detokenizer cost are not measured, so suitability for simultaneous interpretation is untested.
- The paper states that UniST is released and links a demo page, but it does not state whether model code or weights are available.

## Wiki Connections

- [[speech-to-speech|Speech-to-Speech]] — Contributes a single-stage, voice-preserving S2ST system that routes translation through text intermediates inside one LLM pass.
- [[spoken-language-model|Spoken Language Model]] — Adapts a text LLM to consume external speech (Whisper-derived linguistic tokens plus speaker tokens) and emit speech tokens, with ablations on the input tokenizer choice.
- [[autoregressive-codec-tts|Autoregressive Codec TTS]] — Generates BiCodec semantic tokens autoregressively and decodes them with global speaker tokens, the same single-stream decoupled scheme as Spark-TTS.
- [[multilingual-tts|Multilingual TTS]] — Trains one model to synthesize both English and Chinese target speech in either translation direction, carrying the source speaker's timbre and emotion across languages through source speaker tokens (evaluated by speaker- and emotion-similarity MOS).
- [[subjective-evaluation|Subjective Evaluation]] — Runs a webMUSHRA-based MOS study with six bilingual raters on emotion similarity, speaker similarity, and naturalness.
- [[2503.01710|Spark-TTS]] — Supplies the BiCodec speaker/semantic tokenizer and decoder and synthesizes all UniST target speech, and serves as the TTS reference point.
- [[2412.02612|GLM-4-Voice]] — Provides the 12.5 Hz quantized-Whisper speech tokenizer used for source content tokens.
- [[2412.15115|Qwen2.5]] — Provides the 1.5B and 0.5B backbones and the 72B model used to translate UniST text.
- [[2312.05187|Seamless]] — SeamlessM4T-v2 and SeamlessExpressive are the main end-to-end S2ST baselines that UniSS outperforms on translation fidelity and voice preservation.
- [[2412.10117|CosyVoice 2]] — TTS stage of all cascaded baselines.
- [[2507.17527|Seed LiveInterpret 2.0]] — Closed-source expressive S2ST system compared in the subjective study, where it remains slightly ahead on emotion similarity and naturalness.
- [[2410.21276|GPT-4o]] — General-purpose speech LLM baseline that is competitive on translation but does not preserve the source voice.
- [[2503.20215|Qwen2.5-Omni]] — Open speech LLM baseline with high speech quality but unstable EN-ZH speech translation.
