---
id: "icml-2026-hgKQemfVbS"
title: "AgentSteerTTS: A Multi-Agent Closed-Loop Framework for Composite-Instruction Text-to-Speech"
authors: ["Bin Kang", "Shaoguo Wen", "Yang Fan", "Shunlong Wu", "Junjie Wang", "Yulin Li", "Junzhi Zhao", "Junle Wang", "Zhuotao Tian"]
organization: "University of Chinese Academy of Sciences / Shenzhen Loop Area Institute / Tencent Turinglab / Tsinghua University / Southwest Jiaotong University"
venue: ICML
venue_type: conference
year: 2026
month: 7
published_date: "2026-07-06"
conference_date: "2026-07-06"
ingested_date: "2026-11-14"
task: [TTS]
architecture: []
conditioning: [text-conditioned, speaker-conditioned, emotion-conditioned, instruction-conditioned, prompt-conditioned]
training: [supervised, fine-tuning]
model_size: "not reported"
codec_used: "none"
datasets_train: ["ESD", "MSP-Podcast"]
datasets_eval: ["ESD", "in-house fine-grained composite-instruction benchmark (14.18 h)"]
metrics:
  - name: WER
    value: 7.12
    system: AgentSteerTTS (Qwen3 slow-loop backend)
    testset: ESD (Table 1)
  - name: CER
    value: 3.08
    system: AgentSteerTTS (Qwen3 slow-loop backend)
    testset: ESD (Table 1)
  - name: SPK-SIM
    value: 0.861
    system: AgentSteerTTS (Qwen3 slow-loop backend), SECS
    testset: ESD (Table 1)
  - name: MOS
    value: 4.52
    system: AgentSteerTTS (Qwen3 slow-loop backend), SNMOS naturalness
    testset: ESD (Table 1)
  - name: SMOS
    value: 4.31
    system: AgentSteerTTS (Qwen3 slow-loop backend), SSMOS
    testset: ESD (Table 1)
  - name: SPK-SIM
    value: 0.841
    system: AgentSteerTTS (Gemini3 slow-loop backend), S-SIM
    testset: composite-instruction benchmark (Table 2)
  - name: WER
    value: 1.34
    system: AgentSteerTTS (Gemini3 slow-loop backend)
    testset: composite-instruction benchmark (Table 2)
  - name: SMOS
    value: 4.28
    system: AgentSteerTTS (Gemini3 slow-loop backend)
    testset: composite-instruction benchmark (Table 2)
  - name: MOS
    value: 4.38
    system: AgentSteerTTS (Gemini3 slow-loop backend), QMOS quality
    testset: composite-instruction benchmark (Table 2)
code_available: false
demo_available: true
url: "https://openreview.net/forum?id=hgKQemfVbS"
url_preprint: "https://arxiv.org/abs/2605.17583"
related_concepts: [instruction-conditioned-tts, emotion-synthesis, disentanglement, subjective-evaluation]
related_papers: ["2506.21619", "2502.05512", "2412.10117", "2407.05407", "2503.01710", "2025.acl-long.313", "2301.02111", "2312.15185", "2601.03888", "2505.17589", "2504.12867", "2506.16381", "interspeech-2025-1394", "2403.03100", "2025.findings-acl.1051", "2305.11000", "2306.12925", "2407.10759", "2025.acl-long.682"]
field_significance:
  level: "moderate"
  type: [architectural-novelty]
generation:
  schema_version: 2
  date: "2026-11-14"
  runtime: "claude-code"
  provider: "anthropic"
  agent: speech-generation-ingest-agent
  model: "claude-opus-5-5"
  commit: "950d8a6"
---

