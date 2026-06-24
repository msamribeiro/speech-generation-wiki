---
slug: flow-matching
title: Flow Matching
aliases: [CFM, OT-CFM, conditional flow matching, rectified flow, flow-based TTS]
status: dominant
last_reviewed: 2026-06-24
source_digest_date: 2026-06-24
generation:
  date: 2026-06-24
  stage: render
  mode: full
  agent: speech-generation-render-agent
  model: claude-sonnet-4-6
  commit: 323d530
---

> [!abstract]
> Flow matching trains continuous normalizing flows by directly regressing a vector field that transports noise to speech, providing a simulation-free and numerically stable alternative to score matching. It has displaced diffusion as the dominant continuous-output backbone for TTS and VC since 2023, and is now the standard acoustic decoder in both standalone non-autoregressive systems and hybrid autoregressive LM pipelines. Its combination of simple training, straight-line generation trajectories, and compatibility with aggressive inference acceleration makes it the central method in the current TTS landscape.

## Current Assessment

Flow matching has become the organising principle of modern continuous-output speech synthesis. The field has converged on two deployment patterns: pure non-autoregressive systems that apply flow matching directly to mel spectrograms (E2 TTS, F5-TTS, StableVC), and hybrid pipelines in which an autoregressive language model generates discrete semantic tokens while a flow-matching decoder converts those tokens to waveforms (CosyVoice 2, Vevo, MoonCast). The hybrid pattern has become increasingly dominant and now accounts for the majority of frontier systems, appearing in six of the fourteen papers in this corpus. It is the architecture of choice when speaker style control and streaming inference are both required, at the cost of sequential decoding latency from the AR stage.

The speed advantage over diffusion is well established: Optimal Transport paths produce straighter generation trajectories that converge at 10–32 ODE steps rather than hundreds, and post-training acceleration has pushed frontier systems to 1–4 steps without measurable quality loss in controlled evaluations. Post-training alignment is the most active current frontier: DPO-FM, the first systematic formulation of velocity-space preference optimization for flow-matching models, demonstrated intelligibility gains without reward model sampling ([[2025.acl-long.598]]); CosyVoice 2 ([[2412.10117|CosyVoice 2]]) simultaneously demonstrated differentiable ASR reward signals as a complementary approach. Both are 2025 findings that have not yet been independently replicated and should be treated as frontier probes rather than established practice.

## Major Claims

### Strongly Supported

- **Flow matching enables simulation-free training of continuous normalizing flows by regressing conditional vector fields, providing a tractable and stable alternative to score matching for generative modeling.**
  Evidence: [[2210.02747]], [[2312.15821|Audiobox]], [[2406.18009|E2 TTS]].
  Caveat: The theoretical guarantee holds for Gaussian conditional paths; extensions to non-Gaussian paths are not formally established. Concurrent independent work (Rectified Flow, Stochastic Interpolants) arrived at equivalent objectives; relative priority is contested.

- **Optimal Transport paths in flow matching produce straighter generation trajectories than diffusion paths, reducing the number of ODE function evaluations needed to achieve equivalent sample quality.**
  Evidence: [[2210.02747]], [[2412.04724|StableVC]], [[2025.acl-long.313|F5-TTS]].
  Caveat: OT optimality is at the conditional level between paired Gaussians; marginal-level optimality is not theoretically guaranteed, and practical NFE requirements depend on the ODE solver and target distribution complexity.

- **Flow matching surpasses diffusion-based generative models in inference efficiency while maintaining or exceeding sample quality, making it the preferred backbone for continuous-output speech synthesis.**
  Evidence: [[2210.02747]], [[2412.04724|StableVC]], [[2025.acl-long.313|F5-TTS]].
  Caveat: The quality advantage over diffusion is established primarily on WER and speaker similarity metrics; perceptual naturalness differences are smaller and depend on the NFE budget and solver choice.

