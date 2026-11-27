---
name: loom-onboarding
description: Set up Loom for a new user, or redo one part of the setup (machine, providers, models, isolation, projects, merge policy, autonomy, pstack). One short question at a time.
---
# Loom onboarding

Follow **loom-rules**. Ask one short question at a time, on a card where the
answer is a pick (multi-select only where several apply). Save each answer to
profile memory right away. Skip any question memory already answers. The user
can redo any step later ("change my model preferences").

## 1. Where Loom runs
List the user's computers (ListMachines) and ask which one runs, or should run,
loom-server. Options: each computer by name, "A server on my tailnet", "Not
installed yet". Prove access with one read-only command (`uname -a`). If Loom
is not installed, point to Loom's own install docs and stop until it is.

## 2. Connection
Today the bot runs `loom-cli` through shell access on that computer. loom-cli
defaults to `ws://127.0.0.1:8765/ws` and the token file under its state dir, so
no flags are needed. A Loom MCP connector is planned: record the preference if
asked and keep using loom-cli. Before running loom-cli from a shell that may
carry another session's environment, unset `LOOM_URL`, `LOOM_TOKEN_FILE`,
`LOOM_STATE_DIR` and `CARGO_TARGET_DIR`. Check with `loom-cli doctor`. Save the
loom-cli path you used.

## 3. Server
Check `curl -s http://127.0.0.1:8765/health` and `loom-cli node`. If the server
is not running, ask: start it now (fully detached, with logs to a file), or
install it as a user service. Never restart a running server without a yes.
Save the workspace id from `loom-cli ls`.

## 4. Providers
Run `loom-cli providers list` and `loom-cli models list`. If none has a
credential, ask which provider the user has. For an API key, the user writes it
to a file on their computer, then run `loom-cli providers set-key --provider <p>
--key-file <path>`. Never ask for the key in chat and never print it. Prove
inference with a one-word test session. Per provider, ask: plan type
(subscription / prepaid / unknown) and preference (use first / normal / only
when needed / don't use).

## 5. Models per kind of work
From `loom-cli models list` only, ask the preferred model for: main session,
delegated children (one for all, or per kind: cheap, coding, review, vision,
reasoning). Offer "Decide each time" as an option. Save the mapping. Ship no
defaults of your own.

## 6. Isolation
Say in two sentences: on a host install, agent commands run as the user with
access to their files, git logins and environment; the session folder is not a
sandbox. Ask: host install, or Loom's container image with only chosen folders
mounted and a narrowly scoped token. Save it.

## 7. Projects
Ask what Loom will be used for. Offer one project per repo or area. Create each
with `loom-cli run --workspace <ws> --new --name <name> --repo <url> --task
<short first task>` (or `--as-directory <dir>`), after the user confirms the
model card (loom-sessions step 1). Save each project root id with its repo.

## 8. Merge policy (per project)
Ask per project, default first:
- **I merge** (default): sessions open PRs, the bot reports them, merges only on
  the user's explicit say-so.
- **Local only**: commits stay on a local branch, no push.
Also ask the repo's merge method if more than one is allowed (rebase / squash /
merge). Bot self-merge is not offered here: it exists only inside AFK mode, for
repos the user adds to the AFK allowlist (step 9).

## 9. Autonomy
Ask once: "How hands-on should I be?"
- **Monitor** (default): I relay questions and approvals, report finishes, and
  step in only when a session is stuck or broken.
- **Orchestrate**: Monitor plus starting watches and several sessions when you
  ask, without interrupting healthy runs.
Then ask the approval policy for new sessions: `default` (each unlisted tool
call waits for an approval card) or `auto` (session runs tools without asking).
Explain the risk of `auto` in one line: agent commands run unasked inside the
session's checkout and with the user's rights on a host install.
Then: "Want AFK mode available? You say 'afk' when you leave and I take fewer
questions to you." On yes, run **afk** "Setup". On no, AFK stays off.

## 10. pstack (optional)
Ask once: "pstack (Lauren Tan's poteto principles) gives sessions a stricter
quality bar: verify by running, smallest change, adversarial review on demand.
Install it?" On yes, find it with SearchPlugins ("pstack") and give the add
link for the official Cursor pstack plugin. Save `pstack: yes|no`. When yes,
loom-sessions may add "work in poteto-mode" to task text. Never nag again.

## Finish
Return to getting-started, or, when redoing one step, confirm the change in one
line.
