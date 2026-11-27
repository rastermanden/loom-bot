---
name: loom-server-install
description: Install and start loom-server (and loom-cli) on the user's computer, find its token file, and health-check it. Use when loom-onboarding step 1 is "Not installed yet", or when the user asks to install, upgrade or start Loom.
---
# Loom server install

Follow **loom-rules**. Everything here runs on the computer the user picked in
loom-onboarding step 1. **Ask before every install or write**, one step at a
time, and say in one line what it changes and where. Checks are read-only and
need no ask. Never print, read out or paste a token or key.

Source of truth: Loom is open source (public GitHub repo `bearmuckle/loom`; licenses are in its README). Its own
README and `docs/deployment.md` ("Installing from a release") win over this
file. If anything below disagrees with them or with `--help`, trust them and
tell the user. If the user runs a Loom build from somewhere else, or the
repository is not reachable, ask for their Loom source or binary and skip to
step 5 with that binary path.

## 1. Detect (read-only)
Run on the target computer:
- `uname -sm` -> platform. The release script supports Linux (x86_64, aarch64)
  and macOS only. Windows: the release page has a zip, but this skill does not
  automate it; say so and ask what the user wants.
- `command -v loom-server loom-cli curl git cargo rustc systemctl`, and
  `loom-server --version` / `loom-cli doctor` if present.
- `curl -sS -m 5 http://127.0.0.1:8765/health` to see whether a server already
  answers. If it does, do not install or start anything: go to step 6.
- `echo "${XDG_STATE_HOME:-$HOME/.local/state}"` and whether `~/.local/bin`
  is on `PATH` (`case ":$PATH:" in *":$HOME/.local/bin:"*) echo yes;; esac`).
Report what exists and what is missing in three or four lines.

## 2. Choose
One card: how should Loom run here?
- **Release install + user service/process** (recommended): this skill.
- **Loom's desktop client** ("Install a worker on this machine" in its setup
  screen): installs a self-updating worker as a user service. Choose it if the
  user wants the desktop UI too; then return to loom-onboarding step 2.
- **Container image** or **a worker on another host over SSH**: see
  `docs/deployment.md`; out of scope for this skill, help only if the user asks.
If loom-cli is missing, say that the release archives ship loom-server only,
so loom-cli is built from source (step 4).

## 3. Install loom-server from a release
Needs `curl` (or `wget`). Never `sudo`.
1. Download the installer to a temp file, don't pipe it into a shell:
   `tmp=$(mktemp -d) && curl -fsSL --proto '=https' --tlsv1.2 -o "$tmp/install.sh" https://github.com/bearmuckle/loom/releases/latest/download/install.sh`
   (a write to a temp dir; mention it).
2. Preview: `sh "$tmp/install.sh" --dry-run`. It prints the release tag, the
   archive URL, the checksum URL and the install path (default
   `~/.local/bin`), and changes nothing. Show those lines to the user.
3. Ask: "Install loom-server <tag> into <path>?" Options: Install / Use another
   folder (`--prefix <dir>`, binaries go in `<dir>/bin`) / Pin a version
   (`--version <tag>`) / Cancel.
4. On yes: `sh "$tmp/install.sh" [--prefix <dir>] [--version <tag>]`. The
   script checks the download against the release's `SHA256SUMS` before it
   unpacks, never uses sudo and never edits a shell profile. If the folder is
   not on `PATH` it prints the line to add: show it, don't edit the profile
   yourself unless the user asks.
5. Verify: `<path>/loom-server --version`. Remove the temp dir.
Note for later: a server started from an arbitrary path such as `~/.local/bin`
never updates itself (only Loom-managed installs do). Tell the user once and
offer to re-run this step to upgrade. Re-running the installer upgrades.

## 4. Install loom-cli (built from source)
Needs `git`, a Rust toolchain at or above the version Loom requires, and
several minutes of build time.
1. Check `rustc --version` against the `rust-version` in Loom's workspace
   `Cargo.toml` (at the time of writing 1.99). Missing or too old: show the
   user how to install or update Rust with rustup (`https://rustup.rs`) and
   wait. Don't install it unasked.
2. Match the server: `loom-cli` and `loom-server` should come from the same
   tag. Use the tag from step 3 (or `loom-server --version`).
