Northwind Sprint 1 Review + Daily Sync — Friday, September 4, 2026

Theo Marsh (00:00): Okay. Recording. So this is sprint review, I blocked forty-five minutes, we probably don't need it but I'd rather have it. Um. Format is, we go through each of the four pieces, what's done, what's not, what's the plan. Then normal standup stuff, then I've got a couple of things from Dev. Let's start with reconciliation. Jordan.

Jordan Okafor (00:25): Okay. So, reconciliation job. Not done. I want to say that first. The non-rules half is merged and it's running on staging. Let me share, actually. Hang on. Can you see this?

Maya Lindqvist (00:40): You're sharing your whole screen.

Jordan Okafor (00:44): Oh. Hang on. The new client. Okay, now?

Maya Lindqvist (00:49): Yeah. That's a window.

Jordan Okafor (00:52): Okay so this is the staging run from this morning. It reads the ledger, reads the processor settlement file, it's streaming the file now, Maya, it doesn't load it. I did that Wednesday night. Um. It matches on settled amount and merchant and date window. And then this, this bucket down here, this is the unmatched. And you can see it's like eighty-seven percent unmatched, which is because the rules are stubbed, so it can't tell the difference between a chargeback and a fee and a real problem. When the rules are in, most of that bucket goes away. Like, most of what's in there is processor fees, and a fee rule is one line.

Theo Marsh (01:42): And the rules are?

Jordan Okafor (01:45): Compliance. Ravi said around the tenth. I'll have it finished Monday, once the rules land.

Theo Marsh (01:53): Monday.

Jordan Okafor (01:55): Yeah, like, a day after they land.

Theo Marsh (01:59): Okay.

Maya Lindqvist (02:01): Can I ask about the report format? Because finance is going to want to see this and that table is, like, it's a debug table.

Jordan Okafor (02:13): Yeah. The report writer has a summary mode, I just didn't run it. I'll show summary mode at the demo. Summary mode is, per merchant, matched count, unmatched count, unmatched amount. It's a table you'd actually put in front of someone.

Maya Lindqvist (02:32): Okay. Per merchant or per day?

Jordan Okafor (02:36): Per merchant.

Maya Lindqvist (02:38): Finance closes per day.

Jordan Okafor (02:41): Um. I can add per day. It's the same data grouped differently.

Maya Lindqvist (02:48): Add per day.

Jordan Okafor (02:50): Okay. Not today.

Maya Lindqvist (02:53): Not today.

Jordan Okafor (02:55): Um, one more on reconciliation. The window. Right now it's three days, like, it looks at transactions from the last three days and settlement from the last three days. And I picked three because the agent picked three. I don't actually know what the right window is. Settlement's T plus two, so anything from yesterday isn't going to be in the file yet, so it's going to look unmatched.

Maya Lindqvist (03:26): So the window should lag. Like, T minus two to T minus five.

Jordan Okafor (03:33): Something like that. I'll figure it out when the real rules are in and I can see what's actually unmatched versus what's just early.

Maya Lindqvist (03:44): Okay. Write it down. That's, that's the kind of thing that makes the number look bad for no reason.

Jordan Okafor (03:54): Yeah. Written.

Theo Marsh (03:56): Um. And, the missing trailer thing. From Tuesday. Did you look for more?

Jordan Okafor (04:03): I looked. There's three in the last six months. July, May, and one in March. So it's, it's every couple months. The parser fails on them now. Loudly. There's a test with the July file.

Maya Lindqvist (04:19): Good. And when it fails, what happens? Like, on prod.

Jordan Okafor (04:25): Right now, it logs and the job stops. Which is, that's the retry thing again, sort of. Like, if the file's cut off, we want to wait and refetch, because the processor usually resends.

Maya Lindqvist (04:41): Does it?

Jordan Okafor (04:43): I think so? The March one has a second file the same day.

Maya Lindqvist (04:50): Okay. So the retry thing handles it if the retry thing exists.

Jordan Okafor (04:57): If the retry thing exists. Which it doesn't.

Maya Lindqvist (05:01): Phase two.

Jordan Okafor (05:04): Phase two. It's on Theo's one-pager.

