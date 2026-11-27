---
name: loom-sessions
description: Start, watch, steer and report Loom sessions with loom-cli; relay approvals and questions as cards; status on request; archive only after asking. Use for "new session", "status", "what's waiting", "watch", "stop", "archive".
---
# Loom sessions

Follow **loom-rules**, including "Connection health": use the saved primary
path, and the fallback if the primary's `/health` check fails, with a one-line
notice. Read settings from memory (machine, connection primary and fallback,
loom-cli path, token-file path, workspace id, projects, models, approval
policy, merge policy, autonomy). If a setting is missing, ask once
(loom-onboarding has the question) and save it. Flags: **loom-cli-reference**.

## 1. Start a session
1. Find the project. Every session starts inside its project:
   `--project <root id>`. A new area gets a new project (loom-onboarding 7).
   There is no verb to move a session later.
2. Pick the shape:
   - **Ship**: changes code in its own worktree, ends in a PR or a local branch
     per the project's merge policy. Use `--repo <url>` (and `--revision
     <branch>` to continue a branch).
   - **Scout**: investigation only, ends in a report file and a three-line
     summary. No branch, no PR. Use `--as-directory <report dir>`.
3. **One confirmation card, with steps:**
   - Main model: saved preference first, then other models from `loom-cli
     models list`.
   - Children: "Same as main" / saved per-kind models / another model.
   - Approval policy: the saved one, shown, changeable.
   Never start without this card's answer.
4. Write the task text. Include: goal in the user's words, ship or scout, merge
   policy ("open a PR, do not merge" / "local branch only"), the session rules
   from loom-rules "Sessions", and "work in poteto-mode" if pstack is
   installed. Don't add design choices the user didn't make.
5. Run, with a unique name:
   `loom-cli run --workspace <ws> --project <root> --new --name <unique>
   --task-file <file> --model <main> [--child-model <id> | --child-model-for
   <kind>=<id> ...] --approval-policy <policy> --repo <url> --request-id <uuid>`
   Reuse the same `--request-id` if you must retry, so a retry never starts a
   second run. If a start times out, check `loom-cli ls` before retrying.
6. Reply with one line: session name, short id, model, and what happens next.

## 2. Status
"status": `loom-cli ls --workspace <ws>`, then `status --session <id> --tree`
for live ones. Report per project: running (what it does now, from `peek`),
waiting on the user, finished (with `result`: branch, diffstat, PR link if
any), failed. Max about eight lines. "What's waiting on me": only the
`decision` lines, each as a card (section 4).

## 3. Watch
"watch <session>" or a session the user started from chat (when the
session-watch routine is on, or autonomy is Orchestrate): check with `wait
--session <id> --until blocked --timeout <s>` or `status`. Stay silent unless:
- it finished: report result and PR link in one or two lines;
- it failed: one line why, from `peek`;
- it waits on an approval or question: card (section 4);
- dangerous behaviour (writes to the main checkout, live server, tokens, push to
  a default branch, merge, `rm -rf` outside its worktree): `stop` the run, then
  tell the user;
- the same command has run over 15 minutes: say so once, say whether it waits
  on a background process, and offer stop that process / stop the run / leave it.
Never send "still running". Stop watching when the session is terminal.

## 4. Approvals and questions
A blocked tool call (policy `default`): one line with session name, the command
in plain words and why it needs the user, then a card: **Approve** (value
`approve run <run> tool-call <id>`) / **Deny** (danger, value `deny run <run>
tool-call <id>`). On the reply, check the call is still pending, then run
`loom-cli approve|deny --run <run> --tool-call <id>` (deny with `--reason` when
given) and confirm in one line. One card per call.
A session question: forward it as a card of its options (or free text), then
`loom-cli answer --run <run> --text <answer>`.
Never approve or answer on the user's behalf. AFK mode has its own rules.

## 5. Steer and stop
"tell <session> ...": `loom-cli send --run <run> --text ...` after showing the
text. "stop <session>": `loom-cli stop --run <run>` after a yes. A delegated
child: `project send` / `project child interrupt|cancel`.

## 6. Archive
When sessions are finished or replaced, list them once and ask with a
multi-select which to archive. Then `loom-cli session archive <id>` per pick.
Never archive without a pick, never archive a project root that has live
members, and never delete. Worktree cleanup (`project worktree cleanup`) is a
separate pick; `--discard` needs a danger-styled extra confirm.

## 7. PRs
When a ship session ends with a PR: report the link and CI state (`gh pr view
<n> --json state,mergeStateStatus,statusCheckRollup`). Merge only on the user's
explicit say-so, with the repo's merge method, and only when the PR is not
behind or dirty.
