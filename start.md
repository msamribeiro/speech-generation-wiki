---
title: "Start Here"
generation:
  schema_version: 2
  date: "2026-11-07"
  stage: render
  mode: full
  runtime: codex
  provider: openai
  agent: speech-generation-render-agent
  model: "gpt-5"
  commit: "0e1aff0"
---

→ [[index|Home]]

Choose the route that matches your question. Each route links back to the papers and structured
evidence behind its conclusions.

> [!warning]
> **Current coverage.** The integrated evidence runs through Q4 2025. Later papers may already be
> in the catalog, but they will not change the summaries until they have been integrated and
> reviewed.

## I want a gentle introduction to the field

Read the [[overview|Field Overview]]. It connects all 23 concepts and explains the main trade-offs:
what speech tokens preserve, where latency comes from, how control affects identity, and why one
evaluation score is rarely enough.

## I want to understand what changed during a quarter

Open the [[reports/index|reports catalog]]. The latest report,
[[reports/quarterly/2025-Q4|Speech Generation — Q4 2025]], compares Q4 activity with the evidence
available through Q3. The earlier [[reports/quarterly/2025-Q3|Q3 2025 report]] compares Q3 with the
pre-Q3 baseline. Both separate the number of new papers from changes in the strength or scope of a
finding.

Quarterly reports are saved records. Later changes to the concept pages do not rewrite them.

## I want a quick view of one topic

Choose a Concept Overview. It defines the topic, summarizes the main findings, compares the main
method families, and lists the trade-offs and open questions that matter most.

→ [[concepts/index|Browse the concept catalog]]

All 23 concepts have complete Overview and In Depth pairs, including singing and fine-tuning.

## I want to examine one topic in depth

From the same [[concepts/index|concept catalog]], choose an **In Depth** page. It explains where
studies agree, why some comparisons are hard to interpret, how the methods relate, and what the
research still cannot establish.

## I want to find a paper

Use the [[papers/index|paper catalog]] to browse all 1,061 paper pages, or search for a title, author,
system, benchmark, venue, or paper ID.

## I want to inspect the complete evidence structure

The reader pages show the findings that matter most. They do not reproduce the complete evidence
graph. The full concept records live in `_claims/{concept}.yaml`. Reviewed links between concepts
live in `_claims/_reconciliation/registry.yaml`. Saved quarterly assessments live in
`_claims/_reconciliation/snapshots/`.

The Q4 snapshot records the publication cutoff, review date, paper set, claim assessments, method
families, and approved links used by the Q4 report. The Q3 snapshot remains available for the
previous quarter.
