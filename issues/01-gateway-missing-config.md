# Issue #1 — Gateway exits: "Missing config"

**Status:** Blocking — service crashes on every start
**Priority:** High
**Labels:** bug, setup-required

## Symptom

```
2026-02-13T15:25:57 Missing config. Run `openclaw setup` or set gateway.mode=local (or pass --allow-unconfigured).
systemd: openclaw-gateway.service: Main process exited, code=exited, status=1/FAILURE
```

The service restarts every ~6 seconds and immediately dies.

## Root cause

`openclaw gateway` requires a config file at `~/.openclaw/openclaw.json` (or
`$OPENCLAW_STATE_DIR/openclaw.json`) to be initialised before it will start.
The config is normally created by running `openclaw setup` or `openclaw onboard`
interactively. The NixOS service runs as system user `openclaw` with home
`/var/lib/openclaw`, so the expected path is:

```
/var/lib/openclaw/.openclaw/openclaw.json
```

## Options

### A — Run `openclaw setup` as the openclaw user (recommended for now)

```bash
sudo -u openclaw /nix/store/1lj2jradag5b00f9pj9k7jqxzpg777np-openclaw-2.0.0-beta5/bin/openclaw setup
```

This creates the config file interactively. Then restart the service:

```bash
sudo systemctl restart openclaw-gateway
```

### B — Pass `--allow-unconfigured` flag

Modify `ExecStart` in `nixos/modules/openclaw.nix`:

```nix
ExecStart = lib.mkForce "${openclaw-pkg}/bin/openclaw gateway --port 18789 --allow-unconfigured";
```

This starts the gateway without a config. Model/API key must then be set
via environment variables (`OPENROUTER_API_KEY`, `OPENCLAW_MODEL_PROVIDER`).

### C — Pre-generate the config via a NixOS activation script

Write `openclaw.json` to `/var/lib/openclaw/.openclaw/` as part of
`system.activationScripts` or via `systemd.tmpfiles.rules`. Requires
knowing the exact JSON schema — check `openclaw config get --json` after
Option A succeeds once.

## Related files

- `infra/nixos/modules/openclaw.nix` — service definition
- `/var/lib/openclaw/` — openclaw state dir (service home)
- `/var/lib/openclaw/secrets/openrouter-key` — API key on disk (0400 openclaw)
