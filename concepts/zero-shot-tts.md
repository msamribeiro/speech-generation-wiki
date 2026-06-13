---
slug: zero-shot-tts
title: Zero-Shot TTS
aliases: [voice cloning, any-speaker TTS, speaker generalization, few-shot TTS]
related_concepts: [speaker-adaptation, voice-conversion, neural-codec, instruction-conditioned-tts, autoregressive-codec-tts, flow-matching]
last_updated: 2026-06-13
status: dominant
---

## Executive Summary

> [!abstract]
> Zero-shot TTS is the ability to synthesize speech in the voice of an unseen speaker given only a short reference audio clip at inference time, without any fine-tuning — enabling personalized voice synthesis for any speaker at inference time. It is the dominant paradigm in modern TTS research, having largely displaced speaker-ID-conditioned models, and as of early 2026 the frontier is characterized by large-scale open-source systems (Fish Audio S2, Qwen3-TTS, OmniVoice) that are rapidly narrowing the gap to closed-source SOTA on intelligibility, speaker similarity, and multilingual breadth. The central research challenge is balancing speaker similarity, intelligibility, and naturalness — objectives that remain in tension and are governed by training methodology, codec design, and inference-time CFG strategy.

## Current Status

dominant — Zero-shot TTS is the dominant paradigm in TTS research as of early 2026, with open-source systems achieving best-reported WER below 1% and spanning 600+ languages. The gap between open-source and closed-source SOTA has materially narrowed.

## Why This Matters

Zero-shot TTS eliminates the need for per-speaker data collection and training, making personalized voice synthesis accessible for any speaker at inference time. This unlocks applications including personalized voice assistants, audiobook production, dubbing and localization, accessibility tools, and — with appropriate safeguards — voice conversion for entertainment. It is the dominant paradigm in modern TTS research, having largely displaced speaker-ID-conditioned models.

The challenge is balancing three objectives: speaker similarity to the reference (measured by SPK-SIM), intelligibility of the output (measured by WER/CER), and naturalness (MOS). These objectives are in tension, and the CFG trade-off is a key current research area.

## Core Idea

Zero-shot TTS is the ability to synthesize speech in the voice of an unseen speaker — one not seen during training — given only a short reference audio clip (typically 3–10 seconds) at inference time, without any fine-tuning. The system must generalize the speaker's timbre, speaking style, and vocal quality from the reference clip and apply it to arbitrary target text. This is distinct from speaker-conditioned TTS (which uses a fixed speaker embedding from a closed training set) and from fine-tuning approaches (which update model weights for each new speaker).

Zero-shot TTS systems typically learn a disentangled representation space where speaker characteristics and content are separated, allowing the speaker conditioning from one utterance to be combined with text content to generate new speech.

## Methods and Variants

**Flow-matching non-autoregressive (F5-TTS paradigm).** Pure flow matching over mel-spectrograms with text and reference audio as joint conditioning. Inference is non-autoregressive in the temporal sense (all frames generated in parallel via ODE integration). F5-TTS ([[2025.acl-long.313]]) and E2-TTS are the main representatives. F5-TTS introduces ConvNeXt V2 text refinement to resolve E2-TTS's 7% unrecoverable alignment failure rate (Mandarin), and Sway Sampling (biasing ODE steps toward early timesteps) to further improve robustness at inference. Classifier-free guidance (CFG) is critical and [[2509.19668]] shows that the CFG strategy significantly affects the SIM/WER trade-off.

**LLM + flow matching (CosyVoice paradigm).** A pre-trained LLM (Qwen2.5-0.5B in CosyVoice 2) autoregressively generates semantic tokens from text, which then condition a small (71M) flow-matching model for acoustic synthesis. This architecture achieves stronger text adherence by leveraging the LLM's language modeling capabilities, making CFG less critical for text conditioning.

**Autoregressive codec LM (VALL-E paradigm).** An AR LM generates codec RVQ-1 tokens from text + speaker embedding; a NAR model fills RVQ-rest. Speaker conditioning comes from a speaker verification model's embedding. Established by [[2301.02111]] (VALL-E, 2023) and extended with low-frame-rate tokens by FlexiCodec-TTS ([[2510.00981]]).

**One-step flow matching with learned prior (OZSpeech paradigm).** [[2025.acl-long.1043]] reformulates OT-CFM to start from a learned prior distribution (from a prior codes generator) rather than Gaussian noise, enabling single-step inference (NFE=1). Trained on only 500 hours, it achieves WER 0.05% on LibriSpeech test-clean — trading UTMOS for extreme intelligibility. Also demonstrates exceptional noise robustness: VALL-E degrades to 93% WER at 0 dB SNR while OZSpeech holds at 0.05–0.06%.

**Zero-shot VC (content-preserving).** Rather than text input, source speech is used. [[2507.14534]] (Conan) uses Emformer-distilled HuBERT content + CVQ style encoder for zero-shot VC at low latency.

## Major Claims

Claims are generalised propositions aggregated from paper evidence. The full claim registry with supporting paper lists is in `wiki/concepts/_evidence/zero-shot-tts.yaml`.

### Strongly Supported

- Zero-shot TTS quality scales with training data size and model parameters, with systems trained on millions of hours and billions of parameters achieving best-in-class intelligibility and speaker similarity.
  Supporting: [[2603.08823]], [[2601.15621]], [[2604.00688]], [[2406.02430]]

