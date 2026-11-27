---
name: loom-cli-reference
description: loom-cli verbs and flags used by Loom Pilot, with exit codes. Read before composing a loom-cli command. Checked against a specific loom-cli version; verify with `loom-cli --help` when in doubt.
---
# loom-cli reference

Checked against: loom-cli built from Loom workspace version 0.1.0, protocol
11.21 (`loom-cli doctor`), help text of late November 2026. loom-cli has no
`--version` flag; use `doctor` for the protocol. If a flag below is rejected
(exit 2), read `loom-cli <verb> --help` and trust it over this file.

## Global flags
`--connect <url>` (fallback `LOOM_URL`, then `ws://127.0.0.1:8765/ws`),
`--token-file <path>` (fallback `LOOM_TOKEN_FILE`, then
`<state dir>/<host>_<port>/token`). After a Loom-managed install the token is
`<state-dir>/local-worker/token`; pass `--token-file` with that path. Then
`--ca-file <pem>`,
`--allow-insecure-remote`, `--json` (one JSON envelope on stdout). There is no
`--token` and no `--key` flag, by design.

## Read
- `doctor`, `node`, `whoami`
- `ls [--workspace <id>] [--archived]`
- `get <session> [--metadata]`
- `status --session <id> [--format text|verb] [--tree]`
- `peek --session <id> [--lines <n>]`
- `events --session <id> [--after <seq>] [--follow] [--timeout <s>] [--format text|jsonl]`
- `wait --session <id> [--until blocked|terminal] [--timeout <s>]` (always bounded; exit 4 on timeout)
- `liveness --session <id>`
- `result --session <id> [--task <id>]` (branch, head, diffstat, changed paths, evidence)
- `project get|tree|tasks|children|messages|worktrees <session>`
- `project models get <root>`
- `providers list`, `models list [--provider <id>]`

Session ids may be a unique prefix (e.g. the first eight characters). An
ambiguous prefix is exit 2 and lists the matches.

## Start a run
```
run --workspace <id> (--task <text> | --task-file <path>) [--name <name>] [--new]
    [--project <root session id>] [--model <id>]
    [--repo <source> [--revision <rev>] [--path <path>]] | [--as-directory <path>]
    [--approval-policy default|auto]
    [--child-model <id>] [--child-model-for <kind>=<id>]...
    [--request-id <uuid>]
```
- Without `--new` it reuses the workspace session with that name. With `--new`
  it always creates one; names are labels, so use unique ones.
- `--project` creates the session as a sub-project of that root and inherits
  its inputs unless `--repo`/`--as-directory` is given.
- Without `--model` the first credentialed provider's default model starts.
- `--approval-policy` is pinned on a new session; on a reused session it changes
  only when given. A live run can't change policy.
- `--repo` and `--as-directory` are mutually exclusive.
- Prints `session <id> <name>`, `run <id> <state>` and, with `--project`,
  `parent <id>`. It returns as soon as the run is accepted.
- Project child models later: `project models set <root> [--child-model <id> |
  --no-child-model] [--child-model-for <kind>=<id>]... [--unset-kind <kind>]... [--clear]`.

## Steer
- `send --run <id> --text <text>`
- `answer --run <id> --text <text>`
- `approve --run <id> --tool-call <id>`
- `deny --run <id> --tool-call <id> [--reason <text>]`
- `stop --run <id> [--session <id> --archive] [--retry-on-conflict]`
- `project send <session> (--text <t> | --text-file <p>)`
- `project child pause|resume|interrupt|cancel <task-id> --session <id>`

## Lifecycle (always the user's pick)
- `session archive <id> [--yes]` (`--yes` needed for a project root with descendants)
- `project archive <root>`
- `project worktree cleanup <session> --task <id> (--keep | --remove | --discard) [--yes]`
- `session delete` / `delete` / `project delete`: irreversible. The bot does not use them in v0.1.

## Providers (onboarding only)
- `providers add --id <id> --kind openai-compatible --endpoint <url> [--model <id>]... [--default-model <id>]`
- `providers set-key --provider <id> (--from-env <NAME> | --key-file <path>)`
- `providers discover --provider <id>`, `providers remove --id <id>`

## Idempotency
`--request-id <uuid>` on `run`, `send`, `answer`, `approve`, `deny`, `stop`,
archive, delete and worktree verbs makes a retry answer from the server's cache
instead of acting twice (`replayed: true` in `--json`). Reuse the id only for
the same mutation.

## Exit codes
0 ok, 1 internal, 2 usage, 3 blocked, 4 timeout, 5 not found or gone, 6 protocol
or auth mismatch, 7 transport unreachable. Plain-words meanings: loom-rules
"Status words".