- **Flow-matching speech models admit substantial inference acceleration — to as few as 1–4 steps — through post-training solver optimisation, distillation, or learned priors, without requiring architectural changes to the base model.**
  Evidence: [[2312.15821|Audiobox]], [[2406.05551|ARDiT]], [[2025.acl-long.1043|OZSpeech]].
  Caveat: The three acceleration mechanisms (learned ODE solvers, distillation, learned priors) each introduce distinct quality-speed tradeoffs and are not directly comparable. Distillation-based naturalness improvements over the teacher require independent replication.

- **The classifier-free guidance conditioning mechanism from diffusion modeling transfers to flow-matching TTS systems and is sufficient for zero-shot speaker conditioning without discriminative components.**
  Evidence: [[2207.12598]], [[2406.18009|E2 TTS]], [[2407.05407|CosyVoice]], [[2412.10117|CosyVoice 2]].
  Caveat: CFG doubles inference compute per step; in low-NFE regimes this cost is proportionally larger. Guidance strength has no principled selection criterion and is tuned empirically per system.

### Emerging

- **Flow-matching TTS models can learn text-speech alignment implicitly from character sequences and filler tokens, without phoneme-level duration supervision, grapheme-to-phoneme converters, or explicit aligner components.**
  Evidence: [[2406.18009|E2 TTS]], [[2025.acl-long.313|F5-TTS]].
  Caveat: Implicit alignment is prone to systematic failure modes in languages with complex scripts, and requires modality-specific text preprocessing (ConvNeXt refinement in F5-TTS) to be reliable in practice.

- **Explicit phoneme alignment supervision in non-autoregressive TTS imposes a naturalness ceiling: removing alignment constraints and training jointly on character sequences enables flow-matching models to exceed the naturalness of alignment-supervised baselines.**
  Evidence: [[2406.18009|E2 TTS]], [[2025.acl-long.313|F5-TTS]].
  Caveat: The advantage is demonstrated on reading-style English benchmarks; whether it holds for expressive or spontaneous speech is untested. Both supporting papers share the Voicebox-lineage architecture, and independent confirmation from architecturally distinct systems is absent.

- **Non-autoregressive flow-matching TTS achieves competitive or superior intelligibility compared to autoregressive codec-LM systems at training scales of 100K hours or more, while generating speech at substantially lower real-time factors.**
  Evidence: [[2025.acl-long.313|F5-TTS]], [[2412.04724|StableVC]].
  Caveat: The speed advantage is most pronounced for utterance-level generation. AR systems retain a measurable lead on speaker similarity, and AR systems show stronger style imitation at the cost of higher WER.

- **Non-uniform step scheduling at inference — specifically densifying early ODE integration steps — improves both speech fidelity and speaker similarity in flow-matching TTS without any retraining.**
  Evidence: [[2025.acl-long.313|F5-TTS]].
  Caveat: Sway Sampling has been demonstrated on one system (F5-TTS/E2 TTS) with a single architecture family. Theoretical justification for why early steps are more informative in speech flow is not yet established.

- **DPO objectives can be formulated directly in the flow-matching velocity space, enabling post-training intelligibility improvements without explicit reward models or reference speech sampling.**
  Evidence: [[2025.acl-long.598]].
  Caveat: Demonstrated on a single hybrid AR+FM system; applicability to pure flow-matching architectures without an AR stage is untested. The relationship between velocity-space alignment and perceptual naturalness improvements is not characterized.

- **Flow-matching decoders applied as post-quantization refinement stages can substantially recover audio quality lost during discrete speech tokenization, enabling modular integration into codec-LM pipelines without retraining the language model.**
  Evidence: [[2502.17239|Baichuan-Audio]].
  Caveat: Quality recovery is measured primarily on UTMOS and WER; speaker similarity and subjective naturalness MOS comparisons against dedicated TTS systems are absent from the supporting paper.

- **Self-supervised pretraining on large unlabeled audio corpora measurably improves the generalisation of flow-matching speech models to out-of-domain speakers and acoustic conditions.**
  Evidence: [[2312.15821|Audiobox]], [[2406.18009|E2 TTS]].
  Caveat: Both supporting papers use Voicebox-lineage architectures; the benefit may partly reflect the specific SSL objective rather than a general property of flow-matching training. Pretraining gains are most pronounced on out-of-domain test sets.

