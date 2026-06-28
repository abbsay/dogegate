# codex-ccswitch-antigravity

Make local coding agents route through a CC Switch / Antigravity Pool setup.

For the full system map, see [ARCHITECTURE.md](ARCHITECTURE.md).

This project packages the setup that was proven locally:

```text
Codex Desktop / CLI
  -> CC Switch local proxy: http://127.0.0.1:15721/v1
  -> Antigravity Pool gateway: http://127.0.0.1:8045/v1
  -> model: claude-sonnet-4-6
```

The main problem it solves is that recent Codex builds no longer route custom
providers from top-level `base_url` / `wire_api` fields alone. Codex needs an
explicit `model_provider = "custom"` plus a `[model_providers.custom]` block.
Some CC Switch live takeover flows can still write the older shape, causing
Codex Desktop to fall back to the official OpenAI provider.

## Status

Experimental but usable. The local setup has been verified with:

- Codex `0.136.0`
- CC Switch local proxy on `127.0.0.1:15721`
- Antigravity Pool upstream on `127.0.0.1:8045`
- OpenAI Responses-compatible `/v1/responses`

Current v0.2.x automation covers Codex CLI, Codex Desktop, OpenClaw, and Hermes
Agent. The broader Dogegate stack is intended to cover Claude Code, Claude
Cowork, Codex CLI, Codex Desktop, OpenClaw, and Hermes through CC Switch and
Antigravity Tools, with Antigravity Tools LS as a standby upstream.

Stable agent model:

```text
claude-sonnet-4-6
```

Known limitation: `gemini-3.1-pro-high` can answer simple direct
`/v1/chat/completions` requests through the pool, but agent/tool requests from
OpenClaw and Hermes currently fail against the upstream Gemini safety-settings
schema. Dogegate therefore does not install Gemini as a stable OpenClaw/Hermes
agent model.

## Antigravity Tools

Dogegate treats Antigravity Tools as the main local model gateway. On the
verified macOS setup, Antigravity Tools v4.2.6 exposes:

```text
Admin UI:  http://127.0.0.1:8045
API base:  http://127.0.0.1:8045/v1
Protocol:  OpenAI-compatible chat completions and model listing
```

Antigravity Tools combines account management, request scheduling, quota
tracking, protocol adaptation, local API-key auth, LAN access control, and an
optional Cloudflared public-access mode. Its local config is stored under:

```text
~/.antigravity_tools/
```

Useful local files:

```text
~/.antigravity_tools/gui_config.json      proxy, auth, Cloudflared, scheduling
~/.antigravity_tools/accounts.json        account registry
~/.antigravity_tools/token_stats.db       usage accounting
~/.antigravity_tools/security.db          allow/deny lists and IP access logs
~/.antigravity_tools/user_tokens.db       user token metadata
~/.antigravity_tools/proxy_logs.db        request logs when enabled
```

The live model catalog can be larger than Dogegate's curated agent-safe list.
Use:

```bash
./bin/codex-ccswitch-antigravity inspect-antigravity
```

This only reads local config and `/v1/models`; it does not print API keys or
account credentials. Dogegate uses the same Antigravity Tools metadata during
install by default:

- reads the local proxy port and API key from `gui_config.json`
- queries `/v1/models` for the current model catalog
- merges that live catalog with Dogegate's known working agent aliases
- writes the merged catalog to CC Switch, OpenClaw, and Hermes
- keeps OpenClaw on a narrower agent-safe allowlist

Disable this behavior with `--no-auto-antigravity`, or override the catalog
manually with `--pool-models model-a,model-b`.

In the current local environment, Antigravity Tools returns 69 models. After
merging Dogegate's known aliases, the install catalog contains 72 models,
including Claude, Gemini, GPT-compatible, image, and thinking variants. OpenClaw
still uses a narrower agent-safe allowlist because agent/tool-shaped requests
are stricter than simple chat requests.

Antigravity Tools LS is a standby path with a different architecture: it bridges
through the native Antigravity language-server process and exposes standard
OpenAI, Anthropic, and Gemini-style APIs. It is useful as a fallback or future
deep-compatibility route, but Dogegate currently targets Antigravity Tools on
port 8045 as the primary pool.

