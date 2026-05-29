---
slug: evaluation-metrics
title: Evaluation Metrics
aliases: [speech quality metrics, TTS evaluation metrics, objective evaluation, automatic evaluation]
related_concepts: [subjective-evaluation, rlhf-speech, spoken-language-model]
last_updated: 2026-05-29
---

# Evaluation Metrics

## What it is

Evaluation metrics for speech synthesis quantify system performance along several orthogonal dimensions: naturalness (how human-like the speech sounds), intelligibility (whether the content can be understood), speaker similarity (how closely the voice matches a reference), prosody accuracy, and overall quality. Metrics divide into subjective (requiring human listeners) and objective (computed automatically). Objective metrics are preferred for reproducibility and cost, but their correlation with human perception is imperfect.

## Why it matters

Consistent use of evaluation metrics allows fair comparison across systems. However, the field faces persistent issues: different papers use different test sets, different ASR models for WER, and different speaker verification models for SPK-SIM, making comparisons across papers unreliable. [[2025.findings-emnlp.424]] introduces a novel interaction dynamics evaluation task (binary classification of backchannel/interruption/gap/pause events) with its own benchmark, extending the field beyond standard TTS metrics toward dialogue-specific evaluation.

## Current state of the art

**Dominant metrics in 2025 TTS:**
- WER (using HuBERT-Large-LS960-ft or Whisper ASR): primary intelligibility metric in flow-matching TTS papers [[2509.19668]], VC papers [[2507.14534]], and codec papers [[2510.00981]].
- SPK-SIM (WavLM-Large cosine similarity): primary speaker similarity metric for zero-shot TTS [[2509.19668]].
- MOS (1–5 scale, listening test): NMOS (naturalness) and QMOS (quality) are reported with confidence intervals in TTS papers [[2510.00981]]. Small listener panels (6 experts) are common in recent papers.
- UTMOS: automatic MOS predictor used alongside human MOS in codec evaluation [[2510.00981]].
- PESQ: signal-level quality metric used in codec evaluation [[2510.00981]].

**Dialogue / interaction metrics:**
- [[2025.findings-emnlp.424]] establishes binary classification accuracy on 4 interaction event types (backchannel, interruption, gap, pause) as an evaluation framework for spoken dialogue models. GPT-4o scores 54.2% (near-random for a binary task), highlighting that current models lack fine-grained interaction understanding.

## Key variants and sub-approaches

Canonical metrics tracked in this wiki:

| Metric | Full name | Measures | Direction |
|--------|-----------|----------|-----------|
| MOS | Mean Opinion Score | Naturalness | higher is better |
| SMOS | Speaker Similarity MOS | Speaker likeness | higher is better |
| WER | Word Error Rate | Intelligibility (via ASR) | lower is better |
| CER | Character Error Rate | Intelligibility (via ASR) | lower is better |
| SPK-SIM | Speaker Cosine Similarity | Automatic speaker similarity | higher is better |
| UTMOS | Saeki et al. automatic MOS | Predicted naturalness | higher is better |
| DNSMOS | Microsoft DNS MOS | Predicted naturalness | higher is better |
| EER | Equal Error Rate | Anti-spoofing / speaker verification | lower is better |
| MUSHRA | Multiple Stimuli with Hidden Reference and Anchor | Perceptual quality | higher is better |
| PESQ | Perceptual Evaluation of Speech Quality | Signal-level quality | higher is better |
| STOI | Short-Time Objective Intelligibility | Intelligibility | higher is better |
| F0-RMSE | Pitch tracking error | Prosody accuracy | lower is better |

## Comparison to alternatives

Human listening studies (MOS, MUSHRA) remain the gold standard but are expensive and slow. Automatic metrics (UTMOS, DNSMOS) predict human scores and enable rapid iteration. SPK-SIM via WavLM-Large is now accepted as a proxy for speaker similarity MOS (SMOS), though the two do not perfectly correlate. WER via ASR is a reliable intelligibility proxy for English; cross-lingual WER requires language-specific ASR models, complicating multilingual evaluation ([[2509.19668]] uses the same F5-TTS model for English and Mandarin but observes language-specific CFG behavior).

## Year-on-year trajectory

