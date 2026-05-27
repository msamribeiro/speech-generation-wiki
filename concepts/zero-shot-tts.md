---
slug: zero-shot-tts
title: Zero-Shot TTS
aliases: [voice cloning, any-speaker TTS, speaker generalization, few-shot TTS]
related_concepts: [speaker-adaptation, voice-conversion, neural-codec, instruction-conditioned-tts, autoregressive-codec-tts, flow-matching]
last_updated: 2026-05-27
---

# Zero-Shot TTS

## What it is

Zero-shot TTS is the ability to synthesize speech in the voice of an unseen speaker — one not seen during training — given only a short reference audio clip (typically 3–10 seconds) at inference time, without any fine-tuning. The system must generalize the speaker's timbre, speaking style, and vocal quality from the reference clip and apply it to arbitrary target text. This is distinct from speaker-conditioned TTS (which uses a fixed speaker embedding from a closed training set) and from fine-tuning approaches (which update model weights for each new speaker).

Zero-shot TTS systems typically learn a disentangled representation space where speaker characteristics and content are separated, allowing the speaker conditioning from one utterance to be combined with text content to generate new speech.

## Why it matters

Zero-shot TTS eliminates the need for per-speaker data collection and training, making personalized voice synthesis accessible for any speaker at inference time. This unlocks applications including personalized voice assistants, audiobook production, dubbing and localization, accessibility tools, and — with appropriate safeguards — voice conversion for entertainment. It is the dominant paradigm in modern TTS research, having largely displaced speaker-ID-conditioned models.

The challenge is balancing three objectives: speaker similarity to the reference (measured by SPK-SIM), intelligibility of the output (measured by WER/CER), and naturalness (MOS). These objectives are in tension, and the CFG trade-off is a key current research area.

## Current state of the art

As of 2025, the frontier of zero-shot TTS is defined by large-scale flow-matching models. Closed-source systems: Seed-TTS (SIM-o 0.790 Seed-TTS test-en, from the Seed-TTS paper), Minimax-Speech (SIM 0.738). Open-weight systems: F5-TTS ([[2025.acl-long.313]], SIM-o 0.67 Seed-TTS test-en, WER 1.83%, RTF 0.15 at 16 NFE), and with def text CFG [[2509.19668]] (SIM 0.690 English Seed-TTS-eval), CosyVoice 2 with input text CFG [[2509.19668]] (SIM 0.666 English Seed-TTS-eval). CosyVoice 3-1.5B reports SIM 0.720. There remains a notable gap (5–7 SIM points) between open-weight and closed-source SOTA.

For intelligibility, OZSpeech [[2025.acl-long.1043]] achieves WER 0.05% on LibriSpeech test-clean with a single-step (NFE=1) flow-matching approach initialized from a learned prior — the best WER in the corpus for this benchmark, at the cost of lower UTMOS (3.15 vs. 3.7+ for F5-TTS).

For zero-shot VC (a closely related task), [[2507.14534]] (Conan) achieves 85.71% SPK-SIM on LibriTTS→VCTK, exceeding even offline baselines while operating in real-time.

The paradigm was established by VALL-E [[2301.02111]] in 2023, which demonstrated for the first time that zero-shot speaker generalization is achievable through large-scale LM pretraining (60K hours) with an enrolled 3-second reference clip and no per-speaker fine-tuning.

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

2023: VALL-E [[2301.02111]] established the codec LM paradigm with 60K hours of LibriLight training and 3-second enrollment at inference; NaturalSpeech 2/3 introduced flow matching and diffusion with factorized codec. 2024: Voicebox, E2-TTS, F5-TTS ([[2025.acl-long.313]]) established pure flow matching as competitive; F5-TTS specifically resolved E2-TTS's alignment failures via ConvNeXt text refinement and Sway Sampling; CosyVoice introduced supervised semantic token conditioning. 2025: Open-weight systems (F5-TTS, CosyVoice 2) become available for study; inference-time improvements via CFG strategies [[2509.19668]] and flow step scheduling [[2025.acl-long.313]] offer parameter-free gains; closed-source frontier (Seed-TTS, Minimax) remains ahead. A new integration direction emerges: [[2025.acl-long.346]] (ControlSpeech) demonstrates that zero-shot voice cloning and zero-shot language-style control can be achieved simultaneously. OZSpeech [[2025.acl-long.1043]] introduces single-step zero-shot TTS trained on only 500 hours, demonstrating that data efficiency and inference speed can be jointly improved via learned-prior flow matching. Preference alignment emerges as the key post-training technique for robustness: [[2025.acl-long.598]] shows that DPO fine-tuning on ~250K preference pairs reduces cross-lingual and code-switching WER by 31–52% relative across five architectures without sacrificing speaker similarity — establishing RLHF as a practical complement to pre-training scale for zero-shot robustness.

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
