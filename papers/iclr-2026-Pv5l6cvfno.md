---
id: "iclr-2026-Pv5l6cvfno"
title: "Human or Machine? A Preliminary Turing Test for Speech-to-Speech Interaction"
authors: ["Xiang Li", "Jiabao Gao", "Sipei Lin", "Xuan Zhou", "Chi Zhang", "Bo Cheng", "Jiale Han", "Benyou Wang"]
organization: "Beijing University of Posts and Telecommunications; The Chinese University of Hong Kong, Shenzhen"
venue: ICLR
venue_type: conference
year: 2026
month: 4
published_date: "2026-04-27"
ingested_date: "2026-10-20"
task: [SCA, evaluation]
architecture: []
conditioning: []
training: [supervised]
model_size: "7B (Qwen2.5-Omni-7B backbone of the judge model)"
codec_used: "none"
datasets_train: ["Own S2S Turing test dialogue corpus (H-M recordings + H-H from DailyTalk, IEMOCAP, MagicData-RAMC and new recordings)"]
datasets_eval: ["Own S2S Turing test dialogue corpus (test split incl. TTS pseudo-human dialogues)", "CosyVoice2-synthesized dialogues", "Fisher", "MultiDialog"]
metrics:
  - name: Turing success rate (judged human, %)
    value: 86.7
    system: human speaker (H-H)
    testset: Turing test, English
  - name: Turing success rate (judged human, %)
    value: 70.0
    system: human speaker (H-H)
    testset: Turing test, Chinese
  - name: Turing success rate (judged human, %)
    value: 30.8
    system: best S2S system (Kimi-K1.5)
    testset: Turing test, English
  - name: Turing success rate (judged human, %)
    value: 37.8
    system: Nari Dia-1.6B pseudo-human (TTS)
    testset: Turing test, English
  - name: human-vs-machine accuracy
    value: 0.7284
    system: human judges
    testset: Turing test (overall)
  - name: human-vs-machine accuracy
    value: 0.6279
    system: Gemini 2.5 Pro (best off-the-shelf AI judge)
    testset: Turing test (overall)
  - name: human-vs-machine accuracy
    value: 0.9605
    system: proposed (ODL + linear judge)
    testset: in-domain test set (overall)
  - name: human-vs-machine accuracy
    value: 0.974
    system: proposed (ODL + linear judge)
    testset: OOD (CosyVoice2, Fisher, MultiDialog)
code_available: true
demo_available: null
url: "https://openreview.net/forum?id=Pv5l6cvfno"
related_concepts: [spoken-language-model, speech-to-speech, subjective-evaluation, evaluation-metrics]
related_papers: ["2503.20215", "2410.21276", "2507.06261", "2502.17239", "2503.01743", "2408.01800", "2503.19786", "2505.09388", "2503.01710", "2412.10117", "2410.17196", "2508.13992", "2410.19168", "2508.18240", "2503.04721", "2503.05085", "2508.09600"]
field_significance:
  level: "moderate"
  type: [empirical-benchmark, evaluation-contribution, dataset-contribution]
generation:
  schema_version: 2
  date: "2026-10-20"
  runtime: "claude-code"
  provider: "anthropic"
  agent: speech-generation-ingest-agent
  model: "claude-opus-5-5"
  commit: "fb8dafe"
---

