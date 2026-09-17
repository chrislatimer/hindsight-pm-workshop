# Northwind Lab — Track 2: Medium
**A board that stays current: mental models with typed output, and a "what changed" that is computed, not guessed. 40 minutes.**

Any language, any UI. This builds on Track 1's bank configuration and ingest — copy that code (`LAB_GUIDE_1_BASIC.md`, minute 0–5) and start the ingest **before reading the rest of this page**. Team, calendar, and files are in `LAB_GUIDE.md`.

## The 40-minute plan

| Minute | Do this | Why now |
|---|---|---|
| 0–5 | Bank config + **start ingesting 01–10** (Track 1 code) + create the directives below | Consolidation takes ~10 min; mental models refresh after it. |
| 5–12 | Create three mental models with schemas | They refresh automatically once the first consolidation finishes. |
| 12–25 | Build the board from their `structured_output`; add the chat from Track 1 | Render whatever the models say so far; it fills in. |
| 25–30 | Check the board against the seven questions; tighten one query | Use `dry_run_refresh` if a model looks wrong. |
| 30–40 | Transcript 11: snapshot → retain → wait → snapshot → "what changed" | The demo moment. |

## What you're building

1. Track 1's bank + ingest, plus **directives** that encode how a PM reads a standup.
2. **Three mental models** — standing questions Theo asks every day — each with a `response_schema` and `refresh_after_consolidation`.
3. A **board** rendered from the models' JSON: open items with date history, blockers, risks. Each row says which meeting it came from.
4. A **"what changed"** view after each new transcript: what the models said before vs after, narrated by one `reflect` that is anchored on the new meeting's own extracted facts.
5. The Track 1 chat for everything else.

The acceptance questions are the same seven as Track 1; the board should answer 1, 3, and 6 without a question being typed, and "what changed" should say what closed, what opened, what moved, and what contradicts — without restating unchanged items.

## Minute 0–5: directives

Right after `create_bank`, add hard rules. Each targets a specific way answers go wrong on this data:

```python
for name, content in [
    ("current-vs-superseded-dates",
     "Never present a superseded date as current. The most recent statement wins; list earlier dates as history "
     "with the meeting each was stated in. An in-meeting self-correction wins."),
    ("shipped-is-not-live",
     "Merged, shipped, on the train, and behind a feature flag are NOT live. Always say which applies."),
    ("verified-means-owner-said-so",
     "A completion is VERIFIED when the owner states it with specifics (merged, on staging, a number read off a dashboard). "
     "Second-hand reports and a non-owner who 'glanced at it' are UNVERIFIED. Use the word only for those."),
    ("flag-contradictions",
     "When a later statement contradicts a recorded decision, report both and say whether anyone corrected it. Never silently overwrite."),
]:
    client.create_directive(bank_id=BANK, name=name, content=content)
```

## Minute 5–12: three mental models

A mental model is a saved answer to a standing question, re-run after each consolidation. Good ones have a stable *shape* even though the answer moves. "What changed?" is a bad mental model — you'll compute that instead.

Three that earn their place here, with the question each asks:

