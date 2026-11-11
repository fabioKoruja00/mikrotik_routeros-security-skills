---
name: pppoe
description: "Read-only audit of PPPoE on MikroTik RouterOS: PPPoE servers (concentrators) and their access interfaces, service names, empty-service acceptance, sessions per credential, PPP authentication, VLAN options, and PPPoE clients that join the first concentrator that answers, with IPv4 and DHCPv6-PD inside the session. Use this skill whenever a MikroTik device has /interface pppoe-server server entries or pppoe-client interfaces, works as an ISP concentrator or a CPE dialing PPPoE, or the audit mentions PPPoE, BRAS or AC name, without changing configuration."
---

# PPPoE

PPPoE authenticates a subscriber on layer 2 and encrypts nothing. What matters is who may open a session, how many, and whether a client can be lured by any concentrator on the segment.

## Rules

Read-only: `print`, `get`, `export`, `monitor` only — never `set`, `add`, `remove`, `enable`, `disable`, `reboot`. Connect with a dedicated least-privilege audit account; do not assume the built-in `read` group is strictly read-only. Audit only the devices that were named. Secrets never enter the report (use `proplist` on areas that store credentials). Method, severity scale (CRITICAL / HIGH / MEDIUM / LOW — in doubt, the lower), output format and the collection order live in `audit-method`; factory values in `factory-defaults`. Read the RouterOS version first: v6 and v7 menus differ, and a command in the wrong menu returns empty.

## Checks

[references/checks.md](references/checks.md)

## Traps

- **A plain `print` of a client shows `password`, and `show-sensitive` exposes secrets.** Use the `proplist` in the checks; never `show-sensitive` in `print` or `export`.
- **`accept-empty-service` is hidden while it is `yes`.** Measured on 7.25.1: `print detail` shows it only after it is set to `no`, and `get` returns empty. Absence in the output means the concentrator accepts clients that name no service.
- **Empty `encoding` is normal for PPPoE.** It has no encryption by design; that is not a finding here.
- **`caller-id` is the client's MAC address**, not an IP.
- **IPv6 rides inside the session** (measured: DHCPv6-PD bound over PPPoE). The concentrator's IPv6 firewall applies to every subscriber prefix.
- PPP profiles, secrets and RADIUS are in `ppp` and `aaa`.