Pre-2023: MOS and WER were the standard; speaker verification cosine similarity emerged as a faster alternative to SMOS. 2024–2025: UTMOS becomes standard in codec evaluation [[2510.00981]]; Seed-TTS-eval (English and Mandarin cross-sentence prompts) becomes the reference benchmark for zero-shot TTS. 2025: Interaction-specific evaluation emerges [[2025.findings-emnlp.424]]; the controllable TTS survey [[2025.emnlp-main.40]] documents that no benchmark unifies naturalness, intelligibility, similarity, and controllability accuracy in a single protocol; DiFlow-TTS [[2509.09631]] reports a concrete divergence between automatic SIM-O ranking and perceptual similarity MOS ranking, demonstrating that WavLM-based cosine similarity is insufficient for capturing perceptual voice identity. 2026: EmergentTTS-Eval (instruction-following paralinguistic tasks) and Audio Turing Test emerge as new evaluation benchmarks used in Fish Audio S2 [[2603.08823]]; LongCat-AudioDiT [[2603.29339]] uses DNSMOS 3.40 as an alternative naturalness proxy; CV3-Eval for multilingual voice cloning across 9 languages is used by both [[2601.15621]] and [[2603.08823]]. F0-RMSE is used by DiFlow-TTS [[2509.09631]] and Vevo2 [[2508.16332]] as a prosody accuracy metric. The diversity of benchmarks (Seed-TTS-Eval, LibriSpeech-PC, FLEURS-102, MiniMax-Multilingual-24, EmergentTTS-Eval) remains a persistent problem: no single public benchmark adequately captures all dimensions of modern TTS capability.

## Open questions

- Is WavLM-Large-based SPK-SIM a sufficient proxy for human speaker similarity MOS? DiFlow-TTS [[2509.09631]] shows that automatic SIM-O and perceptual similarity MOS can rank systems in opposite order.
- How should the WER/SIM trade-off be optimally balanced for zero-shot TTS? Both metrics are reported independently; no composite metric is standardized.
- What are the right metrics for evaluating spoken dialogue system naturalness beyond event classification accuracy [[2025.findings-emnlp.424]]?
- EmergentTTS-Eval (used by [[2603.08823]]) and Audio Turing Test are newer benchmarks gaining traction for paralinguistic instruction-following evaluation; what is their relationship to standard MOS and WER, and do they capture meaningfully different aspects of system capability?
- The controllable TTS survey [[2025.emnlp-main.40]] identifies no standardized evaluation combining naturalness, intelligibility, similarity, and controllability accuracy; what would such a benchmark look like?

## Papers

