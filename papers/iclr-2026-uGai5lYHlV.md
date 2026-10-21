---
id: "iclr-2026-uGai5lYHlV"
title: "TTSDS2: Resources and Benchmark for Evaluating Human-Quality Text to Speech Systems"
authors: ["Christoph Minixhofer", "Ondrej Klejch", "Peter Bell"]
organization: "Centre for Speech Technology Research, University of Edinburgh"
venue: ICLR
venue_type: conference
year: 2026
month: 4
published_date: "2026-04-27"
ingested_date: "2026-10-21"
task: [TTS, evaluation]
architecture: []
conditioning: []
training: []
model_size: "not applicable (training-free metric)"
codec_used: "none"
datasets_train: ["not applicable (training-free metric)"]
datasets_eval: ["LibriTTS test (CLEAN)", "LibriVox 2025 recordings (NOISY)", "YouTube 2025 English (WILD)", "My Science Tutor (KIDS)", "TTSDS2 multilingual YouTube benchmark (14 languages)"]
metrics:
  - name: MOS
    value: 3.70
    system: Ground Truth
    testset: "Mean over CLEAN/NOISY/WILD/KIDS (Table 2)"
  - name: MOS
    value: 3.41
    system: E2-TTS
    testset: "Mean over CLEAN/NOISY/WILD/KIDS (Table 2)"
  - name: MOS
    value: 1.98
    system: SpeechT5
    testset: "Mean over CLEAN/NOISY/WILD/KIDS (Table 2)"
  - name: SMOS
    value: 4.37
    system: Ground Truth
    testset: "Mean over CLEAN/NOISY/WILD/KIDS (Table 2)"
  - name: SMOS
    value: 4.39
    system: MaskGCT
    testset: "Mean over CLEAN/NOISY/WILD/KIDS (Table 2)"
code_available: true
demo_available: null
url: "https://openreview.net/forum?id=uGai5lYHlV"
related_concepts: [evaluation-metrics, subjective-evaluation, self-supervised-speech]
related_papers: ["2204.02152", "2502.05139", "2406.18009", "2025.acl-long.313", "2409.00750", "2502.07243", "2407.05361", "2402.01912", "2301.02111", "2304.09116", "2305.07243", "2406.04904", "2411.01156", "2403.16973", "2312.01479"]
field_significance:
  level: "moderate"
  type: [evaluation-contribution, empirical-benchmark, dataset-contribution]
generation:
  schema_version: 2
  date: "2026-10-21"
  runtime: "claude-code"
  provider: "anthropic"
  agent: speech-generation-ingest-agent
  model: "claude-opus-5-5"
  commit: "afa19d3"
---

