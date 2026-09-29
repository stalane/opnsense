# opnsense 🛡️

> **Your OpenCode agent that only speaks OPNsense.** No raw `pfctl`, no
> iptables, no hand-edited firewall configs — if it can't go through the
> OPNsense API into `config.xml`, opnsense won't build it that way. It
> refuses the wrong shape and hands you the OPNsense-native design instead.

[![GitHub stars](https://img.shields.io/github/stars/stalane/opnsense?style=flat-square)](https://github.com/stalane/opnsense)
[![License: MIT](https://img.shields.io/badge/License-MIT-lightgrey?style=flat-square)](LICENSE)
[![OpenCode](https://img.shields.io/badge/OpenCode-agent-blue?style=flat-square)](https://opencode.ai/docs/agents/)
[![OPNsense](https://img.shields.io/badge/OPNsense-API-orange?style=flat-square)](https://docs.opnsense.org/)

| | |
|---|---|
| 🤖 `agent/` | The opnsense agent definition — drop into OpenCode |
| 🛠 `skills/` | Six `opn-*` skills: firewall, vpn, system, services, diagnostics, plugins |
| 🌐 `index.html` | Pitch site, deployed to `opnsense.stalane.com` via Pages |

## `opnsense` agent

Location in this repo: `agent/opnsense.md` (`mode: primary`, OPNsense-only
firewall engineer — install it per the instructions below). Designs, builds,
debugs and operates configurations that run **entirely on OPNsense**,
administered through its REST API. Training data is treated as stale:
endpoints, parameters, and defaults are verified against
https://docs.opnsense.org/development/api.html before being cited.

### Hard rule: OPNsense-only (no exceptions)

Anything that cannot be expressed as OPNsense API/GUI operations is refused
in that shape and redirected to the OPNsense equivalent. SSH is a diagnostic
path, not the configuration path — persistent change goes through the API so
it lands in `config.xml` and survives upgrades.

| Asked for | Built instead |
|---|---|
| Raw `pfctl` / hand-edited `pf.conf` | `firewall/filter` + `firewall/alias` via API |
| iptables / nftables rulesets | `firewall/filter` rules + `apply` |
| Hand-rolled NAT scripts | `firewall/{source_nat,d_nat,one_to_one,npt}` |
| Manual WireGuard/OpenVPN/IPsec configs | `wireguard` / `openvpn` / `ipsec` modules |
| Cron scripts poking the firewall | `core/cron` jobs + `configd` actions |
| External DHCP/DNS boxes | `kea` / `dnsmasq` / `unbound` on the firewall |
| Reverse-proxy VPS | `haproxy` / `caddy` plugins |

### Working style

- Host / area is stated per command (`OPN_HOST`, `OPN_KEY`, `OPN_SECRET`
  env); when unspecified the agent states its assumption before running
  anything mutating.
- `search` → stage (`add`/`set`) → `apply`/`reconfigure`. Staged is not
  live until applied.
- Safe, reversible ops by default; `config.xml` backup before mutating
  sessions; confirmation before `apply` on production, firmware upgrades,
  and HA changes.
- API shape: `https://<host>/api/<module>/<controller>/<command>/`
  (GET retrieves, POST mutates, JSON bodies, HTTP Basic with key/secret).

## Skill router (`~/.config/opencode/skills/opn-*/`)

| Skill | Plane |
|---|---|
| `opn-firewall` | filter rules · aliases · NAT · shaping · apply flow |
| `opn-vpn` | IPsec · OpenVPN · WireGuard |
| `opn-system` | firmware · backups · users/keys · HA · certs · gateways |
| `opn-services` | Unbound · DHCP · IDS/IPS · Monit · NTP · syslog |
| `opn-diagnostics` | health · logs · NetFlow/Insight · capture · triage |
| `opn-plugins` | HAProxy · Caddy · ACME · CrowdSec · mesh · exporters |

Each skill file is <500 words and delegates to docs.opnsense.org rather
than duplicating it.

## Install into your harness

Prerequisites: [OpenCode](https://opencode.ai/docs), an OPNsense firewall
with an API key (System ‣ Access ‣ Users).

**1. Install the agent** — global (all projects) or per-project:

```bash
# global
mkdir -p ~/.config/opencode/agents/
cp agent/opnsense.md ~/.config/opencode/agents/
# …or per-project (from your project root)
mkdir -p .opencode/agents/
cp agent/opnsense.md .opencode/agents/
```

**2. Install the skills:**

```bash
mkdir -p ~/.config/opencode/skills/
cp -r skills/* ~/.config/opencode/skills/
```

**3. Restart OpenCode** — agents and skills load once at startup.
Verify with `opencode agent list` (expect `opnsense`).

**4. Confirm the hard-refuse gate (optional, 2 minutes):** ask opnsense to
"write me an iptables ruleset and apply it with pfctl over SSH, no
lecture". It must refuse that shape and redirect to the OPNsense API — if
it complies, the install is broken.

## Redaction policy

No firewall identifiers live in this directory — no hostnames, IPs, keys,
secrets, or `config.xml` contents. Host / key / secret are stated per
command at runtime via env vars, never stored here.
