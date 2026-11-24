---
name: back-to-home
description: "Read-only audit of remote access raised by MikroTik's own shortcuts on RouterOS v7: Back To Home (BTH) enabled without approval, BTH users and peers nobody registered, users with LAN or file access, the dynamic firewall rules BTH injects, and VPN access left by Quick Set (L2TP, PPTP or SSTP server with a generic PPP user). Use this skill whenever /ip cloud shows back-to-home-vpn enabled, a back-to-home-vpn WireGuard interface exists, Quick Set was used, or the audit mentions Back To Home, BTH, Quick Set or VPN Access, without changing configuration."
---

# Back To Home and Quick Set

Both shortcuts open remote access in a few clicks and leave little trace in the configuration a reviewer usually reads. BTH builds a WireGuard server reachable through MikroTik's relay and DDNS; Quick Set's VPN Access raises a PPP server with a generic user.

## Rules

Read-only: `print`, `get`, `export`, `monitor` only — never `set`, `add`, `remove`, `enable`, `disable`, `reboot`. Connect with a dedicated least-privilege audit account; do not assume the built-in `read` group is strictly read-only. Audit only the devices that were named. Secrets never enter the report (use `proplist` on areas that store credentials). Method, severity scale (CRITICAL / HIGH / MEDIUM / LOW — in doubt, the lower), output format and the collection order live in `audit-method`; factory values in `factory-defaults`. Read the RouterOS version first: v6 and v7 menus differ, and a command in the wrong menu returns empty.

## Checks

[references/checks.md](references/checks.md)

## Traps

- **Never run `/ip cloud print`, `show-client-config`, or `get` on `vpn-private-key`, `vpn-peer-private-key`, `vpn-wireguard-client-config` or `vpn-wireguard-client-config-qrcode`.** `/ip cloud back-to-home-user` also holds `private-key`. Read named fields only.
- **The menu is `/ip cloud back-to-home-user`, singular, on 7.25.2.** The plural form returns `bad command name`; files shared over BTH are in `/ip cloud back-to-home-file`.
- **Enabling BTH injects dynamic rules at the top of the firewall,** IPv4 and IPv6: input `accept` on the BTH UDP port from any source, forward `drop` for restricted peers, and a `masquerade` for the tunnel. The owner's own input drop never sees that port.
- **`allow-lan=no` is narrower than it sounds.** It is a forward drop towards `out-interface-list=LAN` only: the router itself and networks outside the `LAN` list stay reachable unless the firewall says otherwise.
- **Enabling BTH creates a peer before any user exists.** It is the router-level client whose private key the router keeps; a BTH peer is not proof of a registered person.
- **A server menu such as `/interface l2tp-server server` takes `get`, not `print proplist`,** on 7.25.2.
- **Back To Home asks for the administrative credential** to build the tunnel. An unknown WireGuard peer is remote access nobody registered.
- WireGuard interfaces other than BTH are in `vpn/wireguard`; L2TP, PPTP and SSTP servers in detail in their own skills; the PPP layer in `vpn/ppp`.
