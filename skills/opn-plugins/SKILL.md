---
name: opn-plugins
description: Use when installing or wiring OPNsense plugins — HAProxy, Caddy, ACME, CrowdSec, Tailscale/ZeroTier, telemetry exporters.
---

# opn-plugins (extensions, OPNsense-only)

## Overview

Plugins extend the firewall through the same API shape under the `plugins/`
tree (e.g. `haproxy`, `caddy`, `acmeclient`, `crowdsec`, `tailscale`,
`zerotier`, `telegraf`, `nodeexporter`, `wazuhagent`). Retrieval-first:
`docs.opnsense.org/development/api/plugins/<name>.html` — community vs
third-party provenance matters. Install via System ‣ Firmware ‣ Plugins
(API-mirrored); configure via the plugin's module; apply; verify.

## When to Use

- Reverse proxying, ACME certificate automation, threat-intel blocking,
  mesh VPN agents, metrics/log export.
- When NOT: core firewall/VPN/services (→ sibling `opn-*` skills).

## Implementation

| Need | Plugin | Notes |
|---|---|---|
| Reverse proxy / TLS offload | `haproxy` / `caddy` | One ingress per firewall; backends via filter rules (→ `opn-firewall`) |
| Certificate automation | `acmeclient` | DNS-01 or HTTP-01; keys stay in `trust` (→ `opn-system`) |
| IDS reinforcement | `crowdsec` | Decisions feed blocklists; review before enforce mode |
| Mesh overlays | `tailscale` / `zerotier` | Agent on the firewall, not beside it |
| Telemetry | `telegraf` / `nodeexporter` / `wazuhagent` | Export out; no inbound management ports |

- Pin the plugin version installed and re-verify after every firmware
  upgrade (plugins follow the branch).
- Third-party plugins get the same scrutiny as any supply chain: source,
  maintainer, update cadence — documented before install.

## Common Mistakes

- Proxying around the firewall instead of through it (rules bypassed).
- ACME HTTP-01 without the matching WAN filter rule, then blaming the CA.
- Enforce-mode blocklists with no allowlist escape hatch.
- Forgetting plugin re-verification after an upgrade.

## Reuses

`opn-firewall` (ingress rules), `opn-system` (trust/certs, upgrades),
`opn-services` (Unbound overrides for proxied names).
