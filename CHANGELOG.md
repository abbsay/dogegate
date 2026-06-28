# Changelog

## 0.3.3 - 2026-06-28

- Add `repair-cowork` for Claude Code's macOS Cowork desktop app.
- Repair the current CC Switch `claude-desktop` Antigravity provider by mapping
  Cowork aliases to Claude-family upstream models instead of Gemini routes that
  can fail desktop beta/tool requests with `INVALID_ARGUMENT`.
- Rebuild the Claude-3p gateway profile on
  `http://127.0.0.1:15721/claude-desktop` while preserving the gateway token.
- If the local route is stopped, start CC Switch's proxy route with a temporary
  Claude provider, then immediately restore `~/.claude/settings.json` and clear
  the temporary takeover state so Claude Code CLI remains direct.

## 0.3.2 - 2026-06-28

- Stop writing Antigravity route fields into CC Switch's global
  `settings.common_config_codex`, because common config is also merged into
  `OpenAI Official` and can make official OAuth sessions hit the local proxy.
- Repair existing Codex common config by removing provider/model/model catalog/
  base URL/token fields while keeping non-routing shared TOML such as plugins,
  MCP servers, and project trust settings.
- Store the Antigravity Pool API key in the Codex provider auth payload and keep
  the provider config as the real upstream route.
- Write `currentProviderCodex` as the CC Switch provider id instead of the
  display name.
- Expand `doctor` output for Codex provider key, current-provider, and common
  config contamination checks, plus API-key environment variables that can
  override official OAuth.

## 0.3.1 - 2026-06-28

- Enable CC Switch Codex official auth preservation by default during install.
- Keep Codex official OAuth login in `~/.codex/auth.json` while routing
  third-party model traffic through CC Switch.
- Write the proxy placeholder token to Codex `config.toml` as
  `experimental_bearer_token = "PROXY_MANAGED"` instead of relying on
  `auth.json` for the proxied third-party route.
- Preserve the CC Switch live backup as the real Antigravity upstream config
  with a provider-scoped token so disabling takeover can restore a working
  third-party setup without replacing official OAuth auth.
- Add `--no-preserve-codex-auth` for users who intentionally want the older
  CC Switch compatibility behavior.

## 0.3.0 - 2026-06-23

- Make Antigravity Tools the source of truth for pool metadata by default.
- Auto-read `~/.antigravity_tools/gui_config.json` for proxy port and local API
  key during install.
- Auto-query Antigravity Tools `/v1/models` and merge the live model catalog
  with Dogegate's known working agent aliases.
- Ensure the CC Switch Codex proxy is enabled on `127.0.0.1:15721` during
  install.
- Stop CC Switch before writing its database when `--restart-ccswitch` is used,
  preventing the app from flushing stale in-memory provider state over the new
  config on exit.
- Add `--no-auto-antigravity` to preserve the previous fixed-catalog behavior.

## 0.2.7 - 2026-06-23

- Add `inspect-antigravity` to summarize the local Antigravity Tools gateway,
  proxy security flags, Cloudflared mode, and live `/v1/models` catalog without
  printing secrets.
- Document Antigravity Tools as Dogegate's primary local model gateway and
  Antigravity Tools LS as the standby native-language-server route.

## 0.2.6 - 2026-06-23

- Add `open-openclaw-dashboard` to recover from the OpenClaw macOS Dashboard
  startup race where the window opens before the gateway sidecars are ready.
- Document the Dashboard unavailable workaround without changing model routing
  or auth.

## 0.2.5 - 2026-06-23

- Add `--agent-models` for the OpenClaw agent-safe model allowlist.
- Keep full pool catalog exposure separate from OpenClaw session model safety.
- Clear stale OpenClaw session overrides that point to pool models known to fail
  agent-shaped requests.
- Avoid writing CC Switch OAuth access tokens into OpenClaw/Hermes API-key
  slots; preserve existing tool auth when CC Switch has no `OPENAI_API_KEY`.

## 0.2.4 - 2026-06-23

- Add `--pool-models` and a default Antigravity Pool model list.
- Configure OpenClaw and Hermes with multiple pool models while keeping
  `claude-sonnet-4-6` as the default.
- Populate CC Switch provider model catalog from the same pool model list.
- Preserve session overrides when they point to an allowed pool model.

## 0.2.3 - 2026-06-23

- Add OpenClaw `agents.defaults.models` allowlist for the configured pool model.
- Keep OpenClaw agent entries constrained to the same pool model.
- Reduce the chance that Dashboard model changes select unsupported built-in catalog models.

## 0.2.2 - 2026-06-23

- Clear stale OpenClaw per-session model/auth overrides during `install-openclaw`.
- Prevent old Dashboard sessions pinned to unsupported models from bypassing the pool model.
- Back up OpenClaw `sessions.json` before repairing session overrides.

## 0.2.1 - 2026-06-23

- Stop installing `gemini-3.1-pro-high` as a stable OpenClaw/Hermes agent model.
- Document Gemini safety-settings schema failures for agent/tool-shaped requests.
- Keep `claude-sonnet-4-6` as the stable OpenClaw/Hermes pool model.
- Clarify that simple smoke tests are not full agent adaptation tests.

## 0.2.0 - 2026-06-23

- Add `install-openclaw` to route OpenClaw through Antigravity Pool.
- Add `install-hermes` to route Hermes Agent through Antigravity Pool.
- Add `verify-agents` for OpenClaw and Hermes smoke tests.
- Document OpenClaw and Hermes configuration surfaces.
- Cap Hermes `model.max_tokens` at 4096 for Antigravity Pool compatibility.

## 0.1.1 - 2026-06-23

- Document the full Dogegate system map and v0.1.0 scope.
- Add architecture diagrams for the current Codex path and broader multi-client
  target.

## 0.1.0 - 2026-06-23

- Add installer for Codex + CC Switch + Antigravity Pool routing.
- Patch CC Switch provider config, Codex common config, live backup, and live
  Codex config.
- Add doctor and verify commands.
- Add backups before mutation.