## Install

Clone or copy this repository, then run:

```bash
chmod +x ./bin/codex-ccswitch-antigravity
./bin/codex-ccswitch-antigravity install --restart-ccswitch --run-verify
```

Defaults:

```text
provider name: Antigravity-Pool
model:         claude-sonnet-4-6
Codex proxy:   http://127.0.0.1:15721/v1
upstream:      http://127.0.0.1:8045/v1
CC Switch DB:  ~/.cc-switch/cc-switch.db
Codex config:  ~/.codex/config.toml
```

By default the installer auto-discovers the Antigravity Tools local proxy
metadata. To keep using only Dogegate's built-in fallback catalog:

```bash
./bin/codex-ccswitch-antigravity install --no-auto-antigravity
```

Custom example:

```bash
./bin/codex-ccswitch-antigravity install \
  --provider-name Antigravity-Pool \
  --model claude-sonnet-4-6 \
  --proxy-url http://127.0.0.1:15721/v1 \
  --upstream-url http://127.0.0.1:8045/v1 \
  --restart-ccswitch \
  --run-verify
```

Install OpenClaw and Hermes against the same pool:

```bash
./bin/codex-ccswitch-antigravity install-openclaw
./bin/codex-ccswitch-antigravity install-hermes
./bin/codex-ccswitch-antigravity verify-agents
```

Inspect the local Antigravity Tools gateway:

```bash
./bin/codex-ccswitch-antigravity inspect-antigravity
```

## What It Changes

The installer backs up files first, then patches:

- `~/.cc-switch/cc-switch.db`
  - `providers.settings_config` for the selected Codex provider
  - sanitizes `settings.common_config_codex` when older Dogegate installs left
    provider/model/model catalog/base URL/token fields there
  - `proxy_live_backup.original_config` when present
- `~/.cc-switch/settings.json`
  - sets `currentProviderCodex` to the provider id
  - enables `preserveCodexOfficialAuthOnSwitch` so Codex official OAuth login
    remains in `~/.codex/auth.json` while third-party traffic uses CC Switch
- `~/.codex/config.toml`
  - writes the current live Codex provider block unless `--no-live-config` is set
  - uses `experimental_bearer_token = "PROXY_MANAGED"` for the local proxy route
    instead of overwriting official OAuth login state
- `~/.openclaw/openclaw.json`
  - points OpenClaw's OpenAI provider at Antigravity Pool
  - sets the default model to `openai/claude-sonnet-4-6`
  - adds an OpenClaw model allowlist for the configured pool model list
- `~/.openclaw/agents/main/agent/openclaw-agent.sqlite`
  - stores the Antigravity Pool API key in OpenClaw's local auth profile
- `~/.openclaw/agents/main/sessions/sessions.json`
  - clears stale per-session model overrides that point away from the configured pool model
- `~/.hermes/config.yaml`
  - sets Hermes to `provider: custom`
  - points `model.base_url` at Antigravity Pool
  - sets `model.max_tokens: 4096` for pool compatibility

Default pool model list:

```text
claude-sonnet-4-6
gemini-3.1-pro-high
gemini-3.1-pro-low
gemini-3-flash-agent
gemini-3-flash
gemini-3.5-flash-low
gemini-3.1-flash-image
gpt-oss-120b-medium
claude-3-5-sonnet-20241022
```

Override it with `--pool-models model-a,model-b`. `claude-sonnet-4-6`,
`gemini-3-flash-agent`, `gemini-3-flash`, `gemini-3.5-flash-low`, and
`claude-3-5-sonnet-20241022` have passed OpenClaw agent smoke tests. Some
Gemini Pro/GPT-OSS routes may answer simple API calls but still reject
agent-shaped requests with upstream schema errors.

OpenClaw uses a narrower default agent allowlist:

```text
claude-sonnet-4-6
gemini-3-flash-agent
gemini-3-flash
gemini-3.5-flash-low
claude-3-5-sonnet-20241022
```

Override it with `--agent-models model-a,model-b`. Dogegate clears stale
OpenClaw session overrides when they point outside this agent-safe list.

The two URLs intentionally differ:

