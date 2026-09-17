Northwind Demo Debrief — Tuesday, September 15, 2026

Theo Marsh (00:00): Okay, we're recording, everyone's back. Um, Ravi, thanks for staying. So. Mostly good. Let's talk about the crash first and then everything else.

Sam Kessler (00:11): Yeah. Okay. So. Um. It's the currency thing.

Maya Lindqvist (00:15): It's the currency thing.

Sam Kessler (00:19): It's the currency thing. So Nadia clicked into a dispute, which, fine, that's the demo, and she clicked accept, and it was the one with the euro transaction. Because I'd loaded the prod-like snapshot into the demo environment last night, Jordan's snapshot, because I wanted real-looking data on the big screen, and the prod-like snapshot has euro transactions in it because prod has euro transactions in it. Staging doesn't. So the modal tried to convert the amount to minor units using the USD helper, and the helper doesn't handle the currency code, and it threw. And the modal doesn't catch that. So it crashed.

Theo Marsh (01:04): Okay.

Sam Kessler (01:06): I know exactly what it is. It's the ticket I filed on the ninth. It's the thing Maya asked about on the second. It's one in a thousand and Nadia found it on the first click.

Maya Lindqvist (01:23): She found it on the first click.

Sam Kessler (01:27): I'll have the fix in tonight. It's, um, it's not a full currency fix, it's, if the currency isn't USD, show a warning and don't let them refund from the UI. Like, escalate it instead. A real fix is converting and that's, that needs the FX rate service, that's not tonight.

Maya Lindqvist (01:50): Tonight meaning before the noon cutoff tomorrow.

Sam Kessler (01:55): Yes. Tonight. It's, like, an hour. I'll do it after this.

Maya Lindqvist (02:01): Walk me through the fix. Like, exactly. Because it's going in the train and I'm not going to have time to read it.

Sam Kessler (02:12): Okay. So. In the detail view, when the dispute loads, I read the currency field, which is already there, I've just been ignoring it. If it's USD, nothing changes. If it's not USD, the accept button is replaced with an escalate button, and there's a banner at the top of the detail view that says "this dispute is in" and the currency, "and can't be refunded from here." Escalate already exists, it's the same escalate as the evidence-requested state, it just moves it to escalated. So a human with ledger access handles it. That's it. It's, the banner's a component I already have, the button swap is a conditional. It's like forty lines.

Maya Lindqvist (03:01): And the modal.

Sam Kessler (03:04): The modal can't open for non-USD because the button that opens it isn't there. But I'm also going to put a guard in the modal itself, so if someone somehow opens it with a non-USD dispute, it refuses instead of converting. Belt and braces.

Maya Lindqvist (03:24): Good. And a test.

Sam Kessler (03:27): And a test. With a euro fixture. Which we've never had. Which is the actual reason this happened.

Maya Lindqvist (03:36): That's the actual reason.

Jordan Okafor (03:39): For what it's worth, the reconciliation side handled the euro ones fine. They matched on settled amount. So the report was right. It was just the UI.

Sam Kessler (03:52): It was just the UI. Yeah. Thanks.

Ravi Chandrasekaran (03:57): Is escalated a state Risk sees?

Maya Lindqvist (04:01): Escalated goes to a queue that, right now, nobody's watching, because nothing's live. When it's live, it's Nadia's team.

Ravi Chandrasekaran (04:10): Okay. So the euro disputes just sit in escalated until someone with ledger access does them by hand.

Maya Lindqvist (04:19): Until the thirtieth, when we do the FX thing. Yes.

Ravi Chandrasekaran (04:25): How many is that?

Sam Kessler (04:28): One in a thousand. So on prod volume, that's, what, a couple a day?

Jordan Okafor (04:35): About that. Two, three a day.

Ravi Chandrasekaran (04:39): Okay. That's fine. That's a spreadsheet's worth. I just want to know.

Theo Marsh (04:46): I'll tell Nadia. Euro disputes go to escalated, a couple a day, until the thirtieth.

