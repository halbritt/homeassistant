# homeassistant

Durable, inspectable, cross-agent **provenance and desired-state for the host
`homeassistant`** — the Home Assistant OS appliance on the home LAN
(`192.168.1.64` / tailnet `100.105.145.26`). One repo per system, one directory
per subsystem. Sibling of [`proximal`](https://github.com/halbritt/proximal),
which documents the workstation this fleet is managed from.

This is operational state, not a codebase. Its job is to remember — across runs
and across agents — what this appliance looks like, how it is reached, what
config it should run, and what was already tried and rejected.

## The box

| fact | value |
|---|---|
| Hardware | Raspberry Pi (MAC `2c:cf:67:fb:66:0d`, RPi Trading OUI) |
| OS | Home Assistant OS (HAOS) — appliance, not a general Linux host |
| LAN | `192.168.1.64` (`enp3s0` ARP on proximal) |
| Tailscale | `100.105.145.26` · hostname `homeassistant` · offers exit node |
| HA core version | 2026.7.2 (454 entities; read via ha-mcp `ha_eval_template`, 2026-07-21) |

Verified 2026-07-21 from `proximal`.

## Access posture (as-found, 2026-07-21)

- **`:8123`** — Home Assistant web UI + REST API. API answers `401` without a
  token (healthy; auth required).
- **`:4357`** — HAOS observer. Reports: Supervisor **Connected**, Support
  **Supported**, Health **Healthy**.
- **`:9583`** — **[ha-mcp](https://github.com/homeassistant-ai/ha-mcp)** add-on
  (v7.14.1 as of 2026-07-21), the agent interface of record — see below.
- **No SSH**: ports 22, 2222, and 22222 (HAOS developer SSH) all closed. Host
  administration happens through the HA UI / Supervisor, not a shell.
- **Native MCP Server integration: not enabled** (`/mcp_server/sse` → `404`) —
  and not wanted: ha-mcp supersedes it.

### Agent access (the plan of record)

The **ha-mcp add-on** (streamable-HTTP MCP, ~89 tools: device control, state
queries, automation management, and more) runs on the appliance at
`:9583`. Its URL contains a **secret path segment** (`/private_…`) that serves
as the credential — **the full URL is a secret; never commit it**. It lives
only in the add-on config on the appliance and in `~/.claude*/.claude.json` on
proximal.

Registered on proximal at **user scope** (2026-07-21) so every Claude Code
session sees it, via the tailnet IP (survives mDNS flakiness):

```bash
claude mcp add --transport http --scope user ha-mcp \
  "http://100.105.145.26:9583/private_<SECRET>"
```

History: originally registered 
project-local to `~/git/ha-mcp` (a clone of the upstream repo), which made it
invisible to sessions launched anywhere else — that's the trap the user-scope
registration fixes. Rotate the secret from the add-on's configuration page if
the URL ever leaks.

Next capture into this repo: HA core/Supervisor/OS versions, installed
add-ons (ha-mcp's own version + config posture), integrations, and an entity
registry snapshot — then grow subsystem directories as they earn them.

## Subsystems

None versioned yet — the appliance was inventoried from the outside only. Add a
directory when a subsystem's config is worth versioning (e.g. `config/` for
YAML packages, `addons/` for add-on settings, `automations/`); don't pre-create
empty ones.

## The one rule

**Values and config, never credentials.** Commit settings, YAML, dashboards,
and rationale. Never commit long-lived access tokens, `secrets.yaml`, `*.env`,
or keys. Tokens live only in `0600` files on proximal or in the HA keyring.

## Conventions

- **One repo per host, one directory per subsystem.**
- **Canonical-in-repo, installed-on-box.** Each subsystem README maps repo
  files → their install path on the appliance.
- **Commit and push often.** `origin` = `github.com/halbritt/homeassistant`.
