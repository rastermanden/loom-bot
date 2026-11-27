---
name: template-update
description: Check this template's GitHub repo for a newer version, tell the user what is new, and install it per their update mode. Use for "check for updates", "update yourself", and the weekly template-update routine.
---
# Template update

Follow **loom-rules**. The template repo is `rastermanden/loom-bot`. Settings in memory: `template version`,
`template update mode: notify|auto|off`.

## Check
1. Read `VERSION` and `CHANGELOG.md` from the repo's default branch (raw file
   over HTTPS, or `gh api repos/rastermanden/loom-bot/contents/VERSION`). Read only;
   no token is needed for a public repo.
2. Compare with the installed version (semver). Not newer: stop. In a routine
   run, send nothing.
3. Newer: collect every CHANGELOG entry above the installed version.

## Tell
One short message: "Loom Pilot <new> is out (you have <old>)." Then the
changes in at most five plain bullets, from the CHANGELOG. Call out anything
that changes behaviour or safety rules.
- **notify**: end with a card: **Update** / **Not now**.
- **auto**: install (below), then send the same message as "updated to <new>".
- **off**: only when the user asked.

## Install
1. Download the release's `skills/` and `routines/` files.
2. For each skill: if the bot's copy differs from the old release's copy (the
   user edited it), show the user and ask before replacing. Otherwise replace.
3. If skills can't be replaced (read-only template copies), say so in one line
   and tell the user to re-import the new version; keep the old version
   recorded.
4. Routines: update the prompt of routines the user has on; never create new
   ones without asking.
5. Save `template version: <new>`.
Never run scripts from the repo. Never change memory beyond the version line
and settings the CHANGELOG says moved.