Theo Marsh (05:08): It's on the one-pager.

Theo Marsh (05:11): Okay. Um. Dispute UI. Sam.

Sam Kessler (05:14): Done. Let me share.

Jordan Okafor (05:17): Let me stop.

Sam Kessler (05:20): Okay. Um. Can you see the, yeah. Okay. So this is the dispute list. This is staging data. You can filter by state, by merchant, by date. Click one. This is the detail view, it's got the transaction, the dispute reason, the evidence if there's any, and the action bar down here changes based on state, so this one's in evidence-requested, so you get, um, escalate, close, and mark received. Let me find one that's pending. Okay, this one. This one gets accept, reject, request evidence. And accept opens the refund confirmation. Which is the one that fought me.

Maya Lindqvist (06:04): Does it post?

Sam Kessler (06:06): It posts to the ledger on staging, yes. I did it, like, twice this morning and then stopped because I don't want to fill staging with fake refunds.

Maya Lindqvist (06:20): Nice. That's one day late but it's solid.

Sam Kessler (06:24): The modal was the whole problem. And the fixture. The fixture was half the problem.

Theo Marsh (06:32): Is the currency thing in there?

Sam Kessler (06:36): No. It's in the tracker. It ignores currency. Everything's USD on staging so it doesn't come up.

Theo Marsh (06:45): Okay.

Priya Raman (06:47): Can I ask a UI thing? The list. Is there a way to see, like, how long a dispute's been sitting? Because that's what I'd want.

Sam Kessler (06:59): There's a created date. There's not an age column. I could add an age column.

Priya Raman (07:07): It's not for me. I just think Nadia's going to ask.

Sam Kessler (07:13): Yeah. I'll add it. It's a computed column, it's nothing.

Maya Lindqvist (07:19): Add it. That's a good one.

Theo Marsh (07:23): Um. I want to talk about the demo date for a second. Because Ravi said the tenth, Jordan said a day after, that's the eleventh, and the demo's the eleventh. So the reconciliation job is done, best case, the morning of the demo.

Maya Lindqvist (07:42): Best case.

Theo Marsh (07:44): Which is, I don't love that.

Maya Lindqvist (07:48): I don't love it either. Options are, demo on the eleventh with stubbed rules and say they're stubbed. Demo on the eleventh with just the dispute UI and evals, and reconciliation the week after. Or move the demo.

Jordan Okafor (08:06): I don't want to show finance stubbed rules. The number's going to be eighty-seven percent unmatched and they're going to remember that number.

Maya Lindqvist (08:17): Agreed.

Sam Kessler (08:19): I can demo the UI on the eleventh. That part's ready.

Maya Lindqvist (08:25): You can, but the UI without reconciliation is, it's half the story. Finance asked for reconciliation. The UI's the thing we added.

Theo Marsh (08:36): So, move it.

Maya Lindqvist (08:39): I don't want to decide that today. Let's see where compliance is on Tuesday. If Ravi says the ninth, we're fine. If he says the eleventh, we move.

Theo Marsh (08:52): Okay. Decision Tuesday.

Maya Lindqvist (08:55): Decision Tuesday.

Theo Marsh (08:57): I'll tell Nadia it might move. So she's not surprised.

Maya Lindqvist (09:02): Tell her it might move. Don't tell her a new date.

Theo Marsh (09:08): Okay.

Priya Raman (09:10): If it moves to the week after, I'm back, so I can do the eval part myself. That's, that's one upside.

Theo Marsh (09:21): That is one upside.

Jordan Okafor (09:24): And I'd have the real rules for a few days instead of a few hours.

Theo Marsh (09:32): Okay. So there's a case for moving it regardless.

Maya Lindqvist (09:37): There's a case. Tuesday.

Theo Marsh (09:40): Tuesday.

Theo Marsh (09:42): Okay. Um. Chargeback rules. Jordan, that's the same status.

Jordan Okafor (09:47): Same status. Written, reviewed by Ravi, in compliance, around the tenth.

Theo Marsh (09:53): Okay. Evals. Priya.

