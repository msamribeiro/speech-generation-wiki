# Wiki Log

Append-only chronological log of all operations.
First token of each entry must be: `ingest` | `integrate` | `filter` | `review` | `query` | `lint` | `discover` | `parse`

---

## 2026-05-12

- filter | arXiv | 404 accepted, 31 review, 69 rejected

## 2026-05-19

- review | arXiv | 31 borderline resolved → 15 accepted, 16 rejected | final corpus: 419 accepted, 85 rejected

## 2026-05-22

- filter | ACL 2025 + EMNLP 2025 + NAACL 2025 + Interspeech 2025 + arXiv + workshops pending batch | 300 accepted, 39 review, 56 rejected
- review | ACL 2025 + cs.CL batch | 39 borderline resolved → 25 accepted, 14 rejected | final corpus: 733 accepted, 166 rejected

## 2026-05-23

- parse | batch queue created | 36 batches × 20 papers (718 total) | batch 1 partial: 8/20 done | workflow: conversion in main session, quality inspection via lightweight sub-agent

## 2026-05-24

- discover | 450 candidates surfaced (86 speech-relevant), 264 corpus papers
- discover | 458 candidates surfaced (112 speech-relevant), 270 corpus papers

## 2026-05-25

- filter | keyword filter expanded (+11 terms: text to speech, speech-to-speech, speech interaction, voice interaction, spoken chatbot, speech foundation model, audio codec, speech tokenizer, voice assistant, voicemos, speech synthesizer) | re-scan triggered
- filter | ISCA 2025 re-scan | 1179 papers, 140 passed title filter, 12 new written
- filter | ACL Anthology 2025 re-scan | 14669 papers, 134 passed filter, 14 new written, 120 arXiv records enriched
- filter | arXiv cs.SD+eess.AS re-scan (2025-08-01→2026-05-25) | 3604 discovered, 555 passed filter, 70 new written
- filter | arxiv.py --ids 2301.02111 2407.05407 2412.10117 2406.02430 2410.06885 | 5 citation-discovery papers fetched (VALL-E, CosyVoice, CosyVoice 2, Seed-TTS, F5-TTS)
- filter | citation-discovery + re-scan batch | 101 papers scored in-conversation | 67 accepted, 7 review, 27 rejected | accept rate 66%

## 2026-05-26

