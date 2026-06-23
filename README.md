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

## What It Changes

The installer backs up files first, then patches:

- `~/.cc-switch/cc-switch.db`
  - `providers.settings_config` for the selected Codex provider
  - `settings.common_config_codex`
  - `proxy_live_backup.original_config` when present
- `~/.cc-switch/settings.json`
  - sets `currentProviderCodex`
- `~/.codex/config.toml`
  - writes the current live Codex provider block unless `--no-live-config` is set
- `~/.openclaw/openclaw.json`
  - points OpenClaw's OpenAI provider at Antigravity Pool
  - sets the default model to `openai/claude-sonnet-4-6`
  - adds an OpenClaw model allowlist for the configured pool model
- `~/.openclaw/agents/main/agent/openclaw-agent.sqlite`
  - stores the Antigravity Pool API key in OpenClaw's local auth profile
- `~/.openclaw/agents/main/sessions/sessions.json`
  - clears stale per-session model overrides that point away from the configured pool model
- `~/.hermes/config.yaml`
  - sets Hermes to `provider: custom`
  - points `model.base_url` at Antigravity Pool
  - sets `model.max_tokens: 4096` for pool compatibility

The two URLs intentionally differ:

```text
~/.codex/config.toml        -> base_url = "http://127.0.0.1:15721/v1"
CC Switch provider upstream -> base_url = "http://127.0.0.1:8045/v1"
```

If both are `15721`, CC Switch forwards to itself and Codex will reconnect in a
loop. If Codex only has top-level `base_url`, it can still use `provider: openai`
and hit the official API.

## Doctor

Inspect the current setup without changing anything:

```bash
./bin/codex-ccswitch-antigravity doctor
```

Useful healthy signs:

```text
provider_found=True
provider_config_has_custom=True
codex_config_has_custom=True
codex_config: model_provider = "custom"
codex_config: base_url = "http://127.0.0.1:15721/v1"
provider_config: base_url = "http://127.0.0.1:8045/v1"
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
