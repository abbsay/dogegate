# Changelog

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
