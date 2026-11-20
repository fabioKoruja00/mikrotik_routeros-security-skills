---
name: access
description: "Read-only audit of administrative access on MikroTik RouterOS: enabled services and their source restriction (Available From), SSH crypto and forwarding, users, groups, SSH keys and active sessions, Protected RouterBOOT, device-mode flags and downgrade, AAA default group, LCD, mode button, supout files, public graphs, RouterOS version. This skill should be used when assessing who can log into a RouterOS device and with what power, without changing configuration."
---

# Administrative access

Who can reach the device, from where, and with what power. The most common CRITICAL findings live here: an administrative service with an empty `available-from` (`address` up to 7.23; "Available From" in Winbox), the `admin` user still enabled, an orphan SSH key, `device-mode` with `flagged=yes`.

## Rules

Read-only: `print`, `get`, `export`, `monitor` only — never `set`, `add`, `remove`, `enable`, `disable`, `reboot`. Connect with a dedicated least-privilege audit account; do not assume the built-in `read` group is strictly read-only. Audit only the devices that were named. Secrets never enter the report (use `proplist` on areas that store credentials). Method, severity scale (CRITICAL / HIGH / MEDIUM / LOW — in doubt, the lower), output format and the collection order live in `audit-method`; factory values in `factory-defaults`. Read the RouterOS version first: v6 and v7 menus differ, and a command in the wrong menu returns empty.

## Checks

[references/checks.md](references/checks.md)

## Traps

- **Changing the service port is obfuscation**, not a control: `available-from` and a filter are.
- **On 7.24+ `proplist=address` on `/ip service` returns nothing.** The field is `available-from`; `address` survives only as a filter alias. A collection script written for 7.23 reports every service as unrestricted.
- **Do not recommend disabling Winbox** — it is the recovery path. Restrict its source.
- **`device-mode` with `flagged=yes`** is a possible compromise, not a configuration backlog.
- **CCR, IP-only, CAP and switch lines do not ship the default firewall.** "No rules" there is factory-normal; the finding is that nobody built the policy.