- Post-training alignment (RLHF, DPO, GRPO) reliably improves zero-shot TTS intelligibility, speaker similarity, and expressiveness without degrading naturalness.
  Supporting: [[2406.02430]], [[2512.14291]], [[2025.acl-long.598]], [[2603.08823]]

- Classifier-free guidance strategy — which conditioning dimensions to apply CFG to and at which timesteps — significantly governs the speaker similarity / intelligibility trade-off in flow-matching zero-shot TTS.
  Supporting: [[2509.19668]], [[2025.acl-long.313]], [[2412.10117]]

### Emerging

- Cross-lingual zero-shot TTS (reference speaker in language A, synthesis in language B) can be substantially improved by preference alignment on cross-lingual preference pairs, even with small alignment datasets.
  Supporting: [[2025.acl-long.598]]

- Zero-shot TTS can be extended to 600+ languages with a single model without per-language adaptation, via full-codebook random masking and LLM initialization in a NAR framework.
  Supporting: [[2604.00688]]

- Zero-shot TTS can be reliably improved for pathological speakers (dysarthria) via teacher-student knowledge anchoring and curriculum learning, without modifying the zero-shot architecture.
  Supporting: [[interspeech-2025-0596]]

- Zero-shot accented TTS benefits from accent label quality more than raw data quantity; geolocation-based filtering consistently outperforms raw self-reported labels.
  Supporting: [[2508.07426]]

- Zero-shot cross-lingual voice conversion with noise robustness is achievable by combining ASR bottleneck features (for noise-robust phonetic content) with SSL features (for paralinguistic expressiveness), regularised through random feature erasure.
  Supporting: [[2508.04996]]

- Flow-matching-based zero-shot TTS models can be efficiently adapted to new languages with fewer than 15 hours of single-speaker data via PEFT, preserving zero-shot capability that full fine-tuning destroys.
  Supporting: [[interspeech-2025-1344]]

- Commercial zero-shot TTS systems approach human-level deception rates on expressive speech, while leading open-source systems remain substantially below this threshold despite reporting competitive CMOS/MUSHRA scores.
  Supporting: [[interspeech-2025-2765]]

- Embedding watermark detection directly into codec encoder training is a viable access-control mechanism for zero-shot TTS systems, creating a structural barrier to voice cloning even in open-source deployments.
  Supporting: [[interspeech-2025-1993]]

### Contested

> [!warning]
> There is a fundamental tension between intelligibility (WER) and naturalness (MOS/UTMOS) in zero-shot TTS: systems optimized for near-zero WER (OZSpeech, Flamed-TTS, DiFlow-TTS on LibriSpeech) achieve this at the cost of UTMOS, while systems achieving high UTMOS scores show higher WER. Whether this trade-off is fundamental to the learned-prior approach or addressable with better codec choice remains unclear.
> Supporting: [[2025.acl-long.1043]], [[2510.02848]], [[2509.09631]] / Contradicting: [[2603.08823]] (high WER and SIM simultaneously at scale)

## Relationship to Other Concepts

### Extends or Builds On
- [[neural-codec]] — Zero-shot TTS systems based on AR codec LMs or flow matching depend on neural audio codecs for both tokenization and waveform reconstruction; codec design (frame rate, codebook size, RVQ layers) directly affects zero-shot quality and speaker generalization.
- [[disentanglement]] — Zero-shot TTS requires learning a disentangled representation space where speaker characteristics and content are separated; disentanglement quality is a fundamental bottleneck for speaker similarity.

### Competes With
- [[speaker-adaptation]] — Speaker-adapted TTS (fine-tuning on target speaker data) achieves higher speaker similarity at the cost of requiring enrollment data and training time. Zero-shot TTS trades some speaker fidelity for generalization without enrollment. The gap between the two is narrowing; recent zero-shot systems are approaching fine-tuned quality on many speakers.

### Commonly Paired With
- [[flow-matching]] — Flow matching is the dominant non-autoregressive method for zero-shot TTS, used in nearly all state-of-the-art zero-shot systems as either the primary model or the acoustic decoder.
- [[autoregressive-codec-tts]] — AR codec LMs are the other dominant paradigm for zero-shot TTS; the AR LM's in-context learning from a short reference clip is the primary mechanism for speaker generalization in this family.
- [[voice-conversion]] — Zero-shot TTS and zero-shot VC share the core challenge of generalizing to unseen speakers at inference; many zero-shot TTS systems also support VC as a secondary capability via content-preserving synthesis from source audio rather than text.

## Representative Papers

### Foundational
- [[2301.02111]] — VALL-E established the zero-shot TTS paradigm via conditional codec language modeling, demonstrating 3-second enrollment without per-speaker fine-tuning and achieving SPK-SIM 0.580 vs. prior SOTA 0.337 on LibriSpeech.
- [[2304.09116]] — NaturalSpeech 2 established latent diffusion over codec representations as a viable zero-shot TTS paradigm, introducing in-context learning via speech prompting and avoiding AR repetition/skipping errors; first work demonstrating singing synthesis in zero-shot TTS.
- [[2406.02430]] — Seed-TTS achieved human-parity synthesis (CMOS -0.07) at foundation-model scale, introduced RL post-training for robustness, and defined Seed-TTS-Eval as the standard benchmark for subsequent work.

