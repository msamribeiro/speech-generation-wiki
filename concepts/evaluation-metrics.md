---
slug: evaluation-metrics
title: Evaluation Metrics
aliases: [speech quality metrics, TTS evaluation metrics, objective evaluation, automatic evaluation]
related_concepts: [subjective-evaluation, rlhf-speech, spoken-language-model]
last_updated: 2026-06-02
status: established
---
## Executive Summary

> [!abstract]
> Evaluation metrics for speech synthesis quantify performance along orthogonal dimensions — naturalness, intelligibility, speaker similarity, prosody accuracy, and overall quality — using both subjective listening tests and objective automatic measures. As of 2025, WER (via Whisper), SPK-SIM (via WavLM-Large), and UTMOS dominate the TTS evaluation landscape, but benchmark fragmentation is a persistent problem: no single public benchmark unifies all relevant dimensions. New interaction-specific and paralinguistic evaluation paradigms are emerging alongside traditional metrics.

## Current Status

established — The core metric vocabulary (MOS, WER, SPK-SIM, UTMOS) is widely adopted across TTS, VC, and codec papers, but evaluation practice remains fragmented: different papers use different test sets, ASR models, and speaker verification backbones, making cross-paper comparisons unreliable. New benchmarks (EmergentTTS-Eval, Audio Turing Test, CV3-Eval) are gaining traction but have not yet consolidated.

## Why This Matters

Consistent use of evaluation metrics allows fair comparison across systems. However, the field faces persistent issues: different papers use different test sets, different ASR models for WER, and different speaker verification models for SPK-SIM, making comparisons across papers unreliable. [[2025.findings-emnlp.424]] introduces a novel interaction dynamics evaluation task (binary classification of backchannel/interruption/gap/pause events) with its own benchmark, extending the field beyond standard TTS metrics toward dialogue-specific evaluation.

## Core Idea

Evaluation metrics for speech synthesis quantify system performance along several orthogonal dimensions: naturalness (how human-like the speech sounds), intelligibility (whether the content can be understood), speaker similarity (how closely the voice matches a reference), prosody accuracy, and overall quality. Metrics divide into subjective (requiring human listeners) and objective (computed automatically). Objective metrics are preferred for reproducibility and cost, but their correlation with human perception is imperfect.

**Dominant metrics in 2025 TTS:**
- WER (using HuBERT-Large-LS960-ft or Whisper ASR): primary intelligibility metric in flow-matching TTS papers [[2509.19668]], VC papers [[2507.14534]], and codec papers [[2510.00981]].
- SPK-SIM (WavLM-Large cosine similarity): primary speaker similarity metric for zero-shot TTS [[2509.19668]].
- MOS (1–5 scale, listening test): NMOS (naturalness) and QMOS (quality) are reported with confidence intervals in TTS papers [[2510.00981]]. Small listener panels (6 experts) are common in recent papers.
- UTMOS: automatic MOS predictor used alongside human MOS in codec evaluation [[2510.00981]].
- PESQ: signal-level quality metric used in codec evaluation [[2510.00981]].

**Dialogue / interaction metrics:**
- [[2025.findings-emnlp.424]] establishes binary classification accuracy on 4 interaction event types (backchannel, interruption, gap, pause) as an evaluation framework for spoken dialogue models. GPT-4o scores 54.2% (near-random for a binary task), highlighting that current models lack fine-grained interaction understanding.

## Methods and Variants

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

## Major Claims

Claims are generalised propositions aggregated from paper evidence. The full claim registry with supporting paper lists is in `wiki/concepts/_evidence/evaluation-metrics.yaml`.

### Strongly Supported

- WER via automatic ASR (Whisper or HuBERT-based) is the dominant intelligibility metric in TTS evaluation, but cross-lingual application requires language-specific ASR models and introduces comparability issues.
  Supporting: [[2025.acl-long.598]], [[2025.acl-industry.42]], [[2025.acl-short.81]]

- Automatic speaker similarity metrics (WavLM-Large cosine SPK-SIM) do not perfectly correlate with perceptual speaker similarity MOS, and can rank systems in opposite order from human listeners.
  Supporting: [[2509.09631]], [[2025.naacl-long.242]]

