# Wiki Log

Append-only chronological log of all operations.
First token of each entry must be: `ingest` | `filter` | `review` | `query` | `lint` | `discover` | `parse`

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