- **Hybrid architectures that separate autoregressive style token generation from flow-matching acoustic decoding achieve stronger decoupling of style and timbre attributes than single-stage flow-matching systems conditioned in-context.**
  Evidence: [[2502.07243|Vevo]], [[2412.10117|CosyVoice 2]].
  Caveat: The decoupling advantage comes at the cost of sequential AR decoding latency, making these systems less suitable for real-time applications. Evaluation relies on automatic speaker similarity metrics whose reliability for this specific task is not independently validated.

- **Standard OT-CFM flow matching for mel-spectrogram TTS is data-hungry, requiring tens of thousands of hours to achieve competitive intelligibility; learned prior variants reduce this requirement by orders of magnitude.**
  Evidence: [[2025.acl-long.1043|OZSpeech]].
  Caveat: The data reduction claim rests on a single comparison between systems that differ in codec type and architecture in addition to the prior design. The naturalness cost of the learned prior (UTMOS 3.15 vs. 3.76 for F5-TTS) is not accounted for in the efficiency framing.

## Method Families

**Pure Flow-Matching Systems.** The clearest expression of the flow-matching paradigm for speech, this family applies the CFM objective directly over mel spectrograms with no autoregressive stage. The theoretical foundation is the conditional flow matching framework ([[2210.02747]]), which established simulation-free training via conditional vector field regression and the efficiency gains of Optimal Transport paths. Applied to speech, the Voicebox infilling objective (inherited by [[2406.18009|E2 TTS]] and then [[2025.acl-long.313|F5-TTS]]) enables zero-shot TTS through masked spectrogram infilling conditioned on a reference audio prompt. The central architectural debate within this family has been whether a flat U-Net transformer or a DiT with ConvNeXt text-refinement handles text-speech alignment more reliably: E2 TTS showed that a flat design could learn alignment implicitly but exhibited a persistent Mandarin failure rate that the ConvNeXt branch in F5-TTS resolved without any alignment supervision. [[2412.04724|StableVC]] extended pure FM to voice conversion, demonstrating that flow matching surpasses diffusion baselines in both quality and inference speed in the VC setting. This family generates at sub-linear RTF scaling with utterance length and is the preferred choice when real-time latency is a constraint.

**Autoregressive LM + Flow-Matching Acoustic Model.** The dominant pipeline pattern for frontier systems as of 2025, this family separates content and style modeling (autoregressive LM on discrete tokens) from fine-grained acoustic synthesis (flow-matching acoustic decoder). [[2407.05407|CosyVoice]] introduced the architecture by conditioning an OT-CFM acoustic model on supervised ASR-encoder speech tokens generated autoregressively by a language model. [[2412.10117|CosyVoice 2]] refined the acoustic stage with chunk-aware causal masking — training a single FM model on four masking patterns simultaneously — achieving near-lossless streaming and offline quality parity in a single deployed model. [[2502.07243|Vevo]] used VQ-VAE bottleneck disentanglement to achieve annotation-free style and accent imitation without parallel corpora or style labels, at the cost of sequential AR decoding latency. [[2503.14345|MoonCast]] pushed the architecture to long-form multi-speaker podcast generation at 40,000-token context windows with chunk-wise causal FM. The AR stage provides prosody and style control that in-context conditioning alone cannot match; the FM stage handles acoustic detail at low per-step cost. The latency of the AR stage is the primary limitation for real-time deployment.

