---
name: ovpn
description: "Read-only audit of OpenVPN (OVPN) on MikroTik RouterOS v7: OVPN servers (one or more) and clients, server and client certificates, auth and cipher lists, TLS version, user authentication method, client verification of the server, pushed routes and bridging mode, over IPv4 and IPv6 transport. Use this skill whenever a MikroTik device has /interface ovpn-server server entries or ovpn-client interfaces, or the audit mentions OpenVPN, OVPN or port 1194, without changing configuration."
---

# OVPN

The factory OVPN settings belong to another decade: a new server accepts `md5` and Blowfish and asks no client certificate, and a new client trusts any server with Blowfish and SHA-1. Every value worth auditing is one somebody had to change.

## Rules

Read-only: `print`, `get`, `export`, `monitor` only — never `set`, `add`, `remove`, `enable`, `disable`, `reboot`. Connect with a dedicated least-privilege audit account; do not assume the built-in `read` group is strictly read-only. Audit only the devices that were named. Secrets never enter the report (use `proplist` on areas that store credentials). Method, severity scale (CRITICAL / HIGH / MEDIUM / LOW — in doubt, the lower), output format and the collection order live in `audit-method`; factory values in `factory-defaults`. Read the RouterOS version first: v6 and v7 menus differ, and a command in the wrong menu returns empty.

## Checks

[references/checks.md](references/checks.md)

## Traps

- **A plain `print` of a client shows `password`.** Use the `proplist` in the checks.
- **On 7.25 the server menu is a list.** `/interface ovpn-server server` holds named entries, each with its own port, certificate and algorithms; an empty print means no server, not defaults.
- **Factory values, measured.** A new server: `protocol=tcp`, `certificate` none, `require-client-certificate=no`, `tls-version=any`, `auth=sha1,md5,sha256,sha512`, `cipher=blowfish128,aes128-cbc`, `user-auth-method=pap`. A new client: `auth=sha1`, `cipher=blowfish128`, `tls-version=any`, `verify-server-certificate=no`, `use-peer-dns=yes`.
- **Each server shows in `/ip service` as dynamic `ovpn-server`** on its port and protocol.
- **OVPN runs over IPv6 transport** (measured: UDP to an IPv6 server address, certificate verified, session up). Read both input chains.
- **A `remote-ipv6-prefix-pool` on the server's PPP profile stalled the session at `link established`**, and a RouterOS client takes no pushed IPv6 address. A tunnel that will not come up is not proof of a firewall block.
- Certificate validity, key and chain are in `certificates`; PPP profiles and secrets in `ppp`.