- review | citation-discovery + re-scan batch | 7 borderline resolved → 3 accepted (2025.clicit-1.81, 2025.coling-industry.29, 2603.02022), 4 rejected | review queue cleared
- ingest | 2509.02020 | FireRedTTS-2: Towards Long Conversational Speech Generation for Podcast and Chatbot | arXiv 2025
- ingest | session-10 | Built native Claude Code multi-agent ingest pipeline (orchestrator + per-paper worker agent specs). Smoke-tested on 2509.02020. Deleted SDK-based scripts/ingest/agent.py and prompts.py.
- parse | batch 21 (queue batch 1) | 40 papers (2025.acl-long.388 … 2512.18706) | 40/40 done | Patched _REFS_HEADER_RE: added French "Références" and modifier-word prefix (e.g. "Bibliographical References"); 2510.03741 + 2510.25577 re-parsed, refs recovered
- parse | batch 22 (queue batch 2) | 40 papers (2512.20156 … 2601.19952) | 40/40 done | Clean run
- parse | batch 23 (queue batch 3) | 40 papers (2601.20094 … 2602.23068) | 40/40 done | Patched _REFS_HEADER_RE: added letter-prefix headings (e.g. "B. REFERENCES"); 2602.06053 re-parsed, 33 refs recovered | total parsed: 531/798
- ingest | 2507.14534 | Conan: A Chunkwise Online Network for Zero-Shot Adaptive Voice Conversion | arXiv 2025
- ingest | 2509.19668 | Selective Classifier-free Guidance for Zero-shot Text-to-speech | arXiv 2025
- ingest | 2510.00981 | FlexiCodec: A Dynamic Neural Audio Codec for Low Frame Rates | arXiv 2025
- ingest | 2412.17048 | Why Do Speech Language Models Fail to Generate Semantically Coherent Outputs? | arXiv 2026
- ingest | 2025.findings-emnlp.424 | InteractSpeech: A Speech Dialogue Interaction Corpus for Spoken Dialogue Model | EMNLP 2025
- ingest-batch | paper-pages pass | 5 ingested, 0 failed | 9 concept pages updated
- query | keyword filter expansion review — 11 new terms mapped to concept gaps | created speech-to-speech.md; expanded self-supervised-speech.md to cover foundation models
- ingest | 2025.acl-demo.37 | RT-VC: Real-Time Zero-Shot Voice Conversion with Speech Articulatory Coding | ACL 2025
- ingest | 2025.acl-industry.42 | Scaling Under-Resourced TTS: A Data-Optimized Framework with Advanced Acoustic Modeling for Thai | ACL 2025
- ingest | 2025.acl-long.1043 | OZSpeech: One-step Zero-shot Speech Synthesis with Learned-Prior-Conditioned Flow Matching | ACL 2025
- ingest | 2025.acl-long.1252 | Finding A Voice: Exploring the Potential of African American Dialect and Voice Generation for Chatbots | ACL 2025
- ingest | 2025.acl-long.1471 | The time scale of redundancy between prosody and linguistic context | ACL 2025
- ingest-batch | paper-pages pass | 5 ingested, 0 failed | 0 concept pages updated (concept pass skipped by user instruction)
- ingest | 2025.acl-long.1498 | Analyzing and Mitigating Inconsistency in Discrete Speech Tokens for Neural Codec Language Models | ACL 2025
- ingest | 2025.acl-long.313 | F5-TTS: A Fairytaler that Fakes Fluent and Faithful Speech with Flow Matching | ACL 2025
- ingest | 2025.acl-long.346 | ControlSpeech: Towards Simultaneous and Independent Zero-shot Speaker Cloning and Zero-shot Language Style Control | ACL 2025
- ingest | 2025.acl-long.388 | Distilling an End-to-End Voice Assistant Without Instruction Training Data | ACL 2025
- ingest | 2025.acl-long.598 | Advancing Zero-shot Text-to-Speech Intelligibility across Diverse Domains via Preference Alignment | ACL 2025

## 2026-05-27

- ingest | session-14 verification | Confirmed batch 3 (2025.acl-long.1498, .313, .346, .388, .598) fully ingested — no data loss. STATUS.md updated: 16 ingested, ~515 ready.
- ingest | interspeech-2025-0253 | Long-Context Speech Synthesis with Context-Aware Memory | Interspeech 2025
- ingest | 2025.emnlp-main.1730 | FillerSpeech: Towards Human-Like Text-to-Speech Synthesis with Filler Insertion and Filler Style Control | EMNLP 2025
- ingest | 2025.findings-naacl.184 | Continuous Speech Tokenizer in Text To Speech | NAACL 2025
- ingest | 2025.emnlp-demos.70 | OpenS2S: Advancing Fully Open-Source End-to-End Empathetic Large Speech Language Model | EMNLP 2025
- ingest-batch | 4 ingested, 0 failed
- ingest | 2301.02111 | Neural Codec Language Models are Zero-Shot Text to Speech Synthesizers | arXiv 2023
- ingest | interspeech-2025-0902 | VoiceQualityVC: A Voice Conversion System for Studying the Perceptual Effects of Voice Quality in Speech | Interspeech 2025
- ingest | 2025.emnlp-main.989 | VocalNet: Speech LLMs with Multi-Token Prediction for Faster and High-Quality Generation | EMNLP 2025
- ingest | 2025.acl-long.682 | Recent Advances in Speech Language Models: A Survey | ACL 2025
- ingest | 2025.americasnlp-1.1 | Text-to-speech system for low-resource languages: A case study in Shipibo-Konibo | workshop 2025
- ingest-batch | 5 ingested, 0 failed
- parse | batch 24 (queue batch 4) | 40 papers (2602.23266 … 2603.14032) | 40/40 done | RapidOCR: 2602.23765, 2603.08574, 2603.08823, 2603.09120, 2603.11589 (non-fatal) | total parsed: 571/798
- integrate | 10 papers | 14 concepts updated | 12 cross-links added
- integrate | 15 papers | 16 concepts updated | 3 cross-links added

## 2026-05-28

