# Routine: session watch (optional, paused)

- Schedule: every 10 minutes (`*/10 * * * *`), flexible. Only useful while
  sessions run; the bot pauses it again when nothing is live.
- Created paused. Turned on in getting-started step 4, or by the bot when the
  user asks it to watch sessions.

Prompt:

> Check the Loom sessions this bot started or was asked to watch, following
> the loom-sessions skill section 3 and loom-rules. Approve, answer, merge,
> archive or stop nothing yourself, except stopping a run that does something
> dangerous as loom-sessions describes. Report to the user in this chat only when
> a session finished, failed, got stuck, or waits on an approval or question
> (then forward it as a card). If nothing changed, send nothing. If no watched
> session is still live, say so once and pause this routine.
