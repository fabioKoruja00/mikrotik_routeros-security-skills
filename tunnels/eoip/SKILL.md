---
name: eoip
description: "Read-only audit of EoIP and EoIPv6 on MikroTik RouterOS: tunnels without ipsec-secret, the weak default proposal behind the dynamic IPsec peer, tunnels bridged into the LAN or trusted as LAN, GRE and EtherIP accepted from any source, keepalive turned off, duplicate tunnel MACs and IPsec objects left behind after the secret is cleared. Use this skill whenever a MikroTik device has /interface eoip or /interface eoipv6 entries, bridges sites at Layer 2 over IP, or the audit mentions EoIP, EtherIP or GRE tunnels, without changing configuration."
---

# EoIP

EoIP carries whole Ethernet frames between two routers: broadcasts, ARP, DHCP, RA and STP cross the tunnel as if both ends shared a switch. The router does not check the source against `remote-address`: frames with the right `tunnel-id` from another source are decapsulated. Without IPsec and a firewall that admits GRE only from the remote, anyone able to send packets to the router who guesses the id injects frames.

## Rules

Read-only: `print`, `get`, `export`, `monitor` only — never `set`, `add`, `remove`, `enable`, `disable`, `reboot`. Connect with a dedicated least-privilege audit account; do not assume the built-in `read` group is strictly read-only. Audit only the devices that were named. Secrets never enter the report (use `proplist` on areas that store credentials). Method, severity scale (CRITICAL / HIGH / MEDIUM / LOW — in doubt, the lower), output format and the collection order live in `audit-method`; factory values in `factory-defaults`. Read the RouterOS version first: v6 and v7 menus differ, and a command in the wrong menu returns empty.

## Checks

[references/checks.md](references/checks.md)

## Traps

Measured on 7.25.2 with EoIP and EoIPv6 tunnels between two routers, carrying IPv4 and IPv6.

- **Never run `print detail` or `export show-sensitive` on `/interface eoip` or `/interface eoipv6`.** `ipsec-secret` is a sensitive field. Use the `proplist` in the checks and `count-only` for presence.
- **EoIPv6 is not GRE.** It travels as IPv6 next header 97, named `etherip` in the IPv6 firewall and in `/ip ipsec policy`. A filter on `protocol=gre` misses it. The IPsec objects of both menus live under `/ip ipsec`.
- **The running flag reports keepalive, not traffic.** A tunnel with `R` can still receive nothing: compare `rx-byte` in `/interface print stats`. Without keepalive (unset, the field disappears from `print`) the flag never drops. With keepalive on one end only, that end stays not running although the remote is up. A different `tunnel-id` on each end carries nothing, and keepalive drops the flag on both.
- **An established IPsec SA does not prove the tunnel.** After decryption the GRE or EtherIP packet goes through the input chain again; a drop there leaves the SA up and the tunnel empty.
- **A secret never set is not an empty secret.** `where ipsec-secret=""` matches only a secret that was set and then cleared; a tunnel created without one matches only `where !ipsec-secret`. Count both.
- **Clearing `ipsec-secret` does not remove the IPsec.** The dynamic peer and policy stay, and fast path stays blocked, until the tunnel is removed. Read the dynamic policies before trusting `ipsec-secret=""`.
- **The IPsec policy binds the input on EoIP, not on EoIPv6.** With a secret on the receiver only, plaintext EoIP from the remote was dropped, but plaintext EoIPv6 was decapsulated. The policy names only the remote's address.
- **`mtu=auto` gives `actual-mtu` 1458**, 65494 with a secret and no SA, 1416 with the SA established. EoIPv6 read 1444 with a secret and no SA, 1406 with the SA.
- **IPsec requires `allow-fast-path=no` on `/interface eoip`.** `/interface eoipv6` has no such property.
- **`tunnel-id` ranges differ.** `/interface eoip` accepts up to 65535; `/interface eoipv6` stops at 4095 on 7.25.2.
- The dynamic peer uses the `default` proposal; its algorithms are audited in `vpn/ipsec`. Bridge-wide controls (RSTP, horizon, DHCP snooping) are in `layer2`.
