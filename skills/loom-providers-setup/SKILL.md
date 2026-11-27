---
name: loom-providers-setup
description: Guide a new user, one short question at a time, to sign in to GitHub Copilot or store a DeepSeek, OpenCode Go, or OpenCode Zen key in Loom, and optionally a TypeSafe key for the bot. Use during onboarding, when no provider has a credential, or when the user asks to add a provider or key.
---
# Provider setup

Follow **loom-rules**. One short click-widget per question (at most six
options). Multi-select only when several answers can apply. Probe first; do
not ask what `providers list` or memory already answers. Never print, log, or
relay a token or API key. Never pass a key as a command-line argument
(`loom-cli` rejects `--key`). Ask before writing a key or starting a login.

This skill sets up at least one **Loom** provider so sessions can start. Run
it from loom-onboarding step 4, or on its own when the user wants another
provider later.

Checked against Loom `origin/main` (Copilot login landed with the #461
commits): `loom-cli providers login` / `login-status`, and MCP tools
`copilot_login_start` / `copilot_login_status`. `providers set-key` is not an
MCP tool; API keys always go through `loom-cli` on the user's connected
computer. If a flag below is rejected (exit 2), trust `loom-cli <verb> --help`.

## 1. What is already configured
On the chosen computer, with `--token-file` after a managed install:

- `loom-cli providers list` (or MCP `providers_list`)
- `loom-cli models list` (or MCP `models_list`)

Both print no secrets. Built-in Loom ids include `openai`, `deepseek`, and
`github-copilot`. `github-copilot` is not API-key configurable
(`api-key-configurable=false`); sign in with the device flow below, never
`set-key`. OpenCode Go and OpenCode Zen are not seeded; add them as
`openai-compatible` gateways (conventional ids `opencode-go` and
`opencode-zen`).

**TypeSafe is not a Loom provider.** Do not run `providers set-key --provider
typesafe`. TypeSafe's Jev model is used by this bot as a box environment
secret, and optionally by loom-cli's local `jev` helper. It cannot start a
Loom session.

If at least one Loom provider already has `credential=true`, say so in one
line and ask whether to add another or continue to models (step 6). If none
does, continue; the user needs at least one Loom credential before sessions
can start.

## 2. Which provider
One widget: "Which should I set up first?" Default: the first option that
fits what they already mentioned. Then set that one up, and ask "Set up
another?" until they have a Loom credential or they choose to stop.

- **GitHub Copilot** — device login in the browser (no API key)
- **DeepSeek** — API key, built-in provider `deepseek`
- **OpenCode Go** — API key, monthly subscription
- **OpenCode Zen** — API key, pay-as-you-go (needs credits on the workspace)
- **TypeSafe** — API key for this bot (and optional loom-cli `jev`), not Loom
- **Later** — only after you have warned that Loom sessions need a credential

If they already have a key for another OpenAI-compatible endpoint, take the
id, endpoint, and a first model id from them (not the key), then use the same
key-file path as step 4.

## 3. GitHub Copilot (device login)
Needs `configure_providers` (exit 3 without it). The worker URL must be
`wss://` or loopback `ws://` (`127.0.0.1`, `localhost`, `[::1]`).
`--allow-insecure-remote` does not override that; an insecure URL exits 2
before any request. The worker performs the OAuth exchange and stores the
credential. This bot and `loom-cli` never handle, print, or store an access
token.

Prefer MCP when the tools exist (they are mutating, so `loom-mcp` must have
been started with `--allow-mutating-tools`; `--read-only` hides them). Older
`loom-mcp` builds from before those tools landed will not list them.

1. One Go-ahead widget: "I'll start a GitHub Copilot login and show you a
   one-time code to enter on GitHub."
2. Start without waiting, so you can show the code before polling:
   - MCP: `copilot_login_start`. `provider` defaults to `github-copilot` and
     must be that id. `no_wait` defaults to true and must not be false;
     `timeout` is rejected. The tool returns immediately with the public
     payload only (never a token).
   - Fallback, on the connected computer:
     `loom-cli providers login --provider github-copilot --no-wait --json`
     Exit 0 on a successful start.
