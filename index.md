---
title: "Speech Generation Wiki"
generation:
  schema_version: 2
  date: "2026-07-24"
  stage: render
  mode: full
  runtime: codex
  provider: openai
  agent: speech-generation-render-agent
  model: "gpt-5"
  commit: "9bf1473"
---

![Speech Generation Wiki](assets/speech-generation-wiki.png)

> [!abstract]
> A living systematic review of synthetic speech, covering **text-to-speech (TTS)**,
> **voice conversion (VC)**, and **spoken conversational agents**. The wiki currently contains
> 888 paper pages and tracks 23 concepts.

The wiki turns a growing paper corpus into three complementary ways to read the research:

- **Concept Overviews** give a concise account of what matters now, for readers scanning the field.
- **Concept In Depth pages** explain what the reviewed research collectively shows, where findings
  disagree, and which conclusions remain uncertain.
- **Structured claim YAML** preserves the complete machine-readable links among claims, evidence,
  method families, and papers.

Concept conclusions describe the literature integrated into this corpus. They are not claims of
exhaustive coverage of the field.

> [!warning]
> **Work in progress.** Concept integration and synthesis currently cover research through
> Q3 2025. The paper catalog may contain later work that has been ingested but not yet incorporated
> into concept conclusions. Coverage will advance chronologically until it catches up with current
> research, then continue on a rolling basis.

→ [[start|Choose how to explore the wiki]]

## Read the Integrated Concepts

Five concepts have completed integration and are available at both reading depths.

| Concept | Overview | In Depth | Papers reviewed |
|---------|----------|----------|-----------------|
| Flow Matching | [[concepts/flow-matching\|Overview]] | [[concepts/flow-matching-in-depth\|In Depth]] | 97 |
| Speech-to-Speech Systems | [[concepts/speech-to-speech\|Overview]] | [[concepts/speech-to-speech-in-depth\|In Depth]] | 60 |
| Evaluation Metrics | [[concepts/evaluation-metrics\|Overview]] | [[concepts/evaluation-metrics-in-depth\|In Depth]] | 285 |
| RLHF for Speech | [[concepts/rlhf-speech\|Overview]] | [[concepts/rlhf-speech-in-depth\|In Depth]] | 29 |
| Disentanglement | [[concepts/disentanglement\|Overview]] | [[concepts/disentanglement-in-depth\|In Depth]] | 100 |

Paper counts overlap because one paper can inform several concepts.

→ [[concepts/index|See all 23 concepts and their integration status]]

## Cross-Concept View

The [[overview|Field Overview]] connects findings across the five completed concept integrations.
It highlights shared design pressures in generation, control, alignment, interaction, and
evaluation while keeping its conclusions bounded to the reviewed corpus.

## Browse the Papers

The corpus contains 888 paper pages from arXiv, Interspeech, ACL, EMNLP, NAACL, ICLR, NeurIPS,
ASRU, workshops, and industry technical reports.

→ [[papers/index|Browse all 888 papers]]

## Project Status

The remaining 18 concepts are awaiting their Q3 2025 integration pass. Their existing pages are
navigation placeholders rather than current research syntheses.

After Q3 integration is complete across the concept catalog, the project will continue through
Q4 2025 and subsequent periods until the synthesis layer catches up with the live research corpus.
Broader analysis is planned alongside that expansion:

- **Time-based reports** will compare how methods, evidence, and open questions change between
  periods such as Q3 and Q4 2025.
- **Venue reports** will synthesize the research represented at venues such as Interspeech,
  NeurIPS, ICLR, and ACL when coverage is sufficient for a meaningful analysis.
- **Citation analysis** will track the most-cited and most influential papers in the corpus,
  including changes in their relative position over time.