> [!abstract] ICLR · 2026 · Conference
> **Xiang Li et al.** (Beijing University of Posts and Telecommunications; The Chinese University of Hong Kong, Shenzhen) · [→ Paper](https://openreview.net/forum?id=Pv5l6cvfno) · Demo: ? · Code: ✓
>
> Runs a large-scale Turing test on nine commercial and open speech-to-speech dialogue systems, finds that none is judged human more often than chance, traces the gap to paralinguistics, emotion, and persona rather than semantics, and trains an interpretable judge that separates human from machine dialogue more accurately than human listeners.

## Problem

Evaluations of spoken conversational agents mostly measure whether a system understands audio or solves reasoning and instruction-following tasks, usually with text as the scored output (VoiceBench, MMAU-Pro) or with arena and rubric protocols for specific abilities. None of these asks the more holistic question of whether a speech-in, speech-out system converses like a person in multi-turn interaction. Prior speech Turing tests targeted TTS output in isolation (text-to-speech), not interactive dialogue. There is also no diagnostic account of *why* a system sounds artificial, and no evidence on whether off-the-shelf audio LLMs can act as reliable judges of human-likeness.

## Method

The study has three parts: a dialogue corpus, a gamified human Turing test with fine-grained diagnosis, and an automatic judge.

**Corpus.** Twenty-eight volunteers from ten countries and regions record multi-turn conversations (typically 20 to 60 seconds) with nine speech-to-speech systems (GPT-4o, Gemini 2.5 Pro, Qwen3, Kimi-K1.5, ChatGLM-4.5, Hunyuan-TurboS, Doubao-Pro 1.5, Claude-Sonnet 4, iFLYTEK-Spark) in a professional studio, in Chinese and English across ten DailyDialog-inspired topics. Because pilot runs showed systems disclosing that they were assistants and adopting a passive assistant stance, three elicitation strategies are used: human-guided initiation, role playing via a spoken prompt, and explicit "be more humanlike" prompting. Dialogues where the system disclosed its identity, switched language, or became aggressive are filtered out. A matched human-human subset is drawn from DailyTalk, IEMOCAP, and MagicData-RAMC (filtered to remove interruptions and imbalanced turns) and from new recordings using the same opening utterances. A third, harder "pseudo-human" subset is synthesized with Nari Dia-1.6B and [[2503.01710|Spark-TTS]], voice-cloning the human participants, from GPT-4o-refined or GPT-4o-generated scripts, with 180 to 230 ms inter-turn gaps and background ambience added. Inter-turn timing and loudness are normalized across all dialogues to remove latency and recording cues. The corpus totals 1,486 dialogues (17.7 h): 669 human-machine, 673 human-human, 144 pseudo-human.

**Human Turing test and diagnosis.** A public web game asks players to listen to dialogues in rounds of five and decide whether Speaker B is human; points and a leaderboard encourage participation. By the reported cutoff, 397 participants contributed 2,968 judgments. The primary metric is success rate (fraction judged human), with 0.5 as the pass threshold. Separately, 36 bilingual graduate-student annotators rate each response on an 18-dimension taxonomy grouped into five categories (semantic and pragmatic habits, non-physiological paralinguistics, physiological paralinguistics, mechanical persona, emotional expression) on a 1 to 5 human-likeness scale; three HCI experts with access to the true labels review and revise the ratings (29.44% of labels changed).

**Interpretable judge.** A fused pooling of [[2503.20215|Qwen2.5-Omni]]-7B hidden states (mean of first-step states plus last-token state) is mapped by an Ordinal Discretization Layer (ODL) to 18 latent scores, each turned into an ordinal distribution over the five rating levels via learnable cumulative-link cut-points and trained with ordinal negative log-likelihood against the human ratings. The 18 scores then feed a regularized linear classifier for the human-vs-machine decision, so each prediction decomposes into per-dimension contributions. The judge is trained on human-human and human-machine dialogues only; pseudo-human dialogues are held out for testing.

## Key Results

- **No system passes.** Human-human dialogues reach success rates of 0.87 (English) and 0.70 (Chinese), while every speech-to-speech system stays well below 0.5; the best per-language figure is Kimi-K1.5 in English at 30.8% (Table 9, Figure 4a). The TTS pseudo-human dialogues score higher than most live systems (Nari Dia 37.8% English, Spark-TTS 36.6% Chinese), which the authors read as evidence that interactive systems are limited by more than synthesis quality.
- **Listener experience matters.** Detection accuracy rises from 64.2% for the least AI-familiar listeners to 78.8% for the most familiar, with little variation by gender or education (Figure 4b). Dialogue length shows no significant trend in accuracy (Cochran-Armitage tests, Table 11).
- **Where systems fail.** Crowd ratings put systems near human level on memory consistency, logical coherence, and pronunciation accuracy, but well below humans on rhythm, intonation, stress, fillers, micro-physiological noise, acoustic emotion, sycophancy, and written-style expression (Figure 5). Acoustic emotion scores fall below textual sentiment scores.
- **Off-the-shelf AI judges are unreliable.** Nine audio LLMs average 0.4527 accuracy versus 0.7284 for humans; the best, Gemini 2.5 Pro, reaches 0.6279. Several models (GPT-4o-Audio-Preview, [[2502.17239|Baichuan-Audio]], Phi-4-Multimodal, Qwen2.5-Omni) label almost everything human, SeaLLMs-Audio shows the opposite bias, and Voxtral Mini is near random (Table 2).
- **The interpretable judge outperforms humans.** The ODL judge reaches 0.9605 overall accuracy (0.9306 on held-out pseudo-human dialogues) versus 0.5744 for LoRA-tuned Qwen2.5-Omni and 0.7284 for human listeners (Table 3). On 64-dialogue samples from CosyVoice2-synthesized, Fisher, and MultiDialog data it reaches 0.974 accuracy and 0.9881 ROC-AUC (Table 4). Replacing the ODL with an unconstrained linear layer drops overall accuracy to 0.9233 (Table 17), and training on pre-expert-revision labels drops it to 0.8791 in-domain and 0.8229 out-of-domain (Table 14).

## Novelty Assessment

The contribution is empirical and evaluative. Turing tests of text LLMs and of TTS output exist; extending the protocol to recorded multi-turn speech-to-speech interaction with commercial systems, at a scale of nearly 3,000 judgments, is new, as is pairing the binary verdict with an 18-dimension diagnostic taxonomy. The judge combines a pretrained audio LLM encoder with a classical cumulative-link ordinal regression head and a linear classifier; none of these components is new, but using ordinal attribute scores as an interpretable bottleneck for human-vs-machine detection is a sensible design that the ablations support. The headline judge accuracy should be read with care: the judge learns from dimension ratings that were revised by experts who knew the ground-truth identity, and the out-of-domain sets are each single-class. The interaction protocol (studio recording, normalized gaps, filtered disclosures) controls for confounds but also removes latency and turn-taking from what is tested.

## Field Significance

Moderate — The paper provides a concrete, reproducible protocol and corpus for measuring human-likeness of spoken conversational agents, and a quantified finding that current systems fall short mainly on prosody, disfluency, emotion, and persona rather than on content. It also offers a cautionary data point that general-purpose audio LLMs are poor zero-shot judges of human-likeness, while a judge supervised on attribute ratings can be strong.

## Claims

- **supports:** Current speech-to-speech dialogue systems remain readily distinguishable from humans in multi-turn spoken interaction, even when prompted to role-play and behave humanlike.
  > *Evidence:* Across nine commercial and open systems and 2,968 judgments, every system's success rate stays below the 0.5 pass threshold (best 30.8%, Kimi-K1.5 English), versus 86.7% and 70.0% for human-human dialogues in English and Chinese. *(§4, Figure 4a; §C.2, Table 9)*
- **refines:** The human-likeness gap of spoken conversational agents lies in paralinguistic delivery, emotional expressivity, and conversational persona rather than in semantic coherence or pronunciation.
  > *Evidence:* Crowd ratings on an 18-dimension taxonomy show systems near human level on memory consistency, logical coherence, and pronunciation accuracy, but low on rhythm, intonation, stress, fillers, breath noise, acoustic emotion, sycophancy, and written-style expression; acoustic emotion scores fall below textual sentiment scores. *(§5, Figure 5)*
- **complicates:** Reasoning and understanding benchmark performance does not predict human-likeness of speech-to-speech systems.
  > *Evidence:* For four systems scored on both MMAU-Pro reasoning and the Turing test, the Pearson correlation between reasoning accuracy and Turing pass rate is 0.0456, though the sample is only four models. *(§A, Table 6)*
- **complicates:** Off-the-shelf audio LLMs are unreliable zero-shot judges of whether spoken dialogue is human or machine, often showing strong systematic label bias.
  > *Evidence:* Nine audio LLMs average 0.4527 accuracy against 0.7284 for human listeners; GPT-4o-Audio-Preview labels 96.5% of human-human but only 0.7% of TTS pseudo-human dialogues correctly, and SeaLLMs-Audio shows the reverse bias. *(§6.1, Table 2)*
- **supports:** Supervising an automatic human-likeness judge on ordinal, attribute-level human ratings yields more accurate and interpretable discrimination than direct fine-tuning on binary labels.
  > *Evidence:* The ODL-plus-linear judge reaches 0.9605 accuracy versus 0.5744 for LoRA-tuned Qwen2.5-Omni; replacing the ordinal layer with a plain linear layer lowers accuracy to 0.9233, and using pre-expert-revision ratings lowers it to 0.8791. *(§6.2.2, Table 3; §E.4, Table 17; §D.3, Table 14)*

## Limitations and Open Questions

> [!warning]
> The judge's 18 dimension labels were revised by experts who were given the true human-or-machine identity of each dialogue, and these revisions raise judge accuracy substantially (0.8791 to 0.9605 in-domain). The headline judge results may therefore partly reflect identity information leaking into the attribute labels, and it is unclear how the judge would perform with ratings collected fully blind.

- The out-of-domain checks use 64 dialogues per set, and each set is single-class (all synthetic or all human), so per-set accuracy measures only one error type; ROC-AUC is reported only for the pooled sets.
- Inter-turn gaps are normalized and interrupting human-human dialogues are filtered out, so latency, overlap, and turn-taking, which are central to real-time interaction, are excluded from the test.
- The paper does not describe how voice interaction was accessed for systems known mainly as text LLMs (for example Qwen3, Kimi-K1.5, Claude-Sonnet 4), so the evaluated pipelines (native speech model vs. app-level cascade) are not specified.
- Some systems are evaluated in only one language, and per-cell sample sizes are small; the reasoning-vs-human-likeness correlation rests on four models.
- Success rates depend on the listener population, which the authors note shifts with AI familiarity, so absolute numbers may drift over time.

## Wiki Connections

- [[spoken-language-model|Spoken Language Model]] — evaluates nine off-the-shelf audio LLMs as human-likeness judges on real dialogue audio and builds its own judge on a speech LM encoder.
- [[speech-to-speech|Speech-to-Speech]] — runs a Turing test on multi-turn speech-to-speech dialogue systems and diagnoses where they fail to sound human.
- [[subjective-evaluation|Subjective Evaluation]] — collects 2,968 human human-vs-machine judgments and 18-dimension crowd ratings, analysing listener demographics as a source of bias.
- [[evaluation-metrics|Evaluation Metrics]] — proposes success rate against a 0.5 threshold, a human-likeness taxonomy, and an interpretable automatic judge as human-likeness measures.
- [[2503.20215|Qwen2.5-Omni]] — serves as the encoder backbone of the proposed judge and as a zero-shot and LoRA-tuned baseline judge.
- [[2503.01710|Spark-TTS]] — one of two TTS systems used to synthesize the voice-cloned pseudo-human dialogues.
- [[2412.10117|CosyVoice 2]] — used to synthesize an out-of-domain pseudo-human test set for the judge.
- [[2410.21276|GPT-4o]] — evaluated both as a speech-to-speech dialogue system and, in its audio preview form, as a biased AI judge.
- [[2507.06261|Gemini 2.5]] — evaluated as a dialogue system and is the strongest off-the-shelf AI judge.
- [[2502.17239|Baichuan-Audio]] — evaluated as an AI judge that labels most dialogues human.
- [[2410.17196|VoiceBench]] — contrasted as an understanding-oriented benchmark with text output, unlike this speech-output human-likeness test.
- [[2508.13992|MMAU-Pro]] — its reasoning scores are compared against Turing pass rates to show they are nearly uncorrelated.
