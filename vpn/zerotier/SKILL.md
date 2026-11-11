---
name: zerotier
description: "Read-only audit of ZeroTier on MikroTik RouterOS v7 (zerotier package): the instance and the interfaces it listens on, networks joined and the routes they may push, ZeroTier interfaces bridged or trusted as LAN, and networks this router controls — public networks, authorized members, bridging members. Use this skill whenever a MikroTik device has the zerotier package, joins or controls a ZeroTier network, or the audit mentions ZeroTier, /zerotier or UDP 9993, without changing configuration."
---

# ZeroTier

A ZeroTier network is a LAN that spans the Internet. Whoever is authorized on it reaches the router as if plugged into the bridge, and a network the router controls with `private=no` authorizes anyone who learns its 16-digit ID.

## Rules

Read-only: `print`, `get`, `export`, `monitor` only — never `set`, `add`, `remove`, `enable`, `disable`, `reboot`. Connect with a dedicated least-privilege audit account; do not assume the built-in `read` group is strictly read-only. Audit only the devices that were named. Secrets never enter the report (use `proplist` on areas that store credentials). Method, severity scale (CRITICAL / HIGH / MEDIUM / LOW — in doubt, the lower), output format and the collection order live in `audit-method`; factory values in `factory-defaults`. Read the RouterOS version first: v6 and v7 menus differ, and a command in the wrong menu returns empty.

## Checks

[references/checks.md](references/checks.md)

## Traps

- **The instance holds the node's private `identity`.** No bare `print`, `print detail` or `export show-sensitive` on `/zerotier`; use the `proplist` in the checks.
- **Empty `allow-default`, `allow-global` and `allow-managed` mean the ZeroTier defaults**, not "off": treat an explicit `yes` as the operator's choice.
- **The package needs `device-mode` `zerotier=yes`.** On `home` and `basic` modes the menu is absent even with the package installed.
- **The UDP port shows in `/ip service` as dynamic `zerotier-one`.** ZeroTier needs it to reach its roots; exposure is about what the overlay reaches, not the port itself.
- The instance contacts ZeroTier's root servers on the Internet (`role=PLANET` peers). That is normal; a controller-only router still depends on them.
</content>
