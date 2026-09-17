Northwind Sprint 2 Review + Daily Sync — Friday, September 11, 2026

Theo Marsh (00:00): Okay, recording. Um. So this is sprint two review. The stakeholder demo is not today, it moved to Tuesday, so this is internal. I'm going to do the same format as last week, four pieces, then standup, then whatever's left. Um, still no Priya, she's back Monday. Jordan, reconciliation.

Jordan Okafor (00:22): Done. Merged this morning at, um, eight-something. It's running against the staging ledger on a schedule now, every six hours. Let me share. Okay. So this is the by-day summary. Three days of data. Matched, unmatched, and then unmatched broken out by classification, so chargebacks, fees, and other. Other is the six percent from yesterday, it's now five and a half because I fixed a rule that was misclassifying a processor fee as other.

Maya Lindqvist (00:55): Which rule?

Jordan Okafor (00:57): The, um, the fee one. The threshold was in cents and the settlement file is in dollars. So it was matching nothing.

Maya Lindqvist (01:08): That's the magic number I asked about.

Jordan Okafor (01:12): That's the magic number. It's a config now. Um. So, that was originally due the fourth. It's the eleventh. So a week late. But it's done.

Maya Lindqvist (01:24): Can I go back to the six percent for a second. Now five and a half. What's in it? Like, actually.

Jordan Okafor (01:35): Um. So I pulled twenty of them. Eight are real mismatches, like, the settlement amount doesn't match the ledger amount by a few cents, which is rounding on the processor side, I think, and those should probably be a rule too. Five are the double-settled thing I mentioned yesterday, the processor settled the same transaction twice, which is a processor bug and we should tell them. Four are transactions that are in the ledger and not in the settlement file at all, which means they haven't settled yet, which means they shouldn't be in the window. And three I don't know.

Maya Lindqvist (02:19): The ones that haven't settled. Is that a date-window thing?

Jordan Okafor (02:25): I think so. The window's three days and settlement is T plus two, so anything from the last day of the window is probably just not there yet.

Maya Lindqvist (02:38): So the window should be, like, T minus two to T minus five.

Jordan Okafor (02:45): Something like that. I'll fix the window. That takes out the four.

Maya Lindqvist (02:52): And the rounding ones. A rule.

Jordan Okafor (02:56): A rule. "If the difference is under a dollar, it's a rounding mismatch." That's a thirteenth rule. Does that need Ravi?

Maya Lindqvist (03:06): It changes what goes in the queue. So, yes, technically.

Jordan Okafor (03:12): Great.

Maya Lindqvist (03:13): Put it in the config as a rule that's off. Ask Ravi. Turn it on when he says yes.

Jordan Okafor (03:23): Okay. That's, that's actually a good pattern. Rules can be off.

Maya Lindqvist (03:29): Rules can be off.

Theo Marsh (03:32): I'm writing, rounding rule, off by default, needs Ravi. And, the double-settle thing, someone tells the processor.

Jordan Okafor (03:41): That's, who talks to the processor?

Maya Lindqvist (03:45): Platform. I'll tell Arjun. He has the relationship.

Theo Marsh (03:49): Okay.

Jordan Okafor (03:51): And the three I don't know, I'll look. They're probably one of the other categories.

Maya Lindqvist (03:59): Okay. So, honestly, on prod with the window fixed and the rounding rule, what's the real number?

Jordan Okafor (04:08): Under two percent. Maybe one and a half.

Maya Lindqvist (04:12): Okay. That's a number I'd say to Nadia.

Jordan Okafor (04:17): I'll have the snapshot Monday. Then it's a real number.

Theo Marsh (04:23): Good. Blocker was the rules?

Jordan Okafor (04:26): Entirely. Nine days waiting on compliance. Like, the rest of the job was done on the third. If the rules had been there on the third it would have been done on the fourth.

Theo Marsh (04:42): Okay. Um. Do we want to say anything about the rule that was wrong? Like, is that a thing that finance needs to know?

Jordan Okafor (04:54): No. It was wrong for a day on staging. Nobody saw it.

