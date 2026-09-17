# Northwind Lab Guide
**Build an AI project manager with Hindsight — 40-minute build**

This guide is written so you can paste it, whole, into your coding agent. Everything it needs is here or in the files next to it: `manifest.json`, `transcripts/`, and the Hindsight skill.

---

## The situation

A payments team has been shipping a feature for two weeks: settlement reconciliation, a dispute UI, chargeback rules, and a fraud-scoring eval harness. There are transcripts of every meeting. There is no status report, no tracker, no list of open items. The transcripts are the only record.

You are building the project manager's assistant. It reads the transcripts into Hindsight, works out the state of the project, and keeps that picture current as new meetings arrive.

**The team**

| Person | Role |
|---|---|
| Theo Marsh | Project manager. Your app is for him. |
| Maya Lindqvist | Engineering lead. Makes the calls. |
| Jordan Okafor | Backend. Reconciliation job, chargeback rules. |
| Sam Kessler | Frontend. Dispute UI. Also transcribed as "Samantha" and "Sam K". |
| Priya Raman | Data / evals. |
| Ravi Chandrasekaran | Risk team. External. Appears occasionally. |

Talked about but rarely speaking: Nadia (finance, the stakeholder), Dev (sales), Arjun (owns the dispute service), Tomás and Kenji (borrowed labelers), Ops, Ravi's boss, Marcus (Nadia's analyst).

**Fixed dates**
- Sprint 1: Aug 31 – Sep 4 (review Sep 4). Sprint 2: Sep 8 – Sep 11 (review Sep 11). Sprint 3: Sep 14 – 18.
- Stakeholder demo: originally Sep 11.
- Release train cutoff: Wed Sep 16, noon PT. Next train: Sep 30.

**The files**
- `transcripts/01`–`10`: Aug 31 – Sep 11. Ingest first. Raw meeting transcripts, `Speaker (MM:SS): text`, 17–21 minutes each. `06` is a Slack thread.
- `transcripts/11`–`15`: Sep 14 – 18. **Do not ingest until released.** They arrive one at a time.
- `manifest.json`: for every file, the `document_id`, `timestamp`, sprint, meeting type, and suggested tags.

---

## Schedule

| Step | Time |
|---|---|
| Step 0 — Register, install the skill, pick a driver | 2 min |
| Step 1 — Decide your approach | 2 min |
| Step 2 — Decide how you'll use Hindsight | 3 min |
| Step 3 — Design the bank | 6 min |
| Milestone 1 — Ingest 1–10, answer the Sep 14 questions | 15 min |
| Milestone 2 — Transcripts 11, 12, 13, released every 4 minutes | 12 min |

40 minutes total. Transcripts 14 and 15 are stretch: ingest them if you have time, they're in the folder.

---

## Step 0 — Register, install, pick a driver (2 min)

1. Sign up at Hindsight Cloud: https://ui.hindsight.vectorize.io/ — promo code `HOUSTON50`. Copy your API key.
2. Install the Hindsight skill in your coding agent, so it has the API reference and doesn't guess parameter names:
   ```
   npx skills add https://github.com/vectorize-io/hindsight --skill hindsight-docs
   ```
3. Pick one person to drive. Everyone else reads transcripts and designs.
4. Your bank ID is `northwind-<team-name>`.

---

## Step 1 — Decide your approach (2 min)

Pick one. You will not have time to change your mind.

- **Agent harness.** Claude Code, Cursor, or similar, with Hindsight connected as a tool. Theo talks to the agent; the agent calls Hindsight. Fastest to stand up. Weakest at "show me what changed."
- **Custom web app.** A page with panels (open items, blockers, risks, decisions) and a question box. Most work. Best demo.
- **CLI app.** Commands like `pm status`, `pm ingest <file>`, `pm diff`. Middle ground. Good if your driver is fast in a terminal.
- **Notebook.** Cells that call Hindsight and print. Fine for answering questions, poor for the "what changed" moment.

Write your choice at the top of your agent's instructions.

---

## Step 2 — Decide how you'll use Hindsight (3 min)

Hindsight gives you four things. Decide which view or command uses which, before building.

