---
name: l2tp
description: "Read-only audit of L2TP on MikroTik RouterOS: L2TP server and client with or without IPsec, the dynamic IPsec peer the server creates and the factory profile it negotiates, cleartext PPP authentication, shared PSK, L2TPv3 and l2tp-ether tunnels, the firewall around udp/1701, over IPv4 and IPv6 transport. Use this skill whenever a MikroTik device runs /interface l2tp-server, l2tp-client or l2tp-ether, serves L2TP/IPsec road-warrior access, or the audit mentions L2TP, L2TPv3 or use-ipsec, without changing configuration."
---

# L2TP

L2TP carries PPP and protects nothing by itself; the protection is the IPsec underneath. `use-ipsec=required` is the only setting that refuses a plain session, and the IPsec the server builds for it is the factory one.

## Rules

Read-only: `print`, `get`, `export`, `monitor` only — never `set`, `add`, `remove`, `enable`, `disable`, `reboot`. Connect with a dedicated least-privilege audit account; do not assume the built-in `read` group is strictly read-only. Audit only the devices that were named. Secrets never enter the report (use `proplist` on areas that store credentials). Method, severity scale (CRITICAL / HIGH / MEDIUM / LOW — in doubt, the lower), output format and the collection order live in `audit-method`; factory values in `factory-defaults`. Read the RouterOS version first: v6 and v7 menus differ, and a command in the wrong menu returns empty.

## Checks

[references/checks.md](references/checks.md)

## Traps

- **A plain `print` of the server or a client shows `ipsec-secret`, and the client also `password`.** Use the `proplist` in the checks and `:len` for presence.
- **`use-ipsec=yes` is not `required`.** It offers IPsec and still accepts a plain L2TP session.
- **The server's IPsec is a dynamic peer on the factory `default` profile.** Measured: `l2tp-in-server` is passive, IKEv1 `main`, `profile=default`, with a PSK identity and `generate-policy=port-strict`; the SA came up with `sha1`. Hardening it means hardening the `default` profile and proposal — audited in `ipsec`.
- **L2TP runs over IPv6 transport too** (measured: client to an IPv6 server address, IPsec established). Read both input chains.
- PPP profiles, secrets and sessions in general are in `ppp`.
