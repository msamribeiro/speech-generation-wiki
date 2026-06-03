---
title: "Overview"
---
*Updated after every ~25 ingests or after a significant query that reveals a new pattern.*

Last updated: 2026-06-03 | Papers ingested: 146

---

## 1. Dominant Paradigms

**Text-to-Speech (TTS).** Two paradigms dominate open-weight zero-shot TTS as of mid-2025:

- **Pure flow-matching** ([[2025.acl-long.313]], F5-TTS; [[2025.acl-long.1043]], OZSpeech): Flow matching over mel-spectrograms with text and reference audio concatenation. F5-TTS (336M, 95K hours EN+ZH) is the reference open-weight system. OZSpeech demonstrates that replacing the Gaussian source distribution with a learned prior enables single-step inference (NFE=1) trained on only 500 hours.
- **LLM + flow matching** (CosyVoice 2, [[2510.00981]] FlexiCodec-TTS): Autoregressive LM predicts semantic tokens from text; a flow-matching or continuous-token acoustic model synthesizes mel/waveform conditioned on those tokens. This hybrid achieves better text adherence at the cost of AR latency. The low-frame-rate direction ([[2510.00981]]: 6.25 Hz AR) makes this practical for streaming.

The VALL-E-family pure AR codec LM ([[2301.02111]]) remains a baseline and conceptual anchor, with active work on improving its codec ([[2025.acl-long.1498]] DRI consistency), decoding ([[2025.emnlp-main.989]] MTP), and post-training alignment ([[2025.acl-long.598]] INTP/DPO).

**Voice Conversion (VC).** Real-time zero-shot VC is the dominant research direction. Two systems define the 2025 frontier: [[2507.14534]] (Conan, SSL-distillation content extraction, 37–140 ms GPU latency, 85.71% SPK-SIM) and [[2025.acl-demo.37]] (RT-VC, articulatory coding, 61.4 ms CPU latency). The gap between streaming and offline VC quality has effectively closed.

**Spoken Conversational Agents (SCA).** Aligned multimodal LLMs (speech encoder + text LLM + speech decoder) dominate over pure end-to-end speech LMs. GPT-4o voice mode, Moshi, Qwen2.5-Omni, and VocalNet ([[2025.emnlp-main.989]]) represent this paradigm. Key advances: cross-modal distillation without instruction data ([[2025.acl-long.388]] DiVA), empathetic response generation ([[2025.emnlp-demos.70]] OpenS2S), and the first interaction-annotation corpus for duplex modeling ([[2025.findings-emnlp.424]] InteractSpeech). A systematic diagnosis of why pure SLMs underperform text LLMs ([[2412.17048]]) reveals paralinguistic variability in tokens (Factor C) as the dominant bottleneck.

## 2. Emerging Trends

**Preference alignment as standard post-training for TTS.** [[2025.acl-long.598]] (INTP) demonstrates that DPO fine-tuning on ~250K paired samples reduces WER by 31–52% relative across five diverse architectures without degrading speaker similarity. DPO has now been extended beyond AR models to flow-matching (DPO-FM) and masked generative models (DPO-MGM), making preference alignment architecture-general. Interspeech 2025 adds two new RLHF approaches: DiffRO ([[interspeech-2025-0704]]) introduces Gumbel-Softmax differentiable token sampling to bypass audio decoding for reward computation (WER 0.78% zh on seed-tts-eval); DLPO ([[interspeech-2025-0063]]) demonstrates RLHF for diffusion TTS via task-specific loss regularization. Together, these papers establish RLHF as applicable across AR codec LMs, flow-matching, and diffusion TTS — no longer limited to any single paradigm.