- No single public benchmark unifies naturalness, intelligibility, speaker similarity, and controllability accuracy in a single evaluation protocol; benchmark fragmentation is a persistent obstacle to fair cross-paper comparison.
  Supporting: [[2025.emnlp-main.40]], [[2025.acl-long.682]], [[interspeech-2025-0779]]

### Emerging

- Domain-specific and language-specific evaluation paradigms (phoneme-level metrics for low-resource TTS, interaction event classification for dialogue systems, mathematical expression intelligibility) are necessary complements to standard MOS/WER for capturing task-relevant quality.
  Supporting: [[interspeech-2025-0469]], [[2025.findings-emnlp.424]], [[interspeech-2025-0779]]

- Automatic naturalness predictors (UTMOS, DNSMOS) are now standard for rapid iteration in codec and TTS evaluation, though their correlation with human MOS degrades outside their training distribution.
  Supporting: [[2510.00981]], [[2603.29339]], [[interspeech-2025-0973]]

- Automatic MOS predictors consistently fail to discriminate among high-quality speech generation systems whose naturalness approaches human levels, indicating a ceiling effect in current SQA methodology.
  Supporting: [[2508.00317]], [[2508.07711]]

- Emotion similarity metrics correlate poorly at the utterance level with human perceptual judgements of emotional expressiveness, limiting their utility for fine-grained emotion TTS evaluation.
  Supporting: [[2504.12867]]

### Contested

> [!warning]
> Whether WavLM-Large-based SPK-SIM is a sufficient proxy for human speaker similarity MOS remains contested. [[2509.09631]] shows automatic SIM-O and perceptual MOS can rank systems in opposite order, but SPK-SIM remains the de facto standard in most zero-shot TTS papers due to cost.
> Challenging sufficiency: [[2509.09631]] / Using as standard: [[2509.19668]], [[2510.00981]]

## Relationship to Other Concepts

### Extends or Builds On
- [[subjective-evaluation]] — objective metrics (UTMOS, SPK-SIM, WER) were developed as cheaper proxies for subjective listening tests; their validity is validated against subjective scores

### Commonly Paired With
- [[rlhf-speech]] — preference optimization and RLHF in speech directly depend on evaluation metrics as reward signals; the choice of metric shapes what the reward model optimizes
- [[spoken-language-model]] — SLM evaluation requires both standard TTS metrics and new interaction-specific metrics; the two domains are converging on a shared evaluation vocabulary

## Representative Papers

### Foundational
- [[2025.emnlp-main.40]] — provides the most comprehensive taxonomy of evaluation metrics for controllable TTS, identifying fragmentation as a fundamental field-level problem

### Influential
- [[2025.acl-long.682]] — surveys SpeechLM evaluation across six categories (representation, linguistic, paralinguistic, generation quality/diversity, real-time interaction, downstream benchmarks), the broadest coverage in the corpus
- [[2509.09631]] — documents a concrete case where automatic SPK-SIM and perceptual similarity MOS rank systems in opposite order, challenging the proxy assumption

### Recent Highlights
- [[2603.08823]] — introduces EmergentTTS-Eval and Audio Turing Test as new paralinguistic instruction-following benchmarks for 2026 evaluation
- [[interspeech-2025-0779]] — demonstrates that standard ASR-based CER correlates below 0.15 with human intelligibility for mathematical TTS, motivating domain-specific metrics
- [[interspeech-2025-0973]] — first Spanish MOS corpus; validates CNN-based wav2vec 2.0 features as more predictive than contextual transformer layers for cross-lingual MOS prediction

## Open Questions

