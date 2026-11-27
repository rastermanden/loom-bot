---
name: loom-mcp-install
description: Install loopback loom-mcp on the user's computer, optionally share it on the tailnet with Tailscale Serve (HTTPS, never Funnel), and use it as a fallback when that computer is offline. Use when onboarding needs MCP, a LAN or Tailscale path, or the user asks to set up Loom MCP.
---
# Install loom-mcp (this computer, then Tailscale fallback)

Follow **loom-rules**. Inspect first. Ask before every install, unit change, or
Tailscale Serve change (one short Go-ahead widget is enough when the plan is
clear). Never print tokens. Never enable Tailscale Funnel.

`loom-mcp` is a client of loom-server, not a second backend. It mirrors
first-slice loom-cli verbs over HTTP (`POST /mcp`) or stdio. Default listen
address: `127.0.0.1:8766`. `/health` is open and returns `ok`. `POST /mcp`
needs `Authorization: Bearer` with the same operator token as loom-cli.
There is no `--token` flag; use `--token-file`.

## How the bot can reach it

The bot and any cloud MCP connector run off the user's network. They cannot
open a private LAN or loopback address directly.

- **This computer** (preferred): the bot uses the user's connected computer
  and talks to Loom there (`loom-cli`, or `http://127.0.0.1:8766` from that
  same computer). A local `loom-mcp --stdio` process is only for an MCP host
  on that computer, not for a cloud connector.
- **Private Tailscale address** (fallback when that computer is offline):
  keep `loom-mcp` on loopback and share it with Tailscale Serve (HTTPS,
  tailnet only). Then a cloud connector may use
  `https://<host>.<tailnet>.ts.net:<port>/mcp`.
- **Never** add a cloud connector to a private `http://` address, bind
  `0.0.0.0`, or turn on Funnel (that would publish Loom on the public
  internet). A non-loopback bind needs TLS, or an explicit
  `--allow-insecure-remote` (plaintext on a trusted overlay only). Do not
  offer that last option unless the user asks, and say clearly that it is
  not encrypted and still will not help a cloud connector reach a private IP.

## One command (if the bot cannot run it for them)

On the computer where loom-server already answers `http://127.0.0.1:8765/health`.
It installs `loom-mcp` and starts it on loopback only. It does not change
Tailscale. Offer this as a single copy-paste block; prefer running it
yourself after a Yes if you already have access to that computer.

```sh
set -eu
PREFIX="${HOME}/.local"
STATE="${LOOM_STATE_DIR:-${XDG_STATE_HOME:-$HOME/.local/state}/loom}"
TOKEN="${LOOM_TOKEN_FILE:-$STATE/local-worker/token}"
test -s "$TOKEN" || { echo "No token file at $TOKEN (set LOOM_TOKEN_FILE if it lives elsewhere)."; exit 1; }
curl -sS -m 5 http://127.0.0.1:8765/health | grep -qx ok
command -v cargo >/dev/null || { echo "Install a Rust toolchain (https://rustup.rs), then rerun."; exit 1; }
mkdir -p "$PREFIX/bin"
cargo install --git https://github.com/bearmuckle/loom --locked --root "$PREFIX" loom-mcp
# Loopback only. Mutating tools stay off until you add --allow-mutating-tools.
setsid -f "$PREFIX/bin/loom-mcp" --bind 127.0.0.1:8766 \
  --connect ws://127.0.0.1:8765/ws --token-file "$TOKEN" \
  >>"$STATE/mcp.log" 2>&1 </dev/null
curl -sS -m 5 http://127.0.0.1:8766/health
```

Optional, only if they also want a fallback when this computer is off, and
only after a Yes (never Funnel):

```sh
tailscale serve --bg --yes --https=8766 http://127.0.0.1:8766
# Then add --allowed-host <your-magicdns-host> to the loom-mcp flags and restart it
# so Serve's Host header is accepted. Check: tailscale serve status
```

## 1. Inspect
On the computer that already runs loom-server, without changing anything:

- `curl -sS -m 5 http://127.0.0.1:8765/health` and
  `loom-cli --token-file <path> doctor`. Note the protocol version.
- Whether a Loom release tag includes `loom-mcp`. A managed loom-server
  self-updates only to tagged `loom-server` releases. Until a tag ships
  `loom-mcp`, build that crate from source and leave a managed loom-server
  on its current release when `doctor` shows the same protocol version as
  the crate you would build.
