# Northwind Lab — Track 3: Advanced
**Derived tags with entity labels, tag-scoped mental models, and hard-filtered views. 40 minutes.**

Any language, any UI. This track changes the bank configuration *before* the first retain, so read the "Minute 0–5" section first and run it immediately — labels are extraction-time config and cannot be added to memories that already exist. Team, calendar, and files are in `LAB_GUIDE.md`; Track 2 (`LAB_GUIDE_2_MEDIUM.md`) covers directives, schemas, and the "what changed" pattern that this track reuses.

## Why this track exists

Track 2's mental models answer one broad question over the whole bank, and a refresh reads a bounded slice of ~800 memories in one pass — so they come out thin. The fix is not a bigger budget; it is scope. If Hindsight labels every fact with the workstream it belongs to and the kind of fact it is, a mental model can be scoped by tag to *only* the reconciliation facts, or *only* the blockers. Each refresh then reads a slice small enough to cover completely, and the same tags give you instant, LLM-free views: every blocker, every decision, everything Sam owns.

The lab guide's rule "don't tag by content" is about tags **you** set at ingest — a tag has to be knowable before anyone reads the file. Entity labels are the other half of that sentence: Hindsight extracts the classification itself, per fact, and (with `tag: true`) writes it as a tag.

## The 40-minute plan

| Minute | Do this | Why now |
|---|---|---|
| 0–5 | Bank config **with entity labels** + directives + **start ingesting 01–10** | Labels must exist before retain. Consolidation takes ~10 min. |
| 5–12 | Create tag-scoped mental models (per workstream + per kind) | They refresh after the first consolidation. |
| 12–25 | Build the memory view: extracted facts per meeting with their tags; filter chips; "all meetings" hard filter. Board from the scoped models. | The facts view works as soon as the first retain lands — no waiting on consolidation. |
| 25–30 | Verify the tags, verify one scoped model with `dry_run_refresh` | Know what your labels actually produced before you demo them. |
| 30–40 | Transcript 11: snapshot → retain → wait → what changed (facts by tag + narration) | The demo moment. |

## Minute 0–5: bank with entity labels

Start from Track 1's `create_bank` (missions, `verbose`, disposition) and Track 2's directives, then add label groups to the bank config **before the first retain**:

```python
ENTITY_LABELS = [
    {"key": "workstream", "type": "value", "tag": True, "optional": True,
     "description": "Which piece of the payments project this fact is about. Leave unset for logistics or chit-chat.",
     "values": [
        {"value": "reconciliation", "description": "settlement reconciliation job, its report, settlement files, retry, mismatch alert"},
        {"value": "chargeback-rules", "description": "chargeback rules, Risk/compliance approval, compliance form, rounding rule, go-live review"},
        {"value": "dispute-ui", "description": "dispute list/detail UI, modals, currency handling, Arjun's dispute service"},
        {"value": "partial-refunds", "description": "the partial refunds feature, its scope, idempotency keys for refunds"},
        {"value": "evals", "description": "fraud-scoring evals, eval dashboard and thresholds, labels and labelers, model candidate"},
        {"value": "release", "description": "stakeholder demo, demo script, release train cutoff, feature flag, go-live"},
     ]},
    {"key": "kind", "type": "value", "tag": True, "optional": True,
     "description": "What kind of project-management fact this is.",
     "values": [{"value": v, "description": d} for v, d in [
        ("commitment", "someone agreed to do something"), ("delivery", "something reported done, merged, shipped, or live"),
        ("blocker", "work waiting on someone or something"), ("decision", "a choice the team made"),
        ("risk", "a concern or something that could go wrong"), ("availability", "PTO, absence, someone joining or leaving"),
        ("scope-change", "a new ask or a change in what is being built"), ("metric", "a number: eval score, PR count, mismatch rate")]]},
    {"key": "owner", "type": "text", "tag": True, "optional": True,
     "description": "Canonical lowercased first name of the person responsible: theo, maya, jordan, sam (also for Samantha / Sam K), priya, ravi, arjun, nadia, ops. Unset if nobody owns it."},
]
client.update_bank_config(BANK, entity_labels=ENTITY_LABELS)
```

Notes before you run it:

- The `description` on each group and value is what the LLM sees. Spend your words there.
- `optional: True` keeps the model from forcing a workstream onto "we're late, recording."
- `owner` as a `text` label with a canonical-name instruction doubles as an alias normalizer: Samantha and Sam K both become `owner:sam`. This workshop's server accepts label types `value`, `multi-values`, `text`, and `map` (not `multi-text`).
- Keep the `sprint:` manual tag from the manifest. You'll be able to show a manual tag and a derived tag doing different jobs.

Then kick off the ingest of 01–10 exactly as in Track 1.

## Minute 5–12: tag-scoped mental models

A mental model with `tags` reads only memories carrying **all** of them (`all_strict`) — or any of them if you set `trigger.tags_match` to `any_strict`. Use that to make each model's slice small:

