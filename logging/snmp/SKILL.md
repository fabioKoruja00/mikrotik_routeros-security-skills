---
name: snmp
description: "Read-only audit of SNMP on MikroTik RouterOS: default communities, communities open to any source, write access, v1/v2c on untrusted networks, weak SNMPv3 algorithms, readable sensitive OIDs, unprotected traps and traps to the wrong collector. Use this skill whenever a MikroTik device has /snmp enabled or communities configured, is polled by a monitoring system, or the audit mentions SNMP, communities, traps or MIB exposure, without changing configuration."
---

# SNMP

SNMP answers with interfaces, addresses, clients, traffic and topology to anyone who holds the community and reaches UDP 161. With write access, the community is administrative access.

## Rules

Read-only: `print`, `get`, `export`, `monitor` only — never `set`, `add`, `remove`, `enable`, `disable`, `reboot`. Connect with a dedicated least-privilege audit account; do not assume the built-in `read` group is strictly read-only. Audit only the devices that were named. Secrets never enter the report (use `proplist` on areas that store credentials). Method, severity scale (CRITICAL / HIGH / MEDIUM / LOW — in doubt, the lower), output format and the collection order live in `audit-method`; factory values in `factory-defaults`. Read the RouterOS version first: v6 and v7 menus differ, and a command in the wrong menu returns empty.

## Checks

[references/checks.md](references/checks.md)

## Traps

- **The community `name` is the credential** on v1/v2c (on v3 it is the user name). Test the well-known names with `count-only where name=...`; never print the field. `/snmp` is a settings menu: read named fields with `get`, because a plain `print` shows `trap-community`.
- **The log records the community name in clear text** when a community is added or changed (`system,info`), and so does any remote syslog that receives that topic. SNMPv3 passwords are left out.
- **The factory community `public` is enabled with `addresses=::/0`.** While `/snmp` stays `enabled=no` nothing answers; turning SNMP on opens it to every source the firewall lets through.
- **An empty field is not shown by `print proplist`.** `denied-oids` exists on 7.25.1 but its column disappears when empty; use `count-only where !denied-oids`.
- **`write-access=yes` with a default community lets anyone who reaches UDP 161 write every writable OID.**
