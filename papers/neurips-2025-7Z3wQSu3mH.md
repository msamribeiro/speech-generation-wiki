---
id: "neurips-2025-7Z3wQSu3mH"
title: "FocalCodec: Low-Bitrate Speech Coding via Focal Modulation Networks"
authors: ["Luca Della Libera", "Francesco Paissan", "Cem Subakan", "Mirco Ravanelli"]
organization: "Concordia University / Mila-Quebec AI Institute"
venue: NeurIPS
venue_type: conference
year: 2025
month: 12
published_date: "2025-12-10"
ingested_date: "2026-08-02"
task: [codec, VC, TTS]
architecture: [VAE, GAN]
conditioning: []
training: [supervised]
model_size: "~142-145M params (encoder + compressor/quantizer/decompressor + decoder, across 50/25/12.5Hz variants)"
codec_used: "none (proposes its own single-codebook BSQ codec)"
datasets_train: ["LibriTTS"]
datasets_eval: ["LibriSpeech test-clean", "Multilingual LibriSpeech (7 languages)", "VoiceBank", "Libri1Mix", "VCTK", "IEMOCAP", "Libri2Mix"]
metrics:
  - name: UTMOS
    value: 4.05
    system: "FocalCodec@50"
    testset: "LibriSpeech test-clean, clean resynthesis (Reference 4.09)"
  - name: SPK-SIM
    value: 98.9
    system: "FocalCodec@50"
    testset: "LibriSpeech test-clean, clean resynthesis"
  - name: WER
    value: 17.63
    system: "FocalCodec@50"
    testset: "LibriSpeech test-clean, downstream ASR probe"
  - name: DNSMOS
    value: 3.16
    system: "FocalCodec@50"
    testset: "VoiceBank, noisy resynthesis"
