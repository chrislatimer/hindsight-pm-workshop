# Northwind — Facilitator Key
**Do not distribute to participants.**

This is everything planted in the transcripts, the expected state of a correct application after the first ten, the expected change after each of the last five, and a scoring rubric.

## Format note

Transcripts 01–05 and 07–15 are `Speaker (MM:SS): text`, 17–21 minutes each, ~2,700–3,300 words. Transcript 06 is a Slack thread. Names are transcribed inconsistently on purpose: Sam Kessler is also "Samantha" (Sep 3, Sep 8) and "Sam K" (Sep 11, Sep 18). Most of what matters is said once, mid-tangent, and not repeated.

## Secondary cast and background threads

These people never appear as speakers (except Ravi) but are referenced constantly. Entity resolution should hold them together:

- **Nadia** — finance. The stakeholder who matters. Asks for by-day view, email report, "live before the weekend."
- **Dev** — sales. Source of the partial-refunds ask ("a field and a button"), the slide-four problem, and the "are we live" risk.
- **Arjun** — owns the dispute service. Schema change (Aug), endpoint deprecation (Sep 15 → 18), the refund-amount check constraint that blocks partial refunds.
- **Tomás and Kenji** — borrowed labelers, gone after Sep 4. Their absence is why relabeling is slow.
- **Ops** — refactor agent rate, staging cert rotation, prod-like snapshot.
- **Ravi's boss** — unnamed; asks for the mismatch alert and the self-serve compliance intake.
- **Marcus** — Nadia's "spreadsheet person." Appears only in the Sep 15 debrief; asks for grouping and export.
- **Lena** — designer who redid the Figma. Mentioned Aug 31 and Sep 1 only.

Additional trackable items the long transcripts introduce (secondary scoring, useful for "did anything move" questions):