3. Show the user the verification `url` and `user_code` (a one-time code; it
   is okay to show). Tell them to open the URL, enter the code, and approve.
   Use the `url` the start response returned; do not invent one. Also keep
   `login_id`, `expires_in`, and `interval` for yourself; no need to read
   those out.
4. Poll until `configured`, `failed`, or the start response's `expires_in`
   elapses. Wait at least `interval` seconds between polls (clamp 1–10, as
   the CLI does). Do not busy-loop.
   - MCP: `copilot_login_status` with the `login_id` from the start.
   - Fallback: `loom-cli providers login-status --login-id <id> --json`
   Status words: `pending`, `configured`, `failed`. Exit codes: `0`
   configured, `4` still `pending` (normal while they are in the browser),
   `1` failed (the message is safe to show; it is never a token). Treat exit
   4 as "keep waiting" until `expires_in` has passed, then stop and say the
   code expired. They can start again.
5. Confirm with `providers list` / `models list` (or the MCP list tools).
   `github-copilot` should show `credential=true`. Then go to step 6, or back
   to "Set up another?"

## 4. API keys (DeepSeek, OpenCode Go, OpenCode Zen)
Always on the connected computer with `loom-cli`. Do not ask them to paste a
key into chat.

### Where to get the key
Give only the official page, then the secret form. Do not invent URLs.

