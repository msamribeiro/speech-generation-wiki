---
id: "iclr-2026-h5AiVx0Aiv"
title: "DrVoice: Parallel Speech-Text Voice Conversation Model via Dual-Resolution Speech Representations"
authors: ["Chao-Hong Tan", "Qian Chen", "Wen Wang", "Chong Deng", "Qinglin Zhang", "Luyao Cheng", "Hai Yu", "Xin Zhang", "Xiang Lyu", "Tianyu Zhao", "Chong Zhang", "Yukun Ma", "Yafeng Chen", "Hui Wang", "Jiaqing Liu", "Xiangang Li", "Jieping Ye"]
organization: "Alibaba Group"
venue: ICLR
venue_type: conference
year: 2026
month: 4
published_date: "2026-04-27"
ingested_date: "2026-10-20"
task: [SCA, TTS]
architecture: [autoregressive-LM, flow-matching]
conditioning: [text-conditioned]
training: [supervised, fine-tuning, RLHF]
model_size: "7B (Qwen2.5-7B-Instruct backbone); DrVoice-Small 1.5B (Qwen2.5-1.5B-Instruct) for ablations; Speech Refined Head initialized from a Qwen2.5-0.5B TTS model"
codec_used: "CosyVoice S3Tokenizer 25Hz supervised semantic tokens, grouped to 5Hz at the LLM (k=5); frozen CosyVoice flow-matching detokenizer + HiFi-GAN vocoder"
datasets_train: ["~100K hours audio-text paired data (SRH pretraining)", "CosyVoice-synthesized speech for ~3B text tokens, WER-filtered to ~26K hours speech-to-speech and ~20K hours speech-to-text conversation", "~10K hours English ASR data (Common Voice, MELD, LibriSpeech, SPGISpeech, VoxPopuli)", "in-house instruction data plus MagpiePro and InfGen text instruction sets"]
datasets_eval: ["OpenAudioBench", "VoiceBench", "UltraEval-Audio", "Big Bench Audio"]
metrics:
  - name: UTMOS
    value: 4.29
    system: "DrVoice-7B"
    testset: "UltraEval-Audio collections (Table 3)"
  - name: WER
    value: 8.36
    system: "DrVoice-7B (ASR-WER of generated speech against generated text, Whisper-large-v3)"
    testset: "UltraEval-Audio collections (Table 3)"
code_available: null
demo_available: null
url: "https://openreview.net/forum?id=h5AiVx0Aiv"
related_concepts: [spoken-language-model, speech-to-speech, autoregressive-codec-tts, flow-matching]
related_papers: ["2410.00037", "2504.18425", "2412.15649", "2410.17196", "2411.17607", "2503.20215", "2507.16632", "2501.15368", "2408.16725", "2410.11190", "2410.17799", "2407.05407", "2412.10117", "2505.17589", "2407.04051", "2501.06282", "2311.07919", "2407.10759", "2409.06666", "2501.01957", "2502.11946", "2505.03739", "2411.00774", "2308.16692", "2305.11000", "2010.05646", "2204.02152", "2212.04356", "2209.03143", "2210.13438", "2210.02747", "2408.01800", "2512.23808"]
field_significance:
  level: "moderate"
  type: [architectural-novelty]
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
> **Chao-Hong Tan et al.** (Alibaba Group) · [→ Paper](https://openreview.net/forum?id=h5AiVx0Aiv) · Demo: ? · Code: ?
>
> A parallel joint speech-text speech LM that runs its LLM backbone at only 5Hz for both speech input and speech output, by grouping 25Hz semantic tokens on the way in and ungrouping them through a small autoregressive Speech Refined Head on the way out.

## Problem

End-to-end speech LMs for spoken dialogue split into text-driven designs, where a separate speech decoder consumes the LLM's text or hidden states so text generation never sees the speech tokens (e.g. Qwen2.5-Omni's Thinker-Talker), and joint speech-text designs, where speech tokens are fed back into the LLM either interleaved or in parallel with text. Joint models keep both modalities mutually aware but tend to degrade the backbone's text abilities, and the leading parallel model the authors compare against ([[2504.18425|Kimi-Audio]]) runs its audio at 12.5Hz. That rate is costly in compute and sits far above the roughly 3Hz rate of text tokens, a mismatch the authors argue dilutes semantic information and keeps the LLM from using its full capability.

