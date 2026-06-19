---
id: "interspeech-2025-0401"
title: "Enabling the replicability of speech synthesis perceptual evaluations"
authors: ["Sébastien Le Maguer", "Gwénolé Lecorvé", "Damien Lolive", "Naomi Harte", "Juraj Šimko"]
organization: null
venue: Interspeech
venue_type: conference
year: 2025
month: 8
published_date: 2025-08-17
ingested_date: 2026-06-03
task: [evaluation]
architecture: []
conditioning: []
training: []
model_size: "not reported"
codec_used: "none"
datasets_train: []
datasets_eval: []
metrics: []
code_available: true
demo_available: null
url: "https://www.isca-archive.org/interspeech_2025/lemaguer25_interspeech.html"
related_concepts: ["subjective-evaluation", "evaluation-metrics"]
related_papers: []
field_significance:
  level: moderate
  type: [evaluation-contribution, conceptual-contribution]
generation:
  date: 2026-06-03
  model: claude-sonnet-4-6
  commit: "55d0339"
---

> [!abstract] Interspeech · 2025 · Conference
> **Le Maguer et al.** · [→ Paper](https://www.isca-archive.org/interspeech_2025/lemaguer25_interspeech.html) · Demo: ? · Code: ✓
>
> An open-source subjective evaluation platform (replikant) and an accompanying set of reporting guidelines that together enable replicability of speech synthesis perceptual evaluations.

## Problem

Despite a decade of awareness — first documented by Wester et al. in 2015 and reconfirmed by Kirkland et al. in 2023 — speech synthesis researchers routinely omit critical evaluation details from published reports. The precise question posed to listeners, the cohort composition, and the rationale for protocol choice are frequently absent, making it impossible to replicate evaluations and dangerous to compare results across papers. Existing platforms (WebMUSHRA, SurveyMonkey, Label Studio, the P.808 open implementation) either lack shareability mechanisms, are proprietary and subscription-based, or require substantial developer effort to extend with new protocols. The field lacks a lightweight, open platform designed specifically around the concept of a shareable evaluation recipe.

## Method

The paper draws a careful distinction between reproducibility (identical inputs, code, and conditions) and replicability (different data, same scientific question). Subjective evaluations are by nature unreproducible — new listeners cannot re-take the same test without bias — so replicability is the appropriate target.

The proposed standardisation decomposes any evaluation report into three parts: the evaluation protocol with its justification; an evaluation recipe (a self-contained bundle of procedure, anonymised data, and configuration); and the analysis of outcomes. The recipe is the key artifact: once shared, it allows any researcher to re-run the exact same evaluation setup.

**replikant** is the open-source platform built around this recipe concept, a refactoring of the authors' earlier FlexEval tool. It is implemented in Python using Flask, SQLite, and Jinja2, deliberately avoiding a GUI frontend in favour of stability and maintainability. Its architecture centres on a `Campaign` class that coordinates activities (pages, forms, authentication, and evaluation tasks) through a configurable activity graph. The `Task` class implements a generic, extensible evaluation step that stores timestamps, sample identifiers, and response values in a structured schema; this design allows entirely new protocols to be introduced by adding templates without modifying the platform core. A recipe directory contains the evaluated material (CSV-referenced audio/text/video), Jinja2-based UI templates, assets, and the campaign database. Recipes for MOS, MUSHRA, AB-test, and Rapid Prosody Transcription (RPT) are provided out of the box.

Complementing the platform, the paper proposes a set of checklist-style reporting guidelines aimed primarily at conference reviewers. The checklist covers three areas: (1) the evaluation campaign — protocol choice and justification, material description, participant cohort; (2) the analysis of outcomes — appropriateness of statistical tests, alignment with the evaluated hypothesis, identification of unexpected effects. These guidelines are grounded in ITU-T P.800, P.85, and the meta-analyses of Kirkland et al. and Wester et al., though they are intentionally kept high-level to avoid placing undue burden on reviewers.

## Key Results

The paper does not include a comparative experiment or numerical evaluation. The contribution is methodological: the platform itself, the recipe schema, and the reviewer checklist. Several published evaluations already use replikant-based recipes, including work on North Sámi TTS and video summarisation, demonstrating real-world deployment.

## Novelty Assessment

The conceptual framing — distinguishing reproducibility from replicability and arguing that the latter is the correct target for subjective evaluations — is the paper's most intellectually distinct contribution. The argument is well-grounded in the 2019 US National Academies report on reproducibility and replicability in science, applied to the speech evaluation context.

The platform itself (replikant) is an engineering contribution of moderate novelty. Its innovation over WebMUSHRA is primarily the recipe abstraction: a self-contained, shareable bundle that separates the evaluation design from the platform implementation. The technology stack (Flask, SQLite, Jinja2) is entirely standard. The reporting guidelines add value as a practical operationalisation of prior meta-analytic findings, but they do not introduce new research — they consolidate known requirements into an actionable reviewer checklist.

## Field Significance

Moderate — A decade of published meta-analyses documenting poor evaluation reporting practices in TTS has not produced broadly adopted remedies. This paper responds by pairing a conceptual framework with a concrete open-source tool and a reviewer checklist, lowering the practical barrier to replicable evaluations. Its impact depends on community uptake of replikant and adoption of the guidelines by programme committees. The conceptual reframing of the target from reproducibility to replicability is a useful clarification that applies across the field.

## Claims

- Standard MOS and MUSHRA protocols are insufficient for evaluating modern high-quality synthetic speech, and evaluation protocol saturation now drives the need for purpose-designed alternatives.
- Missing methodological details in published evaluation reports — particularly the exact question posed to listeners and cohort composition — materially affect the validity of cross-paper comparisons.
- Standardising evaluation into a shareable recipe schema separates the evaluation design from its execution platform, enabling replication without re-implementing the full setup.
- Conference review processes can meaningfully improve evaluation quality by assessing protocol justification, material appropriateness, and alignment between the statistical analysis and the evaluated hypothesis.

## Limitations and Open Questions

The guidelines are deliberately high-level, and the paper acknowledges that reviewers retain the final say and may deviate from them. Without formal adoption by Interspeech, ICASSP, or SSW programme committees, the guidelines remain advisory. The platform requires minimal but non-trivial coding skills (HTML, CSS, JavaScript, Python, Jinja2), which may limit uptake among researchers without web development experience. The paper does not evaluate whether recipe sharing actually increases replication attempts in practice — this is left as implicit future validation. The boundary between replicability and comparability (different platforms, different protocols, same scientific question) is not fully resolved: the framework addresses within-protocol replication but does not address cross-protocol comparability, which remains an open question.

## Wiki Connections

Concept pages most informed: [[subjective-evaluation]], [[evaluation-metrics]]