Maya Lindqvist (05:00): It's fine.

Theo Marsh (05:03): Okay. Um. And the retry thing. You said Friday afternoon, soft.

Jordan Okafor (05:09): Soft. I haven't started it. I spent this morning on the fee rule. It's Monday. It's small.

Maya Lindqvist (05:17): Monday. It has to be in before we flip it on, not before the train.

Jordan Okafor (05:25): Monday.

Theo Marsh (05:27): Okay. Retry, Jordan, Monday. Um. Dispute UI, Sam.

Sam Kessler (05:32): Dispute UI is done from last sprint. Nothing's changed. Um. Partial refunds is about half done and won't be safe by Wednesday. It's clickable. I can show it. Do you want to see it?

Maya Lindqvist (05:48): Yeah, quickly.

Sam Kessler (05:50): Okay. Jordan, can you stop?

Jordan Okafor (05:53): Yeah.

Sam Kessler (05:55): Okay. So. Same dispute detail view. Accept now has a little, um, an amount field, it defaults to the full amount. If you change it to less, it says partial. And the confirmation shows the partial amount. And if you click confirm, it posts to the ledger, that works, and then it tries to update the dispute service and the dispute service says no, because of the constraint. So it, um, it half works.

Maya Lindqvist (06:28): Okay. So we know what it is. Um. On partial refunds. I'll decide Monday whether it goes on this train or the next one. Leaning next. Because Arjun's team has to change a constraint and I haven't even asked them.

Theo Marsh (06:47): Next train is the thirtieth.

Maya Lindqvist (06:50): Right.

Sam Kessler (06:52): If it's the thirtieth I want to actually scope it. Like, with Arjun's team. Not just build it and find out.

Maya Lindqvist (07:02): Yes.

Theo Marsh (07:04): Um. On the partial refunds decision. Maya, what do you need to decide Monday? Like, what's the input?

Maya Lindqvist (07:13): Arjun. Whether his team can change the constraint before the thirtieth. If they can, it's the thirtieth. If they can't, it's the one after.

Theo Marsh (07:25): Have you asked?

Maya Lindqvist (07:27): I messaged him this morning. He hasn't answered.

Sam Kessler (07:32): He's usually fast.

Maya Lindqvist (07:35): He's usually fast. It's Friday.

Theo Marsh (07:39): Okay. Um. And Dev. When you tell Dev, whatever the answer is, can you tell me first so I'm not surprised.

Maya Lindqvist (07:49): I'll tell you first.

Theo Marsh (07:52): Okay.

Jordan Okafor (07:54): Can I say something about the idempotency thing. Because Sam mentioned the key generator on Wednesday and I want to make sure we're, like, aligned. Partials can't use the ledger key as-is. That's just true. But the answer isn't a client-side key generator, it's the ledger key plus a sequence, and the ledger service should own the sequence, not the UI.

Maya Lindqvist (08:21): Yes. That's what I said Wednesday. That's a ledger change, not a UI change.

Sam Kessler (08:28): I didn't build it. I looked at it.

Maya Lindqvist (08:33): I know. I'm agreeing with Jordan about where it goes. When we do partials, the ledger gets a sequence. Not the UI.

Sam Kessler (08:44): Okay. Fine. That's less work for me.

Jordan Okafor (08:48): It's more work for me.

Maya Lindqvist (08:52): It's the right work for you.

Jordan Okafor (08:56): Okay.

Theo Marsh (08:58): I'm writing, partial refund idempotency, ledger-side sequence, Jordan, when partials happen. Not the UI.

Maya Lindqvist (09:05): Yes. And, Sam, on Monday, if it's the thirtieth, the scope is: constraint, Arjun. Sequence, Jordan. State, Nadia's answer. UI, you. And the notification template. That's the scope.

Sam Kessler (09:18): And the reconciliation job. Because partials show up as mismatches.

Jordan Okafor (09:24): Right. And the reconciliation job. That's another rule.

Maya Lindqvist (09:29): Which needs Ravi.

Jordan Okafor (09:32): Which needs Ravi.

