---
name: dude
description: "Read-only audit of The Dude server running on MikroTik RouterOS v7 (the dude package): whether the server should run at all, who can reach it through Winbox, its built-in syslog listener, where its database lives, database exports left on disk and remote agents. Use this skill whenever a MikroTik device has the dude package installed, runs The Dude as network monitor, or the audit mentions The Dude, /dude, dude.db or a router answering syslog on UDP 514, without changing configuration."
---

# The Dude

The Dude server holds the network map plus the credentials it uses to log in to every monitored device. Whoever opens it through the Winbox port with a valid login inherits that reach. Turning it on also opens a syslog listener on UDP 514 with no `available-from`.

## Rules

Read-only: `print`, `get`, `export`, `monitor` only — never `set`, `add`, `remove`, `enable`, `disable`, `reboot`. Connect with a dedicated least-privilege audit account; do not assume the built-in `read` group is strictly read-only. Audit only the devices that were named. Secrets never enter the report (use `proplist` on areas that store credentials). Method, severity scale (CRITICAL / HIGH / MEDIUM / LOW — in doubt, the lower), output format and the collection order live in `audit-method`; factory values in `factory-defaults`. Read the RouterOS version first: v6 and v7 menus differ, and a command in the wrong menu returns empty.

## Checks

[references/checks.md](references/checks.md)

## Traps

- **Never run `/dude export-db`, open `dude.db` or read an exported database.** They carry the credentials the server uses on every monitored device. Judge files by name, size and date only.
- **Package installed is not server running.** `/system package print` lists `dude`; only `enabled=yes` with `status=running` serves clients and syslog.
- **The syslog listener shows in `/ip service` as dynamic `dudes` on UDP 514** with no `available-from`. Judge it from the input chain.
- **The CLI shows only names under `/dude device`, `agent`, `probe` and `notification`.** Devices, credentials and notification actions are managed from the Dude client; the audit sees the inventory, not the settings.
- Who may log in through Winbox is in `access`.
</content>