**Accelerated / Single-Step Flow Matching.** A focused subfield targeting NFE reduction to 1–4 steps via three distinct mechanisms. [[2312.15821|Audiobox]] demonstrated post-training ODE reparameterisation (Bespoke Solvers), achieving 25x NFE reduction without measurable quality loss at 4 steps. [[2406.05551|ARDiT]] demonstrated that distribution matching distillation (DMD) can improve perceptual naturalness over the multi-step teacher, not merely match it, though this finding requires replication. [[2025.acl-long.1043|OZSpeech]] took a different path entirely: replacing the Gaussian noise prior with a learned codec-based prior that absorbs the transport complexity, enabling NFE=1 without any distillation. These mechanisms are not equivalent — distillation-based models retain higher naturalness (UTMOS 3.76 range) while the learned-prior approach prioritises intelligibility (WER 0.05% but UTMOS 3.15). The optimal choice depends on whether naturalness or intelligibility is the binding constraint for a given application.

**Classifier-Free Guidance (Predecessor).** The CFG paper ([[2207.12598]]) established the conditioning mechanism adopted universally across FM-TTS: jointly training a single model for conditional and unconditional generation via random conditioning dropout, then interpolating between the two estimates at inference. All pure FM and hybrid FM-TTS systems in this corpus use this mechanism. The paper's experiments are entirely on image generation (ImageNet); the transfer to flow-matching velocity field estimation was not formally analyzed. Subsequent work treats the transfer as empirically sufficient, which it appears to be, though the optimal guidance strength and dropout fraction differ from image settings.

## How to Interpret Older and Newer Evidence

The foundational CFM paper ([[2210.02747]], 2022) established the theoretical machinery — simulation-free CNF training, OT paths, gradient equivalence with score matching — and remains directly valid for understanding the training objective. The CFG paper ([[2207.12598]], 2022) established the conditioning mechanism. Neither paper contains speech experiments; they are foundational context for the theory but not direct evidence for speech quality claims.

[[2312.15821|Audiobox]] (2023) is the earliest paper in this corpus to apply flow matching at scale to speech generation. Its claims about Bespoke Solver acceleration (25x NFE reduction) and multi-domain generalisation via SSL pretraining are influential, but the Joint-CLAP evaluation metric it introduces has not been independently validated and should not be treated as a benchmark.

The 2024 papers ([[2406.18009|E2 TTS]], [[2025.acl-long.313|F5-TTS]], [[2412.04724|StableVC]], [[2412.10117|CosyVoice 2]], [[2406.05551|ARDiT]], [[2407.05407|CosyVoice]]) constitute the main evidence base for the strongly-supported claims and the core of current best practice. These are the papers to consult for current methodology decisions.

The 2025 papers ([[2025.acl-long.598]], [[2025.acl-long.1043|OZSpeech]], [[2502.07243|Vevo]], [[2502.17239|Baichuan-Audio]], [[2503.14345|MoonCast]]) are active frontier probes. DPO-FM velocity-space alignment ([[2025.acl-long.598]]) and learned-prior single-step synthesis ([[2025.acl-long.1043|OZSpeech]]) each have only a single supporting paper in this corpus. Their importance depends on replication and broader adoption. Treat them as directions to watch rather than settled evidence.

## Current Tensions

**Mel spectrogram vs. codec-token representation for pure FM systems.** Non-autoregressive FM systems operating on mel spectrograms achieve low RTF and simple training pipelines but face a speaker similarity ceiling: [[2025.acl-long.313|F5-TTS]] reaches SPK-SIM 0.66–0.67 on English, well below Seed-TTS DiT (0.790). Codec-based approaches such as [[2025.acl-long.1043|OZSpeech]] improve intelligibility through better semantic grounding (WER 0.05%) but trade perceptual naturalness (UTMOS 3.15 vs. 3.76 for F5-TTS). The trend notes in this corpus identify the mel-spectrogram sequence-length bottleneck as an increasingly recognised limitation, suggesting a gradual shift toward codec-native FM pipelines, but no codec-native pure FM system with competitive naturalness metrics has yet resolved this tradeoff.

**NAR speed vs. AR style control.** Pure NAR FM systems generate at RTF 0.15 and scale sub-linearly with utterance length. AR systems such as [[2502.07243|Vevo]] achieve stronger style and emotion imitation at the cost of significantly higher WER (12.07% vs. 9.41% for Voicebox on the same set) and sequential decoding latency. The hybrid AR+FM pattern attempts to capture both properties, but the AR stage's latency disqualifies hybrid systems from real-time conversational TTS. The optimal architecture depends on the application: latency-critical streaming favours pure NAR FM; style-sensitive long-form generation favours hybrid pipelines.