Maya Lindqvist (04:54): Yes.

Theo Marsh (04:55): Okay. Um. I want to say, Nadia was very nice about it.

Sam Kessler (05:02): She was very nice about it. She said "oh, that's what happens with the euro ones?" like it was a feature.

Ravi Chandrasekaran (05:12): She's seen worse. Trust me.

Theo Marsh (05:16): Okay. Jordan.

Jordan Okafor (05:18): Reconciliation demo was clean. The prod-like snapshot matched, unmatched rate was two point one percent, which is, that's a real number, that's what it would be on prod. Nadia asked about the by-day view and I showed it and she said that's what she wanted. Um, she asked if it could email her. I said not yet.

Maya Lindqvist (05:44): Not yet.

Jordan Okafor (05:46): I said not yet.

Theo Marsh (05:49): Write it down. Um. Priya.

Priya Raman (05:53): Um. The eval number I showed was eighty-nine. I split the dashboard yesterday like I said, and the golden set alone is ninety-one, the rolling window alone is like eighty-four, and the blend is eighty-nine. I showed the blend because that's what the dashboard shows and I didn't want to explain it. Nadia asked about it. I said we're investigating. Which is true.

Maya Lindqvist (06:21): Did she push?

Priya Raman (06:23): No. Dev pushed, actually. He asked if it was better than the vendor. I said the vendor's at eighty-six on the same set and he said "so yes" and I said yes.

Maya Lindqvist (06:38): Okay.

Priya Raman (06:40): Um. On the actual problem, I got through about half the relabeling yesterday. It's the labels. I'm like ninety percent sure. The vendor's format change moved the fraud flag column and about a third of the batch from the twenty-sixth came in with the flag inverted, and the labeling agent used that as a prior. I'll have a clean number tomorrow morning.

Theo Marsh (07:08): Um. Priya. The rerun. You said a clean number this morning. Was there one?

Priya Raman (07:15): There was a number. It wasn't clean. The relabel finished at like eleven last night and the eval job runs at two, and I was asleep, and this morning it said ninety-two, which is better, but I only did half the batch. So it's half fixed. I'll do the other half today and tomorrow morning it's a real number.

Maya Lindqvist (07:42): Ninety-two on half.

Priya Raman (07:45): Ninety-two on half. Which is consistent with, like, ninety-three or ninety-four on all of it. Which is where we were. Which is the theory.

Maya Lindqvist (07:56): Okay. So why did you show eighty-nine?

Priya Raman (08:01): Because the dashboard I showed was the split one and the golden set number on it was ninety-one, and the blend was eighty-nine, and I showed the wrong panel because I was nervous and I clicked the wrong tab. I said eighty-nine. I should have said ninety-one. It's fine. Nadia doesn't care.

Maya Lindqvist (08:24): Nadia doesn't care. Dev cares.

Priya Raman (08:28): Dev heard "better than the vendor" and stopped listening.

Maya Lindqvist (08:33): Okay.

Theo Marsh (08:35): For the record, the number tomorrow is the number. Whatever it is.

Priya Raman (08:41): Whatever it is.

Theo Marsh (08:44): Okay. Um. Ravi.

Ravi Chandrasekaran (08:47): Yeah. Um. So, the lighter thing. The go-live meeting I told Jordan about. I've got it scheduled now and I wanted to say it in the room rather than in a message. Because the chargeback rules change dispute handling, Risk needs a formal sign-off before this goes live to customers. It's not the compliance review, that's done, that's the rules. This is the go-live sign-off. It's a different thing.

Maya Lindqvist (09:17): Is it a form?

Ravi Chandrasekaran (09:20): It's a meeting. It's me, my boss, and someone from compliance, and someone from your side walks us through what's going live. Twenty minutes. I've got one on the calendar for Thursday morning, eight-thirty, because that was the first slot my boss had.

Maya Lindqvist (09:40): Thursday.

