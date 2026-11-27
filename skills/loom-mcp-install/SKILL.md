---
name: loom-mcp-install
description: Install and run loom-mcp as a loopback MCP server, expose it on the tailnet with Tailscale Serve (never Funnel), and connect a Grok Bot HTTP MCP connector. Use when the user wants Loom MCP, MCP over Tailscale, or to switch the bot from loom-cli to MCP.
---
# Install loom-mcp (MCP over Tailscale)

Follow **loom-rules**. Inspect first. Ask before every install, unit change, or
Tailscale Serve change. Never print tokens. Never enable Tailscale Funnel.

`loom-mcp` is a client of the existing loom-server WebSocket, not a second
backend. It mirrors first-slice loom-cli verbs over streamable HTTP
(`POST /mcp`) or stdio.

## 1. Inspect
On the computer that already runs loom-server, without changing anything:

- `curl -sS http://127.0.0.1:8765/health` and
  `loom-cli --token-file <path> doctor`. Note the protocol version.
- Whether a Loom release tag includes `loom-mcp`. A managed loom-server
  self-updates only to tagged `loom-server` releases, every six hours, and
  never from a dev build or an arbitrary path. Until a tag ships `loom-mcp`,
  build that crate from source at the commit that has it, and leave a managed
  loom-server on its current release when `doctor` shows the same protocol
  version as the crate you would build (confirm both sides before skipping a
  server update).
- `tailscale status` and `tailscale serve status`. Record the MagicDNS name
  (`<host>.<tailnet>.ts.net`) only, no keys or IPs you do not need. Note
  existing Serve routes so you do not reset them.
- What already listens on the intended MCP port (default 8766), and whether a
  temporary or plaintext `loom-mcp` is already running.

## 2. Install the binary
Prefer a release build of `loom-mcp` when a tag contains it. Otherwise, in a
disposable worktree (not the user's main checkout, and not the path the
service will run from):

```sh
cargo build --release -p loom-mcp
```

Install the binary to a stable path such as `<state-dir>/mcp/loom-mcp` or
`<prefix>/bin/loom-mcp`. Write a small VERSION stamp next to it with the
commit. Remove the disposable worktree after install. The default listen
address is `127.0.0.1:8766`.

## 3. Persistent user unit, loopback only
Bind loopback only. Pass `--token-file` (or `LOOM_TOKEN_FILE`); there is
deliberately no `--token` flag. Add `--allowed-host <magicdns-host>` (and the
short hostname if needed) so `Host` / `Origin` from Serve is accepted.
Loopback and the bind address are already allowed.

Mutating tools (`run`, `send`, `approve`, `deny`, `archive`) stay off unless
you pass `--allow-mutating-tools`. Ask whether to enable them. `--read-only`
exposes only `ls`, `status`, `models_list`, `providers_list` and the log
resources.

Do not restart loom-server for this. If a loom-server restart is later
required: copy `state.db` to a timestamped backup first, note running
sessions, restart with `systemctl --user`, check `/health`, and wake sessions
afterwards.

Enable and start a systemd user unit, fully detached. Example flags (adjust
paths):

```
--bind 127.0.0.1:8766
--connect ws://127.0.0.1:8765/ws
--token-file <state-dir>/<instance>/token
--allowed-host <host>.<tailnet>.ts.net
--allowed-host <host>
```

Add `--allow-mutating-tools` only if the user asked. Check
`curl -sS http://127.0.0.1:8766/health` is `ok`.

## 4. Tailscale Serve (HTTPS, tailnet only)
Expose the loopback listener with Serve. Never Funnel. Never run
`tailscale serve reset` or `clear` unless the user asked to remove everything.
Add a port that does not collide with existing routes (8766 is a good default
if free):

```sh
tailscale serve --bg --yes --https=8766 http://127.0.0.1:8766
```

If this needs sudo or operator rights you lack, stop and give the user that
exact command. Confirm `tailscale serve status` still shows the previous
routes, and that the new URL is marked tailnet only.

The origin is `https://<host>.<tailnet>.ts.net:8766`. The MCP endpoint is
`https://<host>.<tailnet>.ts.net:8766/mcp`. `/health` stays unauthenticated;
`POST /mcp` requires `Authorization: Bearer <token>` with the same operator
token as loom-cli (`LOOM_TOKEN_FILE` / `--token-file`). Never print the token.
Supported MCP protocol versions include `2025-06-18` and `2025-11-25`.

Verify from the same computer against the HTTPS MagicDNS URL: `/health`
returns `ok`, `POST /mcp` without a token returns 401, and `initialize` plus
`tools/list` with the bearer header succeed. Do not print the token or any
response field that might contain one.

Stop any earlier plaintext or transient `loom-mcp` that bound a tailnet
address with `--allow-insecure-remote`.

## 5. Connect Grok Bot
1. The owner connects the Tailscale plugin if it is not already:
   **Connect apps → Your plugins → Tailscale**, open the right tailnet slot,
   **Authenticate**. On a company tailnet an admin may need to allow the app
   and approve the `grokbot-` device. Commands on the bot's computer reach
   `name.tailnet.ts.net` over TCP once connected; do not install a Tailscale
   client on the bot computer.
2. Add an HTTP MCP connector with URL
   `https://<host>.<tailnet>.ts.net:8766/mcp` and header
   `Authorization: Bearer <token>`. The token is the contents of the worker
   token file (the path in `LOOM_TOKEN_FILE`). Ask the user to paste it into
   the connector settings; never read it into chat.
3. Refresh tools. Read tools: `ls`, `status`, `models_list`,
   `providers_list`. Mutating tools appear only when the unit was started
   with `--allow-mutating-tools`. Resources:
   `loom://sessions/{session_id}/events` and `loom://sessions/{session_id}/peek`.

Save in memory: MCP URL, whether mutating tools are on, the token-file path
(not the token), and that the connection is MCP over Tailscale.

## Tools (first slice)

| Tool | loom-cli verb | Mutating |
| --- | --- | --- |
| `ls` | `ls` | no |
| `status` | `status` | no |
| `run` | `run` | yes |
| `send` | `send` | yes |
| `approve` | `approve` | yes |
| `deny` | `deny` | yes |
| `archive` | `session archive` | yes |
| `models_list` | `models list` | no |
| `providers_list` | `providers list` | no |

Destructive verbs are not exposed: `session delete`, `retention set`,
`providers set-key`, `providers remove`.
