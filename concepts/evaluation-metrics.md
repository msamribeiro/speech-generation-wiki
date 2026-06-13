---
slug: evaluation-metrics
title: Evaluation Metrics
aliases: [speech quality metrics, TTS evaluation metrics, objective evaluation, automatic evaluation]
related_concepts: [subjective-evaluation, rlhf-speech, spoken-language-model]
last_updated: 2026-06-13
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

- WER derived from large-vocabulary ASR systems is not a valid proxy for subjective speech intelligibility in codec benchmarking on isolated word stimuli; STOI and ESTOI are reasonable aggregate proxies but fail to capture gender- and wordlist-specific variation.
  Supporting: [[interspeech-2025-0984]]

- Pairwise ranking objectives with adaptive margins and quality-aware weighting improve automatic MOS predictor system-level correlation over standard regression losses across TTS, TTM, and TTA domains.
  Supporting: [[2508.08957]], [[2509.03292]]

- Automatic empathy evaluation pipelines using LLM scoring and automatic emotion classifiers diverge measurably from human judgements; model rankings remain consistent but absolute scores differ, limiting absolute score interpretation.
  Supporting: [[2508.09600]], [[2508.17623]]

- Current spoken dialogue systems consistently fail to sustain emotionally coherent responses across multi-turn conversations, and this failure is not fully captured by single-turn evaluation protocols.
  Supporting: [[2508.17623]]

- LLM-based evaluation augmented with discrete acoustic features achieves strong alignment with human judgment on multi-dimensional spoken role-playing tasks.
  Supporting: [[2509.03940]]

- Human Fooling Rate (HFR) — the proportion of trials where listeners misclassify synthetic speech as human — reveals performance gaps that CMOS/MUSHRA scores conceal, because preference metrics allow reference matching to inflate perceived quality.
  Supporting: [[interspeech-2025-2765]]

- ABX error rate, a widely-used tokenizer evaluation metric, negatively correlates with downstream SLM performance; n-gram predictability and phoneme mutual information are more informative proxies.
  Supporting: [[interspeech-2025-0246]]

- Evaluation protocol standardization through shareable recipe bundles — separating evaluation design from platform implementation — is a necessary precondition for replicable perceptual evaluation in speech synthesis.
  Supporting: [[interspeech-2025-0401]]

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
- [[2204.02152]] — UTMOS won VoiceMOS Challenge 2022 and became the standard automatic MOS predictor in TTS evaluation pipelines; ranked first on utterance-level MSE, SRCC, and system-level MSE
- [[2212.04356]] — Whisper provides the robust ASR backbone universally used for WER-based intelligibility evaluation in TTS and codec papers
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

Integration pass 7 (Aug–Sep 2025): Three dedicated automatic MOS improvement papers appear — QAMRO [[2508.08957]] introduces pairwise ranking with adaptive margins and quality-aware weighting; AESA-Net [[2509.03292]] applies triplet loss over BEATs embeddings for multi-axis aesthetic assessment; both demonstrate that regression-only training is systematically suboptimal for perceptual quality ordering. SITool [[interspeech-2025-0984]] provides the first systematic correlation study between DRT/MRT subjective intelligibility and objective metrics for neural codecs, with the key negative result that WER (via Whisper) is uncorrelated with subjective intelligibility on isolated-word stimuli. The dialogue evaluation cluster grows further: EMO-Reasoning [[2508.17623]] introduces a unified three-axis framework (continuous VAD, categorical rationality, human perceptual) for emotional coherence in spoken dialogue; VoxRole [[2509.03940]] adds a role-playing dimension with acoustically-aware LLM evaluation; OSUM-EChat [[2508.09600]] documents auto–human divergence in empathy scoring. LibriQuote [[2509.04072]] introduces ContextMOS and Win-Rate as new expressivity-specific metrics for audiobook TTS. M3PDB [[2508.09702]] identifies quality mismatch between training and inference prompts as a systematic degradation source, connecting prompt evaluation to zero-shot TTS methodology. These papers collectively reinforce the trend toward task-specific evaluation frameworks that go beyond MOS/WER/SPK-SIM.

