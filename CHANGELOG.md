# Changelog

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
