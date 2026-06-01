---
slug: zero-shot-tts
title: Zero-Shot TTS
aliases: [voice cloning, any-speaker TTS, speaker generalization, few-shot TTS]
related_concepts: [speaker-adaptation, voice-conversion, neural-codec, instruction-conditioned-tts, autoregressive-codec-tts, flow-matching]
last_updated: 2026-05-30
---
## What it is

Zero-shot TTS is the ability to synthesize speech in the voice of an unseen speaker — one not seen during training — given only a short reference audio clip (typically 3–10 seconds) at inference time, without any fine-tuning. The system must generalize the speaker's timbre, speaking style, and vocal quality from the reference clip and apply it to arbitrary target text. This is distinct from speaker-conditioned TTS (which uses a fixed speaker embedding from a closed training set) and from fine-tuning approaches (which update model weights for each new speaker).

Zero-shot TTS systems typically learn a disentangled representation space where speaker characteristics and content are separated, allowing the speaker conditioning from one utterance to be combined with text content to generate new speech.

## Why it matters

Zero-shot TTS eliminates the need for per-speaker data collection and training, making personalized voice synthesis accessible for any speaker at inference time. This unlocks applications including personalized voice assistants, audiobook production, dubbing and localization, accessibility tools, and — with appropriate safeguards — voice conversion for entertainment. It is the dominant paradigm in modern TTS research, having largely displaced speaker-ID-conditioned models.

The challenge is balancing three objectives: speaker similarity to the reference (measured by SPK-SIM), intelligibility of the output (measured by WER/CER), and naturalness (MOS). These objectives are in tension, and the CFG trade-off is a key current research area.

## Current state of the art

As of early 2026, the frontier has expanded substantially with new large-scale open-source systems narrowing the gap to closed-source SOTA. On the Seed-TTS-Eval benchmark:
- Fish Audio S2 [[2603.08823]]: WER 0.54% (zh) / 0.99% (en) — best open-source WER; achieves 81.88% win rate on EmergentTTS-Eval against GPT-4o-audio and Gemini 2.5 Flash TTS.
- Qwen3-TTS [[2601.15621]]: WER 0.77% (zh) / 1.24% (en), top multilingual speaker similarity across 10 languages.
- OmniVoice [[2604.00688]]: WER 1.30% on LibriSpeech-PC (multilingual), SIM-o 0.741 (en) / 0.777 (zh) on Seed-TTS; subjective CMOS +0.44 (best reported in corpus).
- LongCat-AudioDiT [[2603.29339]]: SIM 0.818 on Seed-ZH (best diffusion NAR in corpus), WER 1.50% on Seed-EN from a 3.5B waveform-latent flow-matching model.
- IndexTTS2 [[2506.21619]]: WER 1.008% (zh) / 1.521% (en), best emotional expressiveness among evaluated open systems (EMOS 4.22).
- M3-TTS [[2512.04720]]: WER 1.36% on Seed-TTS test-en (VAE variant) — best WER among NAR systems evaluated against Seed benchmark including AR baselines.

Established SOTA from prior integration pass: F5-TTS ([[2025.acl-long.313]], SIM-o 0.67, WER 1.83%), CosyVoice 2 [[2412.10117]] (SIM 0.745, WER 2.47%), Seed-TTS [[2406.02430]] (CMOS -0.07 vs. human).

For intelligibility, OZSpeech [[2025.acl-long.1043]] and Flamed-TTS [[2510.02848]] and DiFlow-TTS [[2509.09631]] all achieve WER ~0.04-0.05% on LibriSpeech test-clean, trading UTMOS for near-perfect intelligibility on this specific benchmark.

The paradigm was established by VALL-E [[2301.02111]] in 2023. StyleTTS-ZS [[2025.naacl-long.242]] provides a notable codec-free alternative via distilled time-varying style diffusion, achieving CMOS -0.032 vs. ground truth on LibriSpeech with 10-20x faster inference than prior diffusion SOTA.

## Key variants and sub-approaches

**Flow-matching non-autoregressive (F5-TTS paradigm).** Pure flow matching over mel-spectrograms with text and reference audio as joint conditioning. Inference is non-autoregressive in the temporal sense (all frames generated in parallel via ODE integration). F5-TTS ([[2025.acl-long.313]]) and E2-TTS are the main representatives. F5-TTS introduces ConvNeXt V2 text refinement to resolve E2-TTS's 7% unrecoverable alignment failure rate (Mandarin), and Sway Sampling (biasing ODE steps toward early timesteps) to further improve robustness at inference. Classifier-free guidance (CFG) is critical and [[2509.19668]] shows that the CFG strategy significantly affects the SIM/WER trade-off.

**LLM + flow matching (CosyVoice paradigm).** A pre-trained LLM (Qwen2.5-0.5B in CosyVoice 2) autoregressively generates semantic tokens from text, which then condition a small (71M) flow-matching model for acoustic synthesis. This architecture achieves stronger text adherence by leveraging the LLM's language modeling capabilities, making CFG less critical for text conditioning.

