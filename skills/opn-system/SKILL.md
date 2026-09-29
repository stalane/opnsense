---
name: opn-system
description: Use for OPNsense platform work — firmware upgrades, config backups, users/API keys, high availability, certificates, gateways and routes.
---

# opn-system (firmware/backups/users/HA/trust, OPNsense-only)

## Overview

The firewall itself is the plane: `firmware` (updates), `core` (services,
firmware options, backups), `auth` (users/groups), `trust` (authorities and
certificates), `routes`/`routing`/`gateways`, HA via CARP. Retrieval-first:
`docs.opnsense.org/development/api/core/{firmware,core,auth,trust,routes,routing}.html`
plus System/High Availability docs.

## When to Use

- Upgrade planning, pre-change backups, API key provisioning, user/privilege
  changes, cert issuance/renewal, gateway/multi-WAN work, HA pair changes.
- When NOT: rules/tunnels/services (→ sibling `opn-*` skills).

## Implementation

- **Backups first, always.** System ‣ Configuration ‣ Backups (download
  `config.xml`) before any mutating session; verify the backup restores by
  inspecting its size/revision, and keep the pre-change copy until the change
  is verified live.
- **Firmware:** check for updates, read the release notes for the running
  branch (Community vs Business), upgrade, confirm version + service health
  after reboot. Never upgrade both HA nodes at once.
- **Users/keys:** System ‣ Access ‣ Users; API keys are per-user and bounded
  by that user's effective privileges — provision least-privilege keys per
  automation, rotate by replacing (never by editing in place across two
  systems).
- **Trust:** authorities and certificates via `trust`; private keys never
  leave the firewall except through the documented export flow.
- **Gateways/routes/HA:** gateway groups for multi-WAN, static routes via
  `routes`, CARP/VIPs for HA; config-sync direction is primary → secondary.

## Common Mistakes

- Upgrading without a fresh backup and without reading branch release notes.
- One shared admin API key for every automation (blast radius).
- Changing HA sync direction or upgrading the secondary first.
- Committing `config.xml`, keys, or secrets anywhere (per-command env only).

## Reuses

`opn-diagnostics` (post-upgrade health), all skills (backup prerequisite).
