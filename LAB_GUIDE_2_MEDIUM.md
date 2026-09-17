# Northwind Lab — Track 2: Medium

Goal: load two weeks of meeting transcripts into Hindsight, get a chat working, and add a project board that Hindsight keeps up to date by itself — plus a "what changed" summary each time a new meeting is loaded. About 40 minutes, working with a coding agent (Claude Code, Codex, or OpenCode). Any language, any UI.

You don't need to know Hindsight. Each concept is explained where you meet it, and every step is a prompt to paste into your agent.

## Setup

1. **Sign up for Hindsight Cloud** at https://ui.hindsight.vectorize.io/signup and create an API key.
2. Make a project folder and copy this `northwind` folder into it.
3. **Install the Hindsight documentation skill** so your agent has the API reference:
   ```
   npx skills add vectorize-io/hindsight --skill hindsight-docs
   ```
   Pick your agent when prompted.
4. Export your key where the agent will run:
   ```
   export HINDSIGHT_API_KEY=<your key>
   ```
5. Start your agent in the project folder.

## What Hindsight does

You give it raw text — here, whole meeting transcripts. It extracts the facts (who committed to what, by when, who's blocked on whom), with dates and people, into a **memory bank**. Storing is called **retain**; you steer what gets extracted with a short **retain mission**. A few minutes after loading, a background step called **consolidation** merges related facts; answers are incomplete until it finishes.

Answering a question is called **reflect**: it searches the bank, reasons, and returns an answer plus the memories it used. You steer it with a **reflect mission** (who's asking, what a good answer looks like) and **directives** (hard rules every answer must follow).

A **mental model** is a question you ask once and Hindsight keeps answering for you: it stores the answer and re-runs it automatically after each consolidation. If you give it a JSON schema, the answer comes back as data you can render — that is how the board in this track works without you writing any summarization.

## The transcripts

A payments team called Northwind — Theo (PM), Maya (eng lead), Jordan (backend), Sam (frontend, sometimes "Samantha" or "Sam K"), Priya (evals), and Ravi from Risk, with Nadia (finance), Arjun (dispute service), and Ops in the background — held two weeks of standups about a payments feature: settlement reconciliation, a dispute UI, chargeback rules, and fraud-scoring evals. Sprint 1 ran Aug 31 – Sep 4, Sprint 2 Sep 7 – 11; the stakeholder demo was set for Sep 11 and the release train cutoff is Wed Sep 16 noon. Nobody wrote a status report; the transcripts are all there is.

`transcripts/01`–`10` cover Aug 31 – Sep 11. `transcripts/11`–`15` cover Sep 14 – 18; you'll load them one at a time, after the first ten, to watch the answers change. `manifest.json` lists a date, timestamp, and id for every file.

## What the board should answer

As of Monday Sep 14, 9:00 AM, after transcripts 01–10:

- What's open, who owns it, and when is it currently due — with the earlier dates it was given?
- What is or was blocked, on whom, and for how long?
- What is at risk for the Sep 16 release train?

And after each of transcripts 11–15: what changed — what closed, what opened, what moved.

## Timing

| Minutes | Step |
|---|---|
| 0–5 | Prompt 1: bank, rules, start loading |
| 5–12 | Prompt 2: mental models |
| 12–25 | Prompt 3: board; Prompt 4: chat |
| 25–30 | Check the board; Prompt 5 if a pane is thin |
| 30–40 | Prompt 6: load transcripts 11–15 one at a time and show what changed |

## Prompt 1 — bank, rules, start loading

Creates the bank, tells Hindsight what to extract from a standup, adds four rules for answers, and starts loading. The rules exist because these transcripts have traps: dates that get restated, a feature that ships behind a flag, a "done" that only one person claims.

> Use the hindsight-docs skill. Write a script that connects to Hindsight Cloud with the API key in `HINDSIGHT_API_KEY`, creates a memory bank named `northwind-<team>`, and loads the phase-1 transcripts listed in `northwind/manifest.json`.
>
> Before loading, configure the bank:
> - `retain_mission`: "These are raw meeting transcripts. Extract commitments and their due dates — resolve 'Friday' or 'tomorrow' to actual dates using the meeting date, and keep every restatement of a date, including corrections within the same meeting. Extract blockers and who they are waiting on, decisions, PTO and absences, new requests and scope changes, and numbers such as eval scores. Sam, Samantha, and Sam K are the same person. Ignore greetings and logistics."
> - `retain_extraction_mode`: `verbose`.
> - `reflect_mission`: "You are chief of staff to Theo, the project manager. For every item give the owner (or 'unowned'), the current due date, earlier dates as history, and the meeting it came from. Treat the date of the latest meeting in memory as today."
> - Disposition: skepticism 4, literalism 4, empathy 1.
>
> Create four directives:
> 1. "The most recent statement of a date is current; list earlier dates as history with the meeting each was stated in. A correction made within a meeting wins."
> 2. "Merged, on the release train, and behind a feature flag are not live. Always say which applies."
> 3. "A completion is verified when the owner states it with specifics. A second-hand report, or a non-owner who glanced at a dashboard, is unverified. Use the word 'unverified' only in those cases."
> 4. "When a later statement contradicts a recorded decision, report both and say whether anyone corrected it."
>
> For each transcript whose number is in `phase_1_files`, retain the whole file with the manifest's `document_id` and `timestamp`, a `context` like "Northwind team standup transcript, 2026-09-08", and the `sprint:` tag from the manifest. Use asynchronous retain.
>
> Also write a `wait_until_settled()` function that polls the bank stats until `pending_consolidation` is 0 and no operations are pending, then waits until no mental model is marked stale. Don't call it now. Print "loading started" and exit.

**Check:** the retain call passes the manifest's `timestamp` and only the `sprint:` tag. Don't let the agent add tags like `blocker` — Hindsight extracts those itself.

## Prompt 2 — mental models

Three standing questions, each with a schema so the answer is data. They will refresh on their own once loading finishes. The `exclude_mental_models` setting matters: without it, a model may build its answer from the *other* models instead of from the memories, and they end up repeating each other's mistakes.

> Use the hindsight-docs skill. Create three mental models. On each, set the trigger to `refresh_after_consolidation: true` and `exclude_mental_models: true`, and give it a JSON `response_schema`.
>
> 1. id `open-items`. Question: "Which significant commitments has the team made — deliverables a milestone or stakeholder depends on, decisions someone owes, requests never given an owner — and for each: the owner (whoever most recently agreed to do it, or 'unowned'), open / delivered / lapsed, the current due date, every earlier due date with the meeting it was stated in, when it was delivered, days late, and whether the owner confirmed. About ten items; prioritize ones whose date moved, whose owner changed, or which nobody owns." Schema: `items[]` with `item, owner, status, current_due, due_history[{date, stated_on}], raised_on, delivered_on, days_late (integer), owner_confirmed (boolean)`.
> 2. id `blockers`. Question: "What is or was blocked on someone outside the team — Risk/Ravi, Arjun, Nadia, Ops: the owner of the blocked work, who it waited on, when it opened and closed, how many days, and what it delayed. Has the same outside party gated the team more than once?" Schema: `gates[]` with `what, owner, blocked_on, opened_on, closed_on, duration_days (integer), delayed`; and `patterns[]` of strings.
> 3. id `risk`. Question: "Milestones — demo, train cutoff, go-live — with original and current dates and when they changed; whether the feature is merged, shipped, dark, or live; and what is at risk for the next milestone: unowned items, metrics moving the wrong way with nobody investigating, estimates landing after the milestone, milestones stacked with no buffer." Schema: `milestones[]` with `name, original_date, current_date, changed_on, status`; `release_state` string; `risks[]` with `risk, severity, why, since, owner`.

## Prompt 3 — the board

Renders the three models' data. Nothing here calls a language model; the models already hold the answers.

> Use the hindsight-docs skill. Build a board page. Fetch each mental model with full detail and render its `reflect_response.structured_output`. Until the first refresh has run, show "refreshing…" and poll every 10 seconds.
> - Open items: owner (highlight "unowned"), current due date in bold, and a date trail built from `due_history` plus `current_due`, e.g. `Sep 4 → Sep 7 → Sep 11` with earlier dates struck through. Delivered items below, with days late and whether the owner confirmed.
> - Blockers: a table; 7 days or more in red; patterns above it.
> - Risk: milestones with the original date struck through when it moved; the release state shown prominently; risks by severity.
> Show each model's `last_refreshed_at` under its pane.

## Prompt 4 — the chat

For anything the board doesn't cover. The date in the context matters: without it Hindsight reasons from today's real date.

> Use the hindsight-docs skill. Add a chat box next to the board. Each question is one `reflect` call at budget `mid` with the include-facts option on. Keep an `AS_OF` string, initially "Monday September 14, 2026, 9:00 AM Pacific", and pass it in the `context` as "Today is {AS_OF}. Answer as of this moment." Show the answer, then the memories from `based_on` with the first ten characters of each one's `occurred_start`.

## Check the board

Tell the agent: *"Run `wait_until_settled()` and tell me when it's done."* Then look at the board against the three questions above. For open items, the reconciliation job should show more than one date and a delivery on Sep 11; the demo should show as moved from Sep 11 to Sep 15.

## Prompt 5 — if a pane is thin or wrong

Hindsight can show you what a model's refresh actually read. If it read a lot but used little, the question is too broad.

> Use the hindsight-docs skill. Run `dry_run_refresh` on `open-items` and show me `facts.retrieved`, `facts.used`, and the diff it would write.

Then ask the agent to narrow the question (fewer items, a sharper definition of "owner" and "current") and refresh the model.

## Prompt 6 — load the next meeting and show what changed

This saves the board, loads the transcript, waits, saves the board again, diffs the two, and asks Hindsight to narrate the difference using the new meeting's own facts.

> Use the hindsight-docs skill. Add an `ingest_next(n)` command that:
> 1. Saves every mental model's `structured_output` as a "before" snapshot.
> 2. Retains transcript n from the manifest with the same settings as the others.
> 3. Calls `wait_until_settled()`.
> 4. Saves an "after" snapshot.
> 5. Diffs `open-items` between the two by pairing rows on item name with fuzzy matching, and reports rows that appeared, rows that vanished, and changes to `owner`, `status`, `current_due`, `delivered_on`. Count a "new" row only if it has a date on or after the new meeting's date; count a vanished row only if it was open.
> 6. Lists the memories stored for the new document (filter by `document_id`) and makes one `reflect` call whose context contains those memories and the "before" snapshot, asking: "A new meeting was just loaded. Report only what it changed: items that closed and whether the owner confirmed; new items with owner and due date; anything that got worse or better; dates that moved, old to new; anything contradicting a recorded decision. Do not restate unchanged items."
>
> Show the diff, then the narration with its `based_on` memories.

Run `ingest_next(11)`, set the chat's `AS_OF` to Tuesday Sep 15, and look at what moved. Then run it for 12, 13, 14, and 15, one at a time.

## Demo

The board as of Sep 14, then transcript 11 loaded live: the diff and the narration.
