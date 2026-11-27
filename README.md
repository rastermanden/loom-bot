# Loom Pilot: a Grok Bot template for Loom

**Working name:** Loom Pilot. That name is a placeholder and can change.

A chat companion that drives [Loom](#requirements) from chat. It finds Loom
on your computer when it can, falls back to a private Tailscale address if
you have set that up, and otherwise uses `loom-cli`. It starts agent
sessions, watches them, brings their questions and approvals to you as
cards, and tells you when work finishes.

Version: see `VERSION`. Changes: see `CHANGELOG.md`. License: MIT.

## Quick start

1. Create a Grok Bot, paste the name and description from `bot/profile.md`,
   and add every folder under `skills/` as a skill.
2. Connect the computer that should run Loom to Grok Bot, then say hi in
   the new bot's chat. It looks for a working Loom by itself and only asks
   when it cannot decide. If Loom is not installed yet, it offers one
   confirmation and, if it cannot run the steps for you, one copy-paste
   command.
3. When it says "Loom is ready", ask it to start a session or just say
   **status**.

## What it does

- Installs and starts a Loom-managed loom-server (self-updating, under the
  state directory) and loom-cli if you don't have them yet.
- Gets you set up one question at a time: where loom-server runs, providers,
  model per kind of work, isolation, projects, merge policy and autonomy.
- Starts sessions inside the right project after one model-confirmation card.
- Watches sessions and stays silent unless one finishes, fails, gets stuck, or
  waits on you.
- Sends approvals and agent questions to chat as cards. You decide.
- Reports status on request and archives finished sessions only after asking.
- Optional AFK mode (off by default) for fewer questions while you are away.
- Checks weekly for a new template version and tells you what changed.

Default stance: **monitor only**. The bot never merges, archives, deletes,
restarts a server or approves out-of-scope calls without your explicit pick.

## Requirements

- A computer that runs, or will run, loom-server and loom-cli, registered with
  Grok Bot so the bot can run shell commands there. If Loom is not installed
  yet, the bot installs a Loom-managed worker (the layout that self-updates),
  asking before every step. The default worker URL is
  `ws://127.0.0.1:8765/ws`. After a managed install, pass the token file path
  the onboarding skill records (never print the token).
- At least one model provider with a credential in Loom.
- Optional: `gh` logged in on that computer, for PR links and merges you approve.
- Optional: the pstack plugin (poteto principles) as a quality bar for sessions.
- Optional: `loom-mcp` on your computer (loopback), and Tailscale Serve
  (HTTPS, your tailnet only, never Funnel) as a fallback when that computer
  is offline. The bot cannot reach a private LAN address from the cloud; it
  uses your connected computer, or that private HTTPS URL. See
  loom-mcp-install.

## Install

Follow [Quick start](#quick-start). Optionally add the routines in
`routines/` as **paused** routines (getting-started can create them).

## Update

The bot compares its installed version (saved in memory) with `VERSION` on the
default branch of this repo once a week, if you turn that routine on, or when
you say "check for updates". It then shows you the new `CHANGELOG.md` entries.

- **Notify** (default): it tells you what is new and waits for your yes.
- **Auto**: it installs and tells you afterwards. A skill you edited yourself is
  never replaced without asking.
- **Off**: only when you ask.

Template skills in a bot's library can be read-only copies. If the bot can't
replace them, it says so and you re-import the new version manually.

## Layout

```
VERSION, CHANGELOG.md, LICENSE, README.md
bot/profile.md              name, description, memory keys
skills/getting-started      first conversation
skills/loom-onboarding      setup questions
skills/loom-server-install  install and start loom-server and loom-cli
skills/loom-mcp-install     loopback loom-mcp, optional Tailscale fallback
skills/loom-sessions        start, watch, approve, status, archive
skills/loom-rules           shared rules every skill follows
skills/loom-cli-reference   loom-cli verbs and flags (versioned)
skills/afk                  opt-in away mode
skills/template-update      weekly version check
routines/                   optional routine prompts (paused)
```

## Credits

Ideas adapted from T3 Tots (Grok Bot companion for T3 Code, MIT), firstmate,
t3dotgg/tstack (MIT) and pstack by Lauren Tan (poteto).