| Provider | Loom id | Get a key | Notes |
| --- | --- | --- | --- |
| DeepSeek | `deepseek` (built in) | [platform.deepseek.com/api_keys](https://platform.deepseek.com/api_keys) — create a key on the DeepSeek platform. Docs: [api-docs.deepseek.com](https://api-docs.deepseek.com/) | Then `set-key` only. |
| OpenCode Go | `opencode-go` (add first) | [OpenCode Console](https://opencode.ai/console) — subscribe to Go and copy an API key. Docs: [opencode.ai/docs/console/go](https://opencode.ai/docs/console/go) | Monthly subscription. |
| OpenCode Zen | `opencode-zen` (add first) | Same Console: add credits and copy an API key. Docs: [opencode.ai/docs/zen](https://opencode.ai/docs/zen) (pay-as-you-go Console models; Loom still calls this OpenCode Zen) | Pay-as-you-go; the workspace needs credits. Paid models (or a Go plan) are required from clients other than OpenCode; free-tier chat models are refused. |

OpenCode key docs also mention signing in at [opencode.ai/auth](https://opencode.ai/auth). A Console API key can be used for Go when the workspace has a Go subscription, and for Zen when it has credits.

### Register OpenCode (before the key)
If `providers list` does not already have the id:

```
loom-cli providers add --id opencode-go --kind openai-compatible \
  --endpoint https://opencode.ai/inference/go/openai/v1/chat/completions \
  --model qwen3.8-flash
loom-cli providers add --id opencode-zen --kind openai-compatible \
  --endpoint https://opencode.ai/inference/openai/v1/chat/completions \
  --model kimi-k3
```

`--kind` is only `openai-compatible`. At least one `--model` is required;
`--default-model` must be one of those ids. The example model ids above come
from Loom and OpenCode docs and can go stale: if `add` refuses them, take a
current id from the provider's docs and retry. Built-in ids (`openai`,
`deepseek`, `ollama`, `github-copilot`, `deterministic`) cannot be reused.
After the key is stored, run `loom-cli providers discover --provider <id>` so
the catalog matches the endpoint's `/models`. Loom's `openai-compatible`
kind calls a chat-completions URL; models that only exist on other Console
APIs (Responses, Anthropic, Gemini) may not appear or run through this
registration. A provider with no stored key
is not restored after a worker restart, so add and `set-key` in the same
sitting. The worker sends `User-Agent: loom/<version>` (OpenCode Go refuses a
generic HTTP-library agent).

### Store the key
1. Ask for the key with Grok Bot's masked secret form so it is stored as a
   box environment secret and never appears in chat. Do not ask them to
   paste it into a message. If the form is not available, stop and ask them
   to write the key themselves to a mode-0600 file on the Loom computer
   (path below); you never open that file, you only pass its path to
   `set-key`.
2. Write the secret to a file mode `0600` on the Loom computer, recommended
   path `~/.config/loom/<provider-id>.key` (create `~/.config/loom` if
   needed). Do not put the value on a command line, in chat, or in a tool
   result you would display. Do not `echo`, `printf`, or `cat` the key. A
   safe pattern is: write a mode-0600 file from the box secret on the bot
   side, copy that file to the path on the Loom computer, `chmod 600` it,
   then call `set-key`. Do not export the value into a remote shell command
   (`VAR=secret loom-cli …`).
3. Store it in Loom:

   ```
   loom-cli providers set-key --provider <id> --key-file ~/.config/loom/<id>.key
   ```

   `--from-env <NAME>` is also valid, but only when that variable is already
   set on the Loom computer. Prefer `--key-file`. The command prints
   `provider <id> credential set` and nothing else. A missing, unreadable,
   or blank file exits 2 and sends no request. An unknown provider id exits
   5. `set-key` for `github-copilot` is refused.
4. Do not open the key file afterwards. Confirm with `providers list`
   (`credential=true`) and `models list`. For a newly added OpenCode
   provider, run `providers discover` after `set-key`. Optionally prove
   inference with a one-word test session (loom-sessions model card first).
5. Ask plan type (subscription / prepaid / unknown) and preference (use
   first / normal / only when needed / don't use). Save them.

## 5. TypeSafe (bot secret, not Loom)
TypeSafe is not in `providers list` and cannot be added as a session model.
This bot stores the key as a box environment secret (recommended name
`TYPESAFE_API_KEY`) and uses it directly. Official pages: create a key at
[console.typesafe.ai/keys](https://console.typesafe.ai/keys) (sign in or
create an account). Docs: [docs.typesafe.ai/introduction/quickstart](https://docs.typesafe.ai/introduction/quickstart).

1. Ask for the key with the masked secret form, never in chat.
2. Save it as a box environment secret. Optionally also write
   `~/.config/loom/typesafe.key` mode `0600` on the Loom computer if they
   want loom-cli's local `jev` helper (`jev status` / `jev ask`; that helper
   talks to TypeSafe itself, not through the worker). `jev status` reports
   whether a key file is present and never prints the key.
3. Say clearly that this does not set up a Loom provider; they still need
   Copilot, DeepSeek, OpenCode, or another credentialed Loom provider to
   start sessions.

## 6. Models per kind of work
From `models list` only (text: `provider=<id> model=<id> default=<true|false>`;
`--json` also has `display_name`, `context_window`, `max_output_tokens`).
Offer no defaults of your own. One widget at a time for:

- main session
- delegated children: one model for all, or per kind. Conventional kinds
  (free-form in Loom, max 32 characters): `cheap`, `coding`, `review`,
  `vision`, `long-reasoning`

Offer **Decide each time** as an option. Save the mapping.

**Training on request data.** `models list` has no training field. Before
you propose a model, check that provider's current published policy and say
so clearly when a model trains on request data (or when you could not
confirm). Do not guess. OpenCode documents some SKUs that train (for
example contributor or "free in exchange for training" models) and, for Go,
workspaces block paid endpoints that train unless an owner turns that on in
Settings → Privacy. Re-check the provider's docs; the list changes.

## Finish
Save which Loom providers have a credential, TypeSafe if configured (box
secret name only, never the value), plan type and preference, and the model
mapping. Return to loom-onboarding (step 5 is already done if you just ran
it), or confirm the change in one line when this skill was run on its own.