Priya Raman (09:56): Eval harness is done. Um, it's a nightly job, runs at two a.m., writes to the dashboard. The baseline is ninety-one on the new golden set. The vendor's at eighty-six on the same set, so that's the comparison. The cheaper model candidate came in at ninety. So basically flat. Maya, your call on whether we switch.

Maya Lindqvist (10:21): Not before the demo.

Priya Raman (10:24): Okay. Then after the demo, we can talk. Um. And reminder that I'm out Tuesday through Friday next week. The eval job runs on its own, it doesn't need me, but someone should just glance at the dashboard. Like, once. It's a green number or a red number. If it's red, ping me, I'll look from my phone, I'm not going to be in the woods.

Theo Marsh (10:54): Who's checking the dashboard while Priya's out?

Maya Lindqvist (10:58): We'll keep an eye on it.

Theo Marsh (11:02): Okay.

Priya Raman (11:04): Um. And, on the ninety-one. The late August cluster. I did not get to it. I got the model candidate done instead, and the vendor comparison. Tomás and Kenji are gone as of today, Theo, did you hear back from trust?

Theo Marsh (11:23): I asked. She said she'd let me know Tuesday.

Priya Raman (11:28): Okay. So if it's labels, it waits till I'm back anyway.

Theo Marsh (11:34): Um. Priya, before I forget. While you're out. Is there anything besides the dashboard? Like, is anything going to need you?

Priya Raman (11:44): The nightly job runs itself. The dashboard updates itself. If the warehouse goes down, the job fails, and someone would need to rerun it, but it's a button. Jordan, I'll show you the button.

Jordan Okafor (12:00): Show me the button.

Priya Raman (12:03): After this. Um. And the model candidate, that's done, the number's ninety, nothing to do there till Maya decides. And the labelers are gone. So, no. Nothing needs me. The only thing is if someone wants a different number, like, a slice by category, that's me, and that waits.

Theo Marsh (12:25): Okay. And, the late August thing.

Priya Raman (12:29): Waits till I'm back. Unless someone wants to run the chargeback comparison. It's a SQL query. I can leave it in the channel.

Maya Lindqvist (12:41): Leave it in the channel. If Jordan's bored waiting for Ravi he can run it.

Jordan Okafor (12:48): I'm not going to be bored. I've got twenty-nine PRs.

Maya Lindqvist (12:54): Fifteen. Fourteen are done.

Jordan Okafor (12:57): Fifteen PRs.

Priya Raman (12:59): I'll leave it in the channel anyway.

Theo Marsh (13:04): Okay. Um. And, the dashboard. Green means what? Like, if I look at it, what's the threshold?

Priya Raman (13:12): Um. Green is above eighty-five. Yellow's eighty to eighty-five. Red's below eighty.

Maya Lindqvist (13:19): Eighty-five's low.

Priya Raman (13:21): It's, I set it when we were at ninety-four. It's a default. I can change it.

Maya Lindqvist (13:29): Change it to ninety.

Priya Raman (13:32): I'll change it when I'm back. If I change it now it'll be yellow all week and someone's going to ping me on the boat.

Maya Lindqvist (13:44): Fair. When you're back.

Theo Marsh (13:48): Okay. I'm writing, green is eighty-five, Priya changes to ninety on the fourteenth.

Priya Raman (13:54): Sure.

Theo Marsh (13:56): Okay. Um. What's the dashboard show right now?

Priya Raman (14:01): Ninety-one on the golden set. The rolling window's at eighty-eight. I haven't changed the big number yet, so the big number is still a blend, it's like ninety. I'll fix that when I'm back.

Maya Lindqvist (14:17): Okay.

Theo Marsh (14:19): Um. So. Summary. Dispute UI done, one day late. Eval harness done. Reconciliation, half done, rest waiting on compliance, around the tenth, Jordan says a day after. Rules in compliance. Um. Normal standup stuff. Maya.

Maya Lindqvist (14:35): Agent PRs. Fourteen out of twenty-six reviewed. It opened three more. So fourteen of twenty-nine. I'm going to talk to Ops about turning down the rate on our module because it's, it's more than we can review. Um, the observability lunch was good, they have the PR thing I mentioned, I'm going to trial it on the ledger repo after the demo. It's thirty days free. Um. Oh, and I forgot to say, Arjun's team is deprecating the old dispute service endpoint on the fifteenth. The old one, the one with the flat status field. We're on the new one, so we're fine, I'm just saying it in case anyone has a script hitting the old one.

