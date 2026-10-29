# MATACS-TH-2026-02: AI Proposal Triage Design

**Name:** Ali Hasnain
**Reference:** MATACS-TH-2026-02
**Time actually spent:** About 1 hour

## What is in this repository

| File | What it contains |
|------|------------------|
| [README.md](README.md) | This page |
| [WRITEUP.md](WRITEUP.md) | Design summary, answers to the five questions, and the scoring table for Proposals A to E |
| [PROMPTS.md](PROMPTS.md) | The AI prompts for fact extraction and for writing the one-line reason, exactly as they would be sent |
| [ranking.json](ranking.json) | Expected ranking of Proposals A to E |
| [diagram/flow.png](diagram/flow.png) | Process flow from upload to ranked output, showing AI steps, rule/code steps and human checkpoints |

## Design in one paragraph

The AI reads each proposal and pulls out facts, with a word-for-word quote as evidence for every fact. Plain code then checks those facts, applies the pass/fail rules, does all the scoring maths and sorts the list. The AI never sets a score or a rank. A reviewer checks every flag, can override any position with a written reason, and makes the final award. Every step is logged.

## How I used AI

I made the design decisions myself (pipeline steps, where AI stops and code takes over, how to handle Proposals B, C and D, and the scoring rubric) and wrote them up as a detailed brief for Claude Code. Claude Code then drafted the files, computed the scores from my rubric and generated the diagram. I reviewed every file and checked the scores against the rubric.

AI disagreement: My own estimate put Proposal A in the high 80s, but the AI's calculation from my rubric gave 93.2. The AI noted that switching to fixed anchors would bring A into the 80s. I re-checked the arithmetic and kept my rubric unchanged, because tuning a rubric to hit an expected number defeats the point of a fixed, explainable scoring rule.
