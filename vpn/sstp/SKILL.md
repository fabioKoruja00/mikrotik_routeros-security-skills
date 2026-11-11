---
name: sstp
description: "Read-only audit of SSTP on MikroTik RouterOS v7: SSTP server certificate, TLS version, ciphers and PFS, client certificate verification, PPP authentication inside the tunnel, SSTP clients that do not verify the server, port 443 sharing, over IPv4 and IPv6 transport. Use this skill whenever a MikroTik device has /interface sstp-server or sstp-client, serves SSTP road-warrior access, or the audit mentions SSTP or TLS VPN on 443, without changing configuration."
---

# SSTP

SSTP is PPP inside TLS. It is only as good as the certificate the server shows and the check the client makes of it, and the client's factory setting makes no check at all.

## Rules

Read-only: `print`, `get`, `export`, `monitor` only — never `set`, `add`, `remove`, `enable`, `disable`, `reboot`. Connect with a dedicated least-privilege audit account; do not assume the built-in `read` group is strictly read-only. Audit only the devices that were named. Secrets never enter the report (use `proplist` on areas that store credentials). Method, severity scale (CRITICAL / HIGH / MEDIUM / LOW — in doubt, the lower), output format and the collection order live in `audit-method`; factory values in `factory-defaults`. Read the RouterOS version first: v6 and v7 menus differ, and a command in the wrong menu returns empty.

## Checks

[references/checks.md](references/checks.md)

## Traps

- **A plain `print` of a client shows `password`.** Use the `proplist` in the checks.
- **The client verifies nothing by default.** Measured: a new client is `verify-server-certificate=no`, `tls-version=any`, `ciphers=aes256-sha`, `pfs=no`, `authentication=pap,chap,mschap1,mschap2`.
- **v7 has no `force-aes`.** The server and client negotiate from `ciphers`; measured: the client default `aes256-sha` made a server offering GCM settle on `AES256-CBC`.
- **The server shows up in `/ip service` as dynamic `sstp-server` on its port** (443 by default, the same port as `www-ssl` and `reverse-proxy`).
- **SSTP runs over IPv6 transport** (measured: client to an IPv6 server address, session up). Read both input chains.
- Certificate validity, key and chain are audited in `certificates`; PPP profiles and secrets in `ppp`.