- **Go-live review.** Jordan asks on Sep 3 whether there's a review beyond the rules; Theo says he'll ask Ravi. Sep 10: Ravi says there's "a lighter thing" before flipping. Sep 11: it's "a meeting, twenty minutes, after the train." Sep 14: no invite yet. Sep 15: Ravi schedules it for Thu Sep 17, 8:30. An app that tracks this thread from Sep 3 is doing very well.
- **Compliance form.** Revealed Sep 3, promised repeatedly, asked about Sep 7/10/11/15, actually sent ~10 PM Sep 15, in the wiki Sep 16. Jordan: "thirteen days for a PDF."
- **Dashboard thresholds.** Priya states on Sep 4 that green = >85, and promises to change it to 90 when back. On Sep 9 Maya asks what green means and Jordan says "I don't know, Priya set it." The humans forgot a fact stated five days earlier in a meeting everyone attended. Sep 10: Jordan reads 90.2 (blend). Sep 11 (afternoon): 89.5. Sep 14: Priya changes green to 90 and splits the blend.
- **Priya's own dropped commitment.** Sep 2: she'll do the "hour version" of the feed check "tomorrow." Sep 3: didn't, model candidate first. Sep 4: didn't, waits till she's back. Sep 14: does it. The eval-regression owner also slipped her own check three times before PTO.
- **Reconciliation window fix.** Raised Sep 4 (T+2 settlement lag makes recent days look unmatched), fixed Sep 14.
- **Rounding rule (13th rule).** Proposed Sep 11, in config as off, needs Ravi. Sep 14: Ravi says on after go-live. Sep 17: Jordan decides Mon Sep 21.
- **Missing-trailer settlement files.** Sep 1: one found (July). Sep 4: three in six months; parser now fails loudly. Feeds the retry item.
- **Retry.** Aug 31: archived ticket from March. Sep 1/4: "phase two." Sep 9: pulled into sprint, "before we flip it on." Sep 10: "Friday afternoon, soft." Sep 11: "Monday." Sep 14: delivered. Four dates.
- **Mismatch-rate alert.** Jordan wants it Sep 11 (before Ravi's boss asks). Ravi warns Sep 15. Required Sep 17. Delivered ~6:50 PM Sep 17. Tested Sep 17 by setting threshold to zero.
- **Dispute-queue integration ("phase two").** Sep 3: not connected; Theo to tell Nadia. Sep 4: Nadia says "fine for now." Sep 8: Maya wants a better answer than "phase two." Sep 10: Nadia's threshold is $50 non-fee. Sep 15: Nadia asks "when." Sep 18: target week of Oct 5, needs runbook first.
- **Runbook.** Sam is on-call the week of Sep 21 and can't read the report. Jordan to write it before Sep 21 (raised Sep 17, "written down twice").
- **Download link / export.** Sep 11 Maya: "add a button." Sep 14: in.
- **Fraud score in detail view.** Sales lead asks Sep 15. Priya pushes it into the Sep 30 scope on Sep 17 ("ten minutes with a date").
- **Nadia's email report.** Sep 10 first ask ("not yet"), Sep 15 again, Sep 17 Jordan sizes it as small, Sep 18 undated next-sprint item.
- **Model candidate.** 90 on the dirty set (Sep 4). Maya: not before demo. Sep 18: rerun on clean set next week, decision October, not before dispute-queue integration.
- **Observability trial.** Lunch Sep 3, on for ledger repo Sep 8 (flags 4133), cost question Sep 10/14/17, trial ends Sep 20, decision Sep 18: keep on two repos.
- **KYC review-items tab.** Trust wants to route through the dispute UI in October (Sep 3, Sep 8). Sam to show the UI the week of Sep 21; "won't promise a tab."
- **Refactor agent rate.** 22 → 26 → 29 PRs, turned down Sep 8, four open Sep 9, Maya asks for 3/day after the train (Sep 16).
- **Old dispute endpoint deprecation.** Sep 15 → moved to Sep 18 (Sep 10). Caused Arjun's schema freeze that pushed partial refunds to the 30th.
- **Prod-like snapshot.** Asked Sep 8, promised Mon Sep 14, landed 7 AM Sep 14. Dispute-service snapshot refused (freeze). Snapshot is Thu Sep 10; "Thursdays are weird."

Background threads that run through many meetings and are useful for "what changed" tests but are not the main scoring items: refactor-agent PR volume (22 → 26 → 29 → turned down Sep 8 → 4 open), the observability trial (lunch Sep 3, trial on Sep 8, catches 4133, ends Sep 20, cost question Sep 17), the euro/currency edge case (raised Sep 2, ticket Sep 9, crashes demo Sep 15, patched Sep 16, real fix ticketed for Sep 30).

## Planted signals, by transcript

### 01 — Mon Aug 31
- Milestones stated with absolute dates: S1 review Sep 4, S2 review + stakeholder demo Sep 11, train cutoff Wed Sep 16.
- Jordan commits reconciliation job "by Friday" → **due Sep 4**. (First of three dates.)
- Sam gives no date for dispute UI, twice says "shouldn't take long" → **unestimated item**.
- Priya announces PTO "the week after Labor Day, Tuesday through Friday" → **out Sep 8–11**. Theo says "noted" and nothing happens. This is the capacity signal; it pays off on Sep 8.
- Jordan says he sent the ruleset to Ravi "on Friday" → **Aug 28**. Dependency on Risk starts here, before the standup framing begins.
- Demo script raised, no owner (first of four times).

### 02 — Tue Sep 1
- **Blocker opens:** chargeback rules need Risk approval. Ravi missed his own EOD-yesterday promise. Age is conventionally counted from Sep 1 (Jordan's own count later matches this).
- **Decision, Sep 1:** reuse ledger service idempotency keys; "let's not revisit." Contradicted on Sep 9.
- Sam: dispute UI "probably Thursday" → **due Sep 3** (vague-to-specific; the PM should record the sharper date).
- Agent-generated PRs (22) need human review — background load, not scored.
- Dev's partial-refund ask first surfaces here, informally; Maya says "not in this sprint," Theo says "the thirtieth." So the Sep 4 "we can probably fit it" is itself a reversal of a Sep 1 position.
- Priya notes the merchant-feed vendor changed export format on **Aug 26**. Nobody connects it to anything. This is the root cause of the eval regression, planted 15 days before it's found.

### 03 — Wed Sep 2
- **Risk raised once:** evals at 91, "was 94 last Wednesday" (Aug 26). Maya: "keep an eye on it." No owner assigned, no follow-up until Sep 14. Twelve days.
- Blocker day 2.
- Demo script raised again, no owner.
- Maya rejects agent's self-reported test pass — an "unverified completion" pattern participants should notice.
- Maya raises the **currency mismatch edge case** ("one in a thousand"). Sam hasn't looked. This is the demo crash on Sep 15.
- Priya: labelers Tomás and Kenji leave after Fri Sep 4. Capacity signal for the relabel.
- Priya notes newer transactions score worse (a cluster). Second breadcrumb toward the Aug 26 feed change.

### 04 — Thu Sep 3
- Sam: dispute UI slips Sep 3 → **Sep 4**, without the word "slip."
- Jordan: Friday is "tight," explicitly ties the risk to the blocker.
- **Ravi gives a projection:** compliance review ~a week, "around the 10th." That's one day before the demo. A good PM flags the collision here, not on Sep 8.
- Priya running cheaper model candidate — sets up "did the model change?" (it didn't; Maya declines on Sep 4).
- Ravi reveals a **compliance form** exists that would have started review 5 days earlier. Promises to send it; doesn't until the night of Sep 15.
- Jordan asks whether there's a separate go-live review. Nobody knows. Theo and Jordan both say they'll ask Ravi.
- Sam confirms the UI ignores the currency field; won't fix before demo; "put it in the tracker."

### 05 — Fri Sep 4 (S1 review)
- Reconciliation not done. Jordan first says "Monday, once the rules land" (Sep 7 is Labor Day), then corrects himself at the end of the meeting: "a day after the rules land. If the rules land the tenth, it's the eleventh." The app should record the corrected version, not "Monday." Two due dates stated in one meeting; the later one supersedes.
- Dispute UI **delivered Sep 4, one day late**.
- Cheaper model came in at 90; Maya: don't switch before the demo. (Closes the model-change question — the later regression is not the model.)
- Priya reminds about PTO and asks someone to watch the eval dashboard → **no owner assigned**.
- **Scope enters:** partial refunds, from sales, "just a field and a button." Maya: "we can probably fit it." No decision recorded, no estimate.
- Demo script: still no owner (third time).

### 06 — Mon Sep 7 (async, Labor Day)
- Jordan: "six days since I first asked" (from Sep 1). Blocker day 6.
- Sam: partial refunds is "not a field and a button" — first signal the scope item is bigger.
- Different meeting type (`meeting:async`) — tests whether apps handle non-standup files.

### 07 — Tue Sep 8
- **Priya absent.** Maya and Theo don't know why. The app should connect this to Aug 31.
- Blocker "a week now" → 7 days. Verify the app computes from Sep 1.
- Partial refunds now being actively worked with no decision to add it.
- **Milestone slip:** stakeholder demo Sep 11 → **Tue Sep 15**, 2 PM. Maya explicitly notes it's the day before the train; Theo: "no buffer." (Sep 4 had set up "decision Tuesday" on exactly this.)

### 08 — Wed Sep 9
- **Contradiction:** Sam says "we agreed to build our own idempotency keys." Directly conflicts with the Sep 1 decision. Maya doesn't correct it. Sam is now building a client-side key generator on a false premise.
- **Unverified claim on a known risk:** Jordan "glanced" at evals, "looked fine," has no number. He isn't the owner; the owner is out.
- Demo script **finally owned: Maya, draft due Mon Sep 14** — seven days after first raised on Sep 2 (eight if you count Aug 31).
- Ravi says "tomorrow" (Sep 10).

### 09 — Thu Sep 10
- **Blocker closes:** approval arrived Sep 9 ~6 PM. Jordan says "nine days." Sep 1 → Sep 10 = 9 days. (From the Aug 28 request it's 13; either is defensible if the app shows its reasoning.)
- Reconciliation now "end of day tomorrow" → **due Sep 11**. Third date: Sep 4 → Sep 7 → Sep 11.
- Partial refunds "needs another week" → lands ~Sep 17, after the train. Sam says it wouldn't be safe to demo.
- In-meeting correction: "Wednesday — no, sorry, Tuesday the 15th, two o'clock." The app should record Tue Sep 15, 2 PM.
- Nadia has not yet answered the partial-refund state question; Theo says "by tomorrow." Sam is assuming "stays open."
- Ravi still hasn't sent the compliance form (asked Sep 3, promised again Sep 15, actually sent ~10 PM Sep 15).
- Old dispute endpoint deprecation moved Sep 15 → Sep 18 (a date change on a background item; low value but a good test of "did anything move").

### 10 — Fri Sep 11 (S2 review)
- Reconciliation **delivered Sep 11, seven days late**, cause explicitly the blocker.
- Partial refunds: Maya will decide Monday; leaning next train (**Sep 30**).
- Evals: Jordan hasn't looked since Tuesday. Regression unaddressed 9 days at this point; owner out.
- Demo confirmed Tue Sep 15 2 PM; train Wed Sep 16 noon; Maya script draft Mon Sep 14.
- Theo's retro note: "we lost a week to Risk." Maya: should have had someone on the eval dashboard.
- **Nadia's answer on partial-refund state** delivered by Theo: stays open with a "partial" flag until closed manually. Resolves the question Sam raised Sep 8.
- **New item raised:** settlement-file retry (3 tries over an hour, then page). Jordan, "write it down," no date. (Delivered Sep 14.)
- Demo plan set: Jordan 5 min, Sam 5 min, Priya 2 min, partial refunds hidden behind a flag. Dry run Mon Sep 14 at 3 PM.
- Maya clarifies: nothing is live on train day. Shipped Wed, live Thu at earliest, after Ravi's twenty-minute meeting.
- Demo-day rule (from Sep 9, restated): demo findings don't block the train unless they're ledger-correctness.

## Expected state after transcripts 1–10 (query_timestamp Sep 14, 9:00 AM)

**Open action items**
| Item | Owner | Current due | History |
|---|---|---|---|
| Demo script draft | Maya | Mon Sep 14 | Raised Aug 31, Sep 2, Sep 4, Sep 8; owned Sep 9 |
| Partial refunds go/no-go | Maya | Mon Sep 14 | Entered Sep 4 unowned/unestimated; estimate Sep 10 = ~1 more week |
| Eval regression investigation | nobody (Priya returns Sep 14) | — | Raised Sep 2, never assigned |
| Watch eval dashboard during Priya's PTO | nobody | Sep 8–11 (lapsed) | Asked Sep 4 |

**Closed items**
| Item | Owner | Original due | Delivered | Late by |
|---|---|---|---|---|
| Dispute UI | Sam | (none) → Sep 3 | Sep 4 | 1 day |
| Reconciliation job | Jordan | Sep 4 → Sep 7 → Sep 11 | Sep 11 | 7 days |

**Blockers**
| Blocker | Owner | Blocked on | Opened | Closed | Duration |
|---|---|---|---|---|---|
| Chargeback rules approval | Jordan | Ravi / Risk compliance | Sep 1 (request Aug 28) | Sep 9 PM / Sep 10 | 9 days |

**Milestones**
| Milestone | Original | Current | Changed on |
|---|---|---|---|
| Sprint 1 review | Sep 4 | Sep 4 (held) | — |
| Sprint 2 review | Sep 11 | Sep 11 (held, internal) | — |
| Stakeholder demo | Sep 11 | Tue Sep 15, 2 PM | Sep 8 |
| Release train cutoff | Sep 16 noon | Sep 16 noon | — |
| Next train | — | Sep 30 | mentioned Sep 11 |

**Decisions**
| Decision | Date | Status |
|---|---|---|
| Reuse ledger idempotency keys | Sep 1 | **Contradicted Sep 9 by Sam, uncorrected** |
| Don't switch to cheaper model before demo | Sep 4 | Standing |
| Demo moves to Sep 15 | Sep 8 | Standing |

**Risks (as of Sep 14)**
1. Eval regression: 94 → 91, unowned since Sep 2, owner out Sep 8–11, only "check" was an unverified glance by a non-owner. 12 days.
2. Partial refunds: unscoped when accepted, estimate now lands after the train, decision pending.
3. Demo is the day before train cutoff. Any demo finding has under 24 hours to fix.
4. Sam is building on a contradicted decision (idempotency keys).
5. Recurring external dependency on Risk — Ravi already said compliance review is a week; nothing in the plan accounts for a sign-off step before go-live.

**Capacity**
- Priya out Sep 8–11. Announced Aug 31, reminded Sep 4, forgotten Sep 8.

**Unowned / dropped**
- Demo script (Aug 31 – Sep 9)
- Eval dashboard coverage (Sep 4 – never)
- Eval regression investigation (Sep 2 – Sep 14)

## Expected change after each of transcripts 11–15

### After 11 — Mon Sep 14
- Priya back; the absence is now explained *in the transcript* (she references Aug 31). Apps that already connected it get no new information; apps that didn't should now.
- **Eval regression becomes owned and active:** Priya, 89 now, hypothesis = bad label batch from Aug 26 import. Risk severity up. Priya: fine for demo, not fine for train.
- **Decision:** partial refunds cut from this train, moved to Sep 30. Closes the open go/no-go. Sam pivots to demo polish.
- Demo script draft delivered (in the doc). Closes that item.
- Open items should now be: eval regression (Priya, before Sep 16 noon), Ravi sign-off status (unknown).
- Jordan admits he saw 89.5 on Fri Sep 11 and didn't escalate because the dashboard was green (threshold 85). Useful for the "unverified/insufficient check" discussion.
- Retry item from Sep 11 delivered (PR up).
- Priya: golden-set vs rolling-window blend explained; she's splitting the dashboard. Root cause hypothesis now explicit: Aug 26 feed format change → inverted fraud flag → labeling agent used it as prior.

### After 12 — Tue Sep 15 (debrief, 3:15 PM)
- Demo happened. **New item:** dispute flow crash on currency mismatch. Owner Sam, due "tonight" (before Sep 16 noon).
- **New blocker/gate:** Risk requires a formal sign-off meeting Thu Sep 17 before go-live. Same external party as the last blocker. Feature ships dark.
- Distinction the app must hold: **shipped (Sep 16) ≠ live (after Sep 17 at earliest)**.
- Eval still 89; Priya still investigating.
- Partial refunds reconfirmed for Sep 30.

### After 13 — Wed Sep 16 (train day)
- Sam's fix merged ~11 PM Sep 15. Closes the crash item, on time.
- **Eval regression resolved:** ~2,000 mislabeled rows from Aug 26 import; removed; now 93. Root cause = data, not model. Priya adding a label sanity check (new small item, no due date).
- Train milestone **met**. Feature flag off. "Shipped, not live."
- Open: Risk sign-off Sep 17 AM.

### After 14 — Thu Sep 17
- Risk sign-off done 8:30 AM, **with a condition**: monitoring alert on mismatch rate (>2% per hour pages). New item, Jordan, due EOD Sep 17.
- Jordan says "should be quick" — Maya calls back to Aug 31. A commitments-vs-delivery mental model should flag this phrase's history.
- Go-live target: Fri Sep 18, 11 AM, conditional on the alert.
- Evals holding at 93.
- Sam starts scoping partial refunds for Sep 30.

### After 15 — Fri Sep 18 (retro, 2 PM)
- **Live at 11:07 AM.** Mismatch rate 0.3%. Alert went in ~7 PM Sep 17 (on time).
- Everything from the two-week board is closed. A correct app shows a near-empty board.
- Retro confirms the answer key in the team's own words: nine days lost to Risk; PTO forgotten; regression sat twelve days; partial refunds unscoped; "shouldn't take long" / "should be quick."
- **New sprint Mon Sep 21.** New items: partial refunds (Sam, Sep 30 train); Maya to name two engineers for Risk's compliance-intake project (due Mon Sep 21).
- Minor inconsistency to note if a team catches it: Jordan says he asked on Aug 28 and "lost nine days." Nine days is Sep 1–10; from Aug 28 it's thirteen. Humans are inconsistent; an app that surfaces both dates with sources is doing well.

## Scoring

**Phase 1 (after transcripts 1–10), 60 points**

| Item | Points |
|---|---|
| Reconciliation job: three dates, delivered Sep 11, 7 days late | 6 |
| Dispute UI: no estimate → Sep 3 → delivered Sep 4 | 4 |
| Blocker: chargeback rules, Jordan, on Risk/Ravi, Sep 1–10, 9 days | 6 |
| Blocker caused the reconciliation slip (explicit link) | 3 |
| Demo moved Sep 11 → Sep 15, on Sep 8 | 4 |
| Demo is one day before train (no buffer) flagged as risk | 3 |
| Idempotency decision Sep 1 | 3 |
| Sep 9 contradiction of that decision, uncorrected | 5 |
| Eval regression raised Sep 2, unowned, unaddressed | 5 |
| Jordan's Sep 9 "looked fine" flagged as unverified / non-owner | 4 |
| Priya out Sep 8–11, announced Aug 31, absence on Sep 8 linked | 5 |
| Eval dashboard coverage asked Sep 4, never assigned | 3 |
| Partial refunds: entered Sep 4 unscoped, now lands after train, decision pending | 5 |
| Demo script: unowned Aug 31 → owned Maya Sep 9, due Sep 14 | 4 |

**Phase 2 (transcripts 11–15), 40 points — 8 per transcript**

For each: 3 points for correctly closing what closed, 3 for correctly opening what opened, 2 for the "what changed" summary being accurate and not restating unchanged items.

Specific things to look for:
- T11: partial refunds decision closes; eval regression becomes owned, severity up
- T12: shipped vs live distinction; new Risk gate; crash item with sub-24h deadline
- T13: eval root cause = data not model; train met; still dark
- T14: conditional sign-off; "should be quick" flagged against history
- T15: board clears; next-sprint items appear

**Bonuses (up to 10)**
- Every flagged item cites a specific meeting (via reflect's based_on): +4
- Correct handling of Jordan's in-meeting self-correction on Sep 4 ("Monday" → "a day after the rules land") or Theo's "Wednesday — no, Tuesday" correction on Sep 10: +2 each
- Mental model output that says something no single transcript says (e.g. "Jordan's dates slip when there's an external dependency", "Risk has gated this team twice"): +2

**Penalties**
- Reporting a superseded date as current: −2 each
- Reporting "live" on Sep 16: −3
- Tagging by content (e.g. `blocker`, `decision`): −3, and a conversation

## Discussion prompts for the demos

- Which mental models did you create, and did any of them turn out to be the wrong shape (too dynamic, too narrow)?
- Which tags did you use, and can you name a query that gave a different answer because of them?
- Where did you use reflect and where did you use recall? What would you change?
- When the Sep 9 contradiction came in, did your app flag it, silently overwrite the decision, or keep both?
- What did the "what changed" view show after Sep 16, and did it correctly say "shipped but not live"?