## Method

DrVoice has three parts: input encoders, a multimodal LLM (MLLM), and a frozen speech detokenizer. On the user side, Whisper-Large-v3 features pass through an adapter that downsamples them and projects them to the LLM width. On the assistant side, speech is represented as 25Hz CosyVoice S3Tokenizer supervised semantic tokens. The authors pick 25Hz over 12.5Hz because GLM-4-Voice ([[2411.17607|Scaling Speech-Text Pre-training]]) reported WER degradation below 25Hz. Generated tokens are turned into a mel spectrogram by the frozen CosyVoice flow-matching model, conditioned on a speaker embedding, and a pre-trained HiFi-GAN vocoder produces the waveform.

The MLLM, initialized from Qwen2.5-7B-Instruct, is a parallel joint speech-text model in the style of [[2410.00037|Moshi]]. At each step the embeddings of the current speech and text tokens are summed into one input. The shorter stream is padded with a silence token. The dual-resolution speech representation (DRSR) is the core idea. On input, k=5 consecutive speech tokens are concatenated and linearly projected into one grouped embedding, so the shared LLM sees 5Hz speech. On output, the shared LLM's final hidden state goes to two heads in parallel. A Text Head predicts the text token. The hidden state is also linearly projected and split into k sub-vectors, and a Speech Refined Head (SRH) uses them to generate the k speech tokens autoregressively at the full 25Hz. This differs from [[2412.15649|SLAM-Omni]]-style grouping, which predicts a group's tokens in parallel from a single projection. Training uses equally weighted cross-entropy losses on the text head and the SRH. The SRH is pre-trained on about 100K hours of paired data. The best variant freezes the rest of the model and trains the SRH with a streaming-TTS objective.

![Figure 1: Overview of DRVOICE. User speech inputs are tokenized, grouped, and encoded by the MLLM for autoregressive text and speech token prediction. The MLLM consists of Shared LLM Layer, a Text Head, and a Speech Refined Head (SRH) for token generation. The generated speech tokens are then converted to speech waveform by the speech detokenizer. Note that SRH generates k speech tokens through k autoregressive forward passes, where k is the grouping factor.](assets/iclr-2026-h5AiVx0Aiv/figure-1.png)

Two training strategies sit around this architecture. CoM-Mixing builds each conversation in seven input/output patterns (Table 1). These are direct speech-to-multimodal, text-only, and three chain-of-modality variants that first write a text transcription and/or text response before generating parallel speech and text. A system prompt selects the pattern at inference. Core-Cocktail handles a learning-rate dilemma with three steps. Stage 1 fine-tunes at a high learning rate. The result is then merged with the base LLM, and Stage 2 fine-tunes the merged model at a low learning rate. Appendix A sets the interpolation weight to α=0, so the merge in practice resets the weights to the base LLM before Stage 2. Appendix C also mentions DPO on ASR-derived speech preference data and on instruction-following data, plus GSPO on maths problems, but reports no separate results for this RL stage. Post-training data is mostly CosyVoice-synthesized speech, WER-filtered to about 26K hours of speech-to-speech and 20K hours of speech-to-text conversation, plus about 10K hours of English ASR data.

## Key Results

On Table 2, DrVoice-7B posts the best overall score on all four benchmarks among the 7-9B open models compared: OpenAudioBench 72.04 vs. 69.08 for Kimi-Audio, VoiceBench 80.17 vs. 76.93, UltraEval-Audio speech-to-speech 56.66 vs. 50.46 for [[2503.20215|Qwen2.5-Omni]], and Big Bench Audio 74.0 vs. 55.8 for MiniCPM-o 2.6. It does this with a 5/5 Hz backbone rate, against 12.5 to 25Hz input for every baseline. It does not win every sub-task. Qwen2.5-Omni is higher on Reasoning QA (63.76 vs. 59.90), and Kimi-Audio is higher on OpenBookQA (83.52 vs. 82.20) and ties on Web Q. (70.20). Most baseline numbers come from the authors running released checkpoints. GLM-4-Voice numbers and most Big Bench Audio numbers are cited from other reports. Open-ended items are scored with the G-Eval LLM judge, so these are task-accuracy and judge scores, not human ratings.