> [!abstract] ICLR · 2026 · Conference
> **Christoph Minixhofer et al.** (Centre for Speech Technology Research, University of Edinburgh) · [→ Paper](https://openreview.net/forum?id=uGai5lYHlV) · Demo: ? · Code: ✓
>
> Revises the distributional TTSDS metric into TTSDS2 and validates it against a new 11,846-rating listening test over 20 open-weight voice-cloning TTS systems and four speech domains, where it is the only one of 16 objective metrics with Spearman correlation of at least 0.50 in every domain and rating type; it also releases the ratings and a recurring, contamination-resistant benchmark pipeline covering 14 languages.

## Problem

Recent TTS systems report MOS or CMOS at or above ground truth, which makes subjective scores saturate and leaves objective metrics as the practical alternative. Those objective metrics (signal-based references such as PESQ, STOI and MCD; MOS predictors such as UTMOS and NISQA; distributional scores such as FAD; multi-dimensional predictors such as Audiobox Aesthetics) are rarely validated against listening tests on systems near human quality, and almost never outside clean audiobook speech. Subjective results are also not comparable across papers because listener pools and test sets differ. At the time of writing, the authors note, no public TTS benchmark covered more than one language, and fixed benchmarks risk leaking into the scraped web data modern systems train on.

## Method

TTSDS2 frames TTS evaluation as distributional similarity: for each feature representation, it computes the 2-Wasserstein distance between the synthetic feature distribution and a real reference distribution, and between the synthetic distribution and the closest of four noise distributions (uniform, Gaussian, all-ones, all-zeros). A feature score is `100 × W_noise / (W_real + W_noise)`, so 0 means noise-like and 100 means indistinguishable from the reference; scores above 50 are closer to real speech than to noise. Features are grouped into four factors, each factor score is the mean of its feature scores, and the final TTSDS2 score is the unweighted mean of factor scores. Lexical content of the reference need not match, and the authors state that 50 to 100 samples suffice because several representations are ensembled.

Relative to the original TTSDS (Minixhofer et al., SLT 2024), the factors are revised (Table 1): the ENVIRONMENT factor (VoiceFixer+PESQ, WADA SNR) is dropped; GENERIC now uses WavLM, HuBERT-base and wav2vec 2.0-base activations; SPEAKER keeps d-vector and WeSpeaker embeddings; PROSODY uses WORLD F0, prosody embeddings, and utterance-level speaking rate from deduplicated HuBERT and Allosaurus tokens (replacing HuBERT token lengths); INTELLIGIBILITY replaces WER with final-layer activations of wav2vec 2.0 and Whisper-small ASR models. Feature selection was fixed before any correlation with human judgments was computed: each candidate had to score at least 95 on average when one half of a ground-truth set was scored against the other half, without high variance across datasets. For multilingual use, HuBERT is swapped for mHuBERT-147 and wav2vec 2.0 for XLSR-53.

Validation uses four English domains with 60 speakers each and speaker-matched reference/target pairs: CLEAN (LibriTTS test), NOISY (unfiltered 2025 LibriVox recordings), WILD (2025 YouTube speech, diarized with Whisper) and KIDS (My Science Tutor children's speech). Twenty open-weight systems released 2022 to 2024 synthesize the second utterance's transcript using the first as voice reference. Two hundred Prolific listeners (native UK/US English speakers, 50 per domain, headphones, attention checks) rate MOS, CMOS and SMOS against ground truth, yielding 11,846 ratings, 11,282 of which are released. Sixteen comparison metrics are computed with the VERSA toolkit plus UTMOSv2.

The multilingual benchmark pipeline (Algorithm 1) translates ten search keywords per language, searches YouTube for videos longer than 20 minutes uploaded after the newest evaluated system, diarizes and language-identifies the audio, filters controversial content with an XNLI entailment model, removes crosstalk (Pyannote) and background music (Demucs), selects 50 speaker-matched pairs per language, synthesizes with each system, and scores with multilingual TTSDS2. Re-running it periodically is intended to keep test data newer than any system's training data.

## Key Results

- **Correlation with listening tests (Table 3):** TTSDS2 averages Spearman ρ = 0.67 across the 12 domain-by-rating cells, with a minimum of 0.50 (KIDS CMOS); the original TTSDS averages about 0.60 and drops to 0.49 on NOISY MOS. Speaker-similarity metrics (X-Vector, RawNet3) average about 0.6 and SQUIM MOS about 0.57; every other metric averages below 0.3. TTSDS2 is not the top metric in every cell: RawNet3 reaches 0.85 on WILD MOS and 0.82 on NOISY SMOS where TTSDS2 scores 0.75 and 0.71, and on the pooled out-of-domain data RawNet3 (ρ = 0.73) edges out TTSDS2 (ρ = 0.72) (§Appendix E).
- **Domain fragility of MOS predictors:** UTMOS, UTMOSv2 and the Audiobox Aesthetics subscores correlate moderately on CLEAN and NOISY audiobook speech but fall to near zero or negative on WILD and KIDS (e.g. UTMOS -0.12 on WILD MOS; AE-CE -0.02 on KIDS MOS). NISQA, MCD and WER correlate negatively or weakly throughout.
- **No human parity under a controlled test (Table 2):** averaged over domains, ground truth scores MOS 3.70; the best synthetic MOS is E2-TTS at 3.41, and every system has negative CMOS except Vevo (+0.08 ± 0.18). MaskGCT (4.39) and E2-TTS (4.37) match ground-truth SMOS (4.37). By contrast, 7 of the 13 systems with papers reported MOS or CMOS within 0.05 of, or above, ground truth (§Appendix A).
- **Aggregation ablation (§3.5, Tables 4 and 5):** under leave-one-domain-out cross-validation, the unweighted factor mean beats regression-learned weights on three of four held-out domains (CLEAN 0.747 vs 0.645, NOISY 0.590 vs 0.514, WILD 0.752 vs 0.658) but loses clearly on KIDS (0.606 vs 0.853). Learned weights vary strongly by training domain and sometimes turn negative.
- **Factor contributions (Table 8):** the SPEAKER factor correlates most strongly with MOS on CLEAN and NOISY (r = 0.84, 0.86), while GENERIC and INTELLIGIBILITY matter more on WILD and KIDS; PROSODY peaks on KIDS (r = 0.60).
- **Multilingual proxy validation (§4.2):** TTSDS2 scores between ground-truth datasets of different languages correlate with Uriel+ typological distances at ρ = -0.39 (standard features) and ρ = -0.51 (multilingual features).
- **Compute (§Appendix F):** each TTSDS2 score takes about 9.4 CPU minutes; synthesis for all systems and languages took 28.8 A100 hours.

Comparisons are fair in the sense that all systems are run on the same speakers and texts and rated by the same listener pool, though rank correlations over 20 systems carry wide uncertainty and the paper reports significance only for TTSDS and TTSDS2.

## Novelty Assessment

The metric itself is an incremental revision of the authors' own TTSDS: the Wasserstein-to-real-versus-noise normalization and factor structure are inherited, and the changes are feature swaps (ASR activations for WER, speaking rate for token lengths, added WavLM, dropped ENVIRONMENT factor) plus multilingual encoders. The more substantial contributions are the validation resources: a speaker- and text-controlled listening test spanning 20 open-weight systems and four domains, released ratings on systems close to human quality, and a cross-domain correlation study of 16 metrics that exposes how sharply MOS predictors degrade outside audiobook speech. The recurring, freshly scraped multilingual benchmark is a practical design for contamination resistance, but its validity outside English rests on a typological-distance proxy rather than human ratings. The pre-registration-style choice to fix features before seeing correlations strengthens the result against overfitting claims.

## Field Significance

Moderate — The paper provides a controlled, cross-system listening test on 20 open-weight TTS systems released 2022 to 2024 and shows that most automatic quality predictors fail to rank such systems once speech leaves the audiobook domain, while a factorized distributional score holds up more consistently. Its released ratings can serve as validation data for future MOS predictors on near-human-quality speech, and its controlled results provide a cautionary data point against paper-reported human parity.

## Claims

- **supports:** Distributional metrics that compare feature distributions of synthetic and real speech across several perceptual factors track human rankings of TTS systems more consistently across domains than per-utterance MOS predictors.
  > *Evidence:* TTSDS2 reaches Spearman ρ of at least 0.50 in all 12 domain-by-rating cells (mean 0.67) across CLEAN, NOISY, WILD and KIDS, while UTMOS, UTMOSv2, NISQA, DNSMOS and the Audiobox Aesthetics subscores average below 0.3. *(§3.4, Table 3)*
- **complicates:** Neural MOS predictors trained largely on read or audiobook speech lose most of their ability to rank TTS systems on conversational, in-the-wild or children's speech.
  > *Evidence:* UTMOS correlates at 0.51 with MOS on CLEAN but -0.12 on WILD and -0.02 on KIDS; AE-CE falls from 0.60 on CLEAN to -0.02 on KIDS; UTMOSv2 is at or below 0.16 on WILD and KIDS. *(§3.4, Table 3)*
- **complicates:** Human parity reported in individual TTS papers often does not reproduce when systems are compared under a common listener pool, speaker set and text set.
  > *Evidence:* 7 of 13 published systems reported MOS or CMOS within 0.05 of, or above, ground truth, yet in the controlled test every system scores below ground-truth MOS (best 3.41 vs 3.70) and only Vevo has positive CMOS (+0.08 ± 0.18). *(§Appendix A, Table 6; §2, Table 2)*
- **refines:** Speaker-identity similarity is the single factor most predictive of naturalness ratings for voice-cloning TTS on clean read speech, but other factors become comparably important on spontaneous and children's speech.
  > *Evidence:* The SPEAKER factor correlates with MOS at r = 0.84 (CLEAN) and 0.86 (NOISY) but 0.59 on WILD, where GENERIC and INTELLIGIBILITY reach 0.53 and 0.58; on KIDS, GENERIC reaches 0.80 and PROSODY 0.60. *(§Appendix D, Table 8)*
- **complicates:** Objective TTS metrics still leave a substantial share of listener judgment unexplained, so they cannot replace listening tests for near-human-quality systems.
  > *Evidence:* No metric, TTSDS2 included, exceeds Spearman ρ = 0.85 in any cell and TTSDS2 never exceeds 0.8; the authors state it cannot replace subjective evaluation and misses failures such as unfaithful transcript reproduction. *(§5 Limitations, Table 3)*

## Limitations and Open Questions

> [!warning]
> All human validation is English-only, with native UK/US listeners. The 14-language benchmark is validated only through correlation with typological language distances (ρ = -0.51), so its agreement with human judgments in other languages is untested.

- The aggregation ablation does not uniformly favour the unweighted mean: learned weights perform much better on the held-out KIDS domain (0.853 vs 0.606), so equal weighting is a robustness choice rather than a uniformly optimal one.
- TTSDS2 operates on distributions, not single utterances; it cannot flag individual failures such as hallucinated or skipped words, which the benchmark site reports separately as high-WER counts.
- Context and long-form speech are not covered; utterances are capped at 30 seconds because many evaluated systems cannot generate longer.
- Computation is CPU-bound (about 9.4 minutes per score); the authors suggest MMD as a cheaper alternative.
- Each Spearman correlation is computed over 20 systems; differences such as TTSDS2 versus RawNet3 in individual cells are within the range where rank correlations are noisy.
- ParlerTTS was modified to allow voice cloning, which the authors note could degrade its scores; all checkpoints predate 2025.

## Wiki Connections

- [[evaluation-metrics|Evaluation Metrics]] — Introduces the TTSDS2 distributional metric and benchmarks 16 objective TTS metrics against human ratings across four domains.
- [[subjective-evaluation|Subjective Evaluation]] — Collects and releases 11,846 MOS, CMOS and SMOS ratings from 200 listeners on 20 systems, and contrasts controlled results with paper-reported human parity.
- [[self-supervised-speech|Self-Supervised Speech]] — The metric's GENERIC and PROSODY factors are built on WavLM, HuBERT/mHuBERT-147 and wav2vec 2.0/XLSR-53 representations.
- [[2204.02152|UTMOS]] — Compared MOS predictor; correlates moderately on audiobook domains but collapses on WILD and KIDS.
- [[2502.05139|Meta Audiobox Aesthetics]] — Three of its subscores are compared and correlate more weakly than originally reported, failing on WILD and KIDS.
- [[2406.18009|E2 TTS]] — Evaluated system with the highest synthetic MOS (3.41) and ground-truth-level SMOS in the controlled test.
- [[2409.00750|MaskGCT]] — Evaluated system with the highest TTSDS2 score among synthetic systems and the highest SMOS.
- [[2502.07243|Vevo]] — Evaluated system and the only one with a positive mean CMOS against ground truth.
- [[2025.acl-long.313|F5-TTS]] — Evaluated system that reported surpassing ground truth in its own paper but scores below ground truth MOS and CMOS here.
- [[2402.01912|Parler-TTS (Natural Language Guidance)]] — Evaluated after modification for voice cloning; its paper's above-ground-truth claim is cited as an example of reported parity.
- [[2407.05361|Emilia]] — Its in-the-wild data collection inspires the WILD domain and the YouTube scraping pipeline.
