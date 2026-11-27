---
name: loom-server-install
description: Install and start a Loom-managed loom-server (the layout that self-updates) and loom-cli, then health-check it. Use when loom-onboarding step 1 is "Not installed yet", or when the user asks to install, upgrade or start Loom.
---
# Loom server install

Follow **loom-rules**. Everything here runs on the computer the user picked in
loom-onboarding step 1. **Ask before every install or write**, one step at a
time, and say in one line what it changes and where. Checks are read-only and
need no ask. Never print, read out or paste a token or key.

Source of truth: Loom is open source (public GitHub repo `bearmuckle/loom`;
licenses are in its README). Its README, `docs/deployment.md` and `--help`
win over this file. If the user runs a Loom build from somewhere else, or the
repository is not reachable, ask for their Loom source or binary and skip to
step 5 with that binary, still placing it in the managed layout below.

## The sanctioned layout (read this first)

Only a worker whose executable is
`<state-dir>/server/<tag>/loom-server`, with `<tag>` a real release tag such as
`v0.17.13`, updates itself. That is the layout the desktop client's **"Install
a worker on this machine"** produces
(`docs/deployment.md`, "A worker installed by the desktop client" and
"Worker auto-update"; implemented by `loom_local::local::install_and_start` in
Loom's `crates/loom-local/src/local.rs`). The SSH provisioner (`loom
--ssh-worker`) produces the same `server/<tag>/` shape on a remote host.

There is **no** `install.sh` flag and **no** `loom-server` subcommand that
creates this layout. `scripts/install.sh` puts `loom-server` in `~/.local/bin`
(or `--prefix`). A server started from that path, a development build, or a
container **never** replaces itself; update those through their image or
package (`docs/deployment.md`, "Worker auto-update"). Do not leave the running
worker in `~/.local/bin`.

Self-update, once the managed layout is running: the worker checks shortly
after start and then every six hours, considers only a newer stable release,
verifies the archive against the release's `SHA256SUMS`, waits until no agent
run is active, then either rewrites the systemd user unit's `ExecStart`
(`server/<old>/` → `server/<new>/`) and restarts the unit, or starts the new
binary with `--restart-after <pid>`. **Opt out** with `--no-auto-update` or
`LOOM_NO_AUTO_UPDATE=1`. Don't add either unless the user asks.

`<state-dir>` is `LOOM_STATE_DIR` if set, otherwise
`${XDG_STATE_HOME:-$HOME/.local/state}/loom`.

The official installer also uses, and this skill must use:
- token file `<state-dir>/local-worker/token` (mode 0600), passed as
  `--token-file` (never on the command line as `--token`)
- database `<state-dir>/state.db`, passed as `--persistence`
- bind `127.0.0.1:8765`
- systemd user unit `loom-server.service` when `systemctl --user` works
  (preferred; survives reboot if linger is on); otherwise a detached process
  with a pidfile under `<state-dir>/local-worker/` (does not survive reboot)

`loom-cli` does **not** find this token file by default (it looks under
`<state-dir>/<host>_<port>/token`). After a managed install, always pass
`--token-file <state-dir>/local-worker/token`, or set `LOOM_TOKEN_FILE` to that
path. Never `cat` the file.

## 1. Detect (read-only)
Run on the target computer:
- `uname -sm` -> platform. The release installer supports Linux (x86_64,
  aarch64) and macOS only. Windows: say the skill does not automate it.
- `command -v loom-server loom-cli loom-ui curl git cargo rustc systemctl`,
  and `loom-server --version` / `loom-cli doctor` if present.
- `curl -sS -m 5 http://127.0.0.1:8765/health`. If a server already answers,
  do not install or start anything: report how it is running (managed path
  vs `~/.local/bin` vs a dev build vs a container) and go to step 6. Offer
  to migrate a non-managed worker only if the user asks; never restart one
  without a yes.
- Print `<state-dir>` as above, whether `<state-dir>/server/*/loom-server`
  exists, and whether `systemctl --user show-environment` works.
Report what exists and what is missing in a few lines.

## 2. Choose
One card: how should Loom be installed here?
- **Headless managed install** (recommended for this bot): this skill follows
  the same steps `install_and_start` performs, with a systemd user unit when
  available. Use this when there is no display, or the user wants the bot to
  do it.
- **Desktop client setup** (the official entry point, when a display is
  available): install `loom-ui` with Loom's `install.sh --client` (preview
  with `--dry-run`, then ask), start `loom-ui`, and have the user click
  **"Install a worker on this machine"**. That is `install_and_start`. When
  it is healthy, go to step 6. `install.sh --client` may also put a
  `loom-server` in `~/.local/bin`; that copy is not the running worker.
