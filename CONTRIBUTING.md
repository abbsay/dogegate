# Contributing

## Local Checks

Run the non-mutating checks first:

```bash
./bin/codex-ccswitch-antigravity doctor
./bin/codex-ccswitch-antigravity install --dry-run
```

Test against copies of real files:

```bash
tmpdir="$(mktemp -d)"
cp ~/.cc-switch/cc-switch.db "$tmpdir/cc-switch.db"
cp ~/.codex/config.toml "$tmpdir/config.toml"
cp ~/.cc-switch/settings.json "$tmpdir/settings.json"

./bin/codex-ccswitch-antigravity install \
  --ccswitch-db "$tmpdir/cc-switch.db" \
  --codex-config "$tmpdir/config.toml" \
  --settings-json "$tmpdir/settings.json" \
  --backup-dir "$tmpdir/backups"

./bin/codex-ccswitch-antigravity doctor \
  --ccswitch-db "$tmpdir/cc-switch.db" \
  --codex-config "$tmpdir/config.toml"
```

## Compatibility Notes

When adding support for a new CC Switch or Codex version, record:

- Codex version
- CC Switch version
- provider name
- proxy URL
- upstream URL
- whether `proxy_live_backup` exists
- expected `codex exec` header

