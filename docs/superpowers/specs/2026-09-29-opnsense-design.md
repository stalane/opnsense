# opnsense Design Spec (2026-09-29)

**Goal:** A cfadmin-mirror OpenCode agent project that administers OPNsense
firewalls API-first, plus a dark-theme pitch site on `opnsense.stalane.com`
and an apex card on `stalane.com`.

**Decisions (user-approved, Approach A):**
- Strict **OPNsense-only mirror** of cfadmin: the agent refuses non-OPNsense
  firewall shapes (raw `pfctl`/iptables/nftables, manual `pf.conf` edits,
  consumer-router flows) and redirects to the OPNsense API.
- 6 thin skills, one plane each: `opn-firewall`, `opn-vpn`, `opn-system`,
  `opn-services`, `opn-diagnostics`, `opn-plugins` (each <500 words, curl
  recipes, delegates to https://docs.opnsense.org/ — never vendored docs).
- Single public repo `stalane/opnsense`: root `AGENTS.md`, OpenCode drop-in
  `agent/opnsense.md`, `skills/opn-*/SKILL.md`, `README.md`, MIT `LICENSE`,
  static site at root (`index.html` + favicons, Pages project `opnsense`).

**Ground truths (verified 2026-09-29 against docs.opnsense.org):**
- API shape: `https://<fw>/api/<module>/<controller>/<command>/[params]`;
  GET retrieves, POST mutates/executes; JSON bodies; HTTP Basic with
  key/secret (System ‣ Access ‣ Users ‣ API keys); per-user effective
  privileges gate each endpoint.
- Firewall plane: `firewall/filter/{searchRule,addRule,setRule,delRule,
  toggleRule/<uuid>/<enabled>,apply}`, `firewall/alias/*`,
  `firewall/{d_nat,source_nat,one_to_one,npt}/*`, `firewall/filter_util/
  rule_stats`. Model flow: search → add/set → **apply** (staged until
  applied). Automation-namespace rules only via that API; core GUI rules are
  separate. Unknown params: inspect GUI `/api/` calls in browser devtools.
- Site/api patterns reuse the house stack: dark finance tokens, `wrangler
  pages deploy --branch main`, custom domain via Pages API on the same-account
  zone (DNS + TLS automatic), apex card = verbatim `<section class="panel">`
  + `shots/opnsense.png` (1280px via `bsk`) + footer link.

**Non-goals:** No vendored API snapshots (go stale); no stored credentials
(host/key/secret stated per command, never in repo); no Business-edition-only
flows as defaults; no MCP servers (OPNsense exposes REST, consumed with curl).
