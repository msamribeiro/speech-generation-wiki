# Speech Generation Wiki

![Speech Generation Wiki](assets/speech-generation-wiki.png)

A living systematic review of synthetic speech research, covering **text-to-speech (TTS)**,
**voice conversion (VC)**, and **spoken conversational agents**.

This repository stores the wiki's published content: paper pages, structured concept records, and
research reviews. The [live site](https://msamribeiro.github.io/speech-generation-wiki-site) is the
easiest place to read it.

## How the Knowledge Base Is Organized

Each concept has three layers:

1. **Overview** — the current state, main method families, trade-offs, and open questions.
2. **In Depth** — the evidence behind that account, including disagreements and uncertainty.
3. **Claim YAML** — the complete machine-readable record of claims, paper roles, method families,
   caveats, and reassessment state.

The two reader pages come from the same claim YAML. They explain the findings that matter most
without copying the whole graph into prose.

```text
papers/{id}.md
    ↓ integration
_claims/{concept}.yaml
    ↓ rendering
concepts/{concept}.md
concepts/{concept}-in-depth.md
```

Concept conclusions cover the research integrated into this corpus. They do not claim to cover
every paper in the field.

> **Current coverage:** concept summaries include research through Q4 2025. The paper catalog
> contains later work that has not yet entered those summaries. New integration cycles will move
> the boundary forward.

## Start Reading

- [Home](index.md) — a short guide to the wiki
- [Start Here](start.md) — choose a route based on your question
- [Field Overview](overview.md) — the main findings across all concepts
- [Concept Catalog](concepts/index.md) — short and detailed guides to each concept
- [Paper Catalog](papers/index.md) — every ingested paper page

All 23 concepts are integrated through Q4 2025. Each has an Overview and an In Depth page. The
catalog also shows the current paper count and last render date.

## Current Coverage and Planned Analysis

Q4 2025 is the current synthesis boundary, not the endpoint. Q3 and Q4 2025 have saved snapshots
and quarterly reports. Papers from 2026 already appear in the catalog and will enter the synthesis
in later integration cycles.

Planned work also includes:

- **Quarterly reports** that extend the Q3 and Q4 2025 sequence.
- **Venue reports** for conferences with enough papers to support real synthesis, not just a list.
- **Citation analysis** that tracks highly cited papers and changes across snapshots.

## Repository Structure

```text
index.md                       Reader-facing landing page
start.md                       Intent-based reading guide
overview.md                    Cross-concept field overview
log.md                         Append-only content operations log

papers/
  index.md                     Full paper catalog
  {id}.md                      One reviewed paper page
  assets/{id}/                 Selected paper figures

_claims/
  {concept}.yaml               Structured concept claim graph and source of truth

concepts/
  index.md                     Concept catalog and integration status
  {concept}.md                 Concept Overview
  {concept}-in-depth.md        Concept In Depth review

venues/                        Venue reports generated on demand
reports/                       Snapshot-backed quarterly and future trend reports
```

The former Evidence Dossier format is retired. Use `_claims/` for the complete paper-to-claim
structure.

## Paper Pages

Each full paper page includes:

- a paper card with venue, year, authors, source link, and one-sentence contribution;
- a method summary and selected architecture figure when appropriate;
- key results with paper-local metric provenance;
- an assessment of novelty and field significance;
- two to five generalized claims with source-section citations;
- limitations, open questions, and links to related concepts and papers.

Foundational papers included mainly for citation context may use a shorter Tier 2 page.

## Coverage

The corpus draws from arXiv, Interspeech, ACL, EMNLP, NAACL, NeurIPS, ICLR, ASRU, workshops, and
technical reports from industry labs. Its main rolling window begins in 2025, with earlier
foundational work added through citation discovery. See the [paper catalog](papers/index.md) for
the current set; static counts in documentation may lag active ingestion.

## Pipeline and Companion Repositories

This repository is the content output of a staged evidence pipeline:

```text
Fetch → Filter → Parse → Ingest → Integrate → Reconcile → Render
                                             └→ Snapshot → Report
```

- [speech-generation-wiki-infra](https://github.com/msamribeiro/speech-generation-wiki-infra)
  contains acquisition scripts, parsed-source infrastructure, metadata, schemas, health checks,
  and shared Claude Code/Codex workflows.
- [speech-generation-wiki-site](https://github.com/msamribeiro/speech-generation-wiki-site)
  contains the Quartz configuration that publishes this repository to GitHub Pages.

Generated pages record their provenance in frontmatter. Integration changes claim status and
evidence relationships in `_claims/`, then rendering turns those records into Markdown. The
rendered concept pages can be replaced; they are not the source of truth.

## Reporting Errors or Contributing

When reporting an error, name the paper or concept page and, if possible, the source section,
table, or figure. Do not correct an evidence assessment only in a rendered concept page. Changes
to claims, roles, or support relationships must go through `_claims/{concept}.yaml`.