- **Container image** or **a worker on another host over SSH** (`loom
  --ssh-worker`): see `docs/deployment.md`. Out of scope here unless asked.

If loom-cli is missing, say that release archives ship `loom-server` only, so
loom-cli is built from source (step 4).

## 3. Place loom-server in the managed layout
Needs `curl` (or `wget`). Never `sudo`. Use Loom's official installer only to
download and verify a release; then copy the binary to the managed path.
1. Download the installer to a temp dir, don't pipe it into a shell:
   `tmp=$(mktemp -d) && curl -fsSL --proto '=https' --tlsv1.2 -o "$tmp/install.sh" https://github.com/bearmuckle/loom/releases/latest/download/install.sh`
2. Preview: `sh "$tmp/install.sh" --dry-run --prefix "$tmp/prefix"`. It prints
   the tag, archive URL, checksum URL and a *temporary* prefix, and changes
   nothing. Show those lines. The tag (for example `v0.17.13`) is the
   directory name under `server/`. A name that is not a release tag (such as
   `bundled`) will not self-update.
3. Ask: "Verify and install loom-server <tag> into
   <state-dir>/server/<tag>/, then start it as a Loom-managed worker (user
   service if possible)?" Options: Install / Pin a version (`--version <tag>`)
   / Cancel.
4. On yes: `sh "$tmp/install.sh" --prefix "$tmp/prefix" [--version <tag>]`.
   The script checks the download against `SHA256SUMS` before it unpacks,
   never uses sudo and never edits a shell profile. Then, still after that
   yes: `mkdir -p "<state-dir>/server/<tag>"` and
   `install -m 755 "$tmp/prefix/bin/loom-server" "<state-dir>/server/<tag>/loom-server"`.
   Do not also install into `~/.local/bin`. Remove `$tmp` when done.
5. Verify the path (not a version string alone):
   `test -x "<state-dir>/server/<tag>/loom-server" && "<state-dir>/server/<tag>/loom-server" --version`.

## 4. Install loom-cli (built from source)
Needs `git`, a Rust toolchain at or above the version Loom requires, and
several minutes of build time.
1. Check `rustc --version` against the `rust-version` in Loom's workspace
   `Cargo.toml` (at the time of writing 1.99). Missing or too old: show how
   to install or update Rust with rustup (`https://rustup.rs`) and wait.
   Don't install it unasked.
2. Match the server: use the tag from step 3.
3. Ask: "Build loom-cli <tag> from source into <prefix>/bin? It downloads
   Rust crates and takes a while." On yes, run it detached so nothing hangs:
   `setsid -f sh -c 'env -u CARGO_TARGET_DIR cargo install --git https://github.com/bearmuckle/loom --tag <tag> --locked --root <prefix> loom-cli > "$HOME/loom-cli-build.log" 2>&1 < /dev/null'`
   then poll the end of the log with short commands (`timeout 10 tail -n 5`)
   until it prints a finish line or an error. Report failures in plain words
   with the last log lines; don't retry in a loop.
4. Verify after the server is up (step 6):
   `<prefix>/bin/loom-cli --token-file <state-dir>/local-worker/token doctor`.
Fallback if `cargo install --git` fails: `git clone` the repo at the tag into
a folder the user names, then `cargo build --release -p loom-cli`, and use
the binary under `target/release/`.