| ID | Title | Venue | Year | Key use of this concept |
|----|-------|-------|------|------------------------|
| [[2025.findings-emnlp.424]] | InteractSpeech: A Speech Dialogue Interaction Corpus for Spoken Dialogue Model | EMNLP 2025 | 2025 | Introduces interaction event classification (backchannel/interruption/gap/pause binary accuracy) as a new evaluation paradigm for spoken dialogue models |
| [[2025.acl-long.1498]] | Analyzing and Mitigating Inconsistency in Discrete Speech Tokens for Neural Codec Language Models | ACL | 2025 | Introduces consistency accuracy — a novel codec-level metric measuring token agreement between context-free and context-inclusive encodings of the same audio segment — used alongside PESQ, ViSQOL, WER, SPK-SIM, and UTMOS |
| [[2025.acl-long.598]] | Advancing Zero-shot TTS Intelligibility across Diverse Domains via Preference Alignment | ACL | 2025 | Uses WER (Whisper-large-v3 for EN, Paraformer-zh for ZH/code-switching), SPK-SIM (WavLM TDNN cosine), UTMOS, N-CMOS, and reading accuracy (binary human intelligibility judgment) across four domain types; introduces reading accuracy as a targeted intelligibility evaluation complementary to WER |
| [[2025.acl-long.682]] | Recent Advances in Speech Language Models: A Survey | ACL | 2025 | Surveys evaluation across six categories: representation, linguistic, paralinguistic, generation quality/diversity, real-time interaction, and downstream benchmarks (SUPERB, VoiceBench, OpenAudioBench, VoxEval); provides the most comprehensive taxonomy of SpeechLM evaluation in the corpus |
| [[2025.acl-industry.42]] | Scaling Under-Resourced TTS: A Data-Optimized Framework for Thai | ACL | 2025 | Uses WER, UTMOS, PESQ, STOI, and NMOS on the TSync2 Thai benchmark alongside domain-specific evaluation (finance, healthcare, education, law); demonstrates cross-domain robustness evaluation paradigm for under-resourced TTS |
| [[2025.acl-long.1471]] | The time scale of redundancy between prosody and linguistic context | ACL | 2025 | Proposes mutual information estimation (via BERT-based prosody predictors) as an evaluation framework for prosody-context relationships; shows MI saturates at 3–8 past words — an information-theoretic approach to characterizing prosody modeling requirements |
| [[2025.acl-long.1252]] | Finding A Voice: Exploring the Potential of African American Dialect and Voice Generation for Chatbots | ACL | 2025 | Uses a 15-metric Likert-scale evaluation (covering comprehension, warmth, inoffensiveness, trustworthiness, engagement) for spoken chatbot quality; demonstrates multi-dimensional socially-grounded evaluation for SCA systems |
| [[2025.naacl-long.242]] | StyleTTS-ZS: Efficient High-Quality Zero-Shot TTS with Distilled Time-Varying Style Diffusion | NAACL | 2025 | Uses CMOS-N (comparative MOS for naturalness) and CMOS-S (speaker similarity) alongside UTMOS, WER, RTF; reports expressiveness (pitch+energy CV) as a novel metric for prosodic diversity |
| [[2025.emnlp-main.40]] | Towards Controllable Speech Synthesis in the Era of LLMs: A Systematic Survey | EMNLP | 2025 | Surveys evaluation metrics for controllable TTS: MCD, FDSD, WER, cosine SIM, PESQ (objective); MOS, CMOS, AB, ABX (subjective); identifies fragmentation across control strategies as a major problem |
| [[2509.09631]] | DiFlow-TTS: Compact and Low-Latency Zero-Shot TTS with Factorized Discrete Flow Matching | arXiv | 2025 | Documents divergence between automatic SIM-O (cosine similarity) and perceptual Similarity MOS ranking — SIM-O ranks DiFlow-TTS third but perceptual MOS ranks it first; highlights limitations of embedding-based automatic speaker similarity |
| [[2603.29339]] | LongCat-AudioDiT: High-Fidelity Diffusion TTS in the Waveform Latent Space | arXiv | 2026 | Uses UTMOS, DNSMOS alongside SPK-SIM for naturalness on Seed benchmark; introduces Adaptive Projection Guidance (APG) as an inference-time quality improvement evaluable through UTMOS/DNSMOS |
| [[2603.08823]] | Fish Audio S2 Technical Report | arXiv | 2026 | Uses EmergentTTS-Eval (win rate vs. multiple closed-source systems), Audio Turing Test (posterior mean for human-likeness), and Fish Audio Instruction Benchmark (TAR, Naturalness, Expressiveness) as newer evaluation paradigms for instruction-following TTS |
| [[2025.acl-long.911]] | DNASpeech: A Contextualized and Situated TTS Dataset with Dialogue, Narrative, and Action Prompts | ACL | 2025 | Introduces MOS-D (dialogue naturalness), MOS-S (situated appropriateness), and a 100-utterance leaderboard evaluation protocol with PESQ, WER, CER, and UTMOS for contextualized TTS — first evaluation framework specifically targeting situated speaking style |
| [[2025.acl-short.81]] | Zero-Shot Text-to-Speech for Vietnamese | ACL | 2025 | Uses WER, SMOS (speaker MOS), F0-RMSE, and MOS for Vietnamese zero-shot TTS evaluation; demonstrates the need for language-specific ASR models for WER in tonal low-resource languages |
| [[2508.12001]] | FNH-TTS: A Fast, Natural, and Human-Like Speech Synthesis System | arXiv | 2025 | Uses MUSHRA alongside MOS, WER, and F0-RMSE for comprehensive evaluation on LJSpeech and VCTK; demonstrates that GAN vocoder replacement (VOCOS vs. HiFi-GAN) has measurable impact across all quality dimensions |
| [[2509.00685]] | MPO: Multidimensional Preference Optimization for LM-based TTS | arXiv | 2025 | Uses CER, SPK-SIM, and log F0-RMSE as the three dimensions of a multidimensional preference evaluation; ABX preference test for subjective validation — provides a practical framework for multi-dimensional TTS alignment evaluation |
| [[2510.07979]] | IntMeanFlow: Few-step Speech Generation with Integral Velocity Distillation | arXiv | 2025 | Uses WER, SIM-o (cosine via WavLM), and CMOS to evaluate few-step distilled flow matching TTS against full teacher; demonstrates that 3-NFE distilled model matches teacher on Seed-TTS test-en |
| [[2509.15969]] | VoXtream: Full-Stream Text-to-Speech with Extremely Low Latency | arXiv | 2025 | Uses WER, SPK-SIM, UTMOS, and GPU first-packet latency to evaluate streaming TTS at different model scales; introduces throughput (tokens/second) as a relevant production metric |
| [[2603.26364]] | LLaDA-TTS: Unifying Speech Synthesis and Zero-Shot Editing via Masked Discrete Diffusion | arXiv | 2026 | Uses CER/WER, SIM-o, and UTMOS on Seed-TTS-Eval for both TTS and zero-shot editing evaluation; demonstrates that edit quality can be measured with the same metrics as generation quality |
| [[2604.12438]] | An Ultra-Low Latency End-to-End Streaming Speech Synthesis Architecture | arXiv | 2026 | Uses RTF, average first-byte latency, WER, UTMOS, and PESQ for streaming TTS evaluation; demonstrates that streaming codec-based NAR TTS achieves 303× real-time speed at competitive quality |