- Is WavLM-Large-based SPK-SIM a sufficient proxy for human speaker similarity MOS? DiFlow-TTS [[2509.09631]] shows that automatic SIM-O and perceptual similarity MOS can rank systems in opposite order.
- How should the WER/SIM trade-off be optimally balanced for zero-shot TTS? Both metrics are reported independently; no composite metric is standardized.
- What are the right metrics for evaluating spoken dialogue system naturalness beyond event classification accuracy [[2025.findings-emnlp.424]]?
- EmergentTTS-Eval (used by [[2603.08823]]) and Audio Turing Test are newer benchmarks gaining traction for paralinguistic instruction-following evaluation; what is their relationship to standard MOS and WER, and do they capture meaningfully different aspects of system capability?
- The controllable TTS survey [[2025.emnlp-main.40]] identifies no standardized evaluation combining naturalness, intelligibility, similarity, and controllability accuracy; what would such a benchmark look like?
- Full-duplex spoken dialogue systems need evaluation frameworks beyond standard TTS metrics; [[2503.04721]] (Full-Duplex-Bench) introduces TOR, backchannel frequency, and response latency across four turn-taking scenarios — how should these integrate with standard evaluation practice?
- For turn-taking and interaction benchmarks, human naturalistic conversation (as in [[2503.04721]]) and synthetic stimuli (GPT-4o + ChatTTS) differ in prosodic properties; does this gap affect benchmark validity?

## Trend Summary

Pre-2023: MOS and WER were the standard; speaker verification cosine similarity emerged as a faster alternative to SMOS. 2024–2025: UTMOS becomes standard in codec evaluation [[2510.00981]]; Seed-TTS-eval (English and Mandarin cross-sentence prompts) becomes the reference benchmark for zero-shot TTS. 2025: Interaction-specific evaluation emerges [[2025.findings-emnlp.424]]; the controllable TTS survey [[2025.emnlp-main.40]] documents that no benchmark unifies naturalness, intelligibility, similarity, and controllability accuracy in a single protocol; DiFlow-TTS [[2509.09631]] reports a concrete divergence between automatic SIM-O ranking and perceptual similarity MOS ranking, demonstrating that WavLM-based cosine similarity is insufficient for capturing perceptual voice identity. New frontiers emerging in 2025: Full-Duplex-Bench [[2503.04721]] introduces automated turn-taking metrics (TOR, backchannel frequency, response latency) for full-duplex dialogue evaluation; VoiceMOS/AudioMOS Challenge survey [[2508.00317]] documents that multi-track challenge infrastructure is the strongest accelerant for SQA method development, and that near-human-quality systems elude discrimination by current predictors; FreeGAN [[2508.07711]] provides concrete evidence that UTMOS and subjective MOS rank vocoders differently, adding to accumulated evidence that automatic predictors are insufficient surrogates. SpeechRole [[2508.02013]] introduces a nine-dimension evaluation framework for speech role-playing agents (Instruction Adherence, Conversational Coherence, Speech Fluency/Naturalness, Prosodic Consistency, Emotion Appropriateness, Personality/Knowledge Consistency). The non-verbal speech pipeline [[2508.05385]] introduces CLAP-based tag accuracy and IMOS as evaluation dimensions for non-verbal vocalization synthesis. 2026: EmergentTTS-Eval (instruction-following paralinguistic tasks) and Audio Turing Test emerge as new evaluation benchmarks used in Fish Audio S2 [[2603.08823]]; LongCat-AudioDiT [[2603.29339]] uses DNSMOS 3.40 as an alternative naturalness proxy; CV3-Eval for multilingual voice cloning across 9 languages is used by both [[2601.15621]] and [[2603.08823]]. F0-RMSE is used by DiFlow-TTS [[2509.09631]] and Vevo2 [[2508.16332]] as a prosody accuracy metric. The diversity of benchmarks remains a persistent problem: no single public benchmark adequately captures all dimensions of modern TTS capability.