Ravi Chandrasekaran (09:42): Thursday. Um. So. You can ship on the train tomorrow. That's fine. But it stays dark, feature-flagged off, until we sign off. And then you flip it.

Maya Lindqvist (09:55): So we ride the train tomorrow with the flag off, and flip it after Thursday.

Ravi Chandrasekaran (10:03): Right.

Theo Marsh (10:04): Is there anything that could come out of the sign-off meeting that would block the flip?

Ravi Chandrasekaran (10:13): Um. Honestly, probably monitoring. My boss is going to ask what happens if the reconciliation job starts flagging everything. Like, what's the alert. If you have an answer for that, it's fine.

Jordan Okafor (10:28): We don't have an alert. We have a retry and a page if the file's missing. We don't have an alert on the mismatch rate.

Ravi Chandrasekaran (10:40): Then he's going to ask for one.

Jordan Okafor (10:44): Okay.

Ravi Chandrasekaran (10:46): I'm just telling you now so it's not a surprise Thursday.

Theo Marsh (10:52): Thanks. That's, that's actually really useful.

Ravi Chandrasekaran (10:56): Yeah. I've, um, I've learned from the compliance form thing.

Jordan Okafor (11:02): Did you send me the form, by the way?

Ravi Chandrasekaran (11:07): I did not. I'll send it.

Jordan Okafor (11:11): Okay.

Maya Lindqvist (11:13): Let's talk about tomorrow for a second. The cut. Because I want everyone clear on what "on the train" means for us. The train's a release branch cut at noon. Whatever's merged to main by noon is on it. It deploys to prod Wednesday evening, usually around six. Our stuff is behind the feature flag, "northwind_disputes," which is off. So Wednesday night, the code's on prod, nothing's visible. Thursday, Ravi's sign-off. Then we flip the flag. Flipping the flag is a config change, not a deploy.

Jordan Okafor (11:51): And the reconciliation job. Is that behind the flag too?

Maya Lindqvist (11:56): The job's scheduled. It's, the schedule's off on prod. Turning the schedule on is also a config change. Same flip.

Jordan Okafor (12:06): Okay. So Thursday, two config changes.

Maya Lindqvist (12:10): Two config changes. The flag and the schedule.

Ravi Chandrasekaran (12:15): And the alert. If my boss asks for it.

Maya Lindqvist (12:20): And the alert. Which would be before the flip. Which is, Jordan, if he asks for it Thursday morning, can you have it Thursday?

Jordan Okafor (12:32): If it's a threshold and a pager route, yes. If it's a dashboard, no.

Ravi Chandrasekaran (12:39): It's going to be a threshold. He's going to say "page someone if it goes above X."

Jordan Okafor (12:48): Then Thursday.

Maya Lindqvist (12:50): Okay. Um. Sam, tonight. Who's reviewing your fix?

Sam Kessler (12:55): You?

Maya Lindqvist (12:57): I'll be up. Ping me when it's up. I want it merged tonight, not tomorrow morning, because tomorrow morning is when everything else in the company is trying to merge before noon and CI's going to be a mess.

Sam Kessler (13:15): Okay. Tonight.

Theo Marsh (13:17): Um. And, I want to name it. Sam's fix is a demo finding. We said on the ninth demo findings don't block the train unless they're ledger-correctness. This one's, it's UI, but it would have posted the wrong amount to the ledger. So it's, it's on the line.

Maya Lindqvist (13:39): It's on the line. It's going in. It's small and it's tonight. If it's not in by noon tomorrow, we ship without it and the escalate path is, it's just, non-USD disputes crash the UI until the thirtieth.

Sam Kessler (13:56): It'll be in.

Maya Lindqvist (13:59): I know. I'm saying what the fallback is.

Theo Marsh (14:04): Okay. Written.

Priya Raman (14:06): Um. Can I ask about the eval number and the train. Because Maya said yesterday we ship at eighty-nine with the flag off. But if it's really eighty-nine, do we flip on Thursday?

