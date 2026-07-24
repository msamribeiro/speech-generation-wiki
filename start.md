---
title: "Start Here"
generation:
  schema_version: 2
  date: "2026-07-24"
  stage: render
  mode: light
  runtime: codex
  provider: openai
  agent: speech-generation-render-agent
  model: "gpt-5"
  commit: "39007c6"
---

→ [[index|Home]]

The wiki presents the same concept-level research at two human-readable depths. Choose based on
the question you are trying to answer.

> [!warning]
> **Work in progress.** Concept synthesis currently covers research through Q3 2025. Later papers
> may already appear in the paper catalog but are not yet reflected in concept conclusions. The
> integration boundary will move forward chronologically until it reaches current research.

## I want a quick view of the current research

Start with a **Concept Overview**. Each one defines the concept, summarizes the current state,
maps the main method families, and surfaces the trade-offs and open questions that matter most.
They are designed to be read together when surveying many topics.

- [[concepts/flow-matching|Flow Matching]]
- [[concepts/speech-to-speech|Speech-to-Speech Systems]]
- [[concepts/evaluation-metrics|Evaluation Metrics]]
- [[concepts/rlhf-speech|RLHF for Speech]]
- [[concepts/disentanglement|Disentanglement]]

For connections across these five completed integrations, read the [[overview|Field Overview]].

## I want to understand one concept in detail

Choose an **In Depth** page. These reviews explain where evidence converges, where comparisons are
confounded, how method families relate, and what the literature still cannot establish.

- [[concepts/flow-matching-in-depth|Flow Matching: In Depth]]
- [[concepts/speech-to-speech-in-depth|Speech-to-Speech Systems: In Depth]]
- [[concepts/evaluation-metrics-in-depth|Evaluation Metrics: In Depth]]
- [[concepts/rlhf-speech-in-depth|RLHF for Speech: In Depth]]
- [[concepts/disentanglement-in-depth|Disentanglement: In Depth]]

## I want to find a paper

Use the [[papers/index|paper catalog]] to browse all 640 paper pages, or use search to find a title,
author, system, benchmark, or paper ID.

## I want to inspect the complete evidence structure

The Markdown pages are reader-oriented selections, not full exports of the claim graph. The
machine-readable records live in `_claims/{concept}.yaml` in the content repository and retain
paper-level claims, support roles, method families, caveats, and reassessment state.

## I want to see what is available

The [[concepts/index|concept catalog]] is the authoritative list of completed and pending concept
integrations. Five of the 23 tracked concepts currently have both reading depths; the others remain
navigation placeholders until integration and rendering are complete.

As coverage expands, the wiki is also planned to add time-based comparisons, venue-level research
reports, and citation analyses of influential and highly cited papers.