**TTS inference acceleration as a dedicated research area.** FM acceleration continues to expand: APTTS ([[interspeech-2025-0455]]) via adversarial post-training (4 steps), RapFlow-TTS ([[interspeech-2025-0554]]) via consistency FM (2 NFE), and now EPSS ([[interspeech-2025-2449]]) via empirically pruned non-uniform step schedules (7 NFE, 4× RTF speedup, training-free). For AR TTS, Speech Speculative Decoding ([[interspeech-2025-2447]]) adapts speculative decoding from text LLMs with a tolerance-factor acceptance criterion that exploits the many-to-one mapping from speech tokens to perceptual quality (1.4× AR LM speedup on CosyVoice 2). The two complementary approaches — non-uniform step scheduling for FM and speculative decoding for AR — are both training-free drop-ins for existing deployed models.

**Codec reliability as a first-class design criterion.** [[2025.acl-long.1498]] formalizes Discrete Representation Inconsistency (DRI) — the same audio produces different codec token sequences depending on context — and shows that training-time consistency constraints reduce downstream VALL-E WER by 1.98% absolute at no reconstruction quality cost. Combined with [[2412.17048]]'s Factor C analysis, the field is converging on the view that codec design quality directly determines SLM/TTS quality, not just audio reconstruction fidelity. PAST ([[interspeech-2025-0669]]) challenges SSL distillation as the mechanism, demonstrating that supervised CTC losses on RVQ-1 outperform SSL pseudo-label distillation for phonetic content preservation.

**Dialogue-native TTS and spoken dialogue generation.** [[2509.02020]] (FireRedTTS-2), ZipVoice-Dialog ([[2507.09318]]), and long-context papers demonstrate that TTS systems are being designed for multi-turn conversational scenarios. ZipVoice-Dialog's curriculum learning + speaker-turn embeddings achieves 15× faster inference than AR dialogue baselines (Dia 1.61B), showing that NAR flow-matching is now competitive for the dialogue generation task.

**Large-scale evaluation and annotation infrastructure.** MIKU-PAL ([[interspeech-2025-0648]]) demonstrates automated multimodal emotion labeling at Fleiss κ=0.93 (vs. human 0.43) using Gemini 2.0 Flash, enabling the 131.2h MIKU-EmoBench dataset with 26 emotion categories. ParaSpeechCaps ([[2025.emnlp-main.180]]) provides 2709h of rich style-annotated data using perceptual speaker similarity and audio LLM quality filters. HiFiTTS-2 ([[interspeech-2025-0989]]) provides 36.7k hours at 22kHz from LibriVox. The field is building scalable annotation infrastructure rather than relying on small hand-labeled datasets.

**Social and dialect-aware evaluation.** [[2025.acl-long.1252]] demonstrates that prosodic/acoustic persona (voice accent) substantially outperforms lexical-syntactic style matching (dialect text) for user acceptance in AAE-speaking populations. This introduces socially-grounded subjective evaluation — measuring warmth, trustworthiness, and engagement — as a dimension absent from standard WER/MOS/SPK-SIM evaluations.

**PEFT as the default for cross-lingual continual adaptation.** [[interspeech-2025-1344]] demonstrates that adapter-based fine-tuning at 1.72% of model parameters is not merely computationally efficient but functionally necessary for cross-lingual continual adaptation: full fine-tuning on single-speaker data destroys zero-shot capability for previously learned languages even with only 12 hours of adaptation data. This finding, combined with prior LoRA results (UtterTune, MahaTTS), confirms PEFT-first as the practical default for extending FM-based multilingual TTS to new low-resource languages.

**Evaluation methodology under scrutiny.** [[interspeech-2025-2765]] introduces human fooling rate — measuring how frequently listeners mistake synthesised speech for real — as a new naturalness paradigm that reveals differentiation between leading zero-shot TTS systems invisible to MOS/MUSHRA. [[interspeech-2025-0401]] proposes a replicability checklist for TTS perceptual evaluations, addressing systematic underreporting in published listening tests. CodecBench ([[2508.20660]]) extends this to codec evaluation, separating acoustic reconstruction from semantic content quality. Together, these papers signal growing methodological awareness in the evaluation layer.

**Low-resource tonal language TTS.** [[2025.acl-industry.42]] provides a complete practical framework for Thai TTS, demonstrating that domain-specific acoustic modeling (Phoneme-Tone BERT, LLM-supervised pause prediction) combined with data-efficient pipeline design can achieve competitive results against Google TTS and Microsoft TTS.