Maya Lindqvist (14:21): If it's really eighty-nine, that's a conversation Thursday. Not today.

Priya Raman (14:27): It's not going to be eighty-nine. I'll have a clean number tomorrow morning.

Maya Lindqvist (14:34): Then it's not a conversation.

Priya Raman (14:38): Okay. I just wanted it said.

Theo Marsh (14:42): It's said.

Theo Marsh (14:44): Okay. So. Let me say it back. Sam's fix tonight. Train tomorrow at noon, flag off. Risk sign-off Thursday eight-thirty. Probably need a mismatch rate alert. Flip after that. Um. Maya, who goes to the sign-off?

Maya Lindqvist (15:01): Me and Jordan.

Jordan Okafor (15:03): Okay.

Theo Marsh (15:05): Um. Other questions from the room. Let me go through what I wrote down. The spreadsheet person, whose name is Marcus, asked if the age column could be sorted by, um, by merchant and then age. Like, grouped.

Sam Kessler (15:23): It can be sorted by anything. Grouping's a different thing. I'll write it down.

Theo Marsh (15:30): Marcus also asked about export. Jordan showed the download link. Marcus was happy.

Jordan Okafor (15:37): Everyone asks about export.

Theo Marsh (15:40): Nadia asked about the dispute queue connection. I gave the answer. She said "fifty dollars, non-fee," which she'd already told me, and then she said "when." I said after the rules are live and we've seen a week of real data.

Maya Lindqvist (15:59): That's fine. That's the first week of October.

Theo Marsh (16:04): I didn't say October. I said after a week of data.

Maya Lindqvist (16:10): Okay.

Theo Marsh (16:12): Ravi's boss asked about governance. Maya answered. He seemed fine.

Ravi Chandrasekaran (16:17): He was fine. He liked "config." He likes things that are config.

Theo Marsh (16:24): Dev asked about partial refunds. Maya said the thirtieth. Dev asked if the thirtieth was a hard date. Maya said it was a target. Dev wrote down "the thirtieth."

Maya Lindqvist (16:37): I saw him write it down.

Theo Marsh (16:41): The other sales lead, the new one, asked if the fraud score was shown in the dispute UI. Like, per dispute.

Sam Kessler (16:52): It's not.

Priya Raman (16:54): It could be. The score's in the transaction record. It's a field.

Sam Kessler (17:01): It's a field I don't read.

Priya Raman (17:04): It's a field you could read.

Maya Lindqvist (17:08): Not this train. Write it down. That's, honestly, that's a good idea, that's a thirtieth thing.

Sam Kessler (17:17): Okay. Fraud score in the detail view. Thirtieth. Written.

Theo Marsh (17:22): And, um, Nadia's analyst asked what happens on a day when the settlement file's late.

Jordan Okafor (17:30): The retry thing.

Theo Marsh (17:32): Which you said was Monday.

Jordan Okafor (17:36): Which was yesterday. It's in. It's merged. Maya looked at it. Three tries, an hour, then page.

Theo Marsh (17:44): Okay. So that one's answered.

Jordan Okafor (17:48): I told her that.

Theo Marsh (17:51): You told her that. Good. Um. I think that's all the questions. Nobody asked about the euro thing, because nobody had to, because it crashed.

Sam Kessler (18:03): Thanks.

Theo Marsh (18:05): Sorry.

Theo Marsh (18:07): Um. Anything else from the demo? Dev?

Maya Lindqvist (18:11): Dev asked about partial refunds again. I said the thirtieth. He said "the thirtieth is a Wednesday" like that meant something. I said yes.

Theo Marsh (18:23): Okay.

Maya Lindqvist (18:25): Um. And, finance liked it. Nadia liked it. Like, actually liked it, she asked when she could use it. That's the headline.

Theo Marsh (18:35): That's the headline. Okay. Um. Sam, go fix it. Everyone else, tomorrow, nine-thirty, and then noon.

Sam Kessler (18:44): Going.

Ravi Chandrasekaran (18:45): Good luck.