## All Papers

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
| [[interspeech-2025-0779]] | Intelligibility of Text-to-Speech Systems for Mathematical Expressions | Interspeech | 2025 | Introduces LaTeX CER (LCER) and TeXBLEU for mathematical expression TTS evaluation; shows standard ASR-based metrics (TTS-ASR cascade CER=0.10) correlate <0.15 with human intelligibility for mathematical content |
| [[interspeech-2025-0973]] | A Dataset for Automatic Assessment of TTS Quality in Spanish | Interspeech | 2025 | First Spanish MOS corpus (4326 samples, 52 systems); validates that NISQA fine-tuned on Spanish improves PCC 0.71→0.73; finds CNN-based wav2vec 2.0 features more predictive of MOS than contextual transformer layers |
| [[interspeech-2025-0648]] | MIKU-PAL: Automated Multimodal Method for Speech Paralinguistic and Affect Labeling | Interspeech | 2025 | Demonstrates that annotation consistency (Fleiss κ=0.93) matters more than accuracy (68.5%) for downstream TTS training; introduces 26-category emotion taxonomy beyond Ekman's basic emotions |
| [[interspeech-2025-0143]] | Multimodal Prosody Modeling: A Use Case for Multilingual Sentence Mode Prediction | Interspeech | 2025 | Shows WHISPER fails on exclamatory sentences (near-chance recall), demonstrating that current ASR punctuation-based prosody labels are unreliable for exclamatory TTS prosody; early fusion of WavLM+XLM-RoBERTa achieves UAR 74.22% for German sentence mode |
| [[interspeech-2025-0469]] | Developing High-Quality TTS for Punjabi and Urdu | Interspeech | 2025 | Comprehensive phoneme-level evaluation (DRT, MRT, SUS) for low-resource South Asian language TTS; demonstrates that phoneme-based models substantially outperform MMS character-level approach |
| [[2503.04721]] | Full-Duplex-Bench | arXiv | 2025 | First automated benchmark for full-duplex dialogue systems across four turn-taking scenarios; introduces TOR, backchannel JSD, and response latency as reproducible metrics without human annotation |
| [[2504.12867]] | EmoVoice | arXiv | 2025 | Demonstrates that sentence-level emotion similarity (emotion2vec cosine) has only ~40% Spearman correlation with human MOS; multimodal LLMs fail as emotion judges |
| [[2508.00317]] | Advancing Speech Quality Assessment through Challenges | arXiv | 2025 | Retrospective on four VoiceMOS/AudioMOS Challenge editions; documents that near-human-quality systems defeat current SQA predictors and that cross-domain generalisation remains unsolved |
| [[2508.02013]] | SpeechRole | arXiv | 2025 | Nine-dimension evaluation framework for speech role-playing agents; LLM-as-judge (Gemini-2.5-Pro) achieves Spearman ρ 0.67–0.98 with human experts |
| [[2508.05385]] | Non-Verbal Speech Generation Pipeline | arXiv | 2025 | Introduces CLAP-based semantic score and IMOS for NV vocalization synthesis evaluation; TSA annotation quality drives controllability more than dataset scale |
| [[2508.06870]] | Text to Speech for Meitei Mayek Script | arXiv | 2025 | Native-speaker MOS evaluation with no automated metric baseline for a new script; illustrates the limits of evaluation in zero-comparison scenarios |
| [[2508.06890]] | Maestro-EVC | ASRU | 2025 | Comprehensive VC evaluation: WER, CER, F0-PCC, EECS, and four-dimension subjective MOS across seen and unseen speakers and emotions |
| [[2508.07302]] | XEmoRAG | arXiv | 2025 | Uses custom EMOS metric for cross-lingual emotion transfer evaluation; demonstrates divergence between emotion and naturalness MOS under cross-lingual transfer |
| [[2508.07426]] | Scalable Controllable Accented TTS | ASRU | 2025 | DCF metric for accent similarity alongside naturalness MOS; identifies divergence between automatic accent identification and human perceptual accent evaluation |
| [[2508.07711]] | Is GAN Necessary for Neural Vocoder? | arXiv | 2025 | Provides controlled comparison showing UTMOS and subjective MOS rank vocoders in different order; contributes to evidence that UTMOS is unreliable for fine-grained vocoder comparison |
| [[2508.08715]] | MultiGen | arXiv | 2025 | MOS + CER evaluation for three low-resource Southeast Asian languages; small-scale evaluation (30 samples, 10 listeners) highlights statistical limitations of low-resource TTS evaluation |
| [[interspeech-2025-0196]] | SPCODEC | Interspeech | 2025 | POLQA objective MOS alongside 7-listener subjective MOS for speech codec evaluation at 16 kHz and 32 kHz |