## 3. Points of Tension

**Open-weight vs. closed-source gap in zero-shot TTS.** There is a persistent 5–7 SIM-point gap between the best open-weight systems (F5-TTS: SIM-o 0.67 on Seed-TTS test-en) and closed-source SOTA (Seed-TTS: 0.790, Minimax-Speech: 0.738). The root causes — data scale, model scale, or training recipe — are not yet clear. [[2509.19668]] shows that inference-time CFG tuning alone cannot close this gap.

**Intelligibility vs. naturalness trade-off.** OZSpeech ([[2025.acl-long.1043]]) achieves WER 0.05% (best in corpus on LibriSpeech test-clean) at UTMOS 3.15, while F5-TTS ([[2025.acl-long.313]]) achieves UTMOS 3.89 at WER 2.42%. There is no consensus on the right trade-off for practical applications, and no composite metric standardizes this balance.

**SSL vs. ASR-supervised representations for codec design.** [[2510.00981]] shows that ASR features dramatically outperform SSL features for dynamic frame merging at ultra-low rates. Yet SSL features remain widely used in SSL distillation codecs (Mimi, DualCodec). The question of when to use which foundation model representation type for codec semantic extraction is not yet resolved.

**DPO alignment vs. pre-training scale.** [[2025.acl-long.598]] shows diminishing returns in iterative preference alignment, suggesting base model capability is the ceiling. It remains unclear whether training on more diverse data, larger models, or better preference data construction would break through this ceiling — or whether the base model's inherent expressiveness limits what post-training can achieve.

**Cascade vs. end-to-end SCA.** Pure speech-to-speech LMs ([[2412.17048]] characterizes their coherence failures) remain far below cascade systems on semantic tasks. Commercial systems (GPT-4o, Qwen2.5-Omni) use cascade or hybrid approaches. [[2025.acl-long.388]] (DiVA) demonstrates that distillation from a text LLM narrows the gap using only ASR data, but cannot capture paralinguistic information that has no textual correlate.

**Reward model scope vs. downstream audio quality in RLHF.** DiffRO ([[interspeech-2025-0704]]) shows that codec-token-level reward prediction improves pronunciation WER substantially but has limited impact on final audio MOS, because the FM+vocoder stages largely denoise codec imperfections. This reveals a fundamental scope problem: optimizing the codec LM stage alone cannot improve quality dimensions controlled by the FM and vocoder stages.

**Consistency vs. adversarial acceleration for FM TTS.** RapFlow-TTS ([[interspeech-2025-0554]]) and APTTS ([[interspeech-2025-0455]]) both reduce FM TTS inference to 2–4 steps but via different mechanisms; APTTS achieves better intelligibility (WER 1.73%) at the cost of a 3-stage training pipeline, while RapFlow-TTS achieves simpler training but remains slightly below human-level MOS. There is no consensus on which acceleration mechanism is preferred for production deployment.

## 4. Gaps

- **Multilingual coverage beyond EN+ZH.** The dominant training datasets (Emilia, 95K hours EN+ZH) and evaluation benchmarks (SeedTTS-eval) center on English and Mandarin. Japanese, Korean, European languages, and especially low-resource tonal languages remain underrepresented. [[2025.acl-long.598]] shows INTP alignment generalizes to JA/KO/DE/FR from EN+ZH training, but this is not validated at pre-training scale.
- **Standardized spoken dialogue evaluation.** Turn-taking, interruption handling, backchannel timing, and empathy response are not captured by standard TTS metrics (WER, SPK-SIM, MOS). [[2025.findings-emnlp.424]] provides a starting point but end-to-end duplex evaluation remains standardization-free.
- **Causal/streaming zero-shot TTS.** Streaming VC is solved ([[2507.14534]], [[2025.acl-demo.37]]), but streaming zero-shot TTS with comparable quality to non-streaming systems at <100 ms latency remains undemonstrated.
- **Anti-spoofing robustness under preference alignment.** [[2025.acl-long.598]] notes that intelligibility-focused DPO may inadvertently reduce anti-spoofing resilience. This interaction has not been evaluated.
- **Prosody modeling for non-English, non-CJK languages.** [[2025.acl-long.1471]]'s context-length analysis is validated only on English audiobooks. Duration/pause behavior and prosody prediction requirements for tonal, agglutinative, or non-configurational languages are unstudied in the corpus.

