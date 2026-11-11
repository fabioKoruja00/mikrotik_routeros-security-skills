---
name: pptp
description: "Read-only audit of PPTP on MikroTik RouterOS: PPTP server and client, their exposure on tcp/1723 and GRE, MS-CHAP and MPPE, the device-mode switch that can disable PPTP, the NAT helper and live sessions. Use this skill whenever a MikroTik device has /interface pptp-server or pptp-client, passes PPTP for clients behind NAT, or the audit mentions PPTP, MPPE or MS-CHAPv2, without changing configuration."
---

# PPTP

PPTP has no fix: MS-CHAPv2 falls to a captured handshake and MPPE is RC4. Every enabled server or client is a finding; the only question is how reachable it is.

## Rules

Read-only: `print`, `get`, `export`, `monitor` only — never `set`, `add`, `remove`, `enable`, `disable`, `reboot`. Connect with a dedicated least-privilege audit account; do not assume the built-in `read` group is strictly read-only. Audit only the devices that were named. Secrets never enter the report (use `proplist` on areas that store credentials). Method, severity scale (CRITICAL / HIGH / MEDIUM / LOW — in doubt, the lower), output format and the collection order live in `audit-method`; factory values in `factory-defaults`. Read the RouterOS version first: v6 and v7 menus differ, and a command in the wrong menu returns empty.

## Checks

[references/checks.md](references/checks.md)

## Traps

- **A plain `print` of a client shows `password`.** Use the `proplist` in the checks.
- **PPTP has no IPv6 transport.** Measured: a client with an IPv6 `connect-to` is refused (`bad address or dns name`). IPv6 still flows inside the PPP session (measured: DHCPv6-PD bound over PPTP), so the tunnel exposes IPv6 networks too.
- **`use-encryption=required` only forces MPPE.** It does not make PPTP safe.
- **The `pptp` service-port is a NAT helper for clients behind the router**, not a PPTP server.
- PPP profiles, secrets and sessions in general are in `ppp`.