| id | source_query (the question you'd ask a human PM) |
|---|---|
| `open-items` | Which significant commitments has the team made — deliverables a milestone or stakeholder depends on, decisions someone owes, requests never given an owner — and for each: owner (whoever most recently agreed to do it, or "unowned"), open/delivered/lapsed, the **current** due date, **every earlier** due date with the meeting it was stated in, when it was delivered, days late, and whether the owner confirmed. About ten items; prioritize ones whose date moved, whose owner changed, or which nobody owns. |
| `blockers` | What is or was blocked on someone outside the team (Risk/Ravi, Arjun, Nadia, Ops): owner of the blocked work, who it waited on, opened, closed, days, what it delayed. Has the same party gated the team more than once? |
| `risk` | Milestones (demo, train cutoff, go-live) with original and current dates and when they changed; whether the feature is merged, shipped, dark, or live; and what is at risk for the next milestone — unowned items, metrics that moved the wrong way with nobody investigating, estimates landing after the milestone, stacked milestones with no buffer. |

Create each with a schema so the board renders JSON. For `open-items`:

```python
OPEN_ITEMS_SCHEMA = {
  "type": "object", "required": ["items"],
  "properties": {"items": {"type": "array", "items": {"type": "object",
    "required": ["item", "owner", "status", "current_due"],
    "properties": {
      "item": {"type": "string"}, "owner": {"type": "string"}, "status": {"type": "string", "description": "open | delivered | lapsed"},
      "current_due": {"type": "string"}, "raised_on": {"type": "string"}, "delivered_on": {"type": "string"},
      "days_late": {"type": "integer"}, "owner_confirmed": {"type": "boolean"},
      "due_history": {"type": "array", "items": {"type": "object", "properties": {"date": {"type": "string"}, "stated_on": {"type": "string"}}}}}}}}}

client.create_mental_model(
    bank_id=BANK, id="open-items", name="Open items", source_query=OPEN_ITEMS_QUERY,
    trigger={
        "refresh_after_consolidation": True,
        "exclude_mental_models": True,     # read the memories, not the other models — see below
        "response_schema": OPEN_ITEMS_SCHEMA,
    },
)
```

Write similar schemas for `blockers` (`gates: [{what, owner, blocked_on, opened_on, closed_on, duration_days, delayed}]`, `patterns: [string]`) and `risk` (`milestones: [{name, original_date, current_date, changed_on, status}]`, `release_state`, `risks: [{risk, severity, why, since, owner}]`).

**Why `exclude_mental_models`:** by default a refresh may consult the *other* mental models and synthesize from them instead of from memories. Three models that read each other converge on the same mistakes. Turn it off on every model.

## Minute 12–25: the board

Fetch each model with full detail and read `reflect_response.structured_output`. Until the first consolidation finishes it will be empty — render "refreshing…" and poll. Then:

- **Open items**: owner (highlight "unowned"), current due in bold, and a date trail built from `due_history` + `current_due`: `Sep 4 → Sep 7 → Sep 11`. Delivered items in a second list with days late and whether the owner confirmed.
- **Blockers**: a table; days ≥ 7 in red; patterns as callouts.
- **Risk**: milestones with the original date struck through when moved; the release state prominently; risks sorted by severity.

Under each pane, show the model's `last_refreshed_at`. Add the Track 1 chat box beside or below the board.

If you have time, make rows clickable: run `reflect` at budget `high` with a small schema (`dates: [{date, stated_on, by}]`, `delivered_on`, `days_late`, `cause_of_slip`) and show its `based_on` — that's the answer to "why is that a risk?"

## Minute 25–30: check and tighten

Poll bank stats until `pending_consolidation` is 0 and every model reports `is_stale == false` (auto-refresh fires after consolidation; refresh explicitly if one stays stale). Then compare the board with the seven questions. If a pane is wrong or thin:

1. Run `dry_run_refresh` on the model. It reports what was retrieved, what was used, and the diff it would write. If it used 4 things, the query is too broad for one pass.
2. Tighten the query — fewer items, sharper definitions of "owner" and "current." A query that says "be exhaustive" returns forty chores with no history.
3. Refresh and re-read.

## Minute 30–40: transcript 11 and "what changed"

Two layers. Do them in this order.

**Snapshot and diff.** Before retaining transcript 11, save each model's `structured_output`. Retain, wait for consolidation *and* the refreshes, save again. Diff `open-items` by item name (fuzzy — the LLM paraphrases): rows that appeared, rows that vanished, and changes to `owner`, `status`, `current_due`, `delivered_on`. Two rules keep it honest: a "new" row must carry a date on or after Sep 14, otherwise it was merely omitted before; a vanished row only matters if it was open.

**Narrate, anchored.** List the memories for the new document (filter by `document_id` — no LLM). Hand them to one `reflect` together with the *before* snapshot as context:

> A new meeting was just ingested: the Sep 14 standup. Its extracted facts and the board as it stood before the meeting are in context. Report only what this meeting changed: items that closed and whether the owner confirmed; new items with owner and due date; anything that got worse or better, with the new number or date; dates that moved old → new; anything contradicting a recorded decision. Do not restate unchanged items.

Show the diff first, then the narration and its `based_on`. Move the as-of date to Tue Sep 15 9:00 AM for anything you ask afterwards.

## Things that will bite you

- **Models synthesizing from each other** — `exclude_mental_models` before anything else.
- **The refresh is one pass** over a bounded slice of the bank. Narrow questions and small item counts help. Track 3 fixes it properly with tag scoping.
- **Row names drift**, so a naive diff shows everything as new-and-gone. Pair fuzzily and gate by date.
- **Reflect has no `query_timestamp`.** As-of date goes in `context`.
- **Consolidation time** grows with the bank: ~10 min for ten transcripts, 2–4 min for each later one, plus the model refreshes.

## What to show

The board as of Sep 14; one drill-down with a date trail and citations; then transcript 11 loaded live — the diff and the narration. Be ready to answer: which mental model turned out to be the wrong shape, and what did you change about it?
