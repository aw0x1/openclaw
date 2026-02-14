# OpenClaw Deployment Status

**Last updated:** 2026-02-13
**Host:** optiplex (NixOS 25.11, 192.168.4.51 / 100.122.35.170)

## Service State

| Component | Status |
|-----------|--------|
| `openclaw-gateway.service` | ❌ Crash-looping — missing config (Issue #1) |
| `caddy.service` | ✅ Running |
| `fail2ban.service` | ✅ Running |
| Binary installed | ✅ `openclaw v2.0.0-beta5` at `/nix/store/.../bin/openclaw` |
| Secret provisioned | ✅ `/var/lib/openclaw/secrets/openrouter-key` (0400 openclaw:openclaw) |
| Nix store (pnpm deps) | ✅ Built and cached — future rebuilds are fast |

## What's Done

- [x] `infra/nixos/modules/openclaw.nix` created and deployed
- [x] `infra/flake.nix` updated with both openclaw flake inputs
- [x] OpenRouter API key provisioned to disk
- [x] Caddy virtualHosts for LAN (192.168.4.51) and mesh (100.122.35.170)
- [x] Memory limits: MemoryHigh=4G, MemoryMax=6G
- [x] AF_NETLINK added to RestrictAddressFamilies
- [x] ExecStart corrected to `openclaw gateway --port 18789`
- [x] Workspace repo initialized at `/home/aw/openclaw`
- [x] `workspace/SOUL.md` and `workspace/AGENTS.md` written

## Immediate Next Steps

1. **Resolve Issue #1** — run `openclaw setup` as the openclaw user, or
   pass `--allow-unconfigured` and configure via env vars

2. **Wire OpenRouter API key** — determine the correct env var name
   (`OPENROUTER_API_KEY`?) and load it from `/var/lib/openclaw/secrets/openrouter-key`

3. **Validate Caddy** — confirm `https://openclaw.local` resolves and proxies correctly

4. **Rotate OpenRouter key** — the key was exposed in plain text during
   this session. Regenerate at https://openrouter.ai → Keys before using in production.

5. **Add `gh` to infra packages** — needed for issue management from the CLI

6. **Push both repos to GitHub**
   - `infra` → `github.com/aw0x1/infra` (already exists)
   - `openclaw` → `github.com/aw0x1/openclaw` (needs remote set)

## Issues

- [Issue #1](issues/01-gateway-missing-config.md) — Gateway exits: "Missing config"
- [Issue #2](issues/02-scout-dj-module-compatibility.md) — Scout-DJ module compatibility gaps

## Quick Commands

```bash
# Check service
systemctl status openclaw-gateway

# Read logs
journalctl -u openclaw-gateway -n 30 --no-pager

# Run setup as openclaw user (resolves Issue #1)
sudo -u openclaw /nix/store/1lj2jradag5b00f9pj9k7jqxzpg777np-openclaw-2.0.0-beta5/bin/openclaw setup

# After setup — restart service
sudo systemctl restart openclaw-gateway

# Check gateway health (once running)
curl -s http://127.0.0.1:18789/health
```
