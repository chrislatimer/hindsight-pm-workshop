# Northwind — AI Project Manager Workshop

A 40-minute hands-on challenge: build an agent that reads two weeks of team meeting transcripts into [Hindsight](https://hindsight.vectorize.io/) and tells the project manager what's actually going on — open items, blockers, slips, risks, decisions, and what changed after each new meeting.

## What's here

| Path | What it is |
|---|---|
| `LAB_GUIDE.md` | Start here. Step-by-step build guide with times. Written to be pasted into a coding agent. |
| `transcripts/` | 15 meeting transcripts, Aug 31 – Sep 18, 2026. `01`–`10` are ingested first; `11`–`15` are released one at a time during the session. |
| `manifest.json` | For every transcript: `document_id`, `timestamp`, sprint, meeting type, and tags for `retain`. |
| `PRESENTATION_OUTLINE.md` | The 15-minute intro deck, slide by slide. |
| `FACILITATOR_KEY.md` | **Facilitators only.** Every planted signal, expected state after each ingest, and the scoring rubric. Don't hand this to participants. |

## Setup

1. Sign up at https://ui.hindsight.vectorize.io/ (promo code `HOUSTON50`) and copy your API key.
2. Install the Hindsight skill in your coding agent:
   ```
   npx skills add https://github.com/vectorize-io/hindsight --skill hindsight-docs
   ```
3. Open `LAB_GUIDE.md` and follow it.

## The transcripts

A payments team of five plus one person from Risk, shipping a feature over three sprints. The transcripts are raw: 17–21 minutes each, tangents, crosstalk, names transcribed inconsistently. The things a PM needs to know are said once, in passing, and not repeated. That's the point.

Do not ingest `transcripts/11`–`15` until told.