```text
~/.codex/config.toml        -> base_url = "http://127.0.0.1:15721/v1"
CC Switch provider upstream -> base_url = "http://127.0.0.1:8045/v1"
```

If both are `15721`, CC Switch forwards to itself and Codex will reconnect in a
loop. If Codex only has top-level `base_url`, it can still use `provider: openai`
and hit the official API.

Dogegate follows CC Switch v3.16.1+'s Codex official auth preservation model:

```text
~/.codex/auth.json          -> official ChatGPT / Codex OAuth login cache
~/.codex/config.toml        -> model_provider, proxy base_url, PROXY_MANAGED token
CC Switch provider settings -> real Antigravity upstream URL and API key
```

That means Codex Desktop can continue to see the official account for app
features, while model traffic is routed to the selected third-party provider.
Use `--no-preserve-codex-auth` only if you intentionally want CC Switch's older
compatibility behavior where third-party switching may rewrite `auth.json`.

`settings.common_config_codex` must not contain the Antigravity route. CC Switch
merges common config into every Codex provider that opts into common config,
including `OpenAI Official`. Dogegate therefore keeps the Antigravity route in
the `Antigravity-Pool` provider itself and only leaves non-routing shared TOML
in common config, such as plugin, MCP, desktop, or project trust settings.

## Doctor

Inspect the current setup without changing anything:

```bash
./bin/codex-ccswitch-antigravity doctor
```

Useful healthy signs:

```text
env_CODEX_API_KEY_present=False
env_OPENAI_API_KEY_present=False
preserve_codex_official_auth_on_switch=True
provider_found=True
provider_has_api_key=True
provider_config_has_custom=True
provider_config_has_experimental_bearer_token=False
common_config_has_custom=False
common_config_has_proxy_url=False
common_config_has_model_provider=False
provider_config: base_url = "http://127.0.0.1:8045/v1"
```

When the third-party proxy route is currently selected, the live Codex config
should also show:

```text
codex_config_has_custom=True
codex_config_has_proxy_placeholder=True
codex_config: model_provider = "custom"
codex_config: base_url = "http://127.0.0.1:15721/v1"
```

## Verify

Run:

```bash
./bin/codex-ccswitch-antigravity verify
```

Expected Codex header:

```text
model: claude-sonnet-4-6
provider: custom
```

Expected answer:

```text
pong
```

For OpenClaw and Hermes:

```bash
./bin/codex-ccswitch-antigravity verify-agents
```

Expected outputs include two `pong` responses:

- one from an OpenClaw `agent --local` turn
- one from a Hermes `chat -q` turn with the `hermes-cli` toolset enabled

## OpenClaw Dashboard

OpenClaw for macOS can open its Dashboard window before the local gateway has
finished restarting its sidecars. In that case the app can show:

```text
Dashboard unavailable
Could not connect to the server.
```

If the gateway health check is otherwise OK, use:

```bash
./bin/codex-ccswitch-antigravity open-openclaw-dashboard
```

This command quits the stale OpenClaw app window, restarts the gateway, waits
until `openclaw gateway call health` succeeds, then opens the Dashboard through
OpenClaw's CLI. It does not rewrite model routing or auth.

## Rollback

Backups are written to:

```text
~/.cc-switch/backups/codex-antigravity/
```

Restore manually:

```bash
cp ~/.cc-switch/backups/codex-antigravity/cc-switch.db.YYYYMMDD-HHMMSS.bak ~/.cc-switch/cc-switch.db
cp ~/.cc-switch/backups/codex-antigravity/config.toml.YYYYMMDD-HHMMSS.bak ~/.codex/config.toml
cp ~/.cc-switch/backups/codex-antigravity/settings.json.YYYYMMDD-HHMMSS.bak ~/.cc-switch/settings.json
open -a "CC Switch"
```

## Release Plan

- `0.1.x`: support the proven Codex Antigravity-Pool setup.
- `0.2.x`: add OpenClaw and Hermes Agent support.
- `0.3.x`: add safer TOML/YAML parsing, tests, and multi-provider profiles.
- `1.0.0`: stable installer, rollback command, and documented compatibility matrix.

## Notes

This tool preserves auth payloads stored by CC Switch but never prints them. It
only rewrites provider routing config.
