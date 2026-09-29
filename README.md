# opnsense 🛡️

> **Your OpenCode agent that only speaks OPNsense.** No raw `pfctl`, no
> iptables, no hand-edited firewall configs — if it can't go through the
> OPNsense API into `config.xml`, opnsense won't build it that way.

`AGENTS.md` is the full operator guide (agent behavior, hard-rule table,
skill router, install, redaction policy) — start there.

## Layout

| Path | What |
|---|---|
| `AGENTS.md` | Comprehensive guide: hard rule, API shape, skills, safety |
| `agent/opnsense.md` | OpenCode drop-in (`mode: primary`) |
| `skills/opn-firewall/` | Filter rules, aliases, NAT, shaping, apply flow |
| `skills/opn-vpn/` | IPsec, OpenVPN, WireGuard |
| `skills/opn-system/` | Firmware, backups, users/keys, HA, certs, gateways |
| `skills/opn-services/` | Unbound, DHCP, IDS/IPS, Monit, NTP, syslog |
| `skills/opn-diagnostics/` | Health, logs, NetFlow/Insight, capture, triage |
| `skills/opn-plugins/` | HAProxy, Caddy, ACME, CrowdSec, mesh, exporters |
| `index.html` | Pitch site → `opnsense.stalane.com` |

## Quick start

```bash
cp agent/opnsense.md ~/.config/opencode/agents/
cp -r skills/* ~/.config/opencode/skills/
export OPN_HOST=fw.example.com OPN_KEY=<key> OPN_SECRET=<secret>
curl -s -u "$OPN_KEY:$OPN_SECRET" \
  "https://$OPN_HOST/api/firewall/filter/searchRule" \
  -d '{"current":1,"rowCount":5,"sort":{},"searchPhrase":""}'
```

Docs: https://docs.opnsense.org/ · Site: https://opnsense.stalane.com