Sam Kessler (15:26): I don't think so.

Maya Lindqvist (15:29): Okay.

Jordan Okafor (15:31): The lunch. What did they actually do? Like, is it a product or a deck?

Maya Lindqvist (15:39): It's a product. They showed it running on someone's repo. It flags PRs by, like, category of risk. Retry, timeout, auth, data migration. And it writes a comment. It's not, it doesn't block anything, it just comments.

Jordan Okafor (15:56): Is it any good?

Maya Lindqvist (15:59): On the demo it was good. On the demo everything's good. We'll see on our repo.

Theo Marsh (16:07): Okay. Um. Dev stuff. Two things. One. Partial refunds. Sales asked yesterday, well, Dev asked on Tuesday and then again yesterday, if we can add partial refunds to the dispute flow. Right now it's full refund only. Dev called it small. He said, "just a field and a button."

Maya Lindqvist (16:30): How small?

Theo Marsh (16:32): He said a field and a button.

Maya Lindqvist (16:36): That's the UI. What about the rest?

Theo Marsh (16:41): He doesn't know about the rest. He's sales.

Maya Lindqvist (16:45): Um. Okay. We can probably fit it. Sam, take a look next week. Like, actually look at it, what would it take.

Sam Kessler (16:56): Sure.

Priya Raman (16:58): Wait, isn't that what you said no to on Tuesday?

Maya Lindqvist (17:04): I said not this sprint. Next week is next sprint.

Priya Raman (17:09): Okay.

Maya Lindqvist (17:11): I'm not saying yes. I'm saying look at it.

Jordan Okafor (17:16): The idempotency thing though.

Maya Lindqvist (17:20): I know about the idempotency thing. Look at it means look at the idempotency thing.

Jordan Okafor (17:27): Okay.

Theo Marsh (17:29): Okay. Um. I'm going to write down, Sam looks at partial refunds next week. No commitment.

Maya Lindqvist (17:37): No commitment.

Theo Marsh (17:40): Two. The demo script. Still no owner.

Maya Lindqvist (17:44): I know. Next week.

Theo Marsh (17:47): Okay. Um. And, uh, Dev changed slide four. It now says "streamlined dispute handling." Which is, fine, that's what it is.

Maya Lindqvist (17:57): That's fine.

Theo Marsh (18:00): And I told Nadia the report and the UI aren't connected. She said "oh." And then she said "that's fine for now." So.

Maya Lindqvist (18:11): "For now."

Theo Marsh (18:13): "For now." Yeah. She's going to ask for it.

Maya Lindqvist (18:18): Phase two.

Theo Marsh (18:21): Phase two. Um. Okay. Anything else? Um. Monday's Labor Day, no standup. If anyone wants to post in the channel, post in the channel. Back Tuesday, nine-thirty.

Jordan Okafor (18:33): Wait, did I say Monday for reconciliation? Monday's the holiday.

Theo Marsh (18:39): You said Monday.

Jordan Okafor (18:42): I mean, whenever the rules land. A day after. If the rules land the tenth, it's the eleventh.

Theo Marsh (18:51): Okay. I'll write that.

Jordan Okafor (18:54): Okay. Sorry. I keep thinking Monday's a work day.

Theo Marsh (18:59): It's fine.

Sam Kessler (19:01): Is anyone doing anything for the long weekend?

Jordan Okafor (19:06): Sleeping.

Priya Raman (19:08): Boat.

Sam Kessler (19:10): A boat?

Priya Raman (19:12): My brother has a boat. It's a small boat. It's, it's more of a raft with an engine.

Sam Kessler (19:21): That sounds nice.

Priya Raman (19:24): It's fine. It's a boat.

Theo Marsh (19:27): Okay. Thanks everyone. Have a good long weekend.

Priya Raman (19:32): Bye.

Sam Kessler (19:34): Bye.
