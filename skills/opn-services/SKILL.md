---
name: opn-services
description: Use for OPNsense network services — Unbound DNS, DHCP (ISC/KEA/Dnsmasq/relay), IDS/IPS, Monit, NTP, syslog, traffic shaping.
---

# opn-services (DNS/DHCP/IPS/monitoring, OPNsense-only)

## Overview

Services run on the firewall, configured through their modules: `unbound`
(resolver + blocklists), `dnsmasq`, `dhcrelay`, `kea` (modern DHCP),
`ids` (Suricata IPS), `monit`, `ntpd`, `syslog`, `trafficshaper`.
Retrieval-first: `docs.opnsense.org/development/api/core/{unbound,dnsmasq,kea,dhcrelay,ids,monit,ntpd,syslog,trafficshaper}.html`.

## When to Use

- Resolver/DHCP changes, DNS blocklists, IPS policy tuning, service
  monitoring/alerting, time sync, log forwarding, shaper queues/pipes.
- When NOT: filter rules (→ `opn-firewall`), tunnels (→ `opn-vpn`).

## Implementation

- One service, one module: change via the module's search → add/set →
  apply/reconfigure, then confirm the service state via `core/service`
  search and the service's own status endpoint.
- **Unbound:** overrides and blocklists (DNSBL) are data — review the feed
  list before enabling; a bad feed blackholes real domains.
- **DHCP:** one server per segment (KEA for new deployments); relay only
  where a central server exists. Overlapping pools/scopes are refused, not
  merged.
- **IDS/IPS:** tune by policy + suppress list, not by disabling the engine;
  every suppress entry carries a reason and an expiry review.
- **Shaping:** pipes → queues → rules; measure before and after
  (→ `opn-diagnostics`) so the shaper is proven, not believed.

## Common Mistakes

- Enabling every IPS ruleset at once (false-positive storm) instead of a
  staged policy rollout.
- Two DHCP servers on one broadcast domain.
- DNSBL feeds without an allowlist escape hatch for the operator.
- Service restart assumed sufficient where a reconfigure/apply is required.

## Reuses

`opn-firewall` (service ports/rules), `opn-diagnostics` (logs/verification).
