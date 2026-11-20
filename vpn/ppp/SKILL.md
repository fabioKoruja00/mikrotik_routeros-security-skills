---
name: ppp
description: "Read-only audit of the PPP layer shared by every MikroTik RouterOS tunnel and PPPoE: PPP profiles (encryption, only-one, interface lists, on-up/on-down scripts), local PPP secrets and the services they open, /ppp aaa accounting and RADIUS use, and active sessions by service. Use this skill whenever a MikroTik device has /ppp secret or custom /ppp profile entries, terminates PPTP, L2TP, SSTP, OVPN or PPPoE, or the audit mentions PPP profiles, secrets or accounting, without changing configuration."
---

# PPP

Every PPP-based tunnel inherits its rules from a profile and admits whoever holds a secret. A secret left on `service=any` opens every PPP service on the router, and a profile left on `default` neither encrypts nor limits sessions.

## Rules

Read-only: `print`, `get`, `export`, `monitor` only — never `set`, `add`, `remove`, `enable`, `disable`, `reboot`. Connect with a dedicated least-privilege audit account; do not assume the built-in `read` group is strictly read-only. Audit only the devices that were named. Secrets never enter the report (use `proplist` on areas that store credentials). Method, severity scale (CRITICAL / HIGH / MEDIUM / LOW — in doubt, the lower), output format and the collection order live in `audit-method`; factory values in `factory-defaults`. Read the RouterOS version first: v6 and v7 menus differ, and a command in the wrong menu returns empty.

## Checks

[references/checks.md](references/checks.md)

## Traps

- **Never `print detail` or `export show-sensitive` on `/ppp secret`.** It holds `password`. Use the `proplist` in the checks; password strength is not assessed.
- **Factory values, measured.** Profile `default`: `use-encryption=default`, `only-one=default`, no `interface-list`. Profile `default-encryption`: `use-encryption=yes`. A new secret: `service=any`, `profile=default`. `/ppp aaa`: `use-radius=no`, `accounting=yes`, `interim-update=0s`, `enable-ipv6-accounting=no`.
- **`on-up` and `on-down` run as scripts on every session.** Read their content only through `scripts-storage` rules; here, presence is the finding.
- **With RADIUS, local secrets stop being the policy but are still checked first.** `only-one` on the effective profile keeps applying to RADIUS users. Read `/ppp aaa` before judging secrets.
- Service-specific settings are in `pptp`, `l2tp`, `sstp`, `ovpn` and `pppoe`; RADIUS servers in `aaa`.