code_available: true
demo_available: true
url: "https://openreview.net/forum?id=7Z3wQSu3mH"
related_concepts: [neural-codec, self-supervised-speech, voice-conversion, gan-vocoder]
related_papers: []
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
> **Della Libera et al.** (Concordia University / Mila) · [→ Paper](https://openreview.net/forum?id=7Z3wQSu3mH) · Demo: ✓ · Code: ✓
>
> Introduces a single-binary-codebook neural speech codec, built on focal modulation and binary spherical quantization, that compresses speech to 0.16-0.65 kbps while matching or exceeding multi-codebook state-of-the-art codecs on resynthesis, voice conversion, and downstream discriminative and generative tasks.

## Problem

Neural audio codecs discretize speech into tokens for downstream language-model-style processing, but existing designs force a trade-off: acoustic codecs achieve high-fidelity reconstruction but rely on multiple codebooks and retain little semantic content; semantic codecs preserve linguistic information but lose acoustic detail (poor speaker fidelity); and hybrid codecs that try to capture both typically need complex multi-codebook designs, explicit disentanglement losses, distillation, or supervised fine-tuning, all of which add architectural complexity for anything built on top of the tokens. Single-codebook designs are simpler but have struggled to balance compression with reconstruction quality and downstream-task utility, particularly at low bitrates.

## Method

FocalCodec follows a compressor-quantizer-decompressor design layered onto a frozen encoder and a separate decoder. The encoder is the first 6 layers of a pretrained, frozen WavLM-large, chosen because prior work shows these lower layers retain substantial acoustic detail alongside semantic content, avoiding the need for a second encoder or a distillation loss to capture both. The compressor maps these representations into a compact, optionally temporally downsampled latent space using a novel focal downscaling module: a linear/convolutional downscaling step followed by a focal modulation block, which replaces self-attention with an operation that first builds a compact multi-scale (local-to-global) context summary and then uses it to modulate each token, avoiding the quadratic cost of self-attention while still capturing long-range dependencies. Quantization uses binary spherical quantization (BSQ), applied to speech for the first time here: latent vectors are projected onto a unit hypersphere and then independently sign-quantized per dimension, giving an implicit, parameter-free codebook of size 2^L for an L-dimensional latent (e.g., L=13 gives a codebook of 8192), with bounded quantization error and empirically high codebook utilization. The decompressor mirrors the compressor to reconstruct continuous features from the quantized codes, and a separate, encoder-heavy asymmetric decoder (Vocos-style, using inverse STFT rather than transposed convolutions) resynthesizes the waveform.

Training is two-stage and can run in parallel: stage one trains the compressor, quantizer, and decompressor to reconstruct the frozen encoder's continuous features (reconstruction plus entropy loss encouraging uniform code usage); stage two trains the decoder to resynthesize audio from those same continuous encoder features, using adversarial, reconstruction, and feature-matching losses with HiFi-GAN-style multi-period and multi-scale discriminators. Because the decompressor is trained to approximate the original continuous features from discrete codes, the decoder performs well at inference using dequantized features without any additional fine-tuning. Three variants are trained (50Hz, 25Hz, 12.5Hz token rates) by adjusting the compressor's temporal downsampling, all sharing an 8192-entry codebook and yielding bitrates of 0.65, 0.33, and 0.16 kbps respectively.

![FocalCodec architecture. The encoder extracts features containing both acoustic and semantic information. These features are then mapped to a low-dimensional space by the compressor, binary quantized, and projected back by the decompressor. The decoder resynthesizes the waveform from these features.](assets/neurips-2025-7Z3wQSu3mH/figure-1.png)

## Key Results

On clean LibriSpeech resynthesis, FocalCodec@50 achieves the lowest differential WER (dWER, a measure of intelligibility) among all compared codecs, including multi-codebook acoustic codecs (EnCodec, DAC), hybrid codecs (SpeechTokenizer, Mimi, Stable Codec), and the current low-bitrate state-of-the-art BigCodec, while operating at or below their bitrates; it also generalizes well to seven unseen languages in Multilingual LibriSpeech and to noisy speech (VoiceBank, Libri1Mix), again achieving the lowest dWER by a large margin. In one-shot voice conversion on VCTK, using a training-free k-nearest-neighbor feature-replacement technique, FocalCodec achieves the highest speaker similarity of any compared codec, including hybrid codecs like SpeechTokenizer and Mimi that are explicitly designed with separate content/acoustic codebooks for disentanglement. On downstream discriminative tasks (ASR, speaker identification, emotion recognition) FocalCodec@50 is competitive with codecs using higher bitrates or more complex multi-codebook or fine-tuned designs. On downstream generative tasks, FocalCodec@50 substantially outperforms all baselines on speech enhancement and separation intelligibility (dWER), while the lower-bitrate FocalCodec@25/@12.5 variants perform best on a downstream TTS task, since their shorter token sequences ease autoregressive next-token prediction; the authors note their TTS results are not state-of-the-art in absolute terms since the downstream model is trained on only 460 hours of speech. Ablations confirm each design choice: focal modulation outperforms Conformer, AMP, or linear alternatives for the compression block, and BSQ outperforms FSQ and vanilla LFQ for quantization.

## Novelty Assessment

Two components are genuinely new to the speech domain: focal modulation (previously used for vision/video and general sound classification) applied as the compression backbone, and binary spherical quantization (previously used for image/video compression), which the authors identify as its first successful application to speech. The overall compressor-quantizer-decompressor-plus-frozen-SSL-encoder recipe is a relatively simple combination compared to prior hybrid codecs' distillation losses or multi-codebook disentanglement schemes, and the paper's own ablations directly attribute performance gains to these two specific architectural substitutions rather than to scale or training data alone.

## Field Significance

> [!tip] high — FocalCodec is validated extensively (resynthesis across clean/multilingual/noisy conditions, voice conversion, three discriminative and three generative downstream tasks, and a component-level ablation study) against nine strong recent baselines spanning acoustic, semantic, and hybrid codec families, consistently matching or exceeding them at equal or lower bitrate with a single codebook.

Achieving competitive or superior quality with a single binary codebook removes the hierarchical token-flattening complexity that multi-codebook designs impose on downstream generative and discriminative models, which is a practically significant simplification for anyone building on top of discrete speech tokens, not just an incremental quality gain.

## Claims

- **supports:** A single binary codebook, paired with an encoder that retains both acoustic and semantic information, can match or exceed the reconstruction quality of codecs relying on multiple codebooks or higher bitrates, without the token-flattening complexity multi-codebook designs impose on downstream models.
  > *Evidence:* At 0.65kbps with a single 8192-entry binary codebook, FocalCodec@50 achieves the lowest dWER among all compared codecs, including multi-codebook EnCodec, DAC, and SpeechTokenizer and the higher-bitrate state-of-the-art BigCodec, on clean LibriSpeech resynthesis. *(§4.3, Table 2)*
- **supports:** Replacing self-attention with focal modulation, and standard vector quantization with binary spherical quantization, in a speech codec's compression bottleneck can substantially improve reconstruction fidelity and codebook utilization at matched bitrate.
  > *Evidence:* Ablating focal modulation for Conformer, AMP, or linear compression blocks degrades dWER from 2.54 up to 9.37, and replacing BSQ with FSQ or vanilla LFQ increases dWER and lowers normalized codebook entropy (99.7% for BSQ vs. 97.7% for FSQ). *(§4.6, Table 5)*
- **supports:** A speech codec encoder built on frozen self-supervised features, without any explicit disentanglement loss, can still support effective one-shot voice conversion via post-hoc feature-space nearest-neighbor matching, competitive with codecs explicitly designed to separate content and speaker into different codebooks.
  > *Evidence:* FocalCodec achieves the highest speaker similarity (92.2-92.6%) of any compared codec in one-shot voice conversion on VCTK, outperforming SpeechTokenizer and Mimi, both of which use explicit separate content/acoustic codebooks for disentanglement. *(§4.4, Table 3)*
- **complicates:** Optimizing a speech codec's token rate for downstream generative modeling and for downstream discriminative or restoration tasks pull in different directions: lower token rates ease autoregressive generation but can hurt tasks that need to recover fine content or source detail.
  > *Evidence:* The lowest-bitrate FocalCodec variants (25Hz, 12.5Hz) achieve the best downstream TTS results because shorter token sequences ease autoregressive next-token prediction, but the same variants show degraded ASR, speaker identification, and speech-separation performance relative to the 50Hz variant. *(§4.5, Table 4)*

## Limitations and Open Questions

The downstream TTS evaluation is trained on only 460 hours of speech, which the authors note explains why absolute TTS quality is not state-of-the-art; this limits how much the TTS comparison across bitrate variants generalizes to larger-scale training. Speech separation results, while the best among compared codecs, remain "far from practical utility" in absolute terms, which the authors attribute to information loss during quantization being especially costly for source-separation-style tasks. Emotion recognition showed no clear advantage for any codec, FocalCodec included, suggesting current codec designs (including this one) may not strongly preserve the specific cues needed for that task.

## Wiki Connections

- [[neural-codec|Neural Audio Codec]] — proposes a single-binary-codebook codec using a novel focal-modulation compression architecture and binary spherical quantization, achieving 0.16-0.65 kbps.
- [[self-supervised-speech|Self-Supervised Speech]] — builds its encoder directly on frozen, pretrained WavLM-large layers rather than training an encoder from scratch or via distillation.
- [[voice-conversion|Voice Conversion]] — demonstrates one-shot voice conversion via feature-space nearest-neighbor matching, achieving the highest speaker similarity among compared codecs despite no explicit disentanglement loss.
- [[gan-vocoder|GAN Vocoder]] — trains its Vocos-style decoder with HiFi-GAN-style multi-period and multi-scale adversarial discriminators.
