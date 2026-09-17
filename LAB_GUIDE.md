# Northwind Lab Guide
**Build an AI project manager with Hindsight — 1.5-hour challenge**

## The situation

You've just joined a team called Northwind as an AI engineer. The team is shipping a payments feature: settlement reconciliation, a dispute UI, chargeback rules, and a fraud-scoring eval harness. They've been at it for two weeks. You have transcripts of every standup and review from those two weeks.

Your job is to build an application that reads those transcripts into Hindsight, figures out the state of the project, and then keeps that picture current as five more meetings arrive over the next week.

Nobody on the team has written a status report. Nobody has a list of open items. The transcripts are all there is.

## The team

| Person | Role | Notes |
|---|---|---|
| Theo | Project manager | Runs standup. Your app is for him. |
| Maya | Engineering lead | Makes the calls. |
| Jordan | Backend | Reconciliation job, chargeback rules. |
| Sam | Frontend | Dispute UI. Sometimes referred to as Samantha or Sam K. |
| Priya | Data / evals | Fraud-scoring evals. |
| Ravi | Risk team (external) | Shows up occasionally. |

People who are talked about but rarely speak: **Nadia** (finance, the stakeholder), **Dev** (sales), **Arjun** (owns the dispute service), **Tomás and Kenji** (borrowed labelers), and **Ops**.

## The calendar

- **Sprint 1:** Mon Aug 31 – Fri Sep 4 (review Sep 4)
- **Sprint 2:** Mon Sep 7 – Fri Sep 11 (Labor Day Sep 7; review Sep 11)
- **Sprint 3:** Mon Sep 14 – Fri Sep 18
- **Stakeholder demo:** originally Fri Sep 11 — check the transcripts for what actually happened
- **Release train cutoff:** Wed Sep 16, noon PT
- **Next train:** Sep 30

## The files

`transcripts/01` through `10` cover Aug 31 – Sep 11. Ingest all of these first. They are raw meeting transcripts in `Speaker (MM:SS): text` form, so they're long, they wander, and the things that matter are usually said in passing. That's the point. Transcript 06 is a Slack thread from a holiday, not a meeting.

`transcripts/11` through `15` cover Sep 14 – Sep 18. **Do not ingest these until told to.** They arrive one at a time during the second half of the session, and the point is to watch your application's understanding change with each one.

`manifest.json` has the date, timestamp, suggested `document_id`, and suggested tags for every file. Use the timestamps. Hindsight's temporal retrieval depends on them, and several of the questions below cannot be answered without them.

## What your application must be able to answer

These are your acceptance tests. Pick a `query_timestamp` of **Sep 14, 2026, 9:00 AM** for everything in the first group.

**After the first ten transcripts:**

1. What's open, who owns it, and when is it currently due? (Not the first date it was given. The current one.)
2. What has slipped, and by how much?
3. What is or was blocked, on whom, and for how long?
4. What decisions have been made? Has anything since contradicted them?
5. Who is or was unavailable, and what did that affect?
6. What is at risk for the Sep 16 release train?
7. What was raised once and never followed up on?

**As each of the next five transcripts lands:**

8. What changed since the last meeting?
9. Did anything previously flagged get resolved, or get worse?
10. Is there a new item to track?
11. Does this meeting contradict anything the application currently believes?

You will be asked to demo your application answering a selection of these, and your answers will be checked against what actually happened in the transcripts.

## The three design questions

You'll need to answer these before you write much code. They're in the order you should decide them, which is the reverse of the order you'll build.

### 1. What should the application look like?

Decide whether Theo asks the application questions, or the application tells Theo what changed, or both. The five-transcript sequence rewards the second. The acceptance tests reward the first. A minimum that works: a board with open items, blockers, risks, and decisions (each with owner, date, and age), a "what changed" view after each ingest, and a free-text question box. Every item should be able to show which meeting it came from.

Think about which views need a reasoned, synthesized answer (reflect) and which just need the relevant facts pulled up (recall). Reflect is an agentic loop that costs tokens and time. Recall has no LLM call. An app that puts reflect behind every pane will be slow. An app that only uses recall will show facts with no judgment.

### 2. What mental models would be most useful?

A mental model is a synthesized answer to a standing question, refreshed automatically as new memories are consolidated. Good ones are stable questions the PM asks every day. Bad ones are questions whose answer changes completely between meetings, or that only apply once.

Aim for four or five. Write the `source_query` for each as the question you'd ask a human PM. Enable `refresh_after_consolidation` so you can watch them update as the last five transcripts land.

Some candidates to argue about: open blockers and their age; each person's commitments versus what they delivered; decisions and anything contradicting them; items raised without an owner; schedule risk for the next milestone; recurring external dependencies. Some of these are better as mental models than others. Decide which, and be ready to say why.

### 3. What tags should you use?

Tags are deterministic labels you set at ingest time. Their job is to scope: a mental model or a recall can be restricted to memories carrying certain tags. They are **not** for classifying content. Do not tag a transcript as "blocker" or "decision" — Hindsight extracts those itself, and a tag has to be knowable before anyone reads the file.

The manifest suggests `project:`, `sprint:`, and `meeting:` tags. Decide which of these you actually need. A useful test: name a mental model or a query that would give a different answer with the tag than without it. If you can't, you don't need the tag.

## Suggested flow

| Time | What |
|---|---|
| 0:00–0:30 | Presentation: Hindsight, cloud registration, retain, mental models, tags, recall, reflect |
| 0:30–0:40 | Ingest transcripts 1–10. Verify with one recall. |
| 0:40–1:00 | Decide the three design questions. Create your mental models. |
| 1:00–1:30 | Build the application. |
| 1:30–1:50 | Transcripts 11–15 released one at a time. Ingest each, show what changed. |
| 1:50–2:00 | Demos and scoring. |

## Ingestion sketch (Python)

Check the current SDK reference for exact parameter names before running this.

```python
import json
from datetime import datetime
from hindsight_client import Hindsight

client = Hindsight(base_url=HINDSIGHT_API_URL, api_key=HINDSIGHT_API_KEY)
BANK_ID = "northwind-<your-team-name>"

manifest = json.load(open("manifest.json"))
for t in manifest["transcripts"]:
    if t["file"].split("/")[-1][:2] not in manifest["phase_1_files"]:
        continue  # phase 2 files are ingested later, one at a time
    content = open(t["file"]).read()
    client.retain(
        bank_id=BANK_ID,
        content=content,
        document_id=t["document_id"],
        timestamp=t["timestamp"],
        context="team meeting transcript for the Northwind payments project",
        tags=t["tags"],
    )
```

Retain the **whole transcript** under one `document_id`. If you re-ingest a corrected version later, use the same ID and Hindsight replaces it rather than duplicating.

## Things that will bite you

- **Consolidation is asynchronous.** Observations and auto-refreshing mental models update in the background after retain. If you query the instant after ingesting, you may see stale or empty results. Wait, then retry.
- **Reflect is expensive.** Use `budget: "low"` or `"mid"` while building. Save `"high"` for the final demo. There are a lot of teams on this server.
- **Timestamps.** If you skip them, "last week" means nothing and blocker ages will be wrong.
- **Bank configuration.** Set a `mission` for the bank that describes what a PM cares about. Try a directive like "a completion is unverified until the owner confirms it" and see what changes.
- **Citations.** Reflect can return which memories it based an answer on. Use this. When you're asked "why is that a risk," you'll want to point at a specific meeting.
