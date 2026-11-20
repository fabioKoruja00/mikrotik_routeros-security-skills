---
name: wireguard
description: "Read-only audit of WireGuard on MikroTik RouterOS v7: interfaces and their listen ports, peers with broad or overlapping allowed-address, peers without preshared key, client private keys stored on the router, stale peers and WireGuard interfaces trusted as LAN. Use this skill whenever a MikroTik device has /interface wireguard entries, serves road-warrior or site-to-site WireGuard, or the audit mentions WireGuard, wg-import or wg-export, without changing configuration."
---

# WireGuard

A WireGuard peer is identified by its key and trusted for every address in its `allowed-address`. A peer with `0.0.0.0/0` there can source any address into the router; a peer whose private key the router generated is only as safe as the router's configuration.

## Rules

Read-only: `print`, `get`, `export`, `monitor` only — never `set`, `add`, `remove`, `enable`, `disable`, `reboot`. Connect with a dedicated least-privilege audit account; do not assume the built-in `read` group is strictly read-only. Audit only the devices that were named. Secrets never enter the report (use `proplist` on areas that store credentials). Method, severity scale (CRITICAL / HIGH / MEDIUM / LOW — in doubt, the lower), output format and the collection order live in `audit-method`; factory values in `factory-defaults`. Read the RouterOS version first: v6 and v7 menus differ, and a command in the wrong menu returns empty.

## Checks

[references/checks.md](references/checks.md)

## Traps

- **Never run `wg-export`, `print detail` or `export show-sensitive` on WireGuard.** The interface holds `private-key`; peers can hold `private-key` and `preshared-key`. Use the `proplist` in the checks and `count-only` for presence.
- **`allowed-address` does not install routes.** It is the source filter and the crypto-routing table; routes to the remote networks are separate entries in `/ip route`.
- **The listen port shows in `/ip service` as dynamic `wireguard-<interface>`.** WireGuard answers nothing to a packet without a valid key, so the open port alone is not a finding.
- **Pick test addresses only after reading every `allowed-address`.** An overlap between interfaces or peers steers traffic to the wrong tunnel; 7.24 and later warn about it on the same interface only.
- Back To Home tunnels are in `back-to-home`.
</content>
