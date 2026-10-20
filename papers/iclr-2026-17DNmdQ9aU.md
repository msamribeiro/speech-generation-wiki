---
id: "iclr-2026-17DNmdQ9aU"
title: "StableToken: A Noise-Robust Semantic Speech Tokenizer for Resilient SpeechLLMs"
authors: ["Yuhan Song", "Linhao Zhang", "Chuhan Wu", "Aiwei Liu", "Wei Jia", "Houfeng Wang", "Zhou Xiao"]
organization: "Peking University, Tencent WeChat AI"
venue: ICLR
venue_type: conference
year: 2026
month: 4
published_date: "2026-04-27"
ingested_date: "2026-10-20"
task: [codec, SCA, TTS]
architecture: [autoregressive-LM, flow-matching]
conditioning: [text-conditioned]
training: [supervised, fine-tuning]
model_size: "320.3M params (tokenizer, N=5 voters); downstream speech LMs use a Qwen2.5-3B backbone"
codec_used: "StableToken (proposed) 25Hz, single codebook of 8192 (13-bit Voting-LFQ), 325 bps"
datasets_train: ["LibriSpeech", "Multilingual LibriSpeech", "The People's Speech", "GigaSpeech", "Yodas", "Hi-Fi TTS", "VCTK", "LibriTTS", "VoiceAssistant-400K", "AISHELL-1", "WenetSpeech", "Common Voice", "Emilia", "in-house data (~150k hours total)"]
datasets_eval: ["FLEURS (noise-augmented)", "LibriSpeech test-clean/test-other", "Seed-TTS eval (EN/ZH)", "CHiME-4", "ESD (noise-augmented)"]
metrics:
  - name: UED
    value: 10.17
    system: proposed
    testset: FLEURS, average over six noise conditions (Table 1, %)
  - name: UED
    value: 26.17
    system: S3Tokenizer (CosyVoice)
    testset: FLEURS, average over six noise conditions (Table 1, %; per §4.1.1)
  - name: UED
    value: 16.48
    system: R-Spin
    testset: FLEURS, average over six noise conditions (Table 1, %)
  - name: UED
    value: 10.96
    system: proposed
    testset: FLEURS + OOD real noise ESC-10 (Table 4, %)
  - name: WER
    value: 3.84
    system: proposed (token resynthesis)
    testset: LibriSpeech test-clean (Table 2, %)
  - name: WER
    value: 7.99
    system: proposed (token resynthesis)
    testset: LibriSpeech test-other (Table 2, %)
  - name: WER
    value: 3.44
    system: proposed (token resynthesis)
    testset: Seed-TTS eval EN (Table 2, %)
  - name: WER
    value: 2.62
    system: proposed (token resynthesis)
    testset: Seed-TTS eval ZH (Table 2, %)
  - name: MOS
    value: 4.09
    system: proposed (token resynthesis)
    testset: LibriSpeech test-clean (Table 2; rating protocol not reported)
  - name: MOS
    value: 4.07
    system: GLM-4-Voice tokenizer (token resynthesis)
    testset: LibriSpeech test-clean (Table 2; rating protocol not reported)
  - name: WER
    value: 4.43
    system: proposed tokenizer + Qwen2.5-3B TTS
    testset: Seed-TTS eval EN (Table 3, %)
  - name: WER
    value: 6.19
    system: GLM-4-Voice tokenizer + Qwen2.5-3B TTS
    testset: Seed-TTS eval EN (Table 3, %)
  - name: WER
    value: 3.02
    system: proposed tokenizer + Qwen2.5-3B TTS
    testset: Seed-TTS eval ZH (Table 3, %)
  - name: MOS
    value: 4.12
    system: proposed tokenizer + Qwen2.5-3B TTS
    testset: Seed-TTS eval EN (Table 3; rating protocol not reported)
  - name: MOS
    value: 4.08
    system: proposed tokenizer + Qwen2.5-3B TTS
    testset: Seed-TTS eval ZH (Table 3; rating protocol not reported)
  - name: WER
    value: 35.90
    system: proposed tokenizer + Qwen2.5-3B ASR
    testset: CHiME-4 test real (Table 3, %)
  - name: WER
    value: 51.08
    system: GLM-4-Voice tokenizer + Qwen2.5-3B ASR
    testset: CHiME-4 test real (Table 3, %)
