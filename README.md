# Loom Pilot: a Grok Bot template for Loom

**Working name:** Loom Pilot. That name is a placeholder and can change.

A chat companion that drives [Loom](#requirements) through `loom-cli`. You talk
to one bot. It starts Loom agent sessions on your machine, watches them, brings
their questions and blocked tool calls to chat as tap-to-answer cards, and
tells you when work finishes.

Version: see `VERSION`. Changes: see `CHANGELOG.md`. License: MIT.

## What it does (v0.1)

- Installs and starts loom-server and loom-cli if you don't have them yet.
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
  yet, the bot installs it for you (asking before every step). loom-cli finds the server
  (`ws://127.0.0.1:8765/ws`) and its token file by default.
- At least one model provider with a credential in Loom.
- Optional: `gh` logged in on that computer, for PR links and merges you approve.
- Optional: the pstack plugin (poteto principles) as a quality bar for sessions.

## Install

1. Create a new Grok Bot.
2. Copy the name and description from `bot/profile.md`.
3. Add every folder under `skills/` as a skill (one `SKILL.md` each).
4. Optionally add the routines in `routines/` as **paused** routines. The
   getting-started skill can create them for you instead.
5. Say hi. The bot runs `getting-started`.

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
