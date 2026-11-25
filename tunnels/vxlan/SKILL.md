---
name: vxlan
description: "Read-only audit of VXLAN on MikroTik RouterOS v7: overlays crossing an untrusted underlay without IPsec or WireGuard, the VXLAN UDP port reachable from any source, MAC learning from VTEPs nobody declared, VXLAN bridged into the LAN or trusted as LAN, and VNI or VTEPs outside the inventory. Use this skill whenever a MikroTik device has /interface vxlan entries, extends Layer 2 over UDP, or the audit mentions VXLAN, VNI or VTEP, without changing configuration."
---

# VXLAN

VXLAN puts Ethernet frames inside UDP, with no authentication and no cipher. The router accepts a frame from any source that reaches the VXLAN port with a matching VNI; the VTEP list only says where flooded traffic goes.

## Rules

Read-only: `print`, `get`, `export`, `monitor` only — never `set`, `add`, `remove`, `enable`, `disable`, `reboot`. Connect with a dedicated least-privilege audit account; do not assume the built-in `read` group is strictly read-only. Audit only the devices that were named. Secrets never enter the report (use `proplist` on areas that store credentials). Method, severity scale (CRITICAL / HIGH / MEDIUM / LOW — in doubt, the lower), output format and the collection order live in `audit-method`; factory values in `factory-defaults`. Read the RouterOS version first: v6 and v7 menus differ, and a command in the wrong menu returns empty.

## Checks

[references/checks.md](references/checks.md)

## Traps

Measured on 7.25.2 with a VXLAN between two routers.

- **Fast path skips the firewall.** With `allow-fast-path=yes` (the default), VXLAN packets for a configured VNI reach the interface without passing `raw` or `filter`: a drop on the VXLAN port counted nothing and the tunnel kept working. With `allow-fast-path=no` the same rule dropped them.
- **Static VTEPs are not an access list.** A VTEP left out of `/interface vxlan vteps` still gets its frames accepted and its MAC and IP learned into the FDB, and a ping crossed both ways: the undeclared source gets a two-way session.
- **`learning=no` does not refuse frames.** Frames from an undeclared VTEP are still decapsulated (ARP is learned from them); only the FDB entry is not created, and the undeclared source got no reply.
- **Read `port` on each interface.** The default on 7.25.2 is 4789; older course material uses 8472. Several interfaces can share one port, separated by VNI.
- **Interface counters undercount with fast path.** `/interface print stats` showed a fraction of the packets that crossed a working tunnel; a low or zero count does not prove the tunnel is idle.
- **Tunnel names come from `/interface vxlan print proplist=name`.** A filter on the string `vxlan` misses renamed interfaces.
- Bridge-wide controls (RSTP, horizon, DHCP snooping) are in `layer2`; IPsec proposals and peers in `vpn/ipsec`; WireGuard in `vpn/wireguard`.
