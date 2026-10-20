---
id: "iclr-2026-F7GmbfyVg9"
title: "FlexiVoice: Enabling Flexible Style Control in Zero-Shot TTS with Natural Language Instructions"
authors: ["Dekun Chen", "Xueyao Zhang", "Yuancheng Wang", "Kenan Dai", "Li Ma", "Zhizheng Wu"]
organization: "The Chinese University of Hong Kong, Shenzhen / Huawei Technologies"
venue: ICLR
venue_type: conference
year: 2026
month: 4
published_date: "2026-04-27"
ingested_date: "2026-10-20"
task: [TTS]
architecture: [autoregressive-LM, flow-matching]
conditioning: [text-conditioned, zero-shot, prompt-conditioned, instruction-conditioned, emotion-conditioned]
training: [supervised, RLHF]
model_size: "~3.8B params (Phi-3.5-mini-instruct LLM backbone)"
codec_used: "DualCodec semantic tokens (16,384-entry vocabulary)"
datasets_train: ["Emilia", "FlexiVoice-Instruct", "ParaSpeechCaps", "ChildMandarin", "Debatts", "Emotion set (aggregated emotional corpora)", "KeSpeech", "L2-ARCTIC", "NVSpeech", "ESD (S1 DPO preference pairs)", "NCSSD (S2 GRPO texts)"]
datasets_eval: ["MEAD (English decoupling set)", "CSEMOTIONS (Chinese decoupling set)", "InstructTTSEval"]
metrics:
  - name: WER
    value: 5.23
    system: proposed
    testset: Decoupling set EN, TR-easy (Whisper-Large-V3)
  - name: WER
    value: 6.99
    system: proposed
    testset: Decoupling set EN, TR-hard (Whisper-Large-V3)
  - name: CER
    value: 4.34
    system: proposed
    testset: Decoupling set ZH, TR-easy (Paraformer-zh)
  - name: CER
    value: 7.02
    system: proposed
    testset: Decoupling set ZH, TR-hard (Paraformer-zh)
  - name: MOS
    value: 4.08
    system: proposed
    testset: Decoupling set EN, TO-easy (quality MOS, Table 3)
  - name: MOS
    value: 4.06
    system: proposed
    testset: Decoupling set EN, TR-hard (quality MOS, Table 3)
