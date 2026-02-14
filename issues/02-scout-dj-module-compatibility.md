# Issue #2 — Scout-DJ module incompatible with openclaw v2.x CLI

**Status:** Partially resolved — two overrides applied, more may be needed
**Priority:** Medium
**Labels:** infrastructure, tech-debt

## Background

`infra/flake.nix` uses two flakes:
- `github:Scout-DJ/openclaw-nix` — provides the NixOS systemd/Caddy/hardening module (`services.openclaw.*`)
- `github:openclaw/nix-openclaw` — provides the actual `openclaw` binary via overlay (`pkgs.openclaw`)

The Scout-DJ module was written for an older/different openclaw CLI and has
several mismatches with the actual `openclaw` v2.0.0-beta5 binary.

## Resolved mismatches

| Problem | Fix applied |
|---------|------------|
| `ExecStart` used `openclaw gateway start --config <json>` — `start` and `--config` don't exist | Overridden with `lib.mkForce` to `openclaw gateway --port 18789` in `openclaw.nix` |
| `RestrictAddressFamilies` missing `AF_NETLINK` — libuv `getifaddrs()` needs it | Added `AF_NETLINK` via `lib.mkForce` override in `openclaw.nix` |

## Remaining unknowns

1. **Config injection** — Scout-DJ module generates a `openclaw-gateway.json` and
   passes it via `--config`. Since we dropped `--config`, the `extraGatewayConfig`
   Nix options (`toolSecurity`, `toolAllowlist`, `modelProvider`, etc.) no longer
   have any effect. Configuration must be done through the binary's own config
   file or env vars.

2. **Model API key** — Scout-DJ module sets `OPENCLAW_MODEL_PROVIDER=openrouter`
   but never loads the actual API key from `modelApiKeyFile` into an env var.
   The env var the real binary expects is unknown (`OPENROUTER_API_KEY`?).
   Needs verification after Issue #1 is resolved.

3. **Caddy integration** — The Scout-DJ module sets up Caddy for `openclaw.local`.
   The real binary may handle TLS itself or expect a different proxy config.
   Verify after gateway is running.

## Long-term fix

Consider replacing Scout-DJ module entirely with a minimal hand-written NixOS
module that wraps the official binary correctly. The hardening (memory limits,
RestrictAddressFamilies, etc.) is already partially in `openclaw.nix`. A clean
module would be ~50 lines and avoid all compatibility shims.

## Related files

- `infra/nixos/modules/openclaw.nix`
- `infra/flake.nix` — both flake inputs
