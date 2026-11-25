---
name: 6to4
description: "Read-only audit of 6to4 and 6in4 tunnels on MikroTik RouterOS (/interface 6to4): tunnels open to any IPv4 source, third-party 6to4 relays, tunnels without ipsec-secret, IPsec that does not bind the input, keepalive set on one end only, IPsec left behind, tunnels trusted as LAN, MTU below the IPv6 minimum and protocol 41 accepted from any source. Use this skill whenever a MikroTik device has /interface 6to4 entries, gets IPv6 from a tunnel broker, or the audit mentions 6to4, 6in4, SIT or protocol 41, without changing configuration."
---

# 6to4

`/interface 6to4` carries IPv6 inside IPv4 (protocol 41). With `remote-address` set it is a 6in4 point-to-point tunnel, the usual form for a tunnel broker, and the router decapsulates only packets whose outer source is that remote. With `remote-address` unspecified it is true 6to4: the router sends each packet to the IPv4 address embedded in its `2002::/16` destination and decapsulates protocol 41 from any source. Protocol 41 has no authentication, and an outer source is trivial to forge.

## Rules

Read-only: `print`, `get`, `export`, `monitor` only — never `set`, `add`, `remove`, `enable`, `disable`, `reboot`. Connect with a dedicated least-privilege audit account; do not assume the built-in `read` group is strictly read-only. Audit only the devices that were named. Secrets never enter the report (use `proplist` on areas that store credentials). Method, severity scale (CRITICAL / HIGH / MEDIUM / LOW — in doubt, the lower), output format and the collection order live in `audit-method`; factory values in `factory-defaults`. Read the RouterOS version first: v6 and v7 menus differ, and a command in the wrong menu returns empty.

## Checks

[references/checks.md](references/checks.md)

## Traps

All measured on 7.25.2 with a tunnel between two routers.

- **Never run `export show-sensitive` or `print detail show-sensitive` on `/interface 6to4`.** `ipsec-secret` is a sensitive field; `print detail` and `export` hide it. Use the `proplist` in the checks and `count-only` for presence.
- **An empty `remote-address` reads `unspecified`.** `where !remote-address` matches nothing; count `where remote-address=unspecified`.
- **`ipsec-secret` does not bind the input.** A router with a secret still decapsulates plaintext protocol 41 from its remote; only an input rule that drops protocol 41 with `ipsec-policy=in,none` refuses it. With the secret on both ends the traffic is ESP with the `default` proposal (`aes-cbc`, `sha1`).
- **The established rule opens the tunnel.** Once the router sends anything through the tunnel (neighbor discovery is enough), the remote's protocol 41 enters by `connection-state=established` with no accept rule for it. A missing accept rule does not mean the tunnel is closed.
- **Keepalive must be set on both ends.** The factory value is none (the documentation states both `0,0` and 10 s × 10). With keepalive on one end only, that end stays not running and the tunnel carries nothing although the remote is up. Without keepalive the running flag stays up with the remote dead.
- **`ipsec-secret` with `remote-address` unspecified** creates a dynamic peer `0.0.0.0/32` and a wildcard policy `0.0.0.0/0` to `0.0.0.0/0` `ipv6-encap`, flagged invalid. Read it as a broken setup, not as protection.
- **Clearing `ipsec-secret` does not remove the IPsec.** The dynamic peer and policy stay until the tunnel is removed, and that end keeps sending ESP. A secret that differs between the ends leaves the policy in `ready-to-send` and logs `wrong password`.
- **True 6to4 works on 7.25.2**, although the documentation marks the example as not working: two routers with `2002::` addresses and `remote-address` unspecified exchanged traffic.
- **Public tunnel brokers offer no IPsec.** A 6in4 to a broker without a secret is the normal case; its risk sits in the endpoint and firewall checks.
- **`mtu=auto` gives `actual-mtu` 1480, 1438 with an established SA, and 65516 while a secret is set and no SA exists** (`ph2-state` `no-phase2`); that last value proves nothing. `/interface 6to4` has `dscp` but no `allow-fast-path`. The IPv6 firewall and RA on the tunnel are audited in `ipv6`; the IPsec profile and proposal in `vpn/ipsec`.