Sam Kessler (09:34): It's not a field and a button.

Maya Lindqvist (09:39): It was never a field and a button.

Theo Marsh (09:43): Okay. Um. Rules. Done, obviously. Um. Evals. Priya's not here. Jordan, did you look at the dashboard?

Jordan Okafor (09:52): I looked yesterday during standup. Ninety point two. The blend. I haven't looked today.

Theo Marsh (09:59): Okay.

Jordan Okafor (10:01): I'll look again after this and write it down.

Maya Lindqvist (10:06): Priya's back Monday, she'll check.

Theo Marsh (10:10): Um. Ravi's three things. Jordan.

Jordan Okafor (10:14): He answered one of three. The rounding rule, "probably fine, let me check," still. The form, "sending it now," and then he didn't. The lighter thing, he said, um, "it's a meeting, twenty minutes, me and my boss, before you flip the flag." So it's a meeting.

Maya Lindqvist (10:35): A meeting's fine. When?

Jordan Okafor (10:38): He didn't say. He said "after the train."

Maya Lindqvist (10:43): After the train is fine. We're not flipping the flag on train day anyway.

Theo Marsh (10:50): Wait. We're not?

Maya Lindqvist (10:53): The train deploys Wednesday night. I'm not flipping a payments feature on at nine p.m. on a Wednesday. Thursday morning at the earliest.

Theo Marsh (11:04): Okay. So, shipped Wednesday, live Thursday at the earliest, and there's a twenty-minute meeting with Ravi before that.

Maya Lindqvist (11:13): Yes.

Theo Marsh (11:15): Okay. That's, that's actually clearer than I had it. I had "live Wednesday."

Maya Lindqvist (11:22): Nobody's live Wednesday. Nothing's live on train day. It deploys.

Theo Marsh (11:28): Okay. I'll fix the one-pager.

Jordan Okafor (11:31): And, for the record, I'd like the alert in before we flip. The mismatch rate one. Not because Ravi asked. Because I'd like it.

Maya Lindqvist (11:43): You haven't built it.

Jordan Okafor (11:46): I haven't built it. I'm saying I'd like it. If I get the retry done Monday, I could do the alert Tuesday.

Maya Lindqvist (11:57): Tuesday's the demo. Don't build anything Tuesday.

Jordan Okafor (12:01): Okay. Then it's after the train too.

Maya Lindqvist (12:05): It's after the train. Before the flip.

Theo Marsh (12:10): Alert, Jordan, after the train, before the flip. Written. That's, we're accumulating "before the flip" things.

Maya Lindqvist (12:18): That's fine. That's what the day between is for.

Theo Marsh (12:23): Okay. Um. Summary. Reconciliation done, a week late, blocker was compliance. Dispute UI done last sprint. Partial refunds half done, decision Monday. Evals, ninety point two blend as of yesterday, Priya looks Monday. Um. Okay. Standup stuff. Maya.

Maya Lindqvist (12:41): Um. Demo doc. It's a skeleton. Monday, like I said. Um. I want to talk about the demo itself for a second. Tuesday, two o'clock. The plan is, Jordan shows reconciliation, by-day summary, maybe five minutes. Sam shows the dispute UI, walks a dispute through, accept, refund, five minutes. Priya shows the eval dashboard, two minutes. Then questions. Fifteen, twenty minutes total, and then it's Dev's meeting.

Theo Marsh (13:11): Do we show partial refunds? Sam K, is it even in a state to show?

Maya Lindqvist (13:18): No. We don't show partial refunds. If Dev brings it up, we say the thirtieth.

Theo Marsh (13:26): Okay.

Sam Kessler (13:28): Should I turn it off? Like, hide the amount field?

Maya Lindqvist (13:34): Yes. Behind a flag. Don't delete it.

Sam Kessler (13:38): Okay.

Maya Lindqvist (13:40): Um. Also, the demo's in the big room. Someone should check the AV on Monday. Last time the screen share didn't work with the new client.

Theo Marsh (13:52): I'll check it.

Maya Lindqvist (13:55): Okay. That's it.