**Data efficiency: learned priors vs. scale.** [[2025.acl-long.1043|OZSpeech]] achieves WER 0.05% on LibriSpeech test-clean from 500 hours of training data via a learned codec prior, compared to [[2025.acl-long.313|F5-TTS]]'s WER 0.24% at 95K hours. If this generalises, it would substantially reduce production data requirements. However, the two systems differ in codec, architecture, and naturalness profile, and the comparison has not been independently reproduced. The naturalness cost (UTMOS 3.15 vs. 3.76) is not reflected in the WER framing.

**Inference-time step scheduling composability.** Sway Sampling ([[2025.acl-long.313|F5-TTS]]) and Bespoke Solvers ([[2312.15821|Audiobox]]) both improve FM TTS inference without retraining. Whether these techniques compose — and whether their gains stack when applied to the same system — is an open empirical question.

## Open Questions

- Speaker similarity in zero-shot FM-TTS consistently lags behind speaker verification ground truth, particularly for out-of-domain speakers. Is this a fundamental limitation of the mel-spectrogram representation, the OT-CFM objective, or the scale of training data?
- The naturalness-intelligibility tradeoff between learned-prior systems (OZSpeech) and standard OT-CFM systems (F5-TTS) raises the question of whether a single FM architecture can simultaneously achieve top-tier performance on both axes, or whether codec choice forces a permanent tradeoff.
- Most pure FM-TTS papers train on clean audiobook-style English data. To what extent do current flow-matching architectures generalise to spontaneous, noisy, or code-switched speech?
- Inference-time step scheduling (Sway Sampling) and post-training solver optimisation (Bespoke Solvers) both improve FM TTS without retraining. Are these techniques composable, and do their gains stack when applied together?
- The DPO-FM velocity-space alignment approach is demonstrated on a single hybrid system. Can preference-based alignment improve pure flow-matching systems, without an AR stage, at comparable cost and scale?

## Recommended Reader Path

1. [[2210.02747]] — Start here for the theory: simulation-free CNF training, OT paths, and gradient equivalence with score matching. Experiments are on images, but the mathematical framework transfers directly to speech.
2. [[2406.18009|E2 TTS]] — The clearest demonstration that alignment-free character-level training can match or exceed phoneme-supervised FM-TTS. Read for both the infilling objective and its limitations (convergence speed, duration model dependency).
3. [[2025.acl-long.313|F5-TTS]] — Read directly after E2 TTS: the ConvNeXt text-refinement fix for alignment failures and Sway Sampling together define the current best-practice pure NAR FM-TTS recipe. Also introduces the LibriSpeech-PC benchmark.
4. [[2412.04724|StableVC]] — Best current evidence for flow matching in VC: surpasses diffusion baselines in both quality and speed, with ablations on timbre-style disentanglement via dual cross-attention.
5. [[2412.10117|CosyVoice 2]] — Read for the hybrid AR+FM pattern at production scale: chunk-aware causal FM for streaming, streaming/offline quality parity, and differentiable ASR reward optimization for post-training alignment.
6. [[2025.acl-long.1043|OZSpeech]] — Frontier probe: learned-prior single-step synthesis with competitive intelligibility from 500 hours of training. Read to understand the data-efficiency vs. naturalness tradeoff and how the learned prior differs mechanistically from distillation.
7. [[2025.acl-long.598]] — Read last for DPO-FM velocity-space preference alignment: the first systematic post-training alignment formulation for flow-matching TTS, with cross-lingual generalisation evidence.

---

_This page is generated from `wiki/_claims/flow-matching.yaml` (digest date: 2026-06-24).
For the full paper inventory, claim support matrix, and reassessment queue, see
[[evidence/flow-matching]]._