### Influential
- [[2406.05370]] — VALL-E 2 introduced grouped codec code prediction and repetition-aware sampling to resolve AR instability, and was the first to claim human parity on both LibriSpeech and VCTK benchmarks.
- [[2409.00750]] — MaskGCT demonstrated that NAR masked generative transformers can achieve human-level speaker similarity in zero-shot TTS without text-speech alignment, and substantially outperform AR approaches on hard-text robustness.
- [[2025.acl-long.313]] — F5-TTS established pure non-autoregressive flow matching as a competitive zero-shot TTS paradigm, resolving E2-TTS alignment failures and introducing Sway Sampling for inference-time robustness.
- [[2407.05407]] — CosyVoice demonstrated that supervised semantic tokens substantially improve zero-shot WER and speaker similarity, establishing LLM+FM as a dominant hybrid paradigm.
- [[2509.19668]] — First systematic study of CFG strategies for zero-shot flow-matching TTS, revealing that language-specific behavior and image-domain CFG improvements do not transfer to speech.
- [[2025.acl-long.598]] — DPO alignment across five zero-shot TTS systems demonstrated 31–52% relative WER reduction on challenging domains (code-switching, cross-lingual) while preserving speaker similarity.

### Recent Highlights
- [[2603.08823]] — Fish Audio S2 achieved best open-source WER on Seed-TTS-Eval (0.54% zh / 0.99% en) with 81.88% win rate against GPT-4o-audio on EmergentTTS-Eval.
- [[2601.15621]] — Qwen3-TTS demonstrated sub-100 ms streaming with best open-source SPK-SIM across 10 languages and DPO+GRPO alignment at scale.
- [[2604.00688]] — OmniVoice extended zero-shot TTS to 600+ languages with a single NAR model, achieving CMOS +0.44 (best in corpus) and WER ≤5% for 82/102 languages on FLEURS.
- [[2603.29339]] — LongCat-AudioDiT achieved SOTA speaker similarity for diffusion NAR systems on Seed-ZH (SIM 0.818) by shifting flow matching to waveform VAE latent space.

### Cautionary / Negative Evidence
- [[2025.acl-long.1043]] — OZSpeech demonstrates an extreme intelligibility/naturalness trade-off point: near-zero WER at the cost of UTMOS, raising questions about whether the intelligibility/naturalness frontier is fundamental to the learned-prior approach.
- [[interspeech-2025-0596]] — Highlights that standard zero-shot TTS systems fail significantly on pathological speakers (dysarthric), showing that generalization claims do not extend to all speaker populations without architectural modification.

## Open Questions

- Can inference-time CFG improvements [[2509.19668]] fully close the gap to closed-source SOTA, or is training-time methodology the bottleneck?
- Why does the CFG strategy effectiveness vary by language (English vs. Mandarin for F5-TTS)? Is it the text encoder architecture, training data, or something else?
- What is the right trade-off between SIM and WER for practical applications? Current evaluation focuses on maximizing both independently. OZSpeech [[2025.acl-long.1043]] demonstrates an extreme point (near-zero WER, lower UTMOS) in this trade-off space.
- Cross-lingual zero-shot TTS (reference speaker speaks language A, synthesis in language B) remains an open challenge, though [[2025.acl-long.598]] shows DPO alignment on cross-lingual preference pairs (500 zh2en + 500 en2zh samples) reduces cross-lingual WER by 50%+ for multiple systems.
- [[2025.acl-long.1043]] shows near-perfect WER on LibriSpeech at the cost of UTMOS; is the intelligibility/naturalness trade-off fundamental to the learned-prior approach, or addressable with better codec choice (FACodec is acknowledged as limiting)?
- The HFR evaluation [[interspeech-2025-2765]] reveals a 20+ point gap between commercial and open-source systems on expressive speech; which component (architecture, data, training procedure) is the primary bottleneck?
- PEFT-TTS [[interspeech-2025-1344]] fine-tunes F5-TTS on a single female Korean speaker; how does PEFT-based cross-lingual adaptation perform when the target language is phonologically more distant from the pre-training language, or when multi-speaker target data is available?
- MKL-VC [[interspeech-2025-2043]] achieves near-FACodec quality with no training from 5 seconds of reference; can training-free OT approaches close the remaining gap on naturalness and be evaluated in streaming scenarios?

## Trend Summary

2023: VALL-E [[2301.02111]] established the codec LM paradigm with 60K hours of LibriLight training and 3-second enrollment at inference. 2024: F5-TTS ([[2025.acl-long.313]]) and E2-TTS established pure flow matching as competitive; CosyVoice [[2407.05407]] introduced supervised semantic token conditioning; Seed-TTS [[2406.02430]] reached human-parity synthesis (CMOS -0.07) with RL post-training; CosyVoice 2 [[2412.10117]] added streaming and FSQ tokenizer. 2025: StyleTTS-ZS [[2025.naacl-long.242]] demonstrated codec-free distilled diffusion TTS with 10-20× faster inference; Flamed-TTS [[2510.02848]] and DiFlow-TTS [[2509.09631]] pushed WER to 0.04-0.05% on LibriSpeech test-clean; Marco-Voice [[2508.02038]] and EmoSteer-TTS [[2508.03543]] extended zero-shot to emotional expressiveness; PALLE [[2504.10352]] introduced PAR for 10x faster inference than AR; IndexTTS2 [[2506.21619]] solved AR duration control; M3-TTS [[2512.04720]] demonstrated MMDiT-style joint attention for alignment-free NAR TTS; DisCo-Speech [[2512.13251]] enabled independent prosody/timbre control via disentangled codec; DiSTAR [[2510.12210]] coupled AR drafting with discrete masked diffusion; DiTAR [[2502.03930]] demonstrated patch-based continuous AR+diffusion. 2026: Fish Audio S2 [[2603.08823]] and Qwen3-TTS [[2601.15621]] achieved best open-source WER while adding streaming and multilingual breadth; OmniVoice [[2604.00688]] scaled NAR TTS to 600+ languages; LongCat-AudioDiT [[2603.29339]] achieved SOTA speaker similarity for diffusion NAR on Seed-ZH (SIM 0.818); Vevo2 [[2508.16332]] extended zero-shot TTS to unified singing synthesis. The gap between open-source and closed-source SOTA has materially narrowed.

