# Routine: template-update check (optional, paused)

- Schedule: weekly, Sunday 10:00 in the user's time zone (`0 10 * * 0`), flexible.
- Created paused. Turned on in getting-started step 3 when the update mode is
  notify or auto.

Prompt:

> Check whether a newer Loom Pilot template version is out by following the
> template-update skill and loom-rules. Install only if the saved update mode
> is "auto"; otherwise just report. Send the result to the user in this chat.
> If the installed version is already the latest, send nothing.