## 5. Key Concept Hubs

By paper count (as of this update):

| Concept | Paper count | Key papers |
|---------|------------|-----------|
| [[self-supervised-speech]] | 11 | WavLM for SPK-SIM, HuBERT for VC content, Whisper for SLM encoding |
| [[zero-shot-tts]] | 8 | F5-TTS, VALL-E, INTP, OZSpeech, ControlSpeech, Conan |
| [[autoregressive-codec-tts]] | 9 | VALL-E, FlexiCodec, VocalNet, DRI, FireRedTTS-2 |
| [[neural-codec]] | 9 | EnCodec, FlexiCodec, DRI, Cont-SPT, ControlSpeech, VocalNet |
| [[spoken-language-model]] | 9 | SpeechLM survey, DiVA, VocalNet, OpenS2S, InteractSpeech, 2412.17048 |
| [[evaluation-metrics]] | 7 | InteractSpeech, DRI, INTP, SpeechLM survey, Thai TTS, prosody MI |
| [[flow-matching]] | 4 | F5-TTS, OZSpeech, INTP, Selective CFG |
| [[streaming-tts]] | 6 | Conan, RT-VC, FlexiCodec, VocalNet, OpenS2S, FireRedTTS-2 |

The heaviest hub — [[self-supervised-speech]] — reflects the foundational role of SSL models as content extractors, speaker encoders, evaluation tools (WavLM for SPK-SIM), and codec distillation targets across all three tasks.

## 6. Year-on-Year Perspective

The corpus currently spans 2023 ([[2301.02111]] VALL-E) through 2026 ([[2412.17048]] ICASSP 2026 paper, published 2026-01). The primary year cluster is 2025 (ACL 2025, EMNLP 2025, Interspeech 2025, arXiv preprints).

**2023:** VALL-E establishes codec LM paradigm. 60K hours of training, 3-second enrollment at inference. EnCodec at 75 Hz. Demonstrates zero-shot speaker generalization via scale.

**2024 (pre-corpus, referenced):** Voicebox, E2-TTS lay the groundwork for flow-matching TTS. Moshi achieves real-time duplex SLM. CosyVoice introduces LLM+flow-matching hybrid. Emilia dataset (100K hours EN+ZH) enables F5-TTS training.

**2025:** The dominant trend is post-training refinement and production deployment. F5-TTS (open-weight), preference alignment (INTP/DPO), codec consistency training (DRI), and streaming adaptation (Conan, RT-VC) all represent refinements of paradigms established in 2023–2024. The field shifts from "can we build a zero-shot TTS system?" to "how do we make it robust, efficient, multilingual, and conversational?" Key technical differentiators: Sway Sampling for inference-time flow improvements, DPO-FM/DPO-MGM extending alignment beyond AR models, DRI consistency training for codecs, and Factor C analysis providing a theoretical framing for SLM limitations.

The emerging 2025→2026 direction is dialogue-native TTS (FireRedTTS-2, InteractSpeech) and rigorous analysis of what limits current systems (DRI, Factor C, context-length constraints for prosody from [[2025.acl-long.1471]]). The field is maturing from empirical advances toward theoretical grounding. Integration pass 6 adds 26 Interspeech 2025 and arXiv papers, reinforcing three broad patterns: (1) inference acceleration as a first-class engineering concern (speculative decoding for AR, non-uniform step pruning for FM); (2) PEFT-first as the functional default for multilingual adaptation, not just a computational convenience; (3) evaluation methodology advances (fooling rate, replicability checklist, FD-Bench, CodecBench) emerging in parallel with synthesis quality improvements.
