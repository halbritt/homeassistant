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
| HA core version | unknown until API access (not exposed unauthenticated) |

Verified 2026-07-21 from `proximal`.

## Access posture (as-found, 2026-07-21)

- **`:8123`** — Home Assistant web UI + REST API. API answers `401` without a
  token (healthy; auth required).
- **`:4357`** — HAOS observer. Reports: Supervisor **Connected**, Support
  **Supported**, Health **Healthy**.
- **No SSH**: ports 22, 2222, and 22222 (HAOS developer SSH) all closed. Host
  administration happens through the HA UI / Supervisor, not a shell.
- **MCP Server integration: not enabled** — `/mcp_server/sse` returns `404`.
- **No credentials on proximal**: no long-lived access token or HA CLI config
  found anywhere on the managing workstation.

### Wiring up agent access (the plan of record)

Home Assistant ships a native **MCP Server** integration; that is the intended
agent interface for this fleet, not SSH.

1. In the HA UI: *Settings → Devices & Services → Add Integration → MCP
   Server*. This exposes `http://homeassistant:8123/mcp_server/sse`.
2. Create a **long-lived access token** (user profile → Security). Store it on
   proximal **outside any repo** (e.g. `~/.config/hass/token`, mode `0600`).
3. Register with Claude Code on proximal:

   ```bash
   claude mcp add --transport sse homeassistant \
     http://100.105.145.26:8123/mcp_server/sse \
     --header "Authorization: Bearer $(cat ~/.config/hass/token)"
   ```

4. Once connected, capture into this repo: HA core/Supervisor/OS versions,
   installed add-ons, integrations, and the entity registry snapshot — then
   grow subsystem directories as they earn them.

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
