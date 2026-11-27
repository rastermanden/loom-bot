# Changelog

All notable changes to this template. Versions follow semver. The bot's
template-update skill reads this file to tell you what is new.

## 0.4.0 - unreleased

- New skill loom-providers-setup, wired into onboarding and Quick start: one
  short question at a time to set up at least one Loom provider. GitHub
  Copilot uses the worker device-login flow (MCP `copilot_login_start` /
  `copilot_login_status` when those tools exist, otherwise
  `loom-cli providers login --no-wait --json` and `login-status`). The bot
  shows the verification URL and one-time user code, polls until done or
  expired, and confirms with `providers list` / `models list`.
- API keys for DeepSeek, OpenCode Go, and OpenCode Zen (pay-as-you-go, needs
  credits) are collected with Grok Bot's masked secret form (a box
  environment secret, never in chat), written to a mode-0600 key file, then
  stored with `providers set-key --key-file`. Keys are never printed or
  passed as CLI arguments. Official key-page URLs are in the skill.
- TypeSafe is documented as not a Loom provider: the bot stores the key as a
  box secret and can optionally write the local loom-cli `jev` key file.
- After setup, the bot asks for models per kind of work (`cheap`, `coding`,
  `review`, `vision`, `long-reasoning`) from `models list`, and flags models
  that train on request data when the provider's published policy says so.

## 0.3.0 - unreleased

- New skill loom-mcp-install: run `loom-mcp` on loopback, optionally share it
  with Tailscale Serve (HTTPS, tailnet only, never Funnel), and use that as
  a fallback when the user's computer is offline. Explains that the bot
  cannot reach a private LAN address from the cloud. One copy-paste install
  command; mutating tools off unless the user opts in. Never prints tokens.
- Onboarding auto-detects a working path, in order: the user's connected
  computer, then a Tailscale HTTPS URL, then loom-cli. It only asks when it
  cannot decide, stores a primary and a fallback, and switches with a one-line
  notice if the primary fails a `/health` check. Ends with an "it works"
  status/ls check.
- README Quick start: three steps from template to first session.

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