## 5. Start the managed worker
Only if step 1's health check showed nothing on the port. Never restart a
running server. Keep the listener on loopback. A bind beyond loopback needs
TLS or an explicit insecure opt-in in Loom; don't set one without the user's
informed yes (point to Loom's `docs/security.md`).

1. **Token file.** If `<state-dir>/local-worker/token` is missing, say you
   will create a new owner-only token file there (mode 0600) and will not
   show its contents. On yes, create the `local-worker` directory (0700) and
   write a token of the form `loom-` plus a random hex uuid, mode 0600, with
   no print or log of the value (for example a short Python snippet that
   writes the file and prints only `wrote token file`). A path passed as
   `--token-file` is not created by the server if it is missing; the start
   fails instead. If the file already exists, leave it alone and do not read
   it.
2. Ask which supervisor, defaulting to the user service when
   `systemctl --user` works:
   - **User systemd service** (preferred; survives logout, and reboot if
     linger is on). Show the unit and ask before writing
     `~/.config/systemd/user/loom-server.service`. The contents must match
     what Loom's installer writes (`unit_contents` in
     `crates/loom-local/src/local.rs`), with the paths substituted:
     ```
     [Unit]
     Description=Loom worker
     After=network-online.target

     [Service]
     ExecStart="<state-dir>/server/<tag>/loom-server" --bind 127.0.0.1:8765 --token-file "<state-dir>/local-worker/token" --persistence "<state-dir>/state.db"
     Restart=on-failure

     [Install]
     WantedBy=default.target
     ```
     Quote the three paths as above. After a yes: `systemctl --user
     daemon-reload` and `systemctl --user enable --now loom-server`. For
     start at boot without a login, ask separately before `loginctl
     enable-linger "$USER"`. A service doesn't see the shell's environment:
     provider variables go in the unit or in `systemctl --user
     set-environment`, or are set with loom-cli later (onboarding step 4).
     Do not add `--no-auto-update` unless the user opts out.
   - **Detached process now** (does not survive a reboot): after a yes,
     `timeout 20 sh -c 'setsid -f "<state-dir>/server/<tag>/loom-server" --bind 127.0.0.1:8765 --token-file "<state-dir>/local-worker/token" --persistence "<state-dir>/state.db" > "<state-dir>/local-worker/server.log" 2>&1 < /dev/null'`
     and write the pid to `<state-dir>/local-worker/server.pid`. The
     `timeout` and `setsid -f` mean the command returns at once.
3. Save the choice (unit path or log path) and the binary path to memory.

## 6. Health check and token path
- Health: `curl -sS -m 5 -w '\n%{http_code}\n' http://127.0.0.1:8765/health`.
  `200` with body `ok` means up. `503` ("persistence fail-stop") means the
  server stopped saving state: report it, don't restart. Connection refused
  or timeout: not running; after a start, retry a few times with short
  bounded waits (about 30 seconds in total, matching Loom's installer), then
  show the last log lines (`timeout 10 tail -n 20` of the log or
  `journalctl --user -u loom-server -n 20`). Skim for secrets before
  relaying.
- Token file: check existence and mode with `ls -l
  <state-dir>/local-worker/token` only. Never `cat` it.
- Then `loom-cli --token-file <state-dir>/local-worker/token doctor` (and
  `node`) to prove the client and server talk. Exit 6 means a protocol or
  token mismatch; exit 7 means unreachable. See loom-rules "Status words".
- Confirm the running executable is under `<state-dir>/server/<tag>/` (for
  example `systemctl --user show -p ExecStart loom-server` or the pid's
  `/proc/<pid>/exe`). If it is not, say self-update is off and why
  (arbitrary path, dev build, or container).

## 7. Finish
Summarise in a few lines: release tag, managed binary path, token file path
(path only), how the server is supervised, and that self-update is on unless
the user opted out. Save `loom server install: managed <tag>`, the binary
paths, the token-file path, and the workspace id (`loom-cli --token-file
... ls`). Return to loom-onboarding step 2 (or 4 if connection is already
proven).

## Uninstall (only if asked)
Stopping the service or deleting state needs its own yes. Removing the
managed binary under `<state-dir>/server/` is separate from Loom's
`install.sh --uninstall`, which only removes binaries that script installed
into a prefix and leaves worker state alone.
