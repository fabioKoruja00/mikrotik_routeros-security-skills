---
name: ipip
description: "Read-only audit of IPIP and IPIPv6 tunnels on MikroTik RouterOS: tunnels without ipsec-secret, the weak default proposal behind the dynamic IPsec peer, keepalive turned off, tunnels trusted as LAN, IP-in-IP accepted from any source and IPsec objects left behind after the secret is cleared. Use this skill whenever a MikroTik device has /interface ipip or /interface ipipv6 entries, routes between sites over IP-in-IP, or the audit mentions IPIP, IP-in-IP or protocol 4 / 41 tunnels, without changing configuration."
---

# IPIP

IPIP wraps an IP packet in another IP header (protocol 4, or next header 4 and 41 for IPIPv6). It has no authentication and no state: the router accepts any encapsulated packet whose outer source matches `remote-address`, and an outer source is trivial to forge. Without IPsec, the routes pointed at the tunnel are open to whoever can reach the router.

## Rules

Read-only: `print`, `get`, `export`, `monitor` only — never `set`, `add`, `remove`, `enable`, `disable`, `reboot`. Connect with a dedicated least-privilege audit account; do not assume the built-in `read` group is strictly read-only. Audit only the devices that were named. Secrets never enter the report (use `proplist` on areas that store credentials). Method, severity scale (CRITICAL / HIGH / MEDIUM / LOW — in doubt, the lower), output format and the collection order live in `audit-method`; factory values in `factory-defaults`. Read the RouterOS version first: v6 and v7 menus differ, and a command in the wrong menu returns empty.

## Checks

[references/checks.md](references/checks.md)

## Traps

- **Never run `print detail` or `export show-sensitive` on `/interface ipip` or `/interface ipipv6`.** `ipsec-secret` is a sensitive field. Use the `proplist` in the checks and `count-only` for presence.
- **A secret never set is not an empty secret.** `where ipsec-secret=""` matches only a secret that was set and then cleared; a tunnel created without one matches only `where !ipsec-secret`. Count both.
- **IPIPv6 carries both families.** One tunnel with a secret creates two dynamic policies, `ipencap` (IPv4 inside) and `ipv6-encap` (IPv6 inside); the firewall needs both protocols in view. IPIP over IPv4 uses `ipencap` only.
- **The running flag reports keepalive, not traffic.** A secret set on one end only drops the flag; without keepalive a dead peer does not drop it. Compare `rx-byte` in `/interface print stats`.
- **Clearing `ipsec-secret` does not remove the IPsec.** The dynamic peer, identity and policy stay until the tunnel is removed. Read the dynamic policies before trusting an empty secret.
- **IPsec requires `allow-fast-path=no` on `/interface ipip`.** The router refuses the secret otherwise. `/interface ipipv6` has no such property.
- **`mtu=auto` is the default**, giving `actual-mtu` 1480 on IPIP and 1460 on IPIPv6; IPv6 inside needs at least 1280 (`ipv6`).
- The dynamic peer uses the `default` profile and proposal; their algorithms are audited in `vpn/ipsec`. GRE tunnels have their own menu (`tunnels/gre`).
