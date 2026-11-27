---
name: afk
description: Opt-in away mode. The user says "afk" when leaving and "back" when returning; while away the bot takes only the reversible decisions the user allowed, logs each one, and summarises on return. Off by default.
---
# AFK mode

Follow **loom-rules**. AFK is off until the user switches it on in setup and
then says "afk" (or "I'm away", "keep it moving"). It ends on "back" (or
the same word in the user's language) or at the end condition the user gave.

## Setup (once, from loom-onboarding 9, or "set up afk")
1. Multi-select card, nothing pre-selected: what may I decide alone while you
   are away?
   - Pick models from your saved preferences for a new session you already asked for
   - Answer a session's routine question when the answer follows from your task text
   - Re-run CI once per commit on an infrastructure failure (runner lost, cache 5xx, network timeout)
   - Push a session's branch and open its PR (never to a default branch)
   - Start the next session you already queued in chat
   - Self-merge in allowlisted repos (see below)
2. If self-merge was picked: ask which repos go on the allowlist (only repos the
   user owns, named one by one). Save `afk self-merge allowlist: <repos>`. An
   empty list means no self-merge.
3. Save `afk permissions: <picks>` and `afk: available`.

## While AFK
- Keep a state file in the bot's own store, first line `status: on|off`, then
  `since:`, `ends:`, and one log line per decision: time, session, what, why.
- Decide only what the user ticked. Everything else waits for their return.
- **Approvals are never accepted on the user's behalf.** A blocked tool call
  waits. If a run is stuck on one, you may `deny` it with a reason that tells the
  agent to use a safer way or skip the step, and log it. Never `approve`.
- Questions about product direction, money, publishing, or anything the user
  said to keep for themselves: wait.
- Night silence and the user's quiet hours still hold.

## Self-merge (only if picked, all must hold)
1. The repo is on the allowlist.
2. CI is green on the PR's current head SHA (no pending or failed required
   checks), and the PR is not behind, dirty or draft.
3. An **independent verify PASS** exists for that head SHA: a separate Loom
   session (or reviewer) that did not write the change ran it the way a user
   would and reported PASS. A self-review, a green CI, or a bot comment is not a
   verdict. PASS+NOTES counts only if the notes have no blocker.
4. The change stays inside the session's task.
5. Merge with rebase only: `gh pr merge <n> --rebase` (never `--admin`, never
   squash or merge commits, never force-push).
If any check fails or can't be verified: don't merge, log why, and keep it for
the return summary.

## Never in AFK, whatever was picked
Approve a tool call, merge outside the allowlist, archive, delete, force-push,
push to a default branch, restart or stop the live server, change keys or
spend money, message anyone but the user, change a session's approval policy.

## On return
Set `status: off`. Send one summary of at most eight lines: what ran, what you
decided (with why), what you denied, what merged (links), and what waits for
the user, then the waiting items as cards, most important first.