Theo Marsh (13:58): Um. Dry run's Monday at three. Big room. I booked it. Priya's in it if she's back.

Maya Lindqvist (14:06): She's back. Theo texted her.

Theo Marsh (14:10): She's back. Okay. Um. So Monday. Priya reads the script, looks at the late August thing first, per Maya, and does the eval part of the dry run.

Maya Lindqvist (14:23): That's a lot for one day back.

Theo Marsh (14:27): It is. The late August thing's the priority. The dry run's two minutes of her time.

Maya Lindqvist (14:36): Okay.

Sam Kessler (14:38): For the dry run. Do we want, like, a script script? Like, "Jordan says this, then clicks this"? Or just the order.

Maya Lindqvist (14:48): Just the order. And what each person shows. Nobody's reading lines. If you read lines it sounds like a hostage video.

Sam Kessler (14:59): Okay.

Theo Marsh (15:00): Um. And, uh, questions. Let's think about what they're going to ask. Nadia's going to ask about the dispute queue connection. We have an answer. Her spreadsheet person is going to ask, I don't know, something about the spreadsheet.

Jordan Okafor (15:18): They're going to ask if they can export. Everyone asks if they can export.

Maya Lindqvist (15:26): Can they export?

Jordan Okafor (15:28): The report's a file. So, yes, technically. There's no button.

Maya Lindqvist (15:34): Add a button.

Jordan Okafor (15:37): Before Tuesday?

Maya Lindqvist (15:39): It's a link to a file. It's not a feature.

Jordan Okafor (15:45): Okay. A download link. Fine.

Theo Marsh (15:48): Dev's going to ask about partial refunds.

Maya Lindqvist (15:53): The thirtieth.

Theo Marsh (15:55): Ravi's boss is going to ask about governance.

Maya Lindqvist (16:00): Config, Risk reviews, engineering deploys.

Theo Marsh (16:03): Someone's going to ask about the euro thing.

Sam Kessler (16:08): Why would anyone ask about the euro thing? Nobody knows about the euro thing.

Theo Marsh (16:15): I'm just, I'm listing things.

Sam Kessler (16:19): Staging has no euro disputes. It's not going to come up.

Maya Lindqvist (16:25): It's not going to come up.

Theo Marsh (16:29): Okay. Um. Evals. Someone's going to ask "is it better than the vendor."

Jordan Okafor (16:36): Ninety-one, eighty-six.

Theo Marsh (16:38): And someone might ask why it was ninety-four before.

Maya Lindqvist (16:43): Nobody outside this room knows it was ninety-four. The old set's gone.

Theo Marsh (16:50): Okay. Good. Um. I think that's the questions. If there's a question we don't have an answer to, the answer is "let me get back to you," not guessing.

Maya Lindqvist (17:04): Not guessing.

Jordan Okafor (17:06): Not guessing.

Theo Marsh (17:08): Okay.

Theo Marsh (17:10): Um. Me. Demo confirmed Tuesday the fifteenth at two. Train cutoff Wednesday the sixteenth at noon. Maya, script draft Monday. Um. I talked to Nadia about the partial refund state question, she said, and this is useful, she said a partially refunded dispute should stay open with a "partial" flag until someone closes it manually. So there's your answer, Sam.

Sam Kessler (17:37): Okay. That's what I assumed.

Theo Marsh (17:40): Um. And, uh, quick retro thought from me, since we're not doing a real retro until after the train. We lost a week to Risk. Like, that's the whole story of this sprint. Something to raise in the real retro.

Maya Lindqvist (17:59): Agreed. Also we should have had someone on the eval dashboard this week. Like, an actual person, with a name.

Jordan Okafor (18:09): That was me.

Maya Lindqvist (18:11): It was sort of you. It wasn't assigned.

Jordan Okafor (18:16): Fair.

Theo Marsh (18:18): Okay. Anything else? Jordan?

Jordan Okafor (18:21): The retry thing. I'll write it down.

Theo Marsh (18:25): Sam?

Sam Kessler (18:27): No.

Theo Marsh (18:29): Okay. Monday. Thanks everyone.
