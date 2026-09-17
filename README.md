# Northwind — AI Project Manager Workshop

A 40-minute hands-on challenge: build an agent that reads two weeks of team meeting transcripts into [Hindsight](https://hindsight.vectorize.io/) and tells the project manager what's actually going on — open items, blockers, slips, risks, decisions, and what changed after each new meeting.

## Pick a lab guide

There are three. Pick one based on how far you want to go in 40 minutes. Each is self-contained and written as prompts to paste into your coding agent (Claude Code, Codex, OpenCode).

| Guide | You'll build | Hindsight features | Pick it if |
|---|---|---|---|
| `LAB_GUIDE_1_BASIC.md` | A chat that answers questions about the project with cited evidence, plus one self-updating summary | Bank configuration, retain, reflect with citations, directives, one mental model | You're new to Hindsight and want the core loop working end to end |
| `LAB_GUIDE_2_MEDIUM.md` | A project board that Hindsight keeps current by itself, a chat, and a "what changed" report after each new meeting | Everything in 1, plus mental models with JSON schemas, `dry_run_refresh`, snapshot/diff, narration anchored on one meeting's facts | You've used a memory or RAG API before and want the synthesis layer doing real work |
| `LAB_GUIDE_3_ADVANCED.md` | A memory-first app: every extracted fact tagged by workstream, kind, and owner; filtered views with no LLM; mental models scoped to one topic each; board, chat, and "what changed" | Everything in 2, plus entity labels (derived tags), tag-scoped mental models, hard-filtered memory queries | You want to see why scoping by tag is what makes mental models accurate at scale |

All three load the same transcripts and answer the same questions. Track 2 contains Track 1; Track 3 contains Track 2.

## What's here

| Path | What it is |
|---|---|
| `LAB_GUIDE_1_BASIC.md`, `LAB_GUIDE_2_MEDIUM.md`, `LAB_GUIDE_3_ADVANCED.md` | The three lab guides. Choose one. |
| `transcripts/` | 15 meeting transcripts, Aug 31 – Sep 18, 2026. Every guide loads `01`–`10` first, then `11`–`15` one at a time to watch the answers change. |
| `manifest.json` | For every transcript: `document_id`, `timestamp`, sprint, meeting type, and tags for `retain`. |
| `PRESENTATION_OUTLINE.md` | The 15-minute intro deck, slide by slide. |
| `FACILITATOR_KEY.md` | **Facilitators only.** Every planted signal, expected state after each ingest, and the scoring rubric. Don't hand this to participants. |

## Setup

1. Sign up at https://ui.hindsight.vectorize.io/ (promo code `HOUSTON50`) and copy your API key.
2. Install the Hindsight documentation skill in your coding agent:
   ```
   npx skills add https://github.com/vectorize-io/hindsight --skill hindsight-docs
   ```
3. Export the key where your agent runs: `export HINDSIGHT_API_KEY=<your key>`
4. Open the lab guide you picked and follow it.

## The transcripts

A payments team of five plus one person from Risk, shipping a feature over three sprints. The transcripts are raw: 17–21 minutes each, tangents, crosstalk, names transcribed inconsistently. The things a PM needs to know are said once, in passing, and not repeated. That's the point.