3. Ask: "Build loom-cli <tag> from source into <prefix>/bin? It downloads Rust
   crates and takes a while." On yes, run it detached so nothing hangs:
   `setsid -f sh -c 'cargo install --git https://github.com/bearmuckle/loom --tag <tag> --locked --root <prefix> loom-cli > "$HOME/loom-cli-build.log" 2>&1 < /dev/null'`
   then poll the end of the log with short commands (`timeout 10 tail -n 5`)
   until it prints a finish line or an error. Report failures in plain words
   with the last log lines; don't retry in a loop. Use a clean environment
   (`env -u CARGO_TARGET_DIR`) so the build doesn't land in another tool's cache.
4. Verify: `<prefix>/bin/loom-cli doctor` after the server is up (step 5).
Fallback if `cargo install --git` fails: `git clone` the repo at the tag into a
folder the user names, then `cargo build --release -p loom-cli`, and use the
binary under `target/release/`.

## 5. Start loom-server
Only after step 6's health check shows nothing is listening. Never restart a
running server. loom-server binds `127.0.0.1:8765` by default and needs no
token or persistence flag: it creates its state and a bearer token itself. Keep
it on loopback. A bind beyond loopback needs TLS or an explicit insecure
opt-in in Loom; don't set one without the user's explicit, informed decision
(point to Loom's `docs/security.md`).

Ask which: 
- **Detached process now** (does not survive a reboot):
  `mkdir -p "${XDG_STATE_HOME:-$HOME/.local/state}/loom"` then
  `timeout 20 sh -c 'setsid -f <bin>/loom-server --bind 127.0.0.1:8765 > "${XDG_STATE_HOME:-$HOME/.local/state}/loom/server.log" 2>&1 < /dev/null'`.
  The `timeout` and `setsid -f` mean the command returns at once and never
  holds the shell open.
- **User systemd service** (survives logout; reboot too if linger is on). Only
  if `systemctl --user` works. Show the unit file and ask before writing
  `~/.config/systemd/user/loom-server.service`:
  ```
  [Unit]
  Description=Loom worker
  After=network-online.target

  [Service]
  ExecStart=<bin>/loom-server --bind 127.0.0.1:8765
  Restart=on-failure

  [Install]
  WantedBy=default.target
  ```
  then, after a yes, `systemctl --user daemon-reload` and `systemctl --user
  enable --now loom-server`. For start at boot without a login, ask separately
  before `loginctl enable-linger "$USER"`. A service doesn't see the shell's
  environment: provider variables go in the unit or in `systemctl --user
  set-environment`, or are set with loom-cli later (onboarding step 4).
Save the choice (and the unit path or log path) to memory.

## 6. Health check and token
- Health: `curl -sS -m 5 -w '\n%{http_code}\n' http://127.0.0.1:8765/health`.
  `200` with body `ok` means up. `503` ("persistence fail-stop") means the
  server stopped saving state: report it, don't restart. Connection refused or
  timeout: not running; after a start, retry a few times with short bounded
  waits (about 20 seconds in total), then show the last lines of the log
  (`timeout 10 tail -n 20 <log>`; skim for secrets before relaying).
- Token file (read-only existence check): the state root is `LOOM_STATE_DIR`,
  else `$XDG_STATE_HOME`, else `$HOME/.local/state`, plus `loom`; the token is
  `<state root>/<host>_<port>/token`, e.g. `.../loom/127.0.0.1_8765/token`,
  mode 0600. Check with `ls -l` only. Never `cat` it, and don't pass it on a
  command line. loom-cli finds this file itself.
- Then `loom-cli doctor` (and `loom-cli node`) to prove the client and server
  talk. Exit 6 means a protocol or token mismatch (versions differ or wrong
  token file); exit 7 means unreachable. See loom-rules "Status words".
- Optional, if the user has no provider yet: `loom-server --providers demo`
  on a scratch port serves a deterministic demo model for a smoke test. Only
  offer it if asked.

## 7. Finish
Summarise in three lines: versions, where the binaries and the token live (path
only), how the server runs. Save `loom server install: <how>`, the binary paths
and the workspace id (`loom-cli ls`). Return to loom-onboarding step 2 (or 4 if
connection is already proven).

## Uninstall (only if asked)
`sh <installer> --uninstall` removes the binaries it installed and keeps the
server's state. Stopping a running server or deleting state needs its own yes.