Pre-2023: MOS and WER were the standard; speaker verification cosine similarity emerged as a faster alternative to SMOS. 2024–2025: UTMOS becomes standard in codec evaluation [[2510.00981]]; Seed-TTS-eval (English and Mandarin cross-sentence prompts) becomes the reference benchmark for zero-shot TTS. 2025: Interaction-specific evaluation emerges [[2025.findings-emnlp.424]]; the controllable TTS survey [[2025.emnlp-main.40]] documents that no benchmark unifies naturalness, intelligibility, similarity, and controllability accuracy in a single protocol; DiFlow-TTS [[2509.09631]] reports a concrete divergence between automatic SIM-O ranking and perceptual similarity MOS ranking, demonstrating that WavLM-based cosine similarity is insufficient for capturing perceptual voice identity. New frontiers emerging in 2025: Full-Duplex-Bench [[2503.04721]] introduces automated turn-taking metrics (TOR, backchannel frequency, response latency) for full-duplex dialogue evaluation; VoiceMOS/AudioMOS Challenge survey [[2508.00317]] documents that multi-track challenge infrastructure is the strongest accelerant for SQA method development, and that near-human-quality systems elude discrimination by current predictors; FreeGAN [[2508.07711]] provides concrete evidence that UTMOS and subjective MOS rank vocoders differently, adding to accumulated evidence that automatic predictors are insufficient surrogates. SpeechRole [[2508.02013]] introduces a nine-dimension evaluation framework for speech role-playing agents (Instruction Adherence, Conversational Coherence, Speech Fluency/Naturalness, Prosodic Consistency, Emotion Appropriateness, Personality/Knowledge Consistency). The non-verbal speech pipeline [[2508.05385]] introduces CLAP-based tag accuracy and IMOS as evaluation dimensions for non-verbal vocalization synthesis. 2026: EmergentTTS-Eval (instruction-following paralinguistic tasks) and Audio Turing Test emerge as new evaluation benchmarks used in Fish Audio S2 [[2603.08823]]; LongCat-AudioDiT [[2603.29339]] uses DNSMOS 3.40 as an alternative naturalness proxy; CV3-Eval for multilingual voice cloning across 9 languages is used by both [[2601.15621]] and [[2603.08823]]. F0-RMSE is used by DiFlow-TTS [[2509.09631]] and Vevo2 [[2508.16332]] as a prosody accuracy metric. The diversity of benchmarks remains a persistent problem: no single public benchmark adequately captures all dimensions of modern TTS capability. Integration pass 6 adds three evaluation-specific contributions: HFR [[interspeech-2025-2765]] provides direct empirical evidence that CMOS and MUSHRA overestimate naturalness when evaluating against low-expressivity reference recordings; the replikant platform [[interspeech-2025-0401]] proposes a replicability-by-design framework that the field has discussed but not operationalised for a decade; and DC-Spin [[interspeech-2025-0246]] provides a negative result against ABX, which is the most widely used proxy metric for speech tokenizer evaluation. CodecBench [[2508.20660]] advances multi-domain codec evaluation with acoustic-semantic orthogonality as a key finding. FD-Bench [[interspeech-2025-0739]] adds automated interrupt-handling metrics for full-duplex dialogue evaluation. The convergence of these papers signals a maturing evaluation subfield that is challenging the sufficiency of individual metrics (MOS, UTMOS, SPK-SIM, ABX) for capturing relevant dimensions of modern synthesis quality.

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
| [[2508.20660]] | CodecBench | arXiv | 2025 | Multi-domain codec benchmark: 8 acoustic signal metrics (PESQ, STOI, SDR, ViSQOL, etc.) plus ASR probing WER and classification accuracy as semantic proxies; reveals acoustic and semantic quality as orthogonal evaluation axes |
| [[interspeech-2025-0246]] | DC-Spin | Interspeech | 2025 | Proxy analysis showing n-gram predictability and PNMI/CNMI are stronger SLM downstream proxies than ABX error rate — a negative result for a widely-used evaluation standard |
| [[interspeech-2025-0401]] | Enabling Replicability of Speech Synthesis Perceptual Evaluations | Interspeech | 2025 | Proposes recipe-based replicability framework; replikant platform; reviewer checklist for evaluation protocol reporting; distinguishes reproducibility from replicability in TTS evaluation |
| [[interspeech-2025-2765]] | The State of TTS: Human Fooling Rates Case Study | Interspeech | 2025 | Introduces HFR (Human Fooling Rate) as a binary deception metric; shows CMOS/MUSHRA overestimate naturalness; commercial systems approach but open-source systems fall 20+ points short of human deception rates |
| [[interspeech-2025-1066]] | Score-Based Training for Energy-Based TTS Models | Interspeech | 2025 | UTMOSv2 and subjective MOS diverge for EBM-based TTS; documents a case where automatic naturalness predictor is unreliable for non-standard synthesis paradigms |
| [[2204.02152]] | UTMOS: UTokyo-SaruLab System for VoiceMOS Challenge 2022 | arXiv | 2022 | Introduces the ensemble SSL-MOS predictor that becomes the community's primary automatic speech quality metric; first in-class on VoiceMOS Challenge 2022 main and OOD tracks |
| [[1904.02882]] | LibriTTS: A Corpus Derived from LibriSpeech for Text-to-Speech | arXiv | 2019 | Establishes 24 kHz sentence-level TTS corpus with MOS evaluation methodology; foundational training and evaluation dataset for multi-speaker TTS naturalness benchmarking |
| [[2212.04356]] | Robust Speech Recognition via Large-Scale Weak Supervision (Whisper) | arXiv | 2022 | Whisper is the de facto ASR backbone for WER-based intelligibility evaluation in TTS papers; demonstrates that evaluation on in-distribution benchmarks overstates robustness |
| [[2006.04558]] | FastSpeech 2: Fast and High-Quality End-to-End Text to Speech | arXiv | 2020 | Evaluates TTS with carefully controlled MOS and CMOS protocols; demonstrates that forced-alignment durations measurably improve intelligibility and that end-to-end waveform generation requires explicit variance conditioning |
| [[interspeech-2025-0739]] | FD-Bench: Full-Duplex Benchmarking Pipeline | Interspeech | 2025 | Interrupt-specific metrics (SRRate, IRD, FSED, EIRate, NIRate) for FDSDS evaluation; LLM-as-judge and conditioned perplexity as complementary quality signals |
| [[interspeech-2025-2043]] | MKL-VC: Training-Free VC via Factorized OT | Interspeech | 2025 | Introduces composite total score (weighted sum of WER, CER, and SPK-SIM distance from ideal point) for VC system ranking; cross-lingual evaluation on FLEURS |
| [[interspeech-2025-2449]] | Accelerating Flow-Matching TTS via EPSS | Interspeech | 2025 | Reports RTF alongside WER, SPK-SIM, UTMOS for step-count reduction evaluation; three-benchmark evaluation (Seed-TTS-eval EN/ZH, LibriSpeech-PC) provides more robust speedup-quality characterisation |
| [[interspeech-2025-0816]] | SSANSVC | Interspeech | 2025 | Custom MOS-n (naturalness) and MOS-ts (timbre similarity) alongside SIM, SingMOS, and CER for cross-modal speech-singing VC evaluation |
| [[2508.07273]] | Incorporating Contextual Paralinguistic Understanding in Large Speech-Language Models | arXiv | 2025 | LLM judge validation with classification metric correlation; GPT-4o judge scores correlate reliably with accuracy/F1 on deterministic questions |
| [[2508.07375]] | TurnGuide | arXiv | 2025 | GPT-based automated semantic evaluation of spoken dialogue; validated against human MOS across 25/30 pairwise comparisons |
| [[2508.08957]] | QAMRO | ASRU | 2025 | Pairwise ranking loss with adaptive margins and quality-aware weighting improves automatic MOS system-level SRCC over regression baselines across TTS/TTM/TTA |
| [[2508.09600]] | OSUM-EChat | arXiv | 2025 | Documents divergence between GPT-4o and emotion2vec automatic evaluation vs. human empathy MOS; ranking preserved but absolute scores differ |
| [[2508.09702]] | M3PDB | arXiv | 2025 | Automatic-only evaluation (SS, UTMOS, CER) for prompt selection quality; demonstrates quality mismatch between training-time and inference-time prompts |
| [[2508.11224]] | Benchmarking Prosody Encoding in Discrete Speech Tokens | ASRU | 2025 | Token Error Rate (TER) as a direct proxy for prosody sensitivity in discrete SSL tokens, without downstream model dependence |
| [[interspeech-2025-0984]] | Benchmarking Neural Speech Codec Intelligibility with SITool | Interspeech | 2025 | SITool open platform for DRT/MRT codec benchmarking; shows WER is uncorrelated with subjective DRT intelligibility; STOI adequate for aggregate ranking only |
| [[2508.13028]] | Integrating Feedback Loss from Bi-modal Sarcasm Detector for Sarcastic Speech Synthesis | arXiv | 2025 | Circular evaluation design (detector used as both training signal and evaluation metric); 13-listener subjective study |
| [[2508.15931]] | QvTAD | arXiv | 2025 | Pairwise comparison framing reduces annotation subjectivity in perceptual attribute modelling |
| [[2508.16188]] | AVLM (Seeing is Believing) | EMNLP | 2025 | Emotion F1 via Qwen2-Audio as third-party emotion classifier for synthesized speech evaluation |
| [[2508.17494]] | Improving French Synthetic Speech Quality via SSML Prosody Control | workshop | 2025 | MOS AB test on 18 listeners; automated SSML tag evaluation via F1 and MAE on numerical prosodic parameters |
| [[2508.17623]] | EMO-Reasoning | arXiv | 2025 | Three-axis emotional coherence framework (continuous ERS, categorical rationality, human perceptual ERS) for spoken dialogue systems; first unified multi-turn emotional evaluation |
| [[2508.18006]] | Unseen Speaker and Language Adaptation for Lightweight TTS with Adapters | arXiv | 2025 | Introduces Phoneme Substitution Rate (PSR) as objective proxy for accent nativeness; validated against MUSHRA with >90% pairwise ranking accuracy |
| [[2509.01391]] | MixedG2P-T5 | arXiv | 2025 | UTMOS, CER, WARP-Q, and SDR as automatic-only proxies for G2P-free Japanese TTS naturalness |
| [[2509.02244]] | Spectrogram Patch Codec | arXiv | 2025 | PESQ, STOI, ViSQOL, MCD evaluation across different sampling rates; reveals MCD divergence from perceptual quality when comparing across architectures |
| [[2509.03292]] | Improving Perceptual Audio Aesthetic Assessment via Triplet Loss | arXiv | 2025 | Triplet loss with BEATs embeddings improves SRCC on multi-axis aesthetic prediction; domain shift affects absolute score calibration more than ranking |
| [[2509.03940]] | VoxRole | arXiv | 2025 | Acoustically-aware LLM judge augmented with discrete acoustic features; Pearson correlation 0.762 with human annotators |
| [[2509.04072]] | Computational Narrative Understanding for Expressive TTS | arXiv | 2025 | ContextMOS and Win-Rate (Gemini-2.5-Pro judge) as expressivity-specific metrics for audiobook TTS beyond standard naturalness MOS |
| [[2308.16692]] | SpeechTokenizer: Unified Speech Tokenizer for Speech Large Language Models | arXiv | 2023 | Introduces codec evaluation framework separating semantic (WER, ABX) and acoustic (PESQ, STOI, UTMOS) quality dimensions; demonstrates how unified codec quality correlates with downstream task metrics |
| [[2505.17589]] | CosyVoice 3: Towards In-the-wild Speech Generation via Scaling-up and Post-training | arXiv | 2025 | Differentiable reward optimization using MOS predictors and ASR-based WER as training-time signals; demonstrates treating evaluation metrics as differentiable rewards |
| [[2502.04128]] | Llasa: Scaling Train-Time and Inference-Time Compute for Llama-based Speech Synthesis | arXiv | 2025 | Uses speech understanding verifiers (WER, speaker similarity) as inference-time selection criteria; demonstrates evaluation metrics as best-of-N selection signals |
| [[2406.05370]] | VALL-E 2: Neural Codec Language Models are Human Parity Zero-Shot Text to Speech Synthesizers | arXiv | 2024 | Introduces new evaluation conditions (VCTK multi-speaker, specific benchmark splits) to claim human parity; documents how benchmark framing affects human parity claims |
| [[2408.16532]] | WavTokenizer: an Efficient Acoustic Discrete Codec Tokenizer for Audio Language Modeling | arXiv | 2024 | Comprehensive multi-metric evaluation across PESQ, STOI, UTMOS, MCD for codec comparison; demonstrates evaluation metric disagreement across architecture families |
| [[2410.17196]] | VoiceBench: Benchmarking LLM-Based Voice Assistants | arXiv | 2024 | Introduces VoiceBench multi-capability evaluation framework for voice assistants; reveals substantial gaps between pipeline and E2E systems on robustness metrics |
| [[2306.00814]] | Vocos: Closing the gap between time-domain and Fourier-based neural vocoders | arXiv | 2023 | Ablation-driven vocoder comparison using PESQ, WVMOS, MUSHRA showing Fourier-domain GAN closes quality gap with time-domain at lower computational cost |
| [[2407.05361]] | Emilia: An Extensive, Multilingual, and Diverse Speech Dataset for Large-Scale Speech Generation | arXiv | 2024 | Uses MOS, WER, and speaker similarity to compare in-the-wild and audiobook training data; introduces acoustic diversity metric in SSL feature space |
| [[2406.18009]] | E2 TTS: Embarrassingly Easy Fully Non-Autoregressive Zero-Shot TTS | arXiv | 2024 | Finds that zero-shot TTS SPK-SIM can exceed ground-truth recordings on standard speaker verification metrics, questioning what the metric actually measures |
| [[2406.04904]] | XTTS: a Massively Multilingual Zero-Shot Text-to-Speech Model | arXiv | 2024 | Comparative evaluation showing multilingual vs. monolingual model comparisons produce misleading results when per-language training data is not matched |
| [[2409.05377]] | BigCodec: Pushing the Limits of Low-Bitrate Neural Speech Codec | arXiv | 2024 | PESQ, STOI, WER, and subjective evaluation; notes PESQ and STOI insufficient alone for codec quality characterization |
| [[2305.02765]] | HiFi-Codec: Group-residual Vector Quantization for High Fidelity Audio Codec | arXiv | 2023 | Argues that PESQ and STOI are insufficient alone for codec quality characterization; calls for combined objective and subjective evaluation |
| [[2403.16973]] | VoiceCraft: Zero-Shot Speech Editing and Text-to-Speech in the Wild | arXiv | 2024 | Shows WER from ASR is an unreliable proxy for perceptual intelligibility; systems with lower WER than ground truth can receive substantially lower human intelligibility ratings |
| [[2305.09636]] | SoundStorm: Efficient Parallel Audio Generation | arXiv | 2023 | Proposes generation time evaluation alongside MOS and SPK-SIM for fast TTS systems |
| [[1712.05884]] | Natural TTS Synthesis by Conditioning WaveNet on Mel Spectrogram Predictions (Tacotron 2) | arXiv | 2017 | Establishes crowd-sourced MOS with English native speakers on Amazon Mechanical Turk as a methodological standard for TTS evaluation |
| [[2411.19842]] | Scaling Transformers for Low-Bitrate High-Quality Speech Coding (TAAE) | arXiv | 2024 | Systematic spectral bias analysis showing power-of-two FFT configurations cause periodic reconstruction artifacts measurable via MUSHRA |
| [[2407.08551]] | MELLE: Autoregressive Speech Synthesis without Vector Quantization | arXiv | 2024 | Comparative evaluation across MOS, WER, SPK-SIM on multiple benchmarks; variational sampling compared with top-p sampling via diversity metrics |
| [[1703.10135]] | Tacotron: Towards End-to-End Speech Synthesis | arXiv | 2017 | 5-scale MOS evaluation on Amazon Mechanical Turk; establishes naturalness MOS as primary quality metric for TTS benchmarking |
| [[2507.16632]] | Step-Audio 2 Technical Report | arXiv | 2025 | Notes existing audio LM benchmarks fail to capture fine-grained paralinguistic comprehension and tool invocation, leaving important dimensions unmeasured |
| [[2106.15561]] | A Survey on Neural Speech Synthesis | arXiv | 2021 | Comprehensive survey of evaluation metrics across MOS, WER, CER, PESQ, STOI, F0-RMSE for all TTS paradigms; provides historical taxonomy |
| [[interspeech-2025-0408]] | Improving User Impression of Spoken Dialogue Systems by Controlling Para-linguistic Expression Based on Intimacy | Interspeech | 2025 | Demonstrates that user impression diverges from naturalness MOS when paralinguistic style is mismatched to social context; intimacy as an additional evaluation dimension |