code_available: true
demo_available: null
url: "https://openreview.net/forum?id=17DNmdQ9aU"
related_concepts: [neural-codec, spoken-language-model, autoregressive-codec-tts, flow-matching]
related_papers: ["2407.05407", "2412.10117", "2412.02612", "2411.17607", "2410.00037", "2504.18425", "2308.16692", "2406.02430", "2502.11946", "2507.16632", "2409.06666", "2505.02625", "2411.00774", "2408.16725", "2506.21875", "2407.05361", "2406.14294", "2308.05725", "2502.06490", "2508.02038", "2104.00355", "2408.02622", "2410.21276", "2010.05646", "2412.15115", "2212.04356", "2025.acl-long.1498"]
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
> **Yuhan Song et al.** (Peking University, Tencent WeChat AI) · [→ Paper](https://openreview.net/forum?id=17DNmdQ9aU) · Demo: ? · Code: ✓
>
> Shows that ASR-supervised semantic speech tokenizers change their token sequences heavily under mild, intelligibility-preserving noise, and fixes this with a multi-branch quantizer that merges parallel binary codes by bit-wise majority vote, trained with a clean-majority/noisy-minority consensus loss; the more stable tokens improve downstream speech LMs on noisy ASR and emotion recognition and on TTS intelligibility.

## Problem

Supervised semantic tokenizers (an ASR encoder with a quantizer inserted mid-stack, trained with an ASR loss) are the input and output vocabulary of many current speech LMs and LLM-based TTS systems. The authors find that these tokenizers are fragile: noise at SNRs where speech remains fully intelligible can change a large fraction of output tokens, measured as Unit Edit Distance (UED) between token sequences of clean and perturbed audio. They attribute this to two causes. First, single-path quantization has no fault tolerance, so a small perturbation near a quantization boundary flips the token. Second, the ASR loss supervises only the final transcript and does not penalize instability of the intermediate tokens. Prior noise-robust tokenizers (NAST, R-Spin) target SSL-style unit discovery rather than the supervised paradigm, and naive fixes (offline ensembles of separately trained tokenizers, or a token-level clean/noisy consistency loss on discrete codes) are either expensive, hard to align, or unstable to train (§1, §5).

## Method

StableToken keeps the supervised semantic tokenizer recipe: an encoder-decoder ASR model initialized from Whisper-large-v3, with a quantizer inserted at the encoder midpoint (layer 16), average pooling to 25 Hz, and a cross-entropy ASR loss on transcripts (§2.1, §3, Appendix C). The quantizer is replaced by a **Voting-LFQ** module, extended from look-up-free quantization (LFQ). The pooled hidden state is fed through n parallel linear projections (n = 5 in the main experiments), each producing a d = 13 dimensional vector that is binarized with a sign function and a straight-through estimator. During training the n binary vectors are averaged per bit, giving soft consensus scores; at inference the averaged scores are passed through sign again, so each bit is decided by majority vote among branches. The resulting 13-bit code maps to one of 8192 token indices (§2.2). Because voting operates per bit rather than per token, the case study shows a token can be recovered even when most branches emit a wrong token, as long as their bit errors fall on different bits (§4.3.3, Table 6).

![Figure 2: The architecture of StableToken. Our model replaces the standard single-path quantizer with a multi-branch Voting-LFQ module. The zoomed-in view shows n parallel branches generating independent binary representations. A bit-wise majority vote then aggregates them into a single token. During our Noise-Aware Consensus Training, a randomly selected minority of branches receive perturbed inputs (H perturbed), while the majority receive clean inputs (H clean). A consensus loss forces the perturbed branches to align with the consensus. The yellow paths are only used during training.](assets/iclr-2026-17DNmdQ9aU/figure-2.png)

**Noise-Aware Consensus Training** supplies the intermediate supervision. Each training sample is perturbed with one randomly chosen augmentation (Gaussian, pink, or brown noise, bit-crush distortion, or real-world noise from AudioCaps, FSD50k and ESC-50 excluding the held-out ESC-10 subset), and both clean and perturbed audio pass through the encoder. A random minority of k < n/2 branches receives the perturbed hidden state while the rest receive the clean one (a 3:2 clean:noisy split for n = 5). A consensus loss penalizes the squared L2 distance of each branch's pre-quantization vector from the mean over all branches, so the clean majority anchors the target and the noisy branches learn to match it (§2.3, Appendix B.3). The total objective is ASR cross-entropy plus weighted consensus, LFQ commitment, and codebook-entropy losses (weights 0.25, 0.25, 1.0) (§2.4, Table 8). The tokenizer is trained on roughly 150k hours of open-source and in-house English and Chinese speech (§3, Table 7). The voting branches add about 0.07M parameters over a single-branch model (320.33M vs. 320.26M) and a negligible FLOPs increase (Table 12).

For audio reconstruction, the authors follow the CosyVoice and GLM-4-Voice recipe: a flow-matching model maps tokens to mel-spectrograms and a HiFi-GAN vocoder produces waveforms (Appendix B.5). For downstream evaluation, each tokenizer is plugged into a speech LM built on Qwen2.5-3B and fine-tuned with task prompts for ASR, speech emotion recognition (SER), and TTS, using the same data and hyperparameters for every tokenizer (§3, Appendix F).

## Key Results

- **Token stability.** Average UED over six noise conditions on noise-augmented FLEURS drops to 10.17%, against 26.17% for the best supervised baseline (S3Tokenizer) and 16.48% for the best SSL robust tokenizer (R-Spin); the held-out OOD real-noise condition is 10.96% (§4.1.1, Table 1, Table 4). StableToken does this with a larger codebook (8192) than most baselines. The parsed Table 1 is garbled for several rows, so per-baseline per-condition numbers other than those quoted in the text are not reproduced here.
- **Reconstruction.** Resynthesized speech via the flow-matching decoder reaches WER 3.84 / 7.99 on LibriSpeech test-clean / test-other and 3.44 / 2.62 on Seed-TTS EN / ZH, the best among the supervised semantic tokenizers compared (GLM-4-Voice tokenizer, S3Tokenizer, CosyVoice 2 tokenizer). MOS is 4.09 on test-clean, essentially tied with the GLM-4-Voice tokenizer (4.07), and slightly below it on test-other (3.83 vs. 3.99) and Seed-TTS EN (4.01 vs. 4.16) (Table 2). Multi-codebook acoustic codecs such as X-Codec at 8 codebooks reach lower WER at much higher bitrate (Table 24). How MOS was collected is not reported.
- **Downstream ASR.** On CHiME-4 the StableToken-based speech LM gets 35.90 / 30.61 WER on test real / simulated, vs. 51.08 / 43.09 for the next-best (GLM-4-Voice tokenizer) system (Table 3). Under OOD real noise at 0 dB on LibriSpeech, WER is 20.34% vs. 29.94% for the baseline; all tokenizers are comparable on clean audio (§4.2.1, Figure 3).
- **Downstream SER.** Accuracy on noise-augmented ESD degrades more slowly with noise than for baseline tokenizers (§4.2.2, Figure 3; numbers shown only in the figure).
- **Downstream TTS.** On Seed-TTS eval the StableToken-based TTS model gets WER 4.43 (EN) and 3.02 (ZH), vs. 6.19 and 5.26 for the GLM-4-Voice tokenizer, with MOS 4.12 (EN, vs. 4.19) and 4.08 (ZH, vs. 3.85) (Table 3).
- **Ablation.** Sequentially removing the consensus loss, the noisy-branch training, and multiple branches raises Gaussian-noise UED from 12.93% to 24.80%, 30.77%, and 34.53%, and LibriSpeech test-other ASR WER (measured on the tokenizer's own decoder) from 4.68% to 4.88%, 5.52%, and 5.85% (Table 4). Going from 3 to 5 voters helps; 7 gives marginal further gains (Table 5).
- **Efficiency.** Latency and throughput match the GLM-4-Voice tokenizer on GPU and CPU, with up to about 14% lower memory (Tables 13 and 14).

The downstream comparison is controlled (same LLM backbone, data, and hyperparameters for every tokenizer), but it uses a small 3B backbone, and the tokenizers differ in frame rate (12.5 Hz vs. 25 Hz) and codebook size as well as in robustness.

## Novelty Assessment

The paper's main contribution is architectural: bit-wise voting across parallel LFQ projections, with soft averaging during training and hard majority at inference, is a new quantizer design for speech tokenizers. Combining it with a consensus loss over a clean majority and a noisy minority of branches gives a way to add explicit intermediate token-stability supervision to an ASR-supervised tokenizer without putting a loss on discrete codes. The surrounding components (Whisper initialization, ASR supervision, mid-encoder quantization, flow-matching plus HiFi-GAN decoding, LLM fine-tuning for downstream tasks) follow the existing CosyVoice/GLM-4-Voice recipe. Noise-invariant speech units are not new in themselves (NAST, R-Spin, and augmentation-invariant units all target them); what is new is applying this to the supervised semantic tokenizer paradigm and the voting mechanism.

## Field Significance

moderate — The paper shows that robustness of the discrete tokenizer is an important and largely unmeasured factor for speech LMs. It provides a cheap, drop-in quantizer change that cuts token instability under noise by more than half relative to existing supervised semantic tokenizers, and it shows that this gain carries through to a controlled downstream speech LM setup for noisy understanding and for TTS intelligibility. The main evidence is the tokenizer's own UED and resynthesis results plus small-scale downstream fine-tunes, not integration into a production-scale speech LM.

## Claims

- **supports:** Discrete semantic speech tokens produced by ASR-supervised single-path quantizers are unstable under mild, intelligibility-preserving acoustic perturbation, and this instability is a distinct failure mode from content loss.
  > *Evidence:* At SNRs chosen so noise does not affect human perception (e.g. Gaussian at 25 dB, real noise at 16 dB), existing supervised semantic tokenizers show average UED of 26% or more on FLEURS, while their clean-audio resynthesis WER stays low. *(§4.1.1, Table 1, Table 2, Appendix E, Table 21)*
- **supports:** Redundant multi-branch quantization with bit-level majority voting, paired with explicit consensus supervision between clean and perturbed views, makes speech tokens more noise-invariant without degrading semantic content.
  > *Evidence:* The full model cuts Gaussian-noise UED from 34.53% (single branch, no noise-aware training) to 12.93% while LibriSpeech test-other ASR WER improves from 5.85% to 4.68%; each component (consensus loss, noisy-branch training, multiple branches) contributes in a sequential ablation. *(§4.3.1, Table 4)*
- **supports:** Gains in tokenizer stability transfer to downstream speech LMs, with the largest benefit in the noisiest conditions and little difference on clean input.
  > *Evidence:* With an identical Qwen2.5-3B speech LM setup, the StableToken system reaches CHiME-4 test-real WER 35.90 vs. 51.08 for the next-best tokenizer, and 20.34% vs. 29.94% WER under 0 dB OOD noise on LibriSpeech, while all tokenizers are comparable on clean audio. *(§4.2.1, Table 3, Figure 3)*
- **refines:** For LLM-based TTS on semantic tokens, token consistency of the tokenizer affects the intelligibility of synthesized speech, not only robustness of understanding.
  > *Evidence:* Training the same TTS model on StableToken tokens gives Seed-TTS WER 4.43 (EN) and 3.02 (ZH) vs. 6.19 and 5.26 with the GLM-4-Voice tokenizer; the MOS difference is mixed (higher on ZH, slightly lower on EN). *(§4.2.3, Table 3)*
- **complicates:** Where the quantizer sits in an ASR-supervised encoder trades token robustness against retention of paralinguistic information.
  > *Evidence:* Quantizing at layer 24 instead of layer 16 lowers UED (13.65% vs. 18.68%) but drops speech emotion recognition accuracy from 67.38% to 59.51%; layer 8 is worse on both robustness and ASR. *(Appendix C.1, Table 20)*

## Limitations and Open Questions

- The robustness evaluation uses synthetic and environmental noise at fixed SNRs. Other nuisance factors that should not change semantic tokens (speaker, channel, reverberation, speaking rate) are not tested.
- MOS appears for reconstruction and downstream TTS, but the paper does not say whether it comes from human listeners or a predictor, or give rater counts or confidence intervals. The reconstruction MOS advantage over the GLM-4-Voice tokenizer is small and not consistent across test sets.
- Downstream speech LMs are separate task-specific fine-tunes of a 3B backbone. The paper does not test whether the gains hold in a full-scale spoken dialogue model or for speech output in end-to-end speech-to-speech interaction.
- Baselines differ in frame rate (12.5 Hz vs. 25 Hz) and codebook size, which affects downstream sequence length and modeling difficulty, separately from token robustness.
- The tokenizer inherits Whisper's 30-second window. Longer audio is chunked; boundary UED is only slightly higher than mid-utterance UED (Table 19), but streaming behavior is not evaluated.
- Training data includes undisclosed in-house data, which limits exact reproduction even though code and model are released.

## Wiki Connections

- [[neural-codec|Neural Audio Codec]] — Proposes a single-codebook 25 Hz semantic speech tokenizer with a new voting LFQ quantizer, evaluated on token stability and on speech reconstruction against SpeechTokenizer, X-Codec, Mimi, and supervised semantic tokenizers.
- [[spoken-language-model|Spoken Language Model]] — Measures how tokenizer stability affects speech LMs that consume speech tokens for ASR and emotion recognition, showing larger gains as noise grows.
- [[autoregressive-codec-tts|Autoregressive Codec TTS]] — Trains an LLM-based TTS model on the proposed tokens and reports lower Seed-TTS WER than with other supervised semantic tokenizers.
- [[flow-matching|Flow Matching]] — Uses a flow-matching token-to-mel decoder (CosyVoice/GLM-4-Voice style) to resynthesize audio from the tokens for reconstruction evaluation.
- [[2407.05407|CosyVoice]] — Its S3Tokenizer is the strongest supervised baseline on noise robustness, and StableToken follows its supervised semantic token plus flow-matching decoding recipe.
- [[2412.10117|CosyVoice 2]] — Its FSQ-based supervised tokenizer is a robustness, reconstruction, and downstream baseline.
- [[2411.17607|Scaling Speech-Text Pre-training with Synthetic Interleaved Data]] — Source of the GLM-4-Voice tokenizer, the main downstream baseline and the efficiency comparison target.
- [[2412.02612|GLM-4-Voice]] — Cited as an example of a speech LM built on a supervised semantic tokenizer; its tokenizer is benchmarked throughout.
- [[2308.16692|SpeechTokenizer]] — Semantic-distilled RVQ tokenizer used as a robustness and reconstruction baseline; much less stable under noise.
- [[2410.00037|Moshi]] — Its Mimi codec is a semantic-distilled baseline, showing the highest UED among the compared tokenizers.
- [[2406.02430|Seed-TTS]] — Provides the Seed-TTS eval sets used for reconstruction and downstream TTS evaluation.
- [[2025.acl-long.1498|Analyzing and Mitigating Inconsistency in Discrete Speech Tokens]] — Related consistency-constraint work on acoustic tokenizers, which the authors argue is a different setting from noise invariance in semantic tokenizers.