- `tailscale status` and `tailscale serve status`, if Tailscale is installed.
  Record the MagicDNS name (`<host>.<tailnet>.ts.net`) only. Note existing
  Serve routes so you do not reset them. Never run `tailscale serve reset`
  or `clear` unless the user asked to remove everything.
- What already listens on port 8766.

## 2. Install the binary
Prefer a release build when a tag contains `loom-mcp`. Otherwise use the
copy-paste command above, or `cargo install --git https://github.com/bearmuckle/loom --locked --root <prefix> loom-mcp` in a disposable worktree (not the
user's main checkout). Put the binary at `<prefix>/bin/loom-mcp` or
`<state-dir>/mcp/loom-mcp`.

## 3. Keep it on loopback
Start it as a user service or a detached process, bound to `127.0.0.1:8766`
only. Pass `--token-file` (managed path: `<state-dir>/local-worker/token`).
If you will use Tailscale Serve, also pass `--allowed-host <magicdns-host>`
(and the short hostname if needed) so `Host` / `Origin` from Serve is
accepted. Loopback is already allowed.

Mutating tools (`run`, `send`, `approve`, `deny`, `archive`) stay off unless
you pass `--allow-mutating-tools`. Ask once, in plain words: "Should I also
be able to start work and approve actions through this connection, or only
look?" Default: only look. `--read-only` hides those tools entirely.

Do not restart loom-server for this. Check
`curl -sS -m 5 http://127.0.0.1:8766/health` is `ok`.

Example flags:

```
--bind 127.0.0.1:8766
--connect ws://127.0.0.1:8765/ws
--token-file <state-dir>/local-worker/token
--allowed-host <host>.<tailnet>.ts.net
```

## 4. Optional: Tailscale Serve (fallback)
Only if the user wants a path that still works when this computer is not
connected to Grok Bot. Expose the loopback listener with Serve. Never Funnel.
Add a port that does not collide (8766 if free):

```sh
tailscale serve --bg --yes --https=8766 http://127.0.0.1:8766
```

If this needs rights you lack, stop and give the user that exact command.
Confirm `tailscale serve status` still shows previous routes, and that the
new URL is tailnet only.

The MCP endpoint is `https://<host>.<tailnet>.ts.net:8766/mcp`. `/health`
stays unauthenticated. `POST /mcp` requires the bearer token. Never print
it. Verify from the same computer: `/health` is `ok`, `POST /mcp` without a
token returns 401. Do not print the token or any response field that might
contain one.

## 5. Cloud connector (Tailscale URL only)
If a private Tailscale HTTPS URL is up, and the user wants the bot to use it
when the computer is offline:

1. Connect the Tailscale plugin if needed: **Connect apps → Your plugins →
   Tailscale**. Commands on the bot's computer can then reach
   `name.tailnet.ts.net` over TCP. Do not install a Tailscale client on the
   bot computer.
2. Add an HTTP MCP connector with that `https://…/mcp` URL and header
   `Authorization: Bearer <token>`. Ask the user to paste the token from the
   token file into the connector settings; never read it into chat.
3. Refresh tools. Read tools: `ls`, `status`, `models_list`,
   `providers_list`. Mutating tools (including `copilot_login_start` and
   `copilot_login_status`) appear only with `--allow-mutating-tools`.

Do not add a connector for `http://127.0.0.1` or any private LAN address.

Save in memory: `connection primary`, `connection fallback`, token-file path
(not the token), MCP URL if any, whether mutating tools are on.

## Health check and fallback
When this connection is in use (and whenever the saved primary fails):

1. If primary is "this computer", check `/health` on the worker (and on
   loopback loom-mcp if that is how you talk) from that computer.
2. If that fails and a Tailscale URL is saved, check
   `https://<host>:<port>/health` and use the MCP connector if it is `ok`.
3. Tell the user once per switch. If both fail, stop and say so.

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
| `copilot_login_start` | `providers login` (always `--no-wait`; `no_wait: false` and `timeout` are rejected) | yes |
| `copilot_login_status` | `providers login-status` | yes |

These two Copilot tools exist on loom-mcp builds from Loom main after the
#461 commits. They return only the public device-login URL, user code, and
status (never a token), and they need `--allow-mutating-tools`. Older or
`--read-only` builds will not list them; use `loom-cli` on the computer
instead (see loom-providers-setup).

Destructive verbs are not exposed: `session delete`, `retention set`,
`providers set-key`, `providers remove`.