**Autoregressive codec LM (VALL-E paradigm).** An AR LM generates codec RVQ-1 tokens from text + speaker embedding; a NAR model fills RVQ-rest. Speaker conditioning comes from a speaker verification model's embedding. Established by [[2301.02111]] (VALL-E, 2023) and extended with low-frame-rate tokens by FlexiCodec-TTS ([[2510.00981]]).

**One-step flow matching with learned prior (OZSpeech paradigm).** [[2025.acl-long.1043]] reformulates OT-CFM to start from a learned prior distribution (from a prior codes generator) rather than Gaussian noise, enabling single-step inference (NFE=1). Trained on only 500 hours, it achieves WER 0.05% on LibriSpeech test-clean — trading UTMOS for extreme intelligibility. Also demonstrates exceptional noise robustness: VALL-E degrades to 93% WER at 0 dB SNR while OZSpeech holds at 0.05–0.06%.

**Zero-shot VC (content-preserving).** Rather than text input, source speech is used. [[2507.14534]] (Conan) uses Emformer-distilled HuBERT content + CVQ style encoder for zero-shot VC at low latency.

## Comparison to alternatives

Speaker-adapted TTS (fine-tuning on target speaker data) achieves higher speaker similarity at the cost of requiring enrollment data and training time. Zero-shot TTS trades some speaker fidelity for generalization without enrollment. The gap between the two is narrowing; recent zero-shot systems are approaching fine-tuned quality on many speakers.

Prompt-conditioned TTS (using reference audio as a style prompt but not specifically for voice cloning) is overlapping with zero-shot TTS but focuses more on prosody/style matching than speaker identity.

## Year-on-year trajectory

2023: VALL-E [[2301.02111]] established the codec LM paradigm with 60K hours of LibriLight training and 3-second enrollment at inference. 2024: F5-TTS ([[2025.acl-long.313]]) and E2-TTS established pure flow matching as competitive; CosyVoice [[2407.05407]] introduced supervised semantic token conditioning; Seed-TTS [[2406.02430]] reached human-parity synthesis (CMOS -0.07) with RL post-training; CosyVoice 2 [[2412.10117]] added streaming and FSQ tokenizer. 2025: StyleTTS-ZS [[2025.naacl-long.242]] demonstrated codec-free distilled diffusion TTS with 10-20× faster inference; Flamed-TTS [[2510.02848]] and DiFlow-TTS [[2509.09631]] pushed WER to 0.04-0.05% on LibriSpeech test-clean; Marco-Voice [[2508.02038]] and EmoSteer-TTS [[2508.03543]] extended zero-shot to emotional expressiveness; PALLE [[2504.10352]] introduced PAR for 10x faster inference than AR; IndexTTS2 [[2506.21619]] solved AR duration control; M3-TTS [[2512.04720]] demonstrated MMDiT-style joint attention for alignment-free NAR TTS; DisCo-Speech [[2512.13251]] enabled independent prosody/timbre control via disentangled codec; DiSTAR [[2510.12210]] coupled AR drafting with discrete masked diffusion; DiTAR [[2502.03930]] demonstrated patch-based continuous AR+diffusion. 2026: Fish Audio S2 [[2603.08823]] and Qwen3-TTS [[2601.15621]] achieved best open-source WER while adding streaming and multilingual breadth; OmniVoice [[2604.00688]] scaled NAR TTS to 600+ languages; LongCat-AudioDiT [[2603.29339]] achieved SOTA speaker similarity for diffusion NAR on Seed-ZH (SIM 0.818); Vevo2 [[2508.16332]] extended zero-shot TTS to unified singing synthesis. The gap between open-source and closed-source SOTA has materially narrowed.

## Open questions

- Can inference-time CFG improvements [[2509.19668]] fully close the gap to closed-source SOTA, or is training-time methodology the bottleneck?
- Why does the CFG strategy effectiveness vary by language (English vs. Mandarin for F5-TTS)? Is it the text encoder architecture, training data, or something else?
- What is the right trade-off between SIM and WER for practical applications? Current evaluation focuses on maximizing both independently. OZSpeech [[2025.acl-long.1043]] demonstrates an extreme point (near-zero WER, lower UTMOS) in this trade-off space.
- Cross-lingual zero-shot TTS (reference speaker speaks language A, synthesis in language B) remains an open challenge, though [[2025.acl-long.598]] shows DPO alignment on cross-lingual preference pairs (500 zh2en + 500 en2zh samples) reduces cross-lingual WER by 50%+ for multiple systems.
- [[2025.acl-long.1043]] shows near-perfect WER on LibriSpeech at the cost of UTMOS; is the intelligibility/naturalness trade-off fundamental to the learned-prior approach, or addressable with better codec choice (FACodec is acknowledged as limiting)?

## Papers

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
