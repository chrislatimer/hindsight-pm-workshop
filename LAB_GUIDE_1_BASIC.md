# Northwind Lab — Track 1: Basic

Goal: load two weeks of meeting transcripts into Hindsight and get a chat working that answers questions about them. That's it. About 40 minutes, working with a coding agent (Claude Code, Codex, or OpenCode). Any language; a terminal chat is fine.

You don't need to know Hindsight. Two prompts to your agent get the chat working; two more load the rest of the meetings and add a self-updating summary.

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

You give it raw text — here, whole meeting transcripts. It extracts the facts (who committed to what, by when, who's blocked on whom), with dates and people, into a **memory bank**. Storing is called **retain**; you steer what it extracts with a short **retain mission**. Answering a question is called **reflect**: it searches the bank, reasons, and returns an answer plus the memories it used. Extraction runs in the background for a few minutes after you load; answers improve as it finishes.

## The transcripts

A payments team called Northwind — Theo (PM), Maya (eng lead), Jordan (backend), Sam (frontend, sometimes "Samantha" or "Sam K"), Priya (evals), and Ravi from Risk — held two weeks of standups about a payments feature. `transcripts/01`–`10` cover Aug 31 – Sep 11. Nobody wrote a status report; the transcripts are all there is.

`transcripts/11`–`15` cover Sep 14 – 18. Load them one at a time, after the first ten, to watch the answers change.

`manifest.json` lists a date, timestamp, and id for every file.

## Prompt 1 — create the bank and load the transcripts

Paste this into your agent:

> Use the hindsight-docs skill. Write a script that connects to Hindsight Cloud with the API key in `HINDSIGHT_API_KEY`, creates a memory bank named `northwind-<team>`, and loads the phase-1 transcripts listed in `northwind/manifest.json`.
>
> Before loading, configure the bank:
> - `retain_mission`: "These are raw meeting transcripts. Extract commitments and their due dates — resolve 'Friday' or 'tomorrow' to actual dates using the meeting date, and keep every restatement of a date. Extract blockers and who they are waiting on, decisions, PTO and absences, and new requests. Sam, Samantha, and Sam K are the same person. Ignore greetings and logistics."
> - `retain_extraction_mode`: `verbose`.
> - `reflect_mission`: "You are chief of staff to Theo, the project manager. For every item give the owner, the current due date, any earlier dates, and the meeting it came from. The most recent statement wins."
>
> For each transcript whose number is in `phase_1_files`, retain the whole file with the manifest's `document_id` and `timestamp`, a `context` like "Northwind team standup transcript, 2026-09-08", and the `sprint:` tag from the manifest. Use asynchronous retain. Print "loading started" and exit.

Run it. Loading takes about ten minutes to finish in the background; go straight to Prompt 2.

## Prompt 2 — the chat

> Use the hindsight-docs skill. Write a chat loop over the bank. Each question is one `reflect` call at budget `mid` with the include-facts option on. Pass this in the `context` parameter: "Today is Monday September 14, 2026, 9:00 AM Pacific. Answer as of this moment." Print the answer, then the memories from `based_on` with the first ten characters of each one's `occurred_start` and its text.

Run it and ask things. Start simple — "What is Jordan working on?" — then:

- What's open, who owns it, and when is it due?
- What has slipped, and by how much?
- What is blocked, on whom, and for how long?
- What decisions have been made?
- Who has been out, and what did that affect?
- What is at risk for the Sep 16 release train?

The date in the context matters: without it, Hindsight reasons from today's real date and every "how long" is wrong. The memories printed under each answer are your evidence — each one's date is a meeting.

If answers are thin in the first few minutes, loading hasn't finished. Ask again.

## If something's wrong

- **It reports an old due date as current.** Tell the agent: *"Use the hindsight-docs skill. Add a directive to the bank: 'The most recent statement of a date is current; earlier dates are history.'"* Directives are rules applied to every answer; they take effect immediately.
- **It says something is live.** Nothing is live as of Sep 14. Add a directive: *"Merged, on the release train, and behind a feature flag are not live."*
- **Answers are fast and vague.** Ask the agent to change the budget to `high`.

## Add a mental model

So far every answer is computed on demand. A **mental model** is a question you give Hindsight once; it stores the answer and re-runs it by itself every time new memories are consolidated. Instead of asking "what's open?" over and over, you read the model.

> Use the hindsight-docs skill. Create a mental model on the bank with id `open-items`, named "Open items", with the trigger set to refresh after consolidation and to exclude other mental models. Its question: "What commitments are currently open on the Northwind project? For each: the item, the owner, the current due date, any earlier due dates and the meeting each was stated in, and whether anything is blocking it." Then add a `/model open-items` command to the chat that prints the model's current content and its `last_refreshed_at`.

Run `/model open-items`. If it says it is still refreshing, wait a minute and try again. You'll read it again after each of the next five loads and watch it change without asking anything. That is the difference between reflect (you ask, it answers) and a mental model (it keeps the answer current).

## Load the remaining five transcripts, one at a time

Transcripts 11–15 are the next five meetings (Sep 14, 15, 16, 17, 18). Load them one by one; after each, ask what changed and re-read the mental model. Tell the agent:

> Add a command `/load <n>` that retains transcript n from the manifest with the same settings as the others, then waits until the bank's `pending_consolidation` is 0 before returning.

Then, in the chat:

1. `/load 11` — when it returns, set the date to Tuesday September 15, ask "What changed since the last meeting?", then `/model open-items`.
2. `/load 12` — set the date to Wednesday September 16; same two steps.
3. `/load 13` — Thursday September 17. Release-train day: also ask whether the feature is live.
4. `/load 14` — Friday September 18; same two steps.
5. `/load 15` — Monday September 21. `/model open-items` should now be nearly empty; ask "What's still open?"

Each load takes two to four minutes to settle. Ask other questions while you wait.

## Demo

Two questions with their cited memories; "what changed" after one of the later transcripts; and the open-items mental model before and after a load.
