# Northwind Lab — Track 1: Basic
**Load the transcripts into Hindsight and chat with them. 40 minutes.**

Any language, any UI — a terminal loop is fine. The Python and Node SDKs expose the same operations (Node uses camelCase). Team, calendar, and file descriptions are in `LAB_GUIDE.md`; you don't need to read them to start.

## The 40-minute plan

| Minute | Do this | Why now |
|---|---|---|
| 0–5 | Configure the bank, **start ingesting transcripts 01–10** | Extraction + consolidation takes ~10 min. Start it before anything else. |
| 5–20 | Write the chat loop (below). Test against whatever has landed so far. | Facts appear per transcript as each retain finishes; you don't have to wait for all ten. |
| 20–30 | Run the seven questions. Fix the bank config, re-ask. | This is where the points are. |
| 30–40 | Transcript 11 is released: retain it, wait, ask "what changed?" | The demo moment. |

## The situation in one paragraph

A payments team (Northwind) has held two weeks of standups about settlement reconciliation, a dispute UI, chargeback rules, and a fraud-scoring eval harness. Transcripts 01–10 (Aug 31 – Sep 11) are the only record. The PM, Theo, has no status report and a release train cutoff on Wed Sep 16 noon. Build him a chat that answers questions about the project **with evidence**. Transcripts 11–15 arrive one at a time later; **don't ingest them until told to.**

## What your chat must answer

As of **Mon Sep 14, 2026, 9:00 AM**, after transcripts 01–10:

1. What's open, who owns it, and when is it *currently* due (not the first date it was given)?
2. What has slipped, and by how much?
3. What is or was blocked, on whom, and for how long?
4. What decisions have been made? Has anything since contradicted them?
5. Who is or was unavailable, and what did that affect?
6. What is at risk for the Sep 16 release train?
7. What was raised once and never followed up on?

After each new transcript: **what changed since the last meeting?**

## Minute 0–5: bank + ingest

Banks auto-create with defaults. Don't accept them — the configuration is most of the work in this track. Paste this, fill in your bank id, run it.

```python
import json
from hindsight_client import Hindsight

client = Hindsight(base_url=HINDSIGHT_API_URL, api_key=HINDSIGHT_API_KEY)
BANK = "northwind-<your-team>"

client.create_bank(
    bank_id=BANK,
    name="Northwind PM",
    # What to extract. These are meeting transcripts; the important things are said once, in passing.
    retain_mission=(
        "Raw meeting transcripts. Extract: commitments and their due dates — resolve 'Friday' or 'tomorrow' "
        "to a real date using the meeting date, and keep EVERY restatement of a date, including in-meeting "
        "self-corrections; blockers and who they are waiting on; decisions; PTO and absences; new asks and "
        "scope changes; numbers such as eval scores. Sam, Samantha, and Sam K are the same person. "
        "Ignore greetings and logistics."
    ),
    retain_extraction_mode="verbose",   # concise extraction drops the passing remarks that matter here
    # How to answer.
    reflect_mission=(
        "You are chief of staff to Theo, the project manager. For every item give the owner (or 'unowned'), "
        "the CURRENT due date, any earlier dates as history, and the meeting each came from. The most recent "
        "statement wins. Never call something live if it is merely merged or behind a flag. 'Now' is the date "
        "of the latest meeting in memory."
    ),
    disposition_skepticism=4, disposition_literalism=4, disposition_empathy=1,
)

manifest = json.load(open("manifest.json"))
def retain(t):
    client.retain(
        bank_id=BANK,
        content=open(t["file"]).read(),
        document_id=t["document_id"],          # same id later = replace, not duplicate
        timestamp=t["timestamp"],              # temporal retrieval depends on this
        context=f"Northwind payments team {t['meeting']} transcript, {t['timestamp'][:10]}.",
        tags=[x for x in t["tags"] if x.startswith("sprint:")],
        retain_async=True,
    )
for t in manifest["transcripts"]:
    if t["file"].split("/")[-1][:2] in manifest["phase_1_files"]:
        retain(t)
print("ingest started — build the chat while this runs")
```

Check the SDK reference for exact parameter names. On tags: the manifest suggests three; only `sprint:` changes any answer, so that's the one kept here. Never tag by content — Hindsight extracts blockers and decisions itself.

## Minute 5–20: the chat

One `reflect` per question. Three things separate a demo from a toy, and all three are in this loop:

```python
from datetime import datetime

AS_OF = "Monday September 14, 2026, 9:00 AM Pacific"   # move this forward after each new transcript
history = []

while True:
    q = input("\nTheo> ").strip()
    if not q:
        break
    ctx = f"Today is {AS_OF}. Answer as of this moment; treat anything later as unknown."
    if history:
        ctx += "\nEarlier in this conversation:\n" + "\n".join(history[-6:])
    t0 = datetime.now()
    r = client.reflect(bank_id=BANK, query=q, budget="mid", context=ctx, include_facts=True)
    print(r.text)
    mems = r.based_on.memories if r.based_on else []
    print(f"\n[{(datetime.now()-t0).seconds}s · based on {len(mems)} memories]")
    for m in mems[:12]:                                  # 2. show the evidence, with the meeting date
        print("  ", (m.occurred_start or "")[:10], m.text[:110])
    history += [f"Q: {q}", f"A: {r.text[:500]}"]
```

1. **Tell it what day it is.** `reflect` has no `query_timestamp` (only `recall` does); the as-of date goes in `context`. Without it, ages are computed from today.
2. **Print `based_on`.** Every memory has an `occurred_start`; that date is a meeting. This is your answer to "why do you say that?"
3. **Budget is a dial.** `mid` while building, `high` for the demo. Print the latency so people see the trade-off.

While the ingest is still running, ask something small and watch the answer improve as transcripts land. Then, before the seven questions, **read the raw memories for one document** — list memories filtered by `document_id`. Did it get "a day after the rules land" as a date? Did it notice Priya's PTO? That's what you're reasoning over.

## Minute 20–30: the seven questions

Poll the bank stats until `pending_consolidation` is 0, then run the seven. Expect to change the bank configuration at least once. Things that will go wrong, and the fix:

| Symptom | Fix |
|---|---|
| Old date reported as current | Add to `reflect_mission` (or a directive): "the most recent statement of a date is current; earlier ones are history." |
| Everything labelled "unverified" | Say what verification is: the owner's own specific statement ("merged, on staging") counts; a non-owner's glance doesn't. |
| "The feature is live" | Directive: merged / on the train / behind a flag are not live. |
| Ages wrong ("blocked 3 days") | You forgot the as-of date, or it's in the query instead of the context. |
| Answer is fast and vague | Budget `low`; go to `mid` or `high`. |

Directives are hard rules on every reflect: `client.create_directive(bank_id=BANK, name="...", content="...")`. Two or three good ones beat a long mission.

## Minute 30–40: transcript 11

When it's released: `retain(t)` for that entry, poll until consolidation is 0 again (2–4 minutes), set `AS_OF` to Tuesday Sep 15 9:00 AM, and ask *"What changed since the last meeting?"* Then re-ask question 1. Say what the answer got right and what it missed.

## What to show

Two of the seven questions with their evidence, and the "what changed" after transcript 11. Be ready to say which parts of the answer came from `reflect` and which from how you configured the bank.

## If you finish early

Ask the same question at `low` and `high` and compare the memory counts. Ask *"What has Sam committed to?"* and check whether Samantha's items came along.