Speech quality (Table 3) is competitive but not leading. UTMOS is 4.29, against 4.28 for Qwen2.5-Omni and 4.53 for [[2507.16632|Step-Audio 2]]-mini. ASR-WER between generated speech and generated text is 8.36, well behind Qwen2.5-Omni's 3.48. The authors attribute this gap to Qwen2.5-Omni feeding text directly to its Talker, while DrVoice passes only hidden states to the SRH.

The ablations run on DrVoice-Small (1.5B), the Llama Questions set, and data subsets. Going from grouping factor 1 to 5 improves speech-to-text accuracy from 55.67 to 63.33 and speech-to-multimodal from 4.00/2.67 to 37.67/28.00 (text/speech), and it roughly halves training GPU hours (Table 7, Figure 2). Factor 7 is worse than 5 on speech output. Replacing the SRH with a parallel projection-and-split head drops speech-to-multimodal text accuracy from 38.33 to 21.67, with text-only performance unchanged (Table 4). Removing SRH pretraining, CoM-Mixing, or the Whisper encoder each costs 10 to 38% relative on speech tasks. Chain-of-modality prompting raises speech-to-multimodal text accuracy from 68.67 to 75.67. For Core-Cocktail, Stage 1 drops VoiceBench from 81.77 (text-only training ceiling) to 70.19, and Stage 2 recovers it to 74.73 (Table 6).

## Novelty Assessment

The architectural contribution is real but builds on existing parts. Token grouping to cut the speech frame rate has precedent in SLAM-Omni. Parallel speech-text summation of embeddings follows Moshi, and an autoregressive per-step speech head echoes Moshi's depth transformer. What DrVoice adds is the specific pairing: grouped 5Hz semantic tokens on input with an autoregressive ungrouping head on output. The ablations show this pairing beats parallel group prediction, and that grouping helps rather than hurts in this setup. Core-Cocktail is a direct application of LM-Cocktail model merging, and at α=0 it reduces to a reset-and-retrain schedule. CoM-Mixing is data augmentation over the chain-of-modality idea from [[2305.11000|SpeechGPT]]. The benchmark gains are large, but they also reflect heavy, carefully curated synthetic instruction data. Appendix C shows how strongly the text data mixture alone moves results, and the paper does not separate data effects from architectural effects at 7B scale. The grouping and SRH ablations use a smaller model, no Whisper encoder, and data subsets.

## Field Significance

Moderate — DrVoice shows that a parallel joint speech-text speech LM can run its backbone at 5Hz on both input and output and still achieve the strongest reported results among the compared ~7B open models on four spoken QA and voice-assistant benchmarks, with about half the training compute of an ungrouped model. It gives concrete evidence that lowering the speech frame rate toward the text rate helps rather than hurts, as long as a fine-grained autoregressive head restores full-rate speech tokens. Its speech-text consistency still trails text-driven designs.

## Claims

- **supports:** Reducing the speech token rate seen by an LLM backbone toward the text token rate can improve both spoken-language understanding and spoken response quality, while cutting training compute.
  > *Evidence:* Grouping 25Hz S3 tokens by k=5 raises Llama Q. speech-to-text accuracy from 55.67 to 63.33 and speech-to-multimodal from 4.00/2.67 to 37.67/28.00 relative to k=1, and it roughly halves GPU hours. k=7 is worse than k=5 on speech output. *(§4.3, §Appendix C, Table 7, Figure 2)*
- **refines:** Grouped low-rate speech representations suit comprehension, but generation needs a fine-grained autoregressive decoder at the original token rate, not parallel prediction of grouped tokens.
  > *Evidence:* Replacing the autoregressive Speech Refined Head with a projection-and-split parallel head drops speech-to-multimodal text accuracy from 38.33 to 21.67 and text-to-multimodal from 59.33 to 45.22, while text-to-text stays at 73 to 73.33. *(§4.3, Table 4)*
- **complicates:** Joint speech-text speech LMs that pass only hidden states to a speech head can match text-driven systems on perceived speech quality yet lag them on consistency between generated speech and generated text.
  > *Evidence:* DrVoice-7B reaches UTMOS 4.29 (vs. 4.28 for Qwen2.5-Omni), but its ASR-WER against its own text output is 8.36 vs. 3.48. The authors attribute this to Qwen2.5-Omni's Talker receiving text directly. *(§4.2, Table 3)*
