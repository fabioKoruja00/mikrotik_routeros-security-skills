---
name: gre
description: "Read-only audit of GRE and GRE6 tunnels on MikroTik RouterOS: tunnels without ipsec-secret, the weak default proposal behind the dynamic IPsec peer, keepalive turned off, tunnels trusted as LAN, GRE accepted from any source and IPsec objects left behind after the secret is cleared. Use this skill whenever a MikroTik device has /interface gre or /interface gre6 entries, routes between sites over GRE, or the audit mentions GRE, GRE6 or protocol 47 tunnels, without changing configuration."
---

# GRE

GRE wraps a packet in another IP header (protocol 47, also over IPv6 for GRE6). It has no authentication and no state: the router accepts any encapsulated packet whose outer source matches `remote-address`, and an outer source is trivial to forge. Without IPsec, the routes pointed at the tunnel are open to whoever can reach the router.

## Rules

Read-only: `print`, `get`, `export`, `monitor` only — never `set`, `add`, `remove`, `enable`, `disable`, `reboot`. Connect with a dedicated least-privilege audit account; do not assume the built-in `read` group is strictly read-only. Audit only the devices that were named. Secrets never enter the report (use `proplist` on areas that store credentials). Method, severity scale (CRITICAL / HIGH / MEDIUM / LOW — in doubt, the lower), output format and the collection order live in `audit-method`; factory values in `factory-defaults`. Read the RouterOS version first: v6 and v7 menus differ, and a command in the wrong menu returns empty.

## Checks

[references/checks.md](references/checks.md)

## Traps

- **Never run `print detail` or `export show-sensitive` on `/interface gre` or `/interface gre6`.** `ipsec-secret` is a sensitive field. Use the `proplist` in the checks and `count-only` for presence.
- **A secret never set is not an empty secret.** `where ipsec-secret=""` matches only a secret that was set and then cleared; a tunnel created without one matches only `where !ipsec-secret`. Count both.
- **EoIP is GRE too.** EoIP over IPv4 travels as protocol 47 and its dynamic policies use `protocol=gre`: a firewall rule for GRE also admits EoIP, and a `gre` policy may belong to an EoIP tunnel (`tunnels/eoip`).
- **The running flag reports keepalive, not traffic.** Without keepalive a dead peer does not drop it. Compare `rx-byte` in `/interface print stats`.
- **Clearing `ipsec-secret` does not remove the IPsec.** The dynamic peer, identity and policy stay until the tunnel is removed, and `allow-fast-path=yes` is still refused. Read the dynamic policies before trusting an empty secret.
- **IPsec requires `allow-fast-path=no` on `/interface gre`.** The router refuses the secret otherwise. `/interface gre6` has neither `allow-fast-path` nor `dscp`.
- **`actual-mtu` lies while the IPsec has no SA.** `mtu=auto` gives 1476 on GRE and 1456 on GRE6; with `ipsec-secret` set on a GRE tunnel whose remote does not answer, `actual-mtu` reads 65512. Judge the MTU from `mtu` and the path; IPv6 inside needs at least 1280 (`ipv6`).
- The dynamic peer uses the `default` profile and proposal; their algorithms are audited in `vpn/ipsec`. IPIP tunnels have their own menu (`tunnels/ipip`).