| Operation | What it is | Use it for |
|---|---|---|
| `retain` | Ingest a transcript. Extracts facts, entities, relationships, timestamps. Resolves aliases. Consolidates in the background. | Every transcript, once, whole file, with the meeting timestamp. |
| `recall` | Lookup. No LLM call. Semantic + keyword + graph + temporal retrieval, fused. Supports `query_timestamp`, `tags`, entity-label filters. | Panels that list facts: what's open, who owns what, when. The free-text question box for factual questions. |
| `reflect` | Reasoning loop over memory. Reads mental models, then observations, then facts. Honors the bank's mission, disposition, and directives. Supports `response_schema` and `include_based_on`. | The status report. "What should Theo escalate." "What changed since the last meeting." Anything that needs judgment. |
| Mental models | A standing `source_query` with a stored answer that refreshes when new memories consolidate. | Questions Theo asks every day. Reflect reads these first, so a good set makes reflect faster and more consistent. |

Decide and write down:
- Which panels/commands use recall and which use reflect. Reflect costs tokens and time; a panel that reflects on every load will be slow.
- Whether Theo asks the app questions (pull), the app tells Theo what changed (push), or both. Milestone 2 rewards push.
- Whether "what changed" is a reflect call with a schema, a diff of mental models before and after ingest, or both.

---

## Step 3 — Design the bank (6 min)

Configure the bank **before** ingesting anything. Four decisions.

### 3a. Mission, disposition, directives

These focus reflect on what a PM needs. Set them on the bank, not in your app's prompt.

```python
client.update_bank_config(
    bank_id=BANK,
    mission=(
        "I am the project manager's assistant for the Northwind payments project. "
        "I track commitments with owners and due dates, blockers with age, risks with "
        "when they were raised, and decisions with anything that contradicts them. "
        "I surface slips, dropped items, and unverified claims before they become incidents."
    ),
    disposition={"skepticism": 5, "literalism": 4, "empathy": 2},
    directives=[
        "A completion or status reported by someone other than the item's owner is unverified.",
        "Every flagged item names an owner, or states that it has none.",
        "Report the most recent date given for an item; earlier dates are slips.",
        "Distinguish shipped, live, and approved; they are different states.",
    ],
)
```

Change the wording to match what you decided in Step 2. Reflect follows this. Recall does not.

### 3b. Entity labels — dynamic tags from content

Free-form entity extraction finds names and concepts on its own. Entity labels give the bank a **controlled vocabulary**: dimensions with fixed values that the extractor must classify every fact against. With `tag: true`, the matched `key:value` is also written to the memory's tags, so you can hard-filter recall by it. Label entities resolve by exact match and are never fuzzy-merged.

This is how you get "blocker" or "decision" as a filter without tagging files by hand.

```python
client.update_bank_config(
    bank_id=BANK,
    entity_labels=[
        {
            "key": "item_type",
            "description": "What kind of project-management item this fact is about",
            "type": "multi-values",
            "tag": True,
            "values": [
                {"value": "commitment"},   # someone said they'd do X by Y
                {"value": "blocker"},      # work stopped waiting on someone/something
                {"value": "decision"},     # a choice was made
                {"value": "risk"},         # something that might go wrong, raised but not resolved
                {"value": "milestone"},    # a dated event: review, demo, train, go-live
                {"value": "capacity"},     # availability: PTO, absence, borrowed people
                {"value": "scope"},        # something added, cut, or deferred
            ],
        },
        {
            "key": "person",
            "description": "Team member this fact is primarily about. Sam, Samantha, and Sam K are the same person.",
            "type": "value",
            "tag": True,
            "values": [
                {"value": "theo"}, {"value": "maya"}, {"value": "jordan"},
                {"value": "sam"}, {"value": "priya"}, {"value": "ravi"},
            ],
        },
        {
            "key": "external_party",
            "description": "An outside team or person the fact depends on",
            "type": "value",
            "values": [
                {"value": "risk"}, {"value": "finance"}, {"value": "sales"},
                {"value": "dispute_service"}, {"value": "ops"}, {"value": "vendor"},
            ],
        },
    ],
)
```

Guidance:
- Two dimensions is enough for today. Every label costs extraction attention. Start ingesting the moment the bank config is set; don't tune labels first.
- Prefer enums (`value`, `multi-values`). Use `text` or `multi-text` only for open vocabularies.
- Set `tag: true` only on dimensions you'll filter by.
- The `description` goes straight into the extraction prompt. Write it carefully.
- Declaring the cast as an enum label pins the alias problem: mentions map onto a fixed value by exact match.

### 3c. Explicit tags — set at retain time

These are labels you supply, from things you know before reading the file. They scope recall, reflect, and mental models. Tags use AND matching.

From the manifest:
- `project:northwind`
- `sprint:s1` / `sprint:s2` / `sprint:s3`
- `meeting:standup` / `meeting:review` / `meeting:async` / `meeting:debrief` / `meeting:retro`

