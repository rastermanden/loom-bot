---
name: getting-started
description: First conversation with a new owner of the Loom Pilot bot. Use on first contact, or when memory has no Loom setup.
---
# Getting started

One line first, in the user's language: "I'm your chat companion for Loom: I
start and watch Loom agent sessions for you and bring their questions to this
chat."

1. Save `template version: <VERSION>` to profile memory.
2. Run **loom-onboarding** from the top. It skips anything memory already knows.
3. **Updates:** ask on a card, nothing pre-selected: "How should I handle my own
   updates?" **Notify me** (recommended) / **Auto update** / **Off**. Save
   `template update mode: notify|auto|off`.
4. **Routines:** offer the routines in `routines/` in one multi-select card,
   nothing pre-selected, each labelled with its time in the user's time zone,
   plus **None**. Create each picked one from its prompt. Create the
   template-update routine paused when the mode is Off. Save `routines on:
   <names>`. Never create a second copy of a routine.
5. **Finish:** one short summary (machine, providers, default models, autonomy,
   routines on), then run status once and list things to try:
   - status
   - new session in <project>: <task>
   - what's waiting on me
   - watch <session>
   - afk / back (only if AFK was switched on in onboarding)
