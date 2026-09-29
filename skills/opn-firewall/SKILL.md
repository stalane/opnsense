---
name: opn-firewall
description: Use when managing OPNsense firewall rules, aliases, NAT, or traffic shaping — filter CRUD, alias tables, source/destination NAT, apply flow.
---

# opn-firewall (filter/alias/NAT, OPNsense-only)

## Overview

Packet filtering lives in `firewall/filter`, address sets in
`firewall/alias`, address translation in
`firewall/{source_nat,d_nat,one_to_one,npt}`. Writes are staged until
applied. Retrieval-first: exact commands at
`docs.opnsense.org/development/api/core/firewall.html`.

## When to Use

- New/changed filter rules, aliases (incl. GeoIP/URL tables), NAT port
  forwards, outbound NAT, 1:1, NPTv6, shaping queues.
- When NOT: tunnels (→ `opn-vpn`), DNS/DHCP/IPS services (→ `opn-services`).

## Implementation

Base: `https://$OPN_HOST/api` with `-u "$OPN_KEY:$OPN_SECRET"`. Search-grid
POST body: `{"current":1,"rowCount":25,"sort":{},"searchPhrase":""}`.

```bash
# find a rule by description
curl -s -u "$OPN_KEY:$OPN_SECRET" \
  "https://$OPN_HOST/api/firewall/filter/searchRule" \
  -d '{"current":1,"rowCount":25,"sort":{},"searchPhrase":"allow app"}'
# stage a rule, then activate (nothing is live before apply)
curl -s -u "$OPN_KEY:$OPN_SECRET" \
  "https://$OPN_HOST/api/firewall/filter/addRule" \
  -d '{"rule":{"description":"allow app","source_net":"192.168.1.0/24","protocol":"TCP","destination_net":"10.0.0.0/24"}}'
curl -s -u "$OPN_KEY:$OPN_SECRET" -X POST \
  "https://$OPN_HOST/api/firewall/filter/apply"
```

Rule lifecycle: `searchRule` → `addRule` / `setRule/<uuid>` /
`delRule/<uuid>` / `toggleRule/<uuid>/<0|1>` → `apply`.
`moveRuleBefore/<selected>/<target>` orders rules.
`alias`: `search_item`/`add_item`/`set_item/<uuid>`/`del_item/<uuid>`/
`reconfigure` (+ `alias_util/{add,delete,flush,find_references}` for table
content). Rule hit counts: `filter_util/rule_stats`.

## Common Mistakes

- Forgetting `apply`/`reconfigure` — staged, not live.
- Editing the core GUI ruleset expecting the automation API to see it
  (Firewall ‣ Automation namespace is separate; invisible rules are not
  returned by the API).
- Unknown model fields guessed instead of copied from the GUI: open browser
  devtools, repeat the action in the GUI, copy the `/api/` request verbatim.
- `curl -k` in committed scripts (lab-only; production pins the CA/cert).

## Reuses

`opn-diagnostics` (verify hits/logs), `opn-system` (backup before sessions).
