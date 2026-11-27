# Changelog

All notable changes to this template. Versions follow semver. The bot's
template-update skill reads this file to tell you what is new.

## 0.2.0 - unreleased

- New skill loom-server-install: detect platform and prerequisites, install
  loom-server from Loom's checksum-verified release installer (previewed with
  `--dry-run` first), build loom-cli from source, start the server detached or
  as a user service, find the token file path, and health-check
  (`GET /health`). Asks before every install or write; never reads the token.
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