- ingest | 2406.02430 | Seed-TTS: A Family of High-Quality Versatile Speech Generation Models | arXiv 2024
- ingest | 2407.05407 | CosyVoice: A Scalable Multilingual Zero-shot Text-to-speech Synthesizer based on Supervised Semantic Tokens | arXiv 2024
- ingest | 2025.acl-long.65 | Autoregressive Speech Synthesis without Vector Quantization | ACL 2025
- ingest | 2412.10117 | CosyVoice 2: Scalable Streaming Speech Synthesis with Large Language Models | arXiv 2024
- ingest | 2410.06885 | F5-TTS: A Fairytaler that Fakes Fluent and Faithful Speech with Flow Matching | ACL 2025
- ingest-batch | 5 ingested, 0 failed
- lint | duplicate detected: 2410.06885 is arXiv preprint of 2025.acl-long.313 (ACL canonical) — wiki page and index entry removed, metadata set to rejected
- lint | full corpus duplicate scan — 14 arXiv/proceedings pairs resolved; arXiv IDs rejected, proceedings IDs canonical; 6 parsed output directories remapped from arXiv to proceedings IDs; corpus: 754 accepted, 29 ingested, 217 rejected
- parse | batch 25 (queue batch 5) | 40 papers (2603.14035 … 2604.06356) | 40/40 done | RapidOCR: 2603.14853, 2603.22252, 2603.23938, 2603.24144 (non-fatal) | total parsed: 611/783
- ingest | 2601.15621 | Qwen3-TTS Technical Report | arXiv 2026
- ingest | 2512.14291 | GLM-TTS Technical Report | arXiv 2025
- ingest | 2508.06262 | Llasa+: Free Lunch for Accelerated and Streaming Llama-Based Speech Synthesis | arXiv 2025
- ingest | 2502.03930 | DiTAR: Diffusion Transformer Autoregressive Modeling for Speech Generation | arXiv 2025
- ingest | 2504.10352 | Pseudo-Autoregressive Neural Codec Language Models for Efficient Zero-Shot Text-to-Speech Synthesis | arXiv 2025
- ingest-batch | 5 ingested, 0 failed
- ingest-batch | 5 ingested, 0 failed
- ingest | 2508.16332 | Vevo2: A Unified and Controllable Framework for Speech and Singing Voice Generation | arXiv 2025
- ingest | 2508.02038 | Marco-Voice Technical Report | arXiv 2025
- ingest | 2604.00688 | OmniVoice: Towards Omnilingual Zero-Shot Text-to-Speech with Diffusion Language Models | arXiv 2026
- ingest | 2508.03543 | EmoSteer-TTS: Fine-Grained and Training-Free Emotion-Controllable Text-to-Speech via Activation Steering | arXiv 2025
- ingest | 2510.02848 | Flamed-TTS: Flow Matching Attention-Free Models for Efficient Generating and Dynamic Pacing Zero-shot Text-to-Speech | arXiv 2025
- ingest-batch | 5 ingested, 0 failed
- ingest | 2506.21619 | IndexTTS2: A Breakthrough in Emotionally Expressive and Duration-Controlled Auto-Regressive Zero-Shot Text-to-Speech | arXiv 2025
- ingest | 2025.naacl-long.242 | StyleTTS-ZS: Efficient High-Quality Zero-Shot Text-to-Speech Synthesis with Distilled Time-Varying Style Diffusion | NAACL 2025
- ingest | 2510.12210 | DiSTAR: Diffusion over a Scalable Token Autoregressive Representation for Speech Generation | arXiv 2025
- ingest | 2025.emnlp-main.40 | Towards Controllable Speech Synthesis in the Era of Large Language Models: A Systematic Survey | EMNLP 2025
- ingest | 2603.08823 | Fish Audio S2 Technical Report | arXiv 2026
- ingest-batch | primary-session sequential experiment | 5 ingested (2506.21619, 2025.naacl-long.242, 2510.12210, 2025.emnlp-main.40, 2603.08823) | 0 failed | no subagents used
- parse | batch 26 (queue batch 6) | 40 papers (2604.06871 … 2604.22821) | 40/40 done | RapidOCR: 2604.11424 (×4, non-fatal) | 0 refs: 2604.13288 (no References header, refs in body text, non-blocking) | total parsed: 651/783

## 2026-05-29