Use explicit tags for **provenance** (which sprint, which kind of meeting). Use entity labels for **content** (what kind of item). Don't hand-tag a file `blocker`; the file contains twenty kinds of thing.

Test for a tag: name a query that gives a different answer with it than without. If you can't, drop it.

### 3d. Mental models

Three. Each is a question Theo would ask every day. You don't have time for more. Enable `refresh_after_consolidation` so they update as new transcripts land.

```python
client.create_mental_model(
    bank_id=BANK,
    name="commitments-vs-delivery",
    source_query=(
        "For each team member: what did they commit to, what date did they give, "
        "did the date change, and what actually happened?"
    ),
    refresh_after_consolidation=True,
)
```

Candidates, argue about which:
- Open blockers: what, who, blocked on whom, since when
- Commitments versus delivery, per person
- Decisions made, and anything that has contradicted them since
- Items raised without an owner, or raised once and never mentioned again
- Schedule risk for the next milestone
- What external parties still owe us

Some are stable enough to be mental models. Some change too fast and are better as ad hoc reflect queries. Decide which.

---

## Milestone 1 — Ingest transcripts 1–10 (15 min)

### Ingest

Whole transcript, one call, timestamp from the manifest. Consolidation runs after retain; wait before querying. Verify parameter names against the skill.

```python
import json
from hindsight_client import Hindsight

client = Hindsight(base_url=HINDSIGHT_API_URL, api_key=HINDSIGHT_API_KEY)
BANK = "northwind-<team-name>"
m = json.load(open("manifest.json"))

for t in m["transcripts"]:
    if t["file"].split("/")[-1][:2] not in m["phase_1_files"]:
        continue
    client.retain(
        bank_id=BANK,
        content=open(t["file"]).read(),
        document_id=t["document_id"],
        timestamp=t["timestamp"],
        context="Team meeting transcript for the Northwind payments project. Speakers: Theo (PM), Maya (eng lead), Jordan (backend), Sam/Samantha/Sam K (frontend), Priya (data/evals), Ravi (Risk).",
        tags=t["tags"],
    )
```

### What happens in transcripts 1–10

- **Aug 31.** Kickoff. Dates set: reviews Sep 4 and 11, train Sep 16. Jordan: reconciliation "by Friday." Sam: dispute UI "shouldn't take long," no date. Priya: out Sep 8–11. Jordan sent chargeback rules to Ravi on Aug 28. Demo script has no owner.
- **Sep 1.** Jordan blocked on Risk approval. Decision: reuse ledger idempotency keys, "let's not revisit." Sam: dispute UI "probably Thursday." Priya mentions the merchant-feed vendor changed export format on Aug 26. Dev asks about partial refunds; Maya says not this sprint.
- **Sep 2.** Evals at 91, "was 94." Maya: "keep an eye on it." No owner. Blocker day 2. Maya raises a currency edge case Sam hasn't looked at.
- **Sep 3.** Sam slips to Friday. Ravi joins: compliance review, "around the 10th." There's a form nobody knew about. Nobody knows if there's a separate go-live review.
- **Sep 4 (review).** Reconciliation not done; "a day after the rules land." Dispute UI delivered, one day late. Priya: green on the dashboard means above 85. Partial refunds: Maya says "we can probably fit it." Demo script still unowned. Decision on moving the demo deferred to Tuesday.
- **Sep 7 (Slack).** Jordan: six days blocked. Sam: partial refunds is "not a field and a button."
- **Sep 8.** Priya absent; nobody knows why. Blocker "a week now." Demo moves to Tue Sep 15, the day before the train. Sam sizes partial refunds at a week.
- **Sep 9.** Sam says "since we agreed to build our own idempotency keys" and nobody corrects him. Jordan "glanced" at the eval dashboard, no number. Maya finally owns the demo script, draft Sep 14.
- **Sep 10.** Approval arrived Sep 9 evening; blocker closed after nine days. Reconciliation now due Sep 11. Partial refunds "needs another week," after the train. Ravi: there's "a lighter thing" before go-live. Theo: "Wednesday — no, Tuesday the 15th."
- **Sep 11 (review).** Reconciliation delivered, a week late. Partial refunds decision Monday. Nadia's answer on partial-refund state. The go-live thing is "a twenty-minute meeting, after the train," no date. Maya: nothing is live on train day.

### Questions to answer, as of Sep 14, 9:00 AM (`query_timestamp`)