> [!abstract] ICML · 2026 · Conference
> **Bin Kang et al.** (University of Chinese Academy of Sciences / Shenzhen Loop Area Institute / Tencent Turinglab / Tsinghua University / Southwest Jiaotong University) · [→ Paper](https://openreview.net/forum?id=hgKQemfVbS) · [→ Preprint](https://arxiv.org/abs/2605.17583) · Demo: ✓ · Code: ✗
>
> AgentSteerTTS treats composite emotion instructions such as "Happy but slightly Arrogant" as a closed-loop control problem: it separates speaker and emotion latents adversarially, anchors the instruction to retrieved expressive audio prototypes, and corrects emotion intensity at inference time with a latent gradient step plus a multimodal-LLM critic.

## Problem

Instruction-following expressive TTS systems produce fluent speech but often miss composite instructions that combine two or more emotional or stylistic attributes. A pilot study with IndexTTS2 on six composite intents shows the failure is systematic rather than random (Figure 1, Figure 2). Target emotion dimensions are under-expressed, non-target dimensions receive spurious activation, and several composites almost never reach a joint recognition threshold. The authors trace this to two causes (§2). First, a deterministic instruction-to-acoustics mapping averages over a multi-modal set of valid realizations, so the output drifts toward a diluted or neutral rendering. Second, speaker timbre and emotional prosody share acoustic cues, so pushing harder on emotion degrades speaker similarity (composite fidelity and speaker similarity correlate negatively, Figure 3).

## Method

The framework has three stages, which the paper calls agents: decouple, anchor, refine (§3).

**Adversarial Disentanglement Module (ADM, §3.1).** An identity encoder and an emotion encoder (multi-layer Transformers, hidden size 512, Appendix B.1) map a reference utterance to utterance-level latents z_id and z_emo. A decoder must reconstruct the mel-spectrogram from their concatenation (L1 loss). A Frobenius penalty on the batch cross-covariance between the two latents discourages shared information. Two discriminators behind gradient reversal layers try to predict speaker from z_emo and emotion from z_id, which pushes each encoder to drop the other factor. The loss is the weighted sum of reconstruction, adversarial and cross-covariance terms. The stated goal is speaker-emotion decoupling only, not full factorization of channel, accent or age.

**Dual-Stream Anchoring Controller (DAC, §3.2).** A roughly 100-hour library of high-expressivity clips (role-oriented Chinese speakers, Appendix B.2), each labelled with a coarse emotion and an MLLM-written natural-language description refined by human listening, serves as a set of acoustic prototypes. A Retrieval Agent rewrites the user instruction into several cue-focused queries and retrieves the prototype whose description has the highest text-embedding cosine similarity. A sliding-window "perceptual cropping" step keeps the slice most consistent with the instruction. A General Controller Agent parses the instruction into per-emotion intensity weights that sum to at most 1. The acoustic stream encodes the cropped prototype and the target speaker's reference with a W2V-BERT-based emotion encoder and interpolates the speaker's neutral state toward the prototype. The text stream builds a symbolic control embedding from speaker-consistent per-emotion prototypes weighted by the intensity vector. The final emotion latent is the text embedding plus the acoustic mix scaled by the remaining weight mass, with a confidence-gated fusion weight (Figure 6).

**Fast-Slow Feedback Agent (§3.3).** The Fast Agent is a three-layer MLP, the Latent Consistency Predictor, that maps the intermediate mel output to an embedding and compares it with the text embedding of the instruction. A scalar α scales the injected emotion latent and is updated by gradient descent on the cosine distance before vocoding (default T = 2 steps, γ = 5 × 10⁻³). The Slow Agent vocodes the result, and a Supervisor Agent (a multimodal LLM; Gemini3 or Qwen3 backends are compared) writes a natural-language critique. "Too weak/strong" triggers another intensity calibration; "wrong emotion type" triggers re-retrieval. The paper states the slow loop adds at most 200 ms and the default fast loop about 121 ms (§4.3).

![Figure 4. Overview of the AGENTSTEERTTS architecture. (a) an Adversarial Disentanglement Module that utilizes gradient reversal layers to construct orthogonal speaker and emotion subspaces; (b) a Dual-Stream Anchoring Controller that aligns latent features with retrieved acoustic prototypes via consistency calibration; and (c) a Fast-Slow Feedback Loop that dynamically refines the generation process during inference.](assets/icml-2026-hgKQemfVbS/figure-4.png)

Training has two stages: a "foundational TTS pre-training" stage, then joint optimization of ADM and DAC (Appendix B.1). The model is fine-tuned on about 700 hours of emotional speech from ESD and MSP-Podcast with AdamW (2 × 10⁻⁴, 3k warm-up steps) on 8 H20 GPUs. The paper does not describe the acoustic decoder's architecture, the vocoder, the text front end, or the parameter count, so the `architecture` field is left empty.

## Key Results

Evaluation covers 45,022 utterances (43.25 h, 209 speakers, 7 emotions): the ESD test set (29.07 h, 20 speakers, 5 emotions) and an in-house fine-grained composite-instruction set (14.18 h) (§4.1). Emotion alignment (E-SIM) is measured with emotion2vec; the paper reports several MOS variants with confidence intervals but no listening-test protocol.

**ESD (Table 1).** With the Qwen3 slow-loop backend, AgentSteerTTS has the highest ESMOS (4.42), SNMOS (4.52) and SSMOS (4.31) among the synthesized systems and the lowest MCD (5.815). It does not lead on every metric. UMETTS (FastSpeech) has higher speaker similarity (SECS 0.896 vs. 0.861) and marginally lower CER (3.07 vs. 3.08); AgentSteerTTS has lower WER (7.12 vs. 7.35). The ESD baselines (Emotional-TTS, EmoSpeech, GenerSpeech, two UMETTS variants) are older label-, prompt- or audio-conditioned systems, not recent instruction-following TTS.

**Composite-instruction benchmark (Table 2).** The Gemini3-backend system reaches E-SIM 0.955 against 0.864 for IndexTTS2 and 0.748 for CosyVoice2. It also has the highest S-SIM (0.841 vs. 0.825 for CosyVoice2), the lowest WER (1.34 vs. 1.81 for IndexTTS2) and the highest QMOS (4.38 vs. 4.35 for CosyVoice2). It is not best on the remaining MOS columns. Its instruction-following DMOS is 3.82, below every baseline except VALL-E (CosyVoice2 4.31); its SMOS (4.28) is below CosyVoice2 (4.31); its PMOS (4.40) is below IndexTTS2 (4.42). The authors attribute the low DMOS to a naturalness-expressiveness trade-off. In this table, the baseline DMOS and SMOS values are identical in every row (verified against the PDF), which the paper does not explain.

The comparison is not retrieval-free: the full system uses the prototype library, which the authors acknowledge (Table 2 caption, §4.2). The text-only variant in Table 3 still reaches E-SIM 0.910 and S-SIM 0.836, above IndexTTS2's 0.864 and 0.823. Swapping the critic from Gemini3 to Qwen3 lowers E-SIM from 0.955 to 0.921 on the composite set, but the Qwen3 backend is the better of the two on ESD.

**Ablations (Table 3).** Removing ADM raises timbre drift (ΔS-SIM, lower is better) from 0.021 to 0.048, non-target leakage from 0.14 to 0.22, and lowers composite success rate (CSR) from 0.78 to 0.61, while E-SIM barely moves (0.955 to 0.948). Text-only conditioning without prototype retrieval gives the largest CSR loss (0.49). Removing the fast or slow loop lowers CSR to 0.70 or 0.67 with little E-SIM change. A linear probe (Appendix B.2) shows speaker predictability from z_emo falling from 97.5% to 64.4% after ADM, while emotion predictability from z_emo also falls (94.2% to 70.8%). Shrinking the prototype library from 100 h to 25 h lowers E-SIM to 0.912 and CSR to 0.62 (§4.3).

## Novelty Assessment

The individual components are known: gradient-reversal adversarial disentanglement with a decorrelation penalty, retrieval-augmented reference conditioning, latent interpolation between neutral and emotional embeddings, and LLM self-critique loops. The new part is combining them into a closed loop for composite-emotion instructions, with two pieces that are specific to this paper: the differentiable Latent Consistency Predictor that tunes emotion intensity by gradient steps on a scaling factor before vocoding, and an MLLM critic whose verbal feedback selects between intensity recalibration and re-retrieval. The ADM adds training objectives on top of a TTS model the paper does not describe, which counts as architectural novelty here. The "multi-agent" framing labels functional modules as agents; only the retrieval rewriter and the supervisor are LLM-driven.

The empirical case is weaker than the architecture. The headline comparison is on an in-house benchmark that is not yet released. The system has access to a 100-hour curated prototype library that the baselines do not. The decoder and vocoder are unspecified, so the comparison mixes backbone quality with control quality. The composite-instruction gains are clearest on emotion2vec similarity and CSR, both of which are close to the objectives the fast loop and retrieval step optimize. Human instruction-following ratings (DMOS) favour the baselines.

## Field Significance

Moderate. The paper gives a clear diagnosis of composite-instruction failure in expressive TTS, namely target-attribute suppression plus non-target leakage, and a speaker-emotion trade-off, and shows that explicit disentanglement, audio anchoring and inference-time intensity correction each raise composite success rate. It can serve as a reference design for test-time control loops in instruction-conditioned TTS, but its gains rest on an unreleased benchmark, an automatic emotion metric and an unspecified backbone, and human instruction-following ratings do not confirm them.

## Claims

- **supports:** Text-only instruction conditioning tends to collapse composite emotion requests toward diluted or neutral realizations, and anchoring generation on retrieved expressive audio prototypes reduces this collapse.
  > *Evidence:* Removing prototype retrieval (text-only conditioning) drops composite success rate from 0.78 to 0.49 and E-SIM from 0.955 to 0.910; Figure 10 shows text-only outputs clustering near a neutral compromise zone while the full system concentrates in the target region. *(§4.3, Table 3; Appendix A.2, Figure 10)*
- **supports:** Explicit adversarial speaker-emotion disentanglement reduces timbre drift and non-target emotion leakage when TTS is pushed toward strong expressive targets.
  > *Evidence:* Removing the gradient-reversal plus cross-covariance module raises speaker-similarity drift from 0.021 to 0.048 and leakage from 0.14 to 0.22, and lowers composite success rate from 0.78 to 0.61; a linear probe shows speaker predictability from the emotion latent falling from 97.5% to 64.4%. *(§4.3, Table 3; Appendix B.2)*
- **complicates:** Disentangling speaker from emotion representations also removes some emotion information, so leakage reduction is not free.
  > *Evidence:* After adversarial disentanglement, emotion predictability from the emotion latent falls from 94.2% to 70.8% in the linear-probe test, alongside the drop in speaker leakage. *(Appendix B.2)*
- **supports:** Inference-time feedback that recalibrates emotion intensity improves the reliability of composite-instruction control more than its average alignment.
  > *Evidence:* Removing the fast loop or the critique-based slow loop changes E-SIM only from 0.955 to 0.951 or 0.949 but lowers composite success rate from 0.78 to 0.70 or 0.67; two latent gradient steps raise CSR from 0.70 to 0.78 at about 121 ms latency. *(§4.3, Table 3, Figure 7)*
- **complicates:** Gains in automatic emotion-alignment scores for composite instructions do not necessarily translate into higher human ratings of instruction following.
  > *Evidence:* The system with the highest E-SIM (0.955) receives a DMOS of 3.82, below CosyVoice2 (4.31), IndexTTS2 (4.24) and most other baselines; the authors attribute this to a naturalness-expressiveness trade-off under strict composite control. *(§4.2, Table 2)*

## Limitations and Open Questions

> [!warning]
> The headline composite-instruction results use an in-house benchmark that is not yet released, a curated 100-hour prototype library that the baselines do not use, and an unspecified decoder and vocoder. Human instruction-following ratings (DMOS) favour most baselines, so the composite-control advantage rests mainly on automatic emotion2vec similarity and composite success rate.

- The paper does not describe the listening tests behind its MOS variants (rater count, pool, protocol), and the baseline DMOS and SMOS columns in Table 2 are identical in every row.
- ESD is in both the fine-tuning data and the evaluation set; the paper does not report whether test speakers or utterances were held out, and it makes no unseen-speaker claim.
- The slow loop depends on external commercial or open MLLM critics; the choice of critic changes composite E-SIM by 0.034, and the authors list reducing this dependence as future work (§6).
- Retrieval quality and library coverage limit performance, especially for rare composite styles; the prototype library covers Chinese role-oriented speakers, and the paper does not report language-specific results (§6, Appendix B.2).
- ADM targets only speaker-emotion coupling; channel, accent and age variation may remain in the latents (§6).
- Code and checkpoints are announced for release under licensing constraints but were not available at publication (Impact Statement).

## Wiki Connections

- [[instruction-conditioned-tts|Instruction-Conditioned TTS]] — Addresses composite natural-language emotion instructions with retrieval anchoring and critique-driven correction, and evaluates instruction following with DMOS.
- [[emotion-synthesis|Emotion Synthesis]] — Targets mixed-emotion rendering with continuous intensity control, evaluated on ESD and a composite-emotion benchmark.
- [[disentanglement|Disentanglement]] — Uses gradient-reversal discriminators and a cross-covariance penalty to separate speaker and emotion latents, with ablation and probe evidence.
- [[subjective-evaluation|Subjective Evaluation]] — Reports several MOS variants (DMOS, SMOS, PMOS, QMOS, ESMOS, SNMOS, SSMOS) whose instruction-following scores diverge from automatic emotion similarity.
- [[2506.21619|IndexTTS2]] — Model used for the pilot study of composite-instruction failure and the strongest emotion-alignment baseline on the composite benchmark.
- [[2412.10117|CosyVoice 2]] — Baseline on the composite benchmark that leads on DMOS and SMOS while trailing on E-SIM.
- [[2502.05512|IndexTTS]] — Baseline on the composite benchmark, evaluated though not cited in the reference list.
- [[2503.01710|Spark-TTS]] — LLM-based TTS baseline on the composite benchmark.
- [[2025.acl-long.313|F5-TTS]] — Flow-matching baseline on the composite benchmark.
- [[2407.05407|CosyVoice]] — Baseline on the composite benchmark.
- [[2312.15185|emotion2vec]] — Provides the emotion embeddings used for the E-SIM metric.
- [[2506.16381|InstructTTSEval]] — Cited as evidence that complex natural-language instruction following remains difficult for TTS systems.