- ingest | 2512.04720 | M3-TTS: Multi-modal DiT Alignment & Mel-latent for Zero-shot High-fidelity Speech Synthesis | arXiv 2025
- ingest | 2511.12347 | VoiceCraft-X: Unifying Multilingual, Voice-Cloning Speech Synthesis and Speech Editing | EMNLP 2025
- ingest | 2509.00685 | MPO: Multidimensional Preference Optimization for Language Model-based Text-to-Speech | arXiv 2025
- ingest | 2512.13251 | DisCo-Speech: Controllable Zero-Shot Speech Generation with A Disentangled Speech Codec | arXiv 2025
- ingest | 2509.09631 | DiFlow-TTS: Compact and Low-Latency Zero-Shot Text-to-Speech with Factorized Discrete Flow Matching | arXiv 2025
- ingest | 2603.29339 | LongCat-AudioDiT: High-Fidelity Diffusion Text-to-Speech in the Waveform Latent Space | arXiv 2026
- ingest | 2508.11273 | EmoSSLSphere: Multilingual Emotional Speech Synthesis with Spherical Vectors and Discrete Speech Tokens | arXiv 2025
- ingest | 2604.12438 | An Ultra-Low Latency, End-to-End Streaming Speech Synthesis Architecture via Block-Wise Generation and Depth-Wise Codec Decoding | arXiv 2026
- ingest | 2604.01760 | T5Gemma-TTS Technical Report | arXiv 2026
- ingest | 2508.15442 | Mitigating Hallucinations in LM-Based TTS Models via Distribution Alignment Using GFlowNets | EMNLP 2025
- ingest | 2025.acl-long.654 | Language-Codec: Bridging Discrete Codec Representations and Speech Language Models | ACL 2025
- ingest | 2603.18090 | MOSS-TTS Technical Report | arXiv 2026
- ingest | 2508.04141 | Parallel GPT: Harmonizing the Independence and Interdependence of Acoustic and Semantic Information for Zero-Shot Text-to-Speech | arXiv 2025
- ingest | 2502.11128 | FELLE: Autoregressive Speech Synthesis with Token-Wise Coarse-to-Fine Flow Matching | arXiv 2025
- ingest | 2603.26364 | LLaDA-TTS: Unifying Speech Synthesis and Zero-Shot Editing via Masked Diffusion Modeling | arXiv 2026
- ingest | 2508.19098 | CLEAR: Continuous Latent Autoregressive Modeling for High-quality and Low-latency Speech Synthesis | arXiv 2025
- ingest | 2508.12001 | FNH-TTS: A Fast, Natural, and Human-Like Speech Synthesis System with advanced prosodic modeling based on Mixture of Experts | arXiv 2025
- ingest | 2510.05758 | EMORL-TTS: Reinforcement Learning for Fine-Grained Emotion Control in LLM-based TTS | ICASSP 2026
- ingest | 2601.03888 | IndexTTS 2.5 Technical Report | arXiv 2026
- ingest | 2509.15969 | VoXtream: Full-Stream Text-to-Speech with Extremely Low Latency | arXiv 2025
- ingest | 2510.07979 | IntMeanFlow: Few-step Speech Generation with Integral Velocity Distillation | arXiv 2025
- ingest-experiment | Mitigation B parallel direct subagents validated — workers write paper page only; main session does batch cleanup pass; ~187–281k tokens/5-paper batch, zero shared-file conflicts, ~2 min wall time; adopted as standard ingest workflow
- ingest | 2025.ccl-1.80 | Lao-English Code-Switched Speech Synthesis Via Neural Codec Language Modeling | workshop 2025
- ingest | 2025.coling-main.352 | DiffStyleTTS: Diffusion-based Hierarchical Prosody Modeling for Text-to-Speech with Diverse and Controllable Styles | workshop 2025
- ingest | 2025.acl-long.911 | DNASpeech: A Contextualized and Situated Text-to-Speech Dataset with Dialogues, Narratives and Actions | ACL 2025
- ingest | 2025.acl-short.81 | Zero-Shot Text-to-Speech for Vietnamese | ACL 2025
- ingest | 2025.acl-long.912 | LLaMA-Omni 2: LLM-based Real-time Spoken Chatbot with Autoregressive Streaming Speech Synthesis | ACL 2025
- ingest-batch | 5 ingested, 0 failed | Mitigation B; Write/Edit subagent permissions added to settings.local.json