| id | tags | source_query |
|---|---|---|
| `ws-reconciliation`, `ws-chargeback-rules`, `ws-dispute-ui`, `ws-partial-refunds`, `ws-evals`, `ws-release` | `["workstream:<x>"]` | Track 2's open-items question, prefixed with "for this workstream — <description>". Same schema. |
| `blockers` | `["kind:blocker"]` | Track 2's blockers question. |
| `decisions` | `["kind:decision", "kind:scope-change"]`, `tags_match: "any_strict"` | Decisions, who made them, when, and anything that later contradicts or reverses them. |
| `risk` | `["workstream:release", "kind:risk", "kind:metric", "kind:blocker"]`, `tags_match: "any_strict"` | Track 2's risk question. |

```python
for key, desc in WORKSTREAMS:          # the six (value, description) pairs above
    client.create_mental_model(
        bank_id=BANK, id=f"ws-{key}", name=f"Workstream: {key}",
        source_query=f"For this workstream — {desc} — " + OPEN_ITEMS_QUERY,
        tags=[f"workstream:{key}"],
        trigger={"refresh_after_consolidation": True, "exclude_mental_models": True, "response_schema": OPEN_ITEMS_SCHEMA},
    )
```

Create the six workstream models plus `blockers`, `decisions`, and `risk`. That's nine refreshes per consolidation; each is small.

## Minute 12–25: the memory view and the board

Build the **memory view first** — it works the moment the first retain completes, long before consolidation:

- A list of the 15 meetings; clicking one lists its memories (filter by `document_id`, no LLM). Show each memory's text **and its tags** — `workstream:evals · kind:risk · owner:priya` — so the room sees the classification happen.
- **Filter chips** for each label group, with counts from the current list. Selecting `kind:blocker` narrows the list.
- An **"all meetings"** switch: the same tags run against the whole bank as a hard filter (the memories list endpoint accepts `tags` + `tags_match=all_strict`; `recall` with `tags_match="any_strict"` also works and ranks). `kind:blocker` across all meetings is the entire Ravi thread from Aug 28 onward. `kind:availability` is Priya's PTO and nothing else. `owner:sam` + `kind:commitment` is everything she took on, however she was named.

Then the **board**, read straight from the scoped models' `structured_output`: merge the six `ws-*` models' `items` into one open-items pane (stamp each row with its workstream), and render `blockers` / `decisions` / `risk` from theirs. No app-side reflect is needed in the load path; the models are the board.

Add the Track 1 chat. With scoped models available, `reflect` will often answer from them (fast, no per-meeting citations); give the chat a toggle for `exclude_mental_models` so you can show both behaviours — the pre-computed answer in 5 seconds versus the memory-grounded one with 150+ citations in 15.

## Minute 25–30: verify the labels

Before demoing them, look at what the labels produced:

- Count tags across all memories: how many carry a `workstream`? A good run labels nearly every fact; `workstream` covers ~85–90 %, and `kind:commitment` dominates because it's a standup.
- Check `owner`: you want one value per person. If you see both `owner:sam` and `owner:samantha`, sharpen the description.
- `dry_run_refresh` one `ws-*` model. Compare `facts.retrieved` vs `facts.used` with what an unscoped model gives: scoped, the refresh should use most of what it retrieved. Read the diff it would write — the reconciliation model should already know the job was delivered Sep 11, seven days late, owner-confirmed, blocked on compliance.

## Minute 30–40: transcript 11 and "what changed"

Same shape as Track 2 — snapshot the models' `structured_output`, retain, wait for consolidation and the refreshes, snapshot, diff — with one upgrade: the deterministic half can now be **facts by tag**. List the new document's memories and group them by `kind`: *3 commitments, 1 blocker, 2 decisions, 1 availability, 2 metrics*. Those counts plus the row diff are the report; one anchored `reflect` (Track 2's prompt, with the new facts and the previous board in context) narrates it.

After transcript 11 lands, `kind:decision` in the all-meetings view should show the partial-refunds decision from Sep 14 at the bottom of the list, and `ws-evals` should have changed owner and severity on the regression. Point at both.

## Things that will bite you

- **Labels are extraction-time.** Add them after retaining and nothing old gets tagged. If you must change them, re-ingest (same `document_id` replaces).
- **Multi-tag models default to `all_strict`** — a model tagged `["kind:decision", "kind:scope-change"]` reads nothing unless you set `tags_match: "any_strict"`.
- **`exclude_mental_models`** on every model, still.
- **The label enum drops off-list values silently.** If a workstream is missing from your `values`, those facts get no workstream tag at all — check coverage.
- **Refresh count.** Nine or ten auto-refreshing models add a minute to each settle. Fine for the lab; worth `min_refresh_interval_seconds` in real life.

## What to show

The memory view with `kind:blocker` across all meetings (hard filter, no LLM); one scoped model's content next to its `dry_run_refresh` numbers; the board reading from the models; transcript 11 loaded live with facts-by-kind and the narration. Be ready to argue: which of these should have been a manual tag, which a label, and which neither?