| Question | What a good answer includes |
|---|---|
| Why is Priya absent and what's exposed? | Announced Aug 31 and Sep 4. Owns the eval regression. Nobody assigned to the dashboard. |
| How long was Jordan blocked and what did it cost? | Sep 1 → Sep 9/10, nine days. Pushed reconciliation Sep 4 → 11, demo Sep 11 → 15. |
| Current due date for reconciliation, and the history? | Sep 4 → "a day after the rules land" → Sep 11 → delivered Sep 11. |
| Is anyone building on a contradicted decision? | Sep 1 decision vs Sep 9 statement. Uncorrected in the meeting. |
| What's the eval regression's history? | Aug 26 feed change → Sep 2 drop → Sep 3 suspicion → unowned till Sep 14. |
| What does "green" mean? | Above 85, stated Sep 4. Forgotten by Sep 9. |
| What's unowned or dropped? | Demo script (till Sep 9), dashboard coverage, regression investigation, Ravi's form. |
| What does Risk still owe us? | The form, rounding-rule approval, the go-live meeting date. |
| What's at risk for Sep 16? | Partial refunds after the train, demo one day before cutoff, evals unresolved, go-live gate undated. |

Every answer should cite the meeting it came from.

---

## Milestone 2 — Transcripts 11, 12, 13, one at a time (12 min)

Ingest each as it's released, using the same code and the manifest. After each one, your app should show: what closed, what opened, what moved, what's contradicted. Watch your mental models change. Transcripts 14 and 15 are listed for completeness; do them if you have time.

| File | What happens | Your app should show |
|---|---|---|
| **11 — Sep 14** | Priya's back, references Aug 31. Evals at 89; she owns it, suspects the Aug 26 batch. Partial refunds cut from this train, moved to Sep 30. Demo script delivered. Green threshold changed to 90. | Regression: owned, severity up. Partial-refunds decision: closed. Script: closed. |
| **12 — Sep 15, 3:15 PM** | Demo. Dispute UI crashes on a euro transaction (the Sep 2 edge case). Sam: fix tonight. Ravi: go-live sign-off Thu Sep 17 8:30, ship dark until then. Mismatch alert will probably be required. | New item with a sub-24h deadline. New gate. Shipped ≠ live. |
| **13 — Sep 16** | Fix merged 11 PM. Root cause found: ~2,000 mislabeled rows from Aug 26; evals 93. Train cut at noon. Flag off. Ravi's form finally arrived (13 days). | Crash: closed on time. Regression: resolved, cause was data not model. Train: met. Not live. |
| **14 — Sep 17** | Sign-off done with one condition: alert on mismatch rate before flipping. Jordan: "should be quick." Flip planned Fri 11 AM. Runbook needed before Sep 21. | New conditional item. Flip date. Flag "should be quick" against Aug 31. |
| **15 — Sep 18, 2 PM** | Live at 11:07. Retro confirms: nine days lost to Risk, PTO forgotten, regression sat 12 days, partial refunds unscoped. Next sprint Sep 21; two engineers to Risk's intake project. | Board clears. Next-sprint items appear. |

Sample questions after each:
- After 11: *What changed on the eval regression?* — owner assigned, number worse, hypothesis stated, deadline is the train.
- After 12: *Are we live on Wednesday?* — no. Shipped Wednesday, dark until Thursday sign-off.
- After 13: *Was the regression the model?* — no. Data. 93 is the first clean number.
- After 14: *Has this team said "should be quick" before, and what happened?* — Aug 31, Sam, delivered a day late.
- After 15: *What's open?* — nearly nothing from the two-week board; partial refunds scope doc Sep 23, runbook before Sep 21, names for Risk by Sep 21.

---

## Demo

Two minutes per team. Show one Milestone 1 answer with its citation, and the "what changed" view after one Milestone 2 transcript. Say which mental models you built and which tags you kept, and why.

---

## For the coding agent

Constraints to follow when building:
- Read the Hindsight skill before writing any Hindsight call. Do not guess method or parameter names.
- Retain whole transcripts, one per call, with `document_id` and `timestamp` from `manifest.json`.
- Configure the bank (mission, disposition, directives, entity labels) before the first retain.
- Consolidation is asynchronous. After retain, poll or wait before querying; do not treat an empty result as a failed ingest.
- Use recall for factual panels and reflect for synthesized ones. Do not call reflect on every page load.
- Pass `query_timestamp` on every time-relative question.
- Pass `include_based_on=True` on reflect and surface the cited transcripts in the UI.
- Keep a "what changed" view that compares state before and after each ingest.
- Do not ingest transcripts 11–15 until told.
