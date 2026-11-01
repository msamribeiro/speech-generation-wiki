---
title: "Start Here"
generation:
  schema_version: 2
  date: "2026-11-01"
  stage: render
  mode: full
  runtime: codex
  provider: openai
  agent: speech-generation-render-agent
  model: "gpt-5"
  commit: "d13cdad"
---

→ [[index|Home]]

Choose a route based on what you want to learn. You can move between the field overview, concept
guides, quarterly reports, and individual papers without losing the evidence trail.

> [!warning]
> **Current synthesis boundary.** The integrated evidence runs through Q4 2025. Later papers may
> already appear in the catalog, but they will not affect field or concept conclusions until their
> quarter has been integrated and reconciled.

## I want a gentle introduction to the field

Read the [[overview|Field Overview]]. It brings together all 23 integrated concepts and explains
the main connections and trade-offs in familiar language: what speech tokens preserve, where
latency comes from, how control interacts with identity, and why evaluation needs several
dimensions.

## I want to understand what changed during a quarter

Open the [[reports/index|reports catalog]]. The latest report,
[[reports/quarterly/2025-Q4|Speech Generation — Q4 2025]], compares Q4 activity with the evidence
available through Q3. The earlier [[reports/quarterly/2025-Q3|Q3 2025 report]] provides the same
kind of retrospective view against the pre-Q3 baseline. Both distinguish publication activity
from evidence that strengthened, qualified, or contested an existing conclusion.

Quarterly reports are frozen to reviewed snapshots. Later changes to the living concept pages do
not rewrite them.

## I want a quick view of one topic

Choose a Concept Overview. Each one defines its topic, summarizes the current evidence, maps the
main method families, and surfaces practical trade-offs and open questions.

→ [[concepts/index|Browse the concept catalog]]

All 23 concepts have complete Overview and In Depth pairs, including singing and fine-tuning.

## I want to examine one topic in depth

From the same [[concepts/index|concept catalog]], choose an **In Depth** page. These guides explain
where evidence converges, where comparisons are confounded, how method families relate, and what
the literature cannot yet establish.

## I want to find a paper

Use the [[papers/index|paper catalog]] to browse all 1,061 paper pages, or search for a title, author,
system, benchmark, venue, or paper ID.

## I want to inspect the complete evidence structure

The reader-facing pages select representative findings rather than reproduce the full evidence
graph. The machine-readable records live in `_claims/{concept}.yaml`. Reviewed cross-concept
relationships live in `_claims/_reconciliation/registry.yaml`, and immutable historical
assessments live in `_claims/_reconciliation/snapshots/`.

The latest Q4 snapshot records the publication cutoff, retrospective assessment date, paper set,
local claim assessments, method families, and human-approved broader relationships used by the Q4
report. The Q3 snapshot remains available as the preceding historical assessment.