## All Papers

| ID | Title | Venue | Year | Key use of this concept |
|----|-------|-------|------|------------------------|
| [[2301.02111]] | Neural Codec Language Models are Zero-Shot Text to Speech Synthesizers (VALL-E) | arXiv | 2023 | Foundational zero-shot TTS paper; demonstrates 3-second enrollment without per-speaker fine-tuning via conditional codec LM pretraining on 60K hours; SPK-SIM 0.580 vs. prior SOTA 0.337 on LibriSpeech |
| [[2509.19668]] | Selective Classifier-free Guidance for Zero-shot Text-to-speech | arXiv | 2025 | Evaluates CFG strategies for zero-shot TTS using F5-TTS and CosyVoice 2; proposes timestep-selective CFG (def text) that improves SIM while limiting WER degradation |
| [[2507.14534]] | Conan: A Chunkwise Online Network for Zero-Shot Adaptive Voice Conversion | arXiv (ASRU 2025) | 2025 | Zero-shot VC setting; Conan transfers unseen speaker timbre and style at inference from a reference clip in a streaming context |
| [[2025.acl-long.1498]] | Analyzing and Mitigating Inconsistency in Discrete Speech Tokens for Neural Codec Language Models | ACL | 2025 | Shows that improving codec consistency reduces VALL-E zero-shot TTS WER from 8.51% to 3.51% and increases SPK-SIM from 55.90% to 60.97% on LibriTTS test-clean |
| [[2025.acl-long.313]] | F5-TTS: A Fairytaler that Fakes Fluent and Faithful Speech with Flow Matching | ACL | 2025 | F5-TTS open-weight flow-matching zero-shot TTS; ConvNeXt text refinement resolves E2-TTS alignment failures; Sway Sampling improves robustness at inference; WER 1.83% / SIM-o 0.67 on Seed-TTS test-en |
| [[2025.acl-long.1043]] | OZSpeech: One-step Zero-shot Speech Synthesis with Learned-Prior-Conditioned Flow Matching | ACL | 2025 | Single-step (NFE=1) zero-shot TTS via learned-prior OT-CFM; achieves WER 0.05% on LibriSpeech test-clean (best in corpus); exceptional noise robustness (WER unchanged from 0 to 0 dB SNR vs. 93% WER for VALL-E) |
| [[2025.acl-long.346]] | ControlSpeech: Towards Simultaneous and Independent Zero-shot Speaker Cloning and Zero-shot Language Style Control | ACL | 2025 | First system to jointly perform zero-shot timbre cloning and zero-shot language-style control via disentangled FACodec representations; achieves Spk-sv 0.89 and WER 2.9% while outperforming style-only baselines on speed/volume/emotion accuracy |
| [[2025.acl-long.598]] | Advancing Zero-shot TTS Intelligibility across Diverse Domains via Preference Alignment | ACL | 2025 | DPO alignment of five zero-shot TTS systems on INTP dataset; reduces intelligibility failures in articulatory, code-switching, and cross-lingual scenarios by 31–52% relative WER reduction while preserving speaker similarity |
| [[2406.02430]] | Seed-TTS: A Family of High-Quality Versatile Speech Generation Models | arXiv | 2024 | Foundation-model-scale zero-shot TTS achieving CMOS -0.07 vs. human; introduces self-distillation for VC and RL post-training for robustness; Seed-TTS-Eval becomes the standard benchmark for subsequent work |
| [[2407.05407]] | CosyVoice: A Scalable Multilingual Zero-shot TTS based on Supervised Semantic Tokens | arXiv | 2024 | Demonstrates that supervised (ASR-derived) semantic tokens significantly improve zero-shot WER and speaker similarity over unsupervised alternatives |
| [[2412.10117]] | CosyVoice 2: Scalable Streaming Speech Synthesis with Large Language Models | arXiv | 2024 | Extends CosyVoice to streaming zero-shot TTS with FSQ tokenizer, LLM backbone, and chunk-aware causal flow matching; surpasses human naturalness on LibriSpeech |
| [[2025.naacl-long.242]] | StyleTTS-ZS: Efficient High-Quality Zero-Shot TTS Synthesis with Distilled Time-Varying Style Diffusion | NAACL | 2025 | Codec-free distilled diffusion zero-shot TTS: fixed-length RVQ prosody latent + 10-step distillation, achieves CMOS -0.032 vs. GT with 10-20x faster inference than prior diffusion SOTA |
| [[2506.21619]] | IndexTTS2: A Breakthrough in Emotionally Expressive and Duration-Controlled AR Zero-Shot TTS | arXiv | 2025 | Positional embedding tying for AR duration control; GRL emotion–speaker disentanglement; WER 1.008% zh / 1.521% en on SeedTTS benchmarks |
| [[2510.12210]] | DiSTAR: Diffusion over a Scalable Token Autoregressive Representation | arXiv | 2025 | AR+discrete-masked-diffusion hybrid; WER 1.32% on SeedTTS test-en at 0.3B; test-time bitrate control via RVQ layer pruning |
| [[2502.03930]] | DiTAR: Diffusion Transformer Autoregressive Modeling for Speech Generation | arXiv | 2025 | Patch-based continuous AR+bidirectional-diffusion; WER 1.685% on Seed-EN at 1B; 3-43x lower FLOPs than competing NAR diffusion systems |
| [[2512.14291]] | GLM-TTS Technical Report | arXiv | 2025 | Production-level zero-shot voice cloning with multi-reward GRPO RL; CER 0.89% zh on Seed-TTS-eval after RL; WER 2.23% en |
| [[2504.10352]] | Pseudo-Autoregressive Neural Codec Language Models for Efficient Zero-Shot TTS | arXiv | 2025 | PAR paradigm for span-level causal inference; 10x RTF speedup over AR; best WER among LibriTTS-trained models (WER-H 2.83) |
| [[2508.02038]] | Marco-Voice Technical Report | arXiv | 2025 | Combines zero-shot voice cloning with emotion control via rotational emotion embeddings and cross-orthogonal disentanglement; SPK-SIM 0.8275 vs. CosyVoice2 0.605 |
| [[2508.03543]] | EmoSteer-TTS: Fine-Grained and Training-Free Emotion-Controllable TTS via Activation Steering | arXiv | 2025 | Training-free emotion control for zero-shot TTS by activation steering in DiT layers; enables emotion conversion, interpolation, erasure without retraining |
| [[2510.02848]] | Flamed-TTS: Flow Matching Attention-Free Models for Efficient Generating and Dynamic Pacing Zero-shot TTS | arXiv | 2025 | Attention-free denoiser enabled by semantically-enriched codec prior; WER 0.04%, RTF 0.016 (10x faster than F5-TTS); 4-5x more natural temporal pauses than deterministic NAR |
| [[2512.04720]] | M3-TTS: Multi-modal DiT Alignment and Mel-latent for Zero-shot High-fidelity Speech Synthesis | arXiv | 2025 | MMDiT-style joint text-speech attention for alignment-free NAR TTS; WER 1.36% on Seed-TTS test-en (best NAR and overall at submission) |
| [[2508.16332]] | Vevo2: A Unified and Controllable Framework for Speech and Singing Voice Generation | arXiv | 2025 | Extends zero-shot TTS to unified speech+singing via chromagram prosody tokenizer and GRPO post-training |
| [[2512.13251]] | DisCo-Speech: Controllable Zero-Shot Speech Generation with A Disentangled Speech Codec | arXiv | 2025 | DisCodec FSQ-based factorization enables independent timbre and prosody control in zero-shot synthesis; VC SIM outperforms Vevo and CosyVoice2 |
| [[2509.09631]] | DiFlow-TTS: Compact and Low-Latency Zero-Shot TTS with Factorized Discrete Flow Matching | arXiv | 2025 | Discrete flow matching over factorized FACodec tokens; WER 0.05%, MOS 4.18, best F0-RMSE in corpus; 164M model 5-34x smaller than comparable baselines |
| [[2603.08823]] | Fish Audio S2 Technical Report | arXiv | 2026 | Dual-AR with 4B backbone; best open-source WER 0.54% zh / 0.99% en; 81.88% win rate on EmergentTTS-Eval vs. GPT-4o-audio |
| [[2601.15621]] | Qwen3-TTS Technical Report | arXiv | 2026 | Zero-shot voice cloning across 10 languages with best open-source SPK-SIM in all evaluated languages; DPO+GRPO post-training |
| [[2604.00688]] | OmniVoice: Towards Omnilingual Zero-Shot TTS with Diffusion Language Models | arXiv | 2026 | Single-stage discrete NAR zero-shot TTS covering 600+ languages; CMOS +0.44 (best in corpus); WER ≤5% for 82/102 languages on FLEURS |
| [[2603.29339]] | LongCat-AudioDiT: High-Fidelity Diffusion TTS in the Waveform Latent Space | arXiv | 2026 | Waveform-latent flow-matching NAR; SIM 0.818 on Seed-ZH (best diffusion NAR), WER 1.50% on Seed-EN from a 3.5B model trained on 1M hours |
| [[2025.ccl-1.80]] | Lao-English Code-Switched Speech Synthesis Via Neural Codec Language Modeling | workshop | 2025 | Zero-shot cross-lingual code-switching without bilingual training data via phoneme latent space alignment |
| [[2025.acl-short.81]] | Zero-Shot Text-to-Speech for Vietnamese | ACL | 2025 | Benchmarks VALL-E, VoiceCraft, and XTTS-v2 for zero-shot TTS on Vietnamese PhoAudiobook; XTTS-v2 fine-tuned on PhoAudiobook achieves WER 8.32% and SMOS 3.39 — documents challenges of low-resource zero-shot TTS |
| [[2502.11128]] | FELLE: Autoregressive Speech Synthesis with Token-Wise Coarse-to-Fine Flow Matching | arXiv | 2025 | Codec-free AR + per-token flow matching for zero-shot voice cloning; outperforms MELLE on MOS and WER on LibriSpeech test-clean in both continuation and cross-sentence evaluation protocols |
| [[2508.04141]] | Parallel GPT: Harmonizing the Independence and Interdependence of Acoustic Tokens | arXiv | 2025 | Hybrid AR+NAR zero-shot TTS with parallel tokenizer; MOS 4.11 on LibriTTS, best naturalness among mid-scale compared systems |
| [[2508.15442]] | Mitigating Hallucinations in LM-Based TTS Models via Distribution Alignment | EMNLP | 2025 | GFlowNet post-training for LM-based zero-shot TTS; >50% CER reduction on Seed-TTS-Eval hard subset while maintaining speaker similarity |
| [[2508.19098]] | CLEAR: Continuous Latent Autoregressive Modeling for High-quality and Efficient TTS | arXiv | 2025 | Continuous AR zero-shot TTS; WER 1.88%, SPK-SIM 0.59, UTMOS 4.22 on LibriSpeech-PC test-clean |
| [[2509.15969]] | VoXtream: Full-Stream Text-to-Speech with Extremely Low Latency | arXiv | 2025 | Fully streaming zero-shot TTS with 102 ms GPU first-packet latency; best mid-scale SPK-SIM (0.537) on Seed-TTS test-en |
| [[2511.12347]] | VoiceCraft-X: Unifying Multilingual Voice-Cloning Speech Synthesis and Editing | EMNLP | 2025 | Multilingual zero-shot voice cloning and speech editing across 11 languages; CMOS 0.63 highest among compared systems |
| [[2601.03888]] | IndexTTS 2.5 Technical Report | arXiv | 2026 | Four-language zero-shot TTS with 2.28x faster inference; token-level concatenation for multilingual voice cloning |
| [[2603.18090]] | MOSS-TTS Technical Report | arXiv | 2026 | Open-source large-scale zero-shot TTS with WER 2.04% (EN) and CER 2.07% (ZH) on Seed-TTS-Eval, close to Qwen3-TTS and CosyVoice 3 |
| [[2603.26364]] | LLaDA-TTS: Unifying Speech Synthesis and Zero-Shot Editing via Masked Discrete Diffusion | arXiv | 2026 | Zero-shot TTS and editing via masked discrete diffusion; CER 0.98% (zh) on Seed-TTS-Eval, enabling zero-shot editing without AR generation |
| [[2604.01760]] | T5Gemma-TTS Technical Report | arXiv | 2026 | Encoder-decoder zero-shot TTS; resolves text-conditioning dilution in decoder-only AR TTS via cross-attention; best WER for multiple languages among compared systems |
| [[interspeech-2025-0319]] | Improving Noise Robustness of LLM-based Zero-shot TTS via Discrete Acoustic Token Denoising | Interspeech | 2025 | NR-LauraTTS: codec-domain speech denoiser (two-stage Conformer) replaces waveform-domain SE; recovers clean-prompt zero-shot quality from noisy prompts at only 1.10G additional FLOPs vs. 38.93G for waveform SE |
| [[interspeech-2025-0455]] | APTTS: Adversarial Post-training in Latent Flow Matching for Fast and High-fidelity TTS | Interspeech | 2025 | Latent flow-matching TTS with adversarial post-training reduces inference to 4 steps; competitive zero-shot quality with 918h training (WER 1.73%) — data-efficient zero-shot via adversarial acceleration |
| [[interspeech-2025-0596]] | Facilitating Personalized TTS for Dysarthric Speakers Using Knowledge Anchoring and Curriculum Learning | Interspeech | 2025 | Extends zero-shot TTS to pathological speakers via teacher-student knowledge anchoring and progressive curriculum learning on short dysarthric reference audio; >50% relative PER reduction vs. baselines |
| [[interspeech-2025-0989]] | HiFiTTS-2: A Large-Scale High Bandwidth Speech Dataset | Interspeech | 2025 | 36.7k-hour English dataset at 22 kHz enabling high-bandwidth zero-shot TTS; Koel-TTS trained on HiFiTTS-2 achieves SSIM 0.731 vs. 0.494 for LibriTTS-trained model on unseen speakers |
| [[2507.09318]] | ZipVoice-Dialog: Non-Autoregressive Spoken Dialogue Generation with Flow Matching | arXiv | 2026 | Zero-shot two-speaker dialogue generation via FM with learnable speaker-turn embeddings; cpSIM 0.437 and WER 3.25% at RTF 0.063 — zero-shot voice conditioning for dialogue |
| [[2507.22746]] | Next Tokens Denoising for Speech Synthesis (Dragon-FM) | arXiv | 2025 | Zero-shot voice prompting using same-paragraph speaker reference; chunk-wise AR+FM hybrid; SPK-SIM 0.916 and WER 2.74 on internal podcast data |
| [[2508.04585]] | UniTalker | arXiv | 2025 | Zero-shot speaker generalisation (SIM_SPK 0.702 on IEMOCAP unseen speakers) in conversational speech-visual synthesis |
| [[2508.04996]] | REF-VC | arXiv | 2025 | Zero-shot VC from a single reference utterance; noise robustness via random feature erasure; 10 unseen speakers from seed-tts-eval |
| [[2508.07302]] | XEmoRAG | arXiv | 2025 | Zero-shot cross-lingual emotion TTS for Thai; retrieval-based emotion transfer without speaker fine-tuning |
| [[2508.07426]] | Scalable Controllable Accented TTS | ASRU | 2025 | Zero-shot accented TTS via XTTS-v2 zero-shot cloning; accent conditioning from geolocation-filtered data |
| [[2508.14049]] | MahaTTS | arXiv | 2025 | Zero-shot speaker conditioning via nearest three reference clips; Indic multilingual coverage |
| [[interspeech-2025-2447]] | Accelerating Autoregressive Speech Synthesis Inference with SSD | Interspeech | 2025 | Zero-shot CosyVoice 2 evaluated at multiple tolerance factors; 1.4× LM-RTF speedup while maintaining SIM/MOS at β=0.4 |
| [[interspeech-2025-1641]] | Robust Neural Codec LM with Phoneme Position Prediction | Interspeech | 2025 | Zero-shot Mandarin TTS with 52.7% relative CER reduction on Seed-TTS test-zh via position supervision; 67.2% improvement on hard cases |
| [[interspeech-2025-1779]] | ReFlow-VC: Zero-shot VC via Rectified Flow | Interspeech | 2025 | Zero-shot VC on LibriTTS; SECS 0.843 vs. 0.781 for Diff-VC with fewer inference steps |
| [[interspeech-2025-0815]] | Non-AR Zero-Shot Expressive VC Disentanglement | Interspeech | 2025 | Zero-shot zero-shot expressive VC on RAVDESS/Expresso/ESD; substantially improved emotion transfer via discrete units and mix-style LN |
| [[interspeech-2025-1101]] | ZSDEVC: Zero-Shot Diffusion-based Emotional VC | Interspeech | 2025 | Zero-shot EVC with MI disentanglement; ECA 0.53 in zero-shot setting vs. 0.256 for EMOCONV-DIFF in seen-speaker |
| [[interspeech-2025-2043]] | MKL-VC: Training-Free VC via Factorized OT | Interspeech | 2025 | Training-free zero-shot VC matching FACodec from 5–10 s reference; fixes kNN-VC cross-lingual failure |
| [[interspeech-2025-1344]] | PEFT-TTS: Cross-Lingual Continual Learning | Interspeech | 2025 | Zero-shot multi-speaker Korean TTS via PEFT adapter; full fine-tuning destroys zero-shot while PEFT preserves it |
| [[interspeech-2025-2449]] | Accelerating Flow-Matching TTS via EPSS | Interspeech | 2025 | Zero-shot F5-TTS with 4× inference speedup at 7 NFE; WER 1.74%, SPK-SIM 0.68 maintained |
| [[interspeech-2025-1993]] | Watermark-Aware Codecs | Interspeech | 2025 | Zero-shot voice cloning protection; MOS 3.64 on clean prompts collapses to 1.27 on watermarked prompts — codec-level access control |
| [[interspeech-2025-2765]] | The State of TTS: HFR Case Study | Interspeech | 2025 | Systematic HFR evaluation of 10 zero-shot TTS systems; commercial systems approach human level, open-source systems remain 20+ points behind |
| [[2508.09702]] | M3PDB | arXiv | 2025 | 400K-hour multimodal prompt database with cascaded latency-aware selection; addresses quality mismatch between training and inference-time prompts |
| [[2508.16790]] | TaDiCodec | arXiv | 2025 | Zero-shot TTS via text-conditioned 6.25 Hz codec; WER 2.28% EN outperforming CosyVoice 2 and MaskGCT; documents reconstruction-generation gap across tokenizer families |
| [[2508.17031]] | RephraseTTS | arXiv | 2025 | Speech insertion with zero-shot speaker style transfer from surrounding audio context; cross-modal attention outperforms global speaker embeddings |
| [[2508.19205]] | VibeVoice Technical Report | arXiv | 2025 | Zero-shot multi-speaker long-form TTS; continuous VAE tokenizer at 7.5 Hz; SPK-SIM 0.692 on podcast benchmark |
| [[2509.04072]] | Computational Narrative Understanding for Expressive TTS | arXiv | 2025 | Zero-shot expressive TTS benchmark on LibriQuote; IndexTTS2 achieves human-level ContextMOS; flow-matching models benefit more from expressive fine-tuning |
| [[2403.03100]] | NaturalSpeech 3: Zero-Shot Speech Synthesis with Factorized Codec and Diffusion Models | arXiv | 2024 | Introduces FACodec with disentangled attribute subspaces; achieves human-level naturalness (CMOS 0.00) and SMOS 4.01 on LibriSpeech zero-shot TTS; SMOS exceeds ground truth |
| [[2410.00037]] | Moshi: a speech-text foundation model for real-time dialogue | arXiv | 2024 | Zero-shot voice TTS subsystem within Moshi; 4.7% WER on LibriSpeech test-clean in streaming mode via Inner Monologue; prompt-conditioned speaker voice preservation |
| [[2504.18425]] | Kimi-Audio Technical Report | arXiv | 2025 | Kimi-TTS subsystem enables zero-shot TTS from the same multimodal backbone; flow-matching streaming detokenizer with BigVGAN vocoder |
| [[2308.16692]] | SpeechTokenizer: Unified Speech Tokenizer for Speech Large Language Models | arXiv | 2023 | Foundational disentangled codec enabling better speaker generalization by separating semantic content (RVQ-1) from acoustic detail in downstream AR TTS |
| [[2503.01710]] | Spark-TTS: An Efficient LLM-Based Text-to-Speech Model with Single-Stream Decoupled Speech Tokens | arXiv | 2025 | Zero-shot TTS with small LLM backbone via single-stream decoupled tokens; shows speaker similarity gap remains relative to multi-stage systems |
| [[2409.00750]] | MaskGCT: Zero-Shot Text-to-Speech with Masked Generative Codec Transformer | arXiv | 2024 | NAR masked generative approach achieving human-level speaker similarity in zero-shot TTS without text-speech alignment; strong hard-text robustness |
| [[2505.17589]] | CosyVoice 3: Towards In-the-wild Speech Generation via Scaling-up and Post-training | arXiv | 2025 | Zero-shot TTS scaling and post-training via differentiable reward optimization; extends in-the-wild generalization with multi-task tokenizer |
| [[2502.04128]] | Llasa: Scaling Train-Time and Inference-Time Compute for Llama-based Speech Synthesis | arXiv | 2025 | Zero-shot TTS with inference-time compute scaling via speech understanding verifiers for quality improvement |
| [[2304.09116]] | NaturalSpeech 2: Latent Diffusion Models are Natural and Zero-Shot Speech and Singing Synthesizers | arXiv | 2023 | Latent diffusion zero-shot TTS using in-context learning for speaker adaptation; demonstrates continuous codec latents avoid AR repetition/skipping errors |
| [[2406.05370]] | VALL-E 2: Neural Codec Language Models are Human Parity Zero-Shot Text to Speech Synthesizers | arXiv | 2024 | Claimed human parity on LibriSpeech via grouped code prediction and repetition-aware sampling; proposes new benchmark conditions for zero-shot TTS evaluation |
| [[2407.04051]] | FunAudioLLM: Voice Understanding and Generation Foundation Models for Natural Interaction Between Humans and LLMs | arXiv | 2024 | CosyVoice zero-shot multilingual voice cloning; establishes cross-lingual zero-shot TTS at production scale with instruction-following capabilities |
| [[2409.03283]] | FireRedTTS: A Foundation Text-To-Speech Framework for Industry-Level Generative Speech Applications | arXiv | 2024 | Foundation TTS with zero-shot and few-shot modes; demonstrates few-shot fine-tuning substantially outperforms zero-shot for highly distinctive voices |
| [[2407.05361]] | Emilia: An Extensive, Multilingual, and Diverse Speech Dataset for Large-Scale Speech Generation | arXiv | 2024 | Shows in-the-wild training data enables zero-shot TTS on spontaneous speech; large-scale dataset for zero-shot TTS training across 6 languages |
| [[2406.18009]] | E2 TTS: Embarrassingly Easy Fully Non-Autoregressive Zero-Shot TTS | arXiv | 2024 | Demonstrates flow matching can learn speaker alignment from raw characters for zero-shot TTS without phoneme aligners; unsupervised pretraining improves zero-shot speaker similarity |
| [[2406.04904]] | XTTS: a Massively Multilingual Zero-Shot Text-to-Speech Model | arXiv | 2024 | Zero-shot TTS across 16 languages; trade-off between multilingual coverage and per-language speaker similarity in zero-shot settings |
| [[2403.16973]] | VoiceCraft: Zero-Shot Speech Editing and Text-to-Speech in the Wild | arXiv | 2024 | Zero-shot TTS and speech editing as unified AR infilling; demonstrates WER is unreliable proxy for perceptual intelligibility in zero-shot evaluation |
| [[2305.09636]] | SoundStorm: Efficient Parallel Audio Generation | arXiv | 2023 | Zero-shot voice conditioning via semantic token speaker prompt; parallel masked generation for fast zero-shot synthesis at scale |
| [[2303.03926]] | VALL-E X: Cross-lingual Neural Codec Language Modeling | arXiv | 2023 | Cross-lingual zero-shot voice transfer preserving identity, emotion, and acoustic environment without paired bilingual data |
| [[1712.05884]] | Natural TTS Synthesis by Conditioning WaveNet on Mel Spectrogram Predictions (Tacotron 2) | arXiv | 2017 | Predecessor to zero-shot TTS; single-speaker conditioned synthesis establishing mel-spectrogram as intermediate representation |
| [[2402.01912]] | Natural language guidance of high-fidelity text-to-speech with synthetic annotations | arXiv | 2024 | Zero-shot TTS with natural language style control; speaker similarity evaluated on zero-shot voice cloning benchmarks |
| [[2306.12925]] | AudioPaLM: A Large Language Model That Can Speak and Listen | arXiv | 2023 | In-context speaker voice preservation in cross-lingual speech synthesis exceeds TTS-based references in zero-shot setting |
| [[2305.07243]] | Better speech synthesis through scaling (Tortoise TTS) | arXiv | 2023 | Multi-speaker zero-shot TTS scaled via internet audio; contrastive re-ranking for quality improvement without per-speaker fine-tuning |
| [[2407.08551]] | MELLE: Autoregressive Speech Synthesis without Vector Quantization | arXiv | 2024 | Zero-shot codec-free AR TTS; speaker similarity scores can exceed ground-truth on standard metrics, raising concerns about metric validity |
| [[2402.08093]] | BASE TTS: Lessons from building a billion-parameter TTS model on 100K hours | arXiv | 2024 | Zero-shot speaker generalization with disentangled SSL tokens; SSL-based representations outperform acoustic codecs for zero-shot quality |
| [[2310.00704]] | UniAudio: An Audio Foundation Model Toward Universal Audio Generation | arXiv | 2023 | Zero-shot TTS as one of seven audio generation tasks; multi-task training improves zero-shot TTS relative to task-specific models |
| [[2501.06282]] | MinMo: A Multimodal Large Language Model for Seamless Voice Interaction | arXiv | 2025 | Zero-shot voice cloning within a multi-task spoken dialogue system via streaming voice decoder |
