---
title: "Speech Generation Wiki"
generation:
  schema_version: 2
  date: "2026-11-01"
  stage: render
  mode: light
  runtime: codex
  provider: openai
  agent: speech-generation-render-agent
  model: "gpt-5"
  commit: "eed154b"
---

![Speech Generation Wiki](assets/speech-generation-wiki.png)

> [!abstract]
> A living systematic review of synthetic speech, covering **text-to-speech (TTS)**,
> **voice conversion (VC)**, and **spoken conversational agents**. The wiki contains 1061 paper
> pages and integrates research across 23 concepts.

The wiki offers four complementary ways to read the research:

- The [[overview|Field Overview]] gives a gentle introduction to the current evidence across the
  whole field.
- The [[concepts/index|concept catalog]] provides focused Overview and In Depth guides to individual
  methods and capabilities.
- [[reports/index|Quarterly reports]] preserve what changed within a fixed publication period.
- The [[papers/index|paper catalog]] and structured claim records provide the underlying evidence.

→ [[start|Choose the best place to start]]

> [!abstract]
> **Current synthesis boundary.** The integrated concepts and field overview use research published
> through Q4 2025. The paper catalog contains later work that has been ingested but is not yet part
> of these conclusions. The next synthesis cycle begins after the Q2 2026 ingestion frontier closes.

## Start with the field

The [[overview|Field Overview]] connects all 23 concepts. It explains the recurring choices around
speech representations, generation speed, control, voice transfer, evaluation, and spoken
interaction without requiring readers to begin with one architecture.

To understand what specifically changed during the latest completed period, read
[[reports/quarterly/2025-Q4|Speech Generation — Q4 2025]]. The report separates publication
activity from changes in assessed knowledge and is frozen to its Q4 evidence boundary.

## Explore a concept

All 23 concepts have both a concise Overview and a detailed In Depth page. These include
generation architectures, neural codecs and speech representations, speaker and style control,
multilingual and zero-shot synthesis, streaming and spoken agents, and evaluation.

→ [[concepts/index|Browse all 23 concepts]]

## Browse the papers

The corpus contains 1061 paper pages from arXiv, Interspeech, ACL, EMNLP, NAACL, ICLR, NeurIPS,
ASRU, workshops, and industry technical reports. Search by title, author, system, benchmark, venue,
or paper ID.

→ [[papers/index|Browse all 1061 papers]]

## How to read the dates

The field overview is a replaceable account of the latest integrated evidence. Quarterly reports
are immutable historical views derived from reviewed snapshots. The current Q4 materials were
assessed retrospectively on 2026-11-01 using evidence published through 2025-12-31; they do not
claim that the assessment itself was made at the end of that quarter.

Paper counts show corpus coverage and research attention. They do not by themselves establish
evidence strength, consensus, or adoption.