code_available: null
demo_available: true
url: "https://openreview.net/forum?id=F7GmbfyVg9"
related_concepts: [instruction-conditioned-tts, zero-shot-tts, rlhf-speech, disentanglement, autoregressive-codec-tts, subjective-evaluation]
related_papers: ["2412.10117", "2506.16381", "2508.16332", "2025.acl-long.598", "2402.01912", "2025.acl-long.346", "2506.21619", "2502.07243", "2504.18425", "2505.13000", "2407.05361", "2508.04195", "2508.02038", "2312.15185", "2312.15821", "2409.00750", "2025.acl-long.313", "2210.02747", "2306.00814", "2412.19437"]
field_significance:
  level: "moderate"
  type: [architectural-novelty, dataset-contribution]
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
> **Dekun Chen et al.** (The Chinese University of Hong Kong, Shenzhen / Huawei Technologies) · [→ Paper](https://openreview.net/forum?id=F7GmbfyVg9) · Demo: ✓ · Code: ?
>
> Shows that a staged post-training curriculum (DPO, then multi-objective GRPO with deliberately conflicting inputs, then GRPO with an audio-language-model judge) lets an LLM-based zero-shot TTS system take its speaking style from a natural-language instruction and its timbre from a reference clip, instead of copying style from the reference or inferring it from the text.

## Problem

Zero-shot TTS clones timbre from a short reference, and instruction-based TTS controls style from a text description, but systems that accept both (CosyVoice 2, VoxInstruct) tend to follow neither cleanly. The authors call this the style-timbre-content conflict: under ordinary supervised training the model over-relies on the acoustic prior of the reference (copying its emotion along with its timbre) or infers prosody from the semantics of the target text, and ignores the explicit instruction. Simply adding instruction conditioning does not separate these factors. Existing instruction-speech corpora are also largely templated and drawn from homogeneous sources, which limits coverage of natural, open-ended instructions.

## Method

FlexiVoice follows the CosyVoice-style two-stage design. An autoregressive LLM initialised from Phi-3.5-mini-instruct (about 3.8B parameters), with its vocabulary extended by the 16,384 DualCodec semantic tokens, takes a chat-formatted prompt containing the instruction and target text, with the reference transcript prepended and the reference speech tokens appended. A flow-matching module trained on Emilia converts the generated tokens to mel-spectrograms conditioned on the reference, and Vocos produces the waveform. Only the LLM is trained; the tokenizer, flow-matching decoder, and vocoder stay frozen. Both the instruction and the reference are optional, so the model covers instruction-only, reference-only, and combined use.

Pre-training (FlexiVoice-Base) uses Emilia plus about 9,700 hours of instruction-speech data. The largest component is FlexiVoice-Instruct (4,316 hours), built by prompting DeepSeek-V3 with transcripts and source metadata (video titles and tags for Emilia; character names for voice-acting data from two games) to write natural-language style descriptions, with an LLM-rated information-value filter to discard noisy annotations. ParaSpeechCaps, KeSpeech, L2-ARCTIC, ChildMandarin, Debatts, an aggregated emotion set, and NVSpeech (whose paralinguistic tags extend the vocabulary) fill out the corpus. Data without an instruction receives the default instruction "Speak the following text". The reference speech is not used during pre-training.

Progressive Post-Training (PPT) then runs three stages as a curriculum:

- **S1, multi-modality DPO.** Emotion-template instructions with a neutral reference from the same speaker; preference pairs come from ESD, where the same sentence read with the target emotion is preferred over the same sentence with a different emotion.
- **S2, decoupling GRPO.** About 20,000 inputs built from NCSSD dialogue texts with randomly assigned target emotions (so text and instruction sometimes conflict) and references that are emotional 10% of the time. The advantage sums two group-normalised rewards: an emotion2vec-Large probability for the instructed emotion (penalising style leakage) and a binary CAM++ speaker-verification decision (preserving timbre).
- **S3, instruction GRPO.** About 14,000 open-ended instruction-text inputs (some sampled from pre-training data, most generated by DeepSeek-V3) without references, rewarded by a yes/no consistency judgement from Kimi-Audio-7B-Instruct. A portion of S2 data is mixed in to limit forgetting.

![An overview of FlexiVoice that supports diverse style generation with arbitrary voice timbres. It takes an optional natural language instruction for style and an optional reference speech for timbre. It consists of Pre-Training and Progressive Post-Training (PPT) stages. The PPT process includes three processes, S1: Multi-modality DPO, S2: Decoupling GRPO, S3: Instruction GRPO.](assets/iclr-2026-F7GmbfyVg9/figure-1.png)

Two reward design choices are justified in the appendix. Binary speaker verification is used rather than continuous WavLM speaker similarity because similarity embeddings also encode pitch and emotion, so the similarity reward fights the emotion reward. Kimi-Audio is a cost-driven stand-in for the Gemini-2.5-Pro judge used by InstructTTSEval, with Macro-F1 agreement of 0.60 to 0.65 across task types and languages. Post-training takes about 3.5 days on 8 A800 GPUs.

## Key Results

On the emotion decoupling set (500 English samples from MEAD, 500 Chinese samples from CSEMOTIONS), scored by emotion2vec-Large classification, PPT lifts instruction accuracy (ACC-I) far above both the base model and the baselines (Table 2):

- **Text-only, conflicting text (TO-hard):** ACC-I 89.4% EN / 98.4% ZH, with only 6.6% / 0.8% of outputs following the text's implied emotion. FlexiVoice-Base reaches 39.4% / 66.8%, VoxInstruct 17.8% / 29.0%.
- **Reference plus instruction, conflicting reference (TR-hard):** ACC-I 78.2% EN / 75.8% ZH, versus 61.0% / 47.8% for CosyVoice 2 and 48.8% / 22.4% for FlexiVoice-Base. Leakage of the reference emotion (ACC-R) is 10.6% / 13.2%, against 14.4% / 15.3% for CosyVoice 2.
- **Timbre:** speaker verification falls slightly relative to the base model in English (91.0 vs 99.2 on TR-easy, 95.8 vs 99.4 on TR-hard) and stays at 98 to 99 in Chinese. CosyVoice 2 holds 99.8 to 100 throughout, so its reference adherence is stronger even as its instruction adherence is weaker.

On InstructTTSEval (Gemini judge, Table 4), FlexiVoice averages 79.3 in English and 70.8 in Chinese. That is above every open-source baseline, including MiMo-Audio-7B-Instruct (72.6 / 70.5), close to Gemini-pro TTS in English (80.3), and below Gemini-flash TTS (88.7 / 85.4). Speed and pitch instructions, which were never optimised directly, show Spearman correlations of 0.86 to 0.91 with measured speaking rate and median F0, against at most 0.79 for any baseline (Table 8).

The intelligibility cost is real but modest. English WER rises from 5.01 to 5.99 on TO-easy and Chinese CER from 3.08 to 4.34 on TR-easy relative to FlexiVoice-Base, and CosyVoice 2 remains more intelligible (3.71 WER EN TR-easy, 0.78 CER ZH). The authors attribute this to ASR weakness on expressive speech, citing human quality MOS of 3.62 to 4.08 (Table 3). The listening tests are small: five paired samples per model per task and language, each rated by at least three listeners with speech expertise. Comparative MOS against ground truth is positive for FlexiVoice in most conditions (up to +0.91) and negative for the baselines, though the ± intervals are wide.

The ablation (Table 5, English) supports the curriculum ordering. Starting with S3 alone gives 72.3 on InstructTTSEval, running S3 before S1 and S2 gives 74.8, and training S2 and S3 jointly after S1 gives 75.5 (decoupling average 84.1), against 79.3 (decoupling 88.7) for the S1 → S2 → S3 order. S2 adds most of its gain on the hard decoupling tasks (TO-hard 81.4 → 88.0, TR-hard 72.2 → 80.2), and S3 adds 7.6 points on InstructTTSEval while holding decoupling.

## Novelty Assessment

The architecture is a standard LLM-plus-flow-matching TTS stack and is not the contribution. The novelty lies in the post-training recipe and its targeting. DPO for emotional TTS (following Emo-DPO's preference construction) and multi-objective GRPO (following [[2508.16332|Vevo2]]'s advantage formulation) are both prior techniques. What is new is sequencing them as a curriculum and building conflicting-input training scenarios so the reward directly penalises style leakage from the reference and the text. The ablation is unusually thorough for this kind of claim, testing orderings and joint training rather than only adding stages. The paper also shows why the reward choice matters: binary verification works where continuous speaker similarity fails. FlexiVoice-Instruct is a substantial secondary contribution, though its descriptions are inferred from metadata and transcripts by an LLM, not from the audio itself.

Two caveats limit the comparisons. The decoupling benchmark is author-built, emotion-only, and scored by the same emotion2vec model family used as the S2 reward, so it partly measures how well the policy optimised its own training signal. And CosyVoice 2, the only baseline that accepts both instruction and reference, keeps a clear edge in timbre preservation and intelligibility.

## Field Significance

Moderate — The paper provides controlled evidence that, in an LLM-based TTS system, conflict between instruction-specified style and reference or text cues is a post-training problem that targeted RL rewards can largely resolve, rather than something to fix through architectural factorisation. Its ablation of stage order and joint versus sequential optimisation is a reusable data point for anyone designing multi-reward preference alignment for controllable speech generation.

## Claims

- **supports:** Preference and reinforcement-based post-training can substantially improve instruction adherence in LLM-based TTS beyond what supervised pre-training on instruction-speech data achieves.
  > *Evidence:* Progressive Post-Training raises emotion instruction accuracy on text-only input from 72.4% to 97.4% (EN) and InstructTTSEval average from 66.4 to 79.3 (EN) and 58.4 to 70.8 (ZH) over the same pre-trained model. *(§5.2, Table 2; §5.3, Table 4)*
- **supports:** Training on deliberately conflicting inputs, with a reward that penalises style leakage, can separate instruction-specified style from reference timbre and from the emotional content of the target text.
  > *Evidence:* Decoupling GRPO with an emotion-recognition reward and speaker-verification reward on NCSSD texts with randomly mismatched emotions raises TO-hard accuracy from 81.4% to 88.0% and TR-hard from 72.2% to 80.2% over DPO alone; the final model leaks the text emotion in only 6.6% (EN) and 0.8% (ZH) of TO-hard outputs. *(§3.2.2; §5.4, Table 5; Table 2)*
- **complicates:** Continuous speaker-embedding similarity is a poor timbre reward for expressive or emotional speech generation because the embeddings also encode prosodic and emotional cues.
  > *Evidence:* Using WavLM speaker similarity alongside an emotion reward in multi-objective GRPO leaves TR accuracy largely unchanged or reduced, while a binary CAM++ verification reward yields clear gains; speaker verification also drops slightly after post-training (EN TR-easy 99.2 to 91.0) as the model alters pitch and energy to follow the instruction. *(§Appendix A.6, Figure 4; §5.2, Table 2)*
- **refines:** When multiple RL objectives for controllable TTS conflict, staged optimisation can outperform joint optimisation, and stage order matters.
  > *Evidence:* S1 → S2 → S3 reaches 88.7 decoupling average and 79.3 InstructTTSEval average, versus 84.1 / 75.5 for joint S2+S3 training and 84.4 / 74.8 for S3 → S1 → S2. *(§5.4, Table 5)*
- **complicates:** Gains in expressive style control from post-training can come with a small loss in ASR-measured intelligibility.
  > *Evidence:* After post-training, English WER on TO-easy rises from 5.01 to 5.99 and Chinese CER on TO-easy from 5.01 to 7.59 relative to the base model, while human quality MOS stays similar or improves; the authors attribute the gap to ASR weakness on expressive speech. *(§5.2, Table 3)*

## Limitations and Open Questions

> [!warning]
> The disentanglement evaluation covers five emotion categories only and is scored by emotion2vec-Large, the same model family used as the S2 training reward, so decoupling gains may partly reflect reward fitting rather than perceptually verified separation; the human listening tests use only five samples per model per condition.

- The S3 reward model agrees with the Gemini benchmark judge only moderately (Macro-F1 about 0.6), and binary yes/no rewards are sparse; how far LLM-judge rewards can push open-ended style control without reward hacking is not examined.
- Timbre preservation trails CosyVoice 2 under conflicting references in English, and the paper treats the drop as an inherent trade-off of changing prosody rather than measuring it with human speaker-similarity judgements.
- FlexiVoice-Instruct descriptions are written by an LLM from metadata and transcripts without listening to the audio, so labels can mismatch the actual delivery despite the information-value filter.
- S3 discards the reference to avoid conflicts such as gender, so complex instructions combined with a cloned voice are not trained or evaluated.
- Code, checkpoints, and the dataset are promised but not confirmed as released in the paper.
- Evaluation covers English and Chinese only.

## Wiki Connections

- [[instruction-conditioned-tts|Instruction-Conditioned TTS]] — Shows that post-training with targeted rewards, not just more instruction data, closes much of the instruction-following gap for open-source instruction TTS.
- [[zero-shot-tts|Zero-Shot TTS]] — Adds instruction-driven style control on top of reference-based voice cloning and measures how much reference emotion leaks into the output.
- [[rlhf-speech|RLHF Speech]] — Contributes a staged DPO-then-GRPO curriculum with ablations of stage order, joint training, and timbre reward choice.
- [[disentanglement|Disentanglement]] — Separates style from timbre and text content through a multi-objective RL reward on conflicting inputs, with ablation evidence for the decoupling stage.
- [[autoregressive-codec-tts|Autoregressive Codec TTS]] — Uses an LLM backbone generating DualCodec semantic tokens, the model whose post-training is studied.
- [[subjective-evaluation|Subjective Evaluation]] — Reports Q-MOS and CMOS from compensated listeners with speech and audio expertise, comparing FlexiVoice with its base model and two baselines (§4, Table 3; App. A.9).
- [[2412.10117|CosyVoice 2]] — Architectural template and the main baseline accepting both instruction and reference; it keeps stronger timbre and intelligibility but weaker instruction adherence.
- [[2506.16381|InstructTTSEval]] — The complex instruction-following benchmark and Gemini-judge protocol used for evaluation.
- [[2508.16332|Vevo2]] — Source of the multi-objective GRPO advantage formulation adapted for the decoupling stage.
- [[2025.acl-long.598|INTP]] — Prior preference-alignment work for zero-shot TTS intelligibility from the same group, cited as RL precedent.
- [[2402.01912|Parler-TTS]] — Instruction TTS baseline without timbre control, outperformed on both the decoupling set and InstructTTSEval.
- [[2504.18425|Kimi-Audio]] — Serves as the audio-language-model reward judge for S3 instruction GRPO.
- [[2505.13000|DualCodec]] — Provides the semantic speech tokens the LLM generates.
- [[2407.05361|Emilia]] — Primary pre-training corpus and a source of FlexiVoice-Instruct annotations.