- **supports:** Adapting a text LLM to joint speech-text generation degrades its text abilities, and model merging with the base LLM followed by low-learning-rate fine-tuning recovers part of the loss.
  > *Evidence:* Stage 1 high-learning-rate training drops VoiceBench from a text-only training ceiling of 81.77 to 70.19. After merging and Stage 2 low-learning-rate training it recovers to 74.73, still short of the ceiling. *(§4.3, §Appendix C, Table 6)*
- **supports:** Generating an intermediate text transcription or text response before parallel speech-text output improves spoken response quality in speech LMs.
  > *Evidence:* The speech-text chain pattern scores 75.67/68.33 (text/speech) on Llama Q., against 68.67/56.00 for direct speech-to-multimodal generation. Removing CoM-Mixing data lowers direct speech-to-multimodal text accuracy to 58.00. *(§4.3, Table 4)*

## Limitations and Open Questions

- The model is half-duplex. The authors list full-duplex interaction (via time-division multiplexing input) as future work, and they do not measure latency or first-packet delay despite arguing that the lower frame rate reduces it.
- Training speech is almost entirely CosyVoice-synthesized and filtered by WER. The assistant voice comes from a frozen detokenizer, and expressive, paralinguistic, or emotion-aware output is neither targeted nor evaluated.
- Evaluation is benchmark accuracy plus LLM-judge scoring (G-Eval) and automatic speech metrics (UTMOS, ASR-WER). There is no human listening test. Most baselines were re-run by the authors, while GLM-4-Voice and most Big Bench Audio numbers are cited from other reports.
- The RL stage (DPO, GSPO) described in Appendix C has no reported isolated effect. It is unclear which reported numbers include it.
- The key architectural ablations are on a 1.5B model without the Whisper encoder, on data subsets and a single benchmark. Whether the k=5 optimum and the SRH gains hold at 7B with the full data is not shown.
- The code and checkpoints are promised for release upon publication. No URL is given in the paper.

## Wiki Connections

- [[spoken-language-model|Spoken Language Model]] — DrVoice adapts Qwen2.5-7B-Instruct to consume Whisper speech features and grouped speech tokens and to emit parallel text and speech, as a joint speech-text speech LM.
- [[speech-to-speech|Speech-to-Speech]] — The model targets end-to-end spoken dialogue and is evaluated on speech-to-speech QA (UltraEval-Audio, Big Bench Audio S2S).
- [[autoregressive-codec-tts|Autoregressive Codec TTS]] — Its Speech Refined Head is a TTS-pretrained autoregressive generator of 25Hz discrete semantic tokens, conditioned on LLM hidden states.
- [[flow-matching|Flow Matching]] — Waveform generation relies on the frozen CosyVoice flow-matching token-to-mel detokenizer.
- [[2504.18425|Kimi-Audio]] — The main parallel joint speech-text baseline, which DrVoice beats on the overall score of all four benchmarks while running at 5Hz instead of 12.5Hz.
- [[2410.00037|Moshi]] — DrVoice adopts Moshi's parallel speech-text design with explicit text streaming as a semantic scaffold.
- [[2412.15649|SLAM-Omni]] — The prior grouping approach, which DrVoice contrasts with its ungrouping Speech Refined Head in place of parallel multi-token prediction.
- [[2503.20215|Qwen2.5-Omni]] — The strongest text-driven baseline, which leads DrVoice on speech-text consistency (ASR-WER) and Reasoning QA.
- [[2411.17607|Scaling Speech-Text Pre-training]] — GLM-4-Voice, an interleaved joint baseline whose frame-rate study motivates DrVoice's choice of 25Hz tokens.
- [[2505.17589|CosyVoice 3]] — Supplies, with its CosyVoice predecessors, the frozen S3Tokenizer, the flow-matching detokenizer, and the synthesized training speech.
- [[2410.17196|VoiceBench]] — One of the four benchmarks, on which DrVoice reports 80.17 overall.
- [[2305.11000|SpeechGPT]] — Origin of the chain-of-modality strategy that DrVoice extends into CoM-Mixing training.
