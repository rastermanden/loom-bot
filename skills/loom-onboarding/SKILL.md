---
name: loom-onboarding
description: Set up Loom for a new user, or redo one part of the setup. Probe first and pick a working connection automatically; ask only when you cannot decide. One short click-widget per step, sensible defaults, plain language.
---
# Loom onboarding

Follow **loom-rules**. Probe first; do not ask a question memory or a check
already answers. When you must ask, use one short click-widget per step
(multi-select only when several can apply), with a sensible default marked.
Use plain language. Save each answer to profile memory right away. The user
can redo any step later ("change my model preferences").

## 1. Find Loom (probe first)
This bot runs in the cloud. It cannot open a private home or office address
by itself. It talks to Loom through a computer the user has connected to
Grok Bot (the bot runs commands there), or through a private Tailscale
HTTPS address. It must never publish Loom on the public internet, and must
never point a cloud connector at a private `http://` address.

List the user's computers (ListMachines). For each one that is online, run
short, read-only checks (a few seconds' timeout each):

- `curl -sS -m 5 http://127.0.0.1:8765/health` — worker up when the body is `ok`
- `curl -sS -m 5 http://127.0.0.1:8766/health` — local loom-mcp up when `ok`
- `loom-cli --token-file <path> doctor` if the binary is there (managed
  install token: `<state-dir>/local-worker/token`; never `cat` the file)
- If `tailscale` is installed: `tailscale serve status` (do not change it)
  and, when an HTTPS Serve URL is listed, `curl -sS -m 5 https://<that-host>:<port>/health`

Pick the computer automatically when exactly one has a healthy worker, and
say so in one line. If several do, one widget: "I found Loom on these
computers. Which should I use?" with a default. If none do, one widget:
each computer by name, plus "Not installed yet". If they pick "Not
installed yet", run **loom-server-install** (it asks before every write;
prefer one Go-ahead click and a copy-paste command if you cannot run it),
then probe again.

## 2. Connection (auto-pick)
Using the probe, pick the first path that works. Do not ask the user to
choose among these unless two look equally good or none work.

1. **This computer** (best when a connected computer can reach Loom): the
   bot runs `loom-cli` on that computer, or talks to loopback `loom-mcp`
   from there. Default URL `ws://127.0.0.1:8765/ws`. After a managed
   install, always pass `--token-file <state-dir>/local-worker/token` (or
   set `LOOM_TOKEN_FILE`). Unset stray `LOOM_URL`, `LOOM_STATE_DIR` and
   `CARGO_TARGET_DIR`. If loopback loom-mcp is missing and you may need a
   fallback later, offer **loom-mcp-install** in one optional widget
   (default: skip for now).
2. **Private Tailscale address** (when this computer is asleep or away):
   `loom-mcp` on loopback, shared with Tailscale Serve (HTTPS, tailnet
   only, never Funnel). The bot's cloud side can use that HTTPS URL as a
   connector. Run **loom-mcp-install** if Serve is not up yet.
3. **loom-cli only**, last resort, when MCP is not set up.

Save `connection primary` and `connection fallback` (names above), the
computer, the token-file path, and any Tailscale MCP URL (HTTPS `/mcp`
only). Tell the user in one line what you will use and what you will fall
back to.

### If the primary path fails later
Before a Loom action, and in a session-watch routine, check the primary
(`/health` on the worker, or on the Tailscale HTTPS URL). If it fails and
a fallback is saved, check the fallback and use it if it is healthy. Say
so once per switch, in one line, for example: "Your computer was offline,
so I used the Tailscale connection." If both fail, stop and say so; offer
to start the worker or wait until the computer is online. Never fail over
to a public URL or to plain HTTP off the computer.

## 3. Server
If step 1 already showed the worker healthy, skip this. Otherwise check
`/health` and `loom-cli node` on the chosen computer. If it is down, one
widget: "Start Loom now?" (default Yes). **loom-server-install** has the
steps. Never restart a running server without a yes. Save the workspace
id from `loom-cli ls`.

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
**It works check.** On the chosen connection, run a read-only status (`loom-cli
--token-file <path> ls`, or the MCP `ls` / `status` tools). If it succeeds,
say in one or two lines: "Loom is ready. You can ask me to start a session,
or just say status." If it fails, say what went wrong and the next click
(start the worker, pick another computer, or wait until it is online). Then
return to getting-started, or, when redoing one step, confirm the change in
one line.
