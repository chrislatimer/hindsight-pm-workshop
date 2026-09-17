# Northwind Lab — Track 3: Advanced

Goal: load two weeks of meeting transcripts into Hindsight with **entity labels**, so every extracted fact is tagged by workstream, kind, and owner as it's stored. Use those tags to build filtered views with no language model, mental models scoped to one topic each, a board that reads from them, a chat, and a "what changed" summary after each new meeting. About 40 minutes, working with a coding agent (Claude Code, Codex, or OpenCode). Any language, any UI.

You don't need to know Hindsight. Each concept is explained where you meet it, and every step is a prompt to paste into your agent. One thing is different from the other tracks: the labels have to be configured **before** the first transcript is loaded, so Prompt 1 is a little longer. Don't skip ahead.

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

You give it raw text — here, whole meeting transcripts. It extracts the facts (who committed to what, by when, who's blocked on whom), with dates and people, into a **memory bank**. Storing is called **retain**; a **retain mission** steers what gets extracted. A few minutes after loading, **consolidation** merges related facts in the background; answers are incomplete until it finishes.

Answering a question is **reflect**: it searches the bank, reasons, and returns an answer plus the memories it used. A **reflect mission** and **directives** (hard rules) steer it.

A **mental model** is a question you ask once and Hindsight keeps answering: it stores the answer and re-runs it after each consolidation. With a JSON schema, the answer is data you can render.

**Tags** are labels on memories that let you narrow a question or a mental model to a subset. You can set tags yourself when you load a file — tonight, which sprint the meeting was in. The subject of this track is the other kind: **entity labels**, where you define categories up front (a list of workstreams, a list of fact kinds, an owner field) and Hindsight assigns them to each fact as it extracts it. With `tag: true`, each assigned label is written as a tag. So you get tags like `kind:blocker` and `owner:sam` on individual facts without anyone reading the transcripts — and a mental model scoped to `workstream:reconciliation` reads only the reconciliation facts, which is small enough for it to be thorough.

## The transcripts

A payments team called Northwind — Theo (PM), Maya (eng lead), Jordan (backend), Sam (frontend, sometimes "Samantha" or "Sam K"), Priya (evals), and Ravi from Risk, with Nadia (finance), Arjun (dispute service), and Ops in the background — held two weeks of standups about a payments feature: settlement reconciliation, a dispute UI, chargeback rules, and fraud-scoring evals. Sprint 1 ran Aug 31 – Sep 4, Sprint 2 Sep 7 – 11; the stakeholder demo was set for Sep 11 and the release train cutoff is Wed Sep 16 noon. Nobody wrote a status report; the transcripts are all there is.

`transcripts/01`–`10` cover Aug 31 – Sep 11. `transcripts/11`–`15` cover Sep 14 – 18; you'll load them one at a time, after the first ten, to watch the answers change. `manifest.json` lists a date, timestamp, and id for every file.

## What the app should answer

As of Monday Sep 14, 9:00 AM, after transcripts 01–10:

- What's open, who owns it, and when is it currently due — with the earlier dates it was given?
- What is or was blocked, on whom, and for how long? (The filtered view should show this with no language model call.)
- What decisions have been made, and has anything contradicted them?
- Who has been out, and what did that affect?
- What is at risk for the Sep 16 release train?

After each of transcripts 11–15: what changed — what closed, what opened, what moved, what contradicts.

## Timing

| Minutes | Step |
|---|---|
| 0–5 | Prompt 1: bank with labels, rules, start loading |
| 5–12 | Prompt 2: scoped mental models |
| 12–25 | Prompt 3: memory view with filters; Prompt 4: board; Prompt 5: chat |
| 25–30 | Prompt 6: check the labels and one model |
| 30–40 | Prompt 7: load transcripts 11–15 one at a time and show what changed |

## Prompt 1 — bank with labels, rules, start loading

Creates the bank, defines the three label groups, adds four answer rules, and starts loading. The label descriptions are what the extraction model reads, so they carry the meaning.

> Use the hindsight-docs skill. Write a script that connects to Hindsight Cloud with the API key in `HINDSIGHT_API_KEY`, creates a memory bank named `northwind-<team>`, configures it — entity labels included — and only then loads the phase-1 transcripts listed in `northwind/manifest.json`. Read the bank config back and confirm the labels are present before the first retain.
>
> Bank configuration:
> - `retain_mission`: "These are raw meeting transcripts. Extract commitments and their due dates — resolve 'Friday' or 'tomorrow' to actual dates using the meeting date, and keep every restatement of a date, including corrections within the same meeting. Extract blockers and who they are waiting on, decisions, PTO and absences, new requests and scope changes, and numbers such as eval scores. Sam, Samantha, and Sam K are the same person. Ignore greetings and logistics."
> - `retain_extraction_mode`: `verbose`.
> - `reflect_mission`: "You are chief of staff to Theo, the project manager. For every item give the owner (or 'unowned'), the current due date, earlier dates as history, and the meeting it came from. Treat the date of the latest meeting in memory as today."
> - Disposition: skepticism 4, literalism 4, empathy 1.
>
> Entity labels (`entity_labels` in the bank config), each with `tag: true` and `optional: true`:
> 1. key `workstream`, type `value`, description "Which piece of the payments project this fact is about. Leave unset for logistics or chit-chat." Values, each with a description: `reconciliation` (settlement reconciliation job, its report, settlement files, retry, mismatch alert); `chargeback-rules` (chargeback rules, Risk/compliance approval, compliance form, rounding rule, go-live review); `dispute-ui` (dispute list and detail UI, modals, currency handling, Arjun's dispute service); `partial-refunds` (the partial refunds feature, its scope, idempotency keys for refunds); `evals` (fraud-scoring evals, eval dashboard and thresholds, labels and labelers, model candidate); `release` (stakeholder demo, demo script, release train cutoff, feature flag, go-live).
> 2. key `kind`, type `value`, description "What kind of project-management fact this is." Values: `commitment` (someone agreed to do something), `delivery` (something reported done, merged, shipped, or live), `blocker` (work waiting on someone or something), `decision` (a choice the team made), `risk` (a concern or something that could go wrong), `availability` (PTO, absence, someone joining or leaving), `scope-change` (a new ask or a change in what is being built), `metric` (a number: eval score, PR count, mismatch rate).
> 3. key `owner`, type `text`, description "Canonical lowercased first name of the person responsible: theo, maya, jordan, sam (also for Samantha and Sam K), priya, ravi, arjun, nadia, ops. Unset if nobody owns it."
>
> Four directives:
> 1. "The most recent statement of a date is current; list earlier dates as history with the meeting each was stated in. A correction made within a meeting wins."
> 2. "Merged, on the release train, and behind a feature flag are not live. Always say which applies."
> 3. "A completion is verified when the owner states it with specifics. A second-hand report, or a non-owner who glanced at a dashboard, is unverified. Use the word 'unverified' only in those cases."
> 4. "When a later statement contradicts a recorded decision, report both and say whether anyone corrected it."
>
> Then, for each transcript whose number is in `phase_1_files`, retain the whole file with the manifest's `document_id` and `timestamp`, a `context` like "Northwind team standup transcript, 2026-09-08", and the `sprint:` tag from the manifest. Use asynchronous retain.
>
> Also write a `wait_until_settled()` function that polls the bank stats until `pending_consolidation` is 0 and no operations are pending, then waits until no mental model is marked stale. Don't call it now. Print "loading started" and exit.

**Check:** the script read the config back and the three label groups were there *before* the first retain. Labels added afterwards don't apply to facts already stored. Label types available on this server: `value`, `multi-values`, `text`, `map`.

## Prompt 2 — scoped mental models

Nine standing questions, each limited by tag to one slice of the bank. A model with several tags needs `tags_match: any_strict` or it reads only facts that carry all of them at once, which is nothing. `exclude_mental_models` stops models from building answers out of each other.

> Use the hindsight-docs skill. Create nine mental models. On every one, set the trigger to `refresh_after_consolidation: true` and `exclude_mental_models: true`, and give it a JSON `response_schema`.
>
> Six workstream models, ids `ws-reconciliation`, `ws-chargeback-rules`, `ws-dispute-ui`, `ws-partial-refunds`, `ws-evals`, `ws-release`, each tagged `workstream:<name>`. Question for each: "For the <name> workstream — <the description from the label> — which commitments has the team made, and for each: the owner (whoever most recently agreed to do it, or 'unowned'), open / delivered / lapsed, the current due date, every earlier due date with the meeting it was stated in, when it was delivered, days late, and whether the owner confirmed. Include requests that were never given an owner. Merge restatements of the same item into one row." Schema: `items[]` with `item, owner, status, current_due, due_history[{date, stated_on}], raised_on, delivered_on, days_late (integer), owner_confirmed (boolean)`.
>
> `blockers`, tagged `kind:blocker`. Question: "What is or was blocked on someone outside the team — Risk/Ravi, Arjun, Nadia, Ops: the owner of the blocked work, who it waited on, when it opened and closed, how many days, what it delayed. Has the same outside party gated the team more than once?" Schema: `gates[]` with `what, owner, blocked_on, opened_on, closed_on, duration_days (integer), delayed`; `patterns[]` of strings.
>
> `decisions`, tagged `kind:decision` and `kind:scope-change`, with `tags_match: any_strict`. Question: "What decisions has the team made, who made them, in which meeting, and has anything said later contradicted or reversed them — who said it, and did anyone correct them?" Schema: `decisions[]` with `decision, decided_by, decided_on, status (standing | contradicted | reversed), contradicted_by`.
>
> `risk`, tagged `workstream:release`, `kind:risk`, `kind:metric`, `kind:blocker`, with `tags_match: any_strict`. Question: "Milestones — demo, train cutoff, go-live — with original and current dates and when they changed; whether the feature is merged, shipped, dark, or live; and what is at risk for the next milestone: unowned items, metrics moving the wrong way with nobody investigating, estimates landing after the milestone, milestones stacked with no buffer." Schema: `milestones[]` with `name, original_date, current_date, changed_on, status`; `release_state` string; `risks[]` with `risk, severity, why, since, owner`.

## Prompt 3 — memory view with filters

Shows what Hindsight extracted from each transcript and how it labelled it, with filters that run as plain tag lookups — no language model. This works as soon as the first transcript lands, before consolidation.

> Use the hindsight-docs skill. Build a memory page. On the left, list the 15 transcripts from the manifest and mark which are loaded. Selecting one lists its stored memories (filter by `document_id`) with each memory's date, text, and tags; show the `workstream:`, `kind:`, and `owner:` tags as small labels on each memory.
>
> Above the list, show filter chips for each label group with counts from the current list; selecting chips narrows the list to memories carrying all selected tags. Add a switch, "this meeting / all meetings": in all-meetings mode, query the bank's memory list endpoint with the selected tags and `tags_match=all_strict`, and show the results from every meeting, each with the meeting it came from.

Once a few transcripts are in: `kind:blocker` in all-meetings mode should be the chargeback-rules thread with Ravi; `kind:availability` should be Priya's time off; `owner:sam` with `kind:commitment` should be everything Sam took on, however she was named.

## Prompt 4 — board

Reads the nine models' data. The six workstream models merge into one open-items list.

> Use the hindsight-docs skill. Build a board page that reads each mental model's `reflect_response.structured_output`. Merge the six `ws-*` models' `items` into one open-items pane, stamping each row with its workstream. Render `blockers`, `decisions`, and `risk` from their own models.
> - Open items: owner (highlight "unowned"), current due date in bold, workstream label, and a date trail built from `due_history` plus `current_due`, e.g. `Sep 4 → Sep 7 → Sep 11` with earlier dates struck through. Delivered items below, with days late and whether the owner confirmed.
> - Blockers: a table; 7 days or more in red; patterns above it.
> - Decisions: contradicted or reversed ones first, with the contradicting statement; standing decisions in a collapsed list.
> - Risk: milestones with the original date struck through when it moved; the release state shown prominently; risks by severity.
> Show each model's `last_refreshed_at` and whether it is stale. Make each open item clickable: on click, run `reflect` at budget `high` for that item's full history with a schema `dates[{date, stated_on, by, note}], current_due, delivered_on, days_late, cause_of_slip, owner_confirmed, release_state`, and show the answer with its `based_on` memories and their dates.

## Prompt 5 — chat

With mental models in the bank, reflect will often answer from them — fast, but without per-memory citations. The toggle lets you show both behaviours.

> Use the hindsight-docs skill. Add a chat page. Each question is one `reflect` call with the include-facts option on. Keep an `AS_OF` string, initially "Monday September 14, 2026, 9:00 AM Pacific", and pass it in the `context` as "Today is {AS_OF}. Answer as of this moment." Add a budget selector (low / mid / high) and a toggle for `exclude_mental_models`. Show the answer, the latency, which mental models it consulted, and the `based_on` memories with the first ten characters of each one's `occurred_start`.

## Prompt 6 — check the labels and one model

Tell the agent *"Run `wait_until_settled()` and tell me when it's done"*, then:

> Use the hindsight-docs skill. Count tags across all memories in the bank: how many carry a `workstream` tag, a `kind` tag, an `owner` tag; the distribution of values for each; and how many carry no label at all. Then run `dry_run_refresh` on `ws-reconciliation` and show `facts.retrieved`, `facts.used`, and the diff it would write.

What you want to see: nearly every memory labelled; `workstream` on most of them; one `owner` value per person (if both `owner:sam` and `owner:samantha` appear, sharpen the owner description and re-run the labels on a fresh bank); and the scoped model using most of what it retrieved. Its content should already say the reconciliation job was delivered Sep 11, seven days late, owner-confirmed, blocked on compliance. Then check the board against the questions above.

## Prompt 7 — load the next meeting and show what changed

Saves the board, loads the transcript, waits, saves again, counts the new meeting's facts by kind, diffs the open items, and asks Hindsight to narrate the difference.

> Use the hindsight-docs skill. Add an `ingest_next(n)` command that:
> 1. Saves every mental model's `structured_output` as a "before" snapshot.
> 2. Retains transcript n from the manifest with the same settings as the others.
> 3. Calls `wait_until_settled()`.
> 4. Saves an "after" snapshot.
> 5. Lists the memories stored for the new document (filter by `document_id`) and groups them by `kind` tag, e.g. "3 commitments, 1 blocker, 2 decisions".
> 6. Diffs the merged open items between the two snapshots by pairing rows on item name with fuzzy matching, and reports rows that appeared, rows that vanished, and changes to `owner`, `status`, `current_due`, `delivered_on`. Count a "new" row only if it has a date on or after the new meeting's date; count a vanished row only if it was open.
> 7. Makes one `reflect` call whose context contains the new document's memories and the "before" snapshot, asking: "A new meeting was just loaded. Report only what it changed: items that closed and whether the owner confirmed; new items with owner and due date; anything that got worse or better; dates that moved, old to new; anything contradicting a recorded decision. Do not restate unchanged items."
>
> Show the counts by kind, then the diff, then the narration with its `based_on` memories.

Run `ingest_next(11)` and set the chat's `AS_OF` to Tuesday Sep 15. Afterwards, `kind:decision` in the all-meetings view should show a Sep 14 decision about partial refunds at the end of the list, and `ws-evals` should show the eval regression with an owner and a higher severity. Then run it for 12, 13, 14, and 15, one at a time.

## Things that go wrong

- Labels added after loading apply to nothing already stored. If you must change them, start a fresh bank.
- A model with several tags reads nothing unless `tags_match` is `any_strict`.
- A value not in a label's list is dropped silently; if a workstream is missing from the list, those facts get no workstream tag. That is what Prompt 6 checks.
- Nine auto-refreshing models add about a minute to each wait.

## Demo

The memory view with `kind:blocker` across all meetings; one scoped model's content next to its `dry_run_refresh` numbers; the board; transcript 11 loaded live with counts by kind, the diff, and the narration.
