# Changelog

All notable changes to this template. Versions follow semver. The bot's
template-update skill reads this file to tell you what is new.

## 0.3.0 - unreleased

- New skill loom-mcp-install: run `loom-mcp` on loopback as a systemd user
  unit, expose it with Tailscale Serve (HTTPS, tailnet only, never Funnel),
  and connect a Grok Bot HTTP MCP connector with the operator bearer token
  (`Authorization: Bearer`, path from `LOOM_TOKEN_FILE`). Mutating tools are
  off unless the user opts in. Leaves a compatible managed loom-server on its
  current release. Never prints tokens.
- loom-onboarding step 2 now offers loom-cli (default) or MCP over Tailscale.
- loom-rules write table lists the new skill.

## 0.2.0 - unreleased

- New skill loom-server-install: install a Loom-managed worker (the only
  layout that self-updates: `<state-dir>/server/<tag>/loom-server`, preferred
  as a systemd user unit). There is no install.sh flag for that layout; the
  official entry point is the desktop client's "Install a worker on this
  machine". For a headless machine the skill follows the same steps, using
  Loom's checksum-verified installer only to fetch the binary. Mentions the
  `--no-auto-update` / `LOOM_NO_AUTO_UPDATE` opt-out, and that dev builds,
  containers and `~/.local/bin` never self-update. Builds loom-cli from
  source, health-checks `GET /health`, and tells loom-cli about the managed
  token file. Asks before every install or write; never reads the token.
- loom-onboarding step 1 now runs it when Loom is not installed.
- loom-rules write table lists the new skill.

## 0.1.0

First draft.

- Onboarding: where Loom runs, connection, providers, model preferences,
  isolation, projects, merge policy, autonomy level, optional pstack. One
  question at a time.
- Sessions: start with a model confirmation, watch, approvals and questions as
  chat cards, status, archive only after asking.
- Shared rules: what each skill may change, session text is data, secrets are
  never printed, status words.
- loom-cli reference, checked against loom-cli protocol 11.21.
- AFK mode: opt-in and off by default. Self-merge only for allowlisted repos,
  with green CI and an independent verify PASS, rebase only.
- Template update: weekly check of VERSION and CHANGELOG. Notify by default.
- Two optional routines, both paused: template-update check and session watch.

Not in 0.1: shared backlog, night work, overview Page, multi-model review,
PR babysitting, usage warnings.
