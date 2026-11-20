---
name: user-manager
description: "Read-only audit of the RouterOS v7 User Manager (the built-in RADIUS server): whether it should be on at all, who can reach its RADIUS ports, Message-Authenticator requirement, NAS list and transport, authentication methods each user group accepts, shared users, database location and the PayPal and private web settings. Use this skill whenever a MikroTik device has the user-manager package installed, acts as RADIUS server for PPPoE, hotspot or Wi-Fi, or the audit mentions User Manager, /user-manager or a router answering on UDP 1812/1813, without changing configuration."
---

# User Manager

User Manager turns the router into a RADIUS server. Whoever reaches UDP 1812/1813 and holds, or forges past, a NAS secret can authenticate as any subscriber. The service has no `available-from`: only the input chain limits who talks to it.

## Rules

Read-only: `print`, `get`, `export`, `monitor` only — never `set`, `add`, `remove`, `enable`, `disable`, `reboot`. Connect with a dedicated least-privilege audit account; do not assume the built-in `read` group is strictly read-only. Audit only the devices that were named. Secrets never enter the report (use `proplist` on areas that store credentials). Method, severity scale (CRITICAL / HIGH / MEDIUM / LOW — in doubt, the lower), output format and the collection order live in `audit-method`; factory values in `factory-defaults`. Read the RouterOS version first: v6 and v7 menus differ, and a command in the wrong menu returns empty.

## Checks

[references/checks.md](references/checks.md)

## Traps

- **Never a bare `print` on `/user-manager router`, `/user-manager user` or `/user-manager advanced`.** They hold `shared-secret`, `password`, `otp-secret`, `paypal-password`, `paypal-signature` and `web-private-password`. Use the `proplist` in the checks, `count-only` for presence, `:put [:len [... get <field>]]` on `advanced`. `/user-manager export` only without `show-sensitive`. Secret length is strength analysis, which `audit-method` keeps out of the agent: report presence, never length.
- **Package installed is not server on.** `/system package print` lists `user-manager`; only `enabled=yes` opens the ports.
- **The RADIUS ports show in `/ip service` as dynamic `manager` entries** with no `available-from` to read. Judge exposure from the input chain, not from the service list.
- v6 has a different product under `/tool user-manager`; these checks cover v7 `/user-manager`.
- The router as RADIUS *client* (`/radius`, CoA, `default-group`) is in `aaa`.
</content>
