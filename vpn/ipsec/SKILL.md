---
name: ipsec
description: "Read-only audit of IPsec on MikroTik RouterOS: weak phase 1 profiles and phase 2 proposals (including the factory defaults), IKEv1 main and aggressive mode, peers open to any address with a shared PSK, generate-policy templates, policies at level use, mode-config, NAT and firewall interaction, live peers and installed SAs, over IPv4 and IPv6. Use this skill whenever a MikroTik device has /ip ipsec peers, identities or policies, terminates site-to-site or road-warrior IPsec, or the audit mentions IKE, IKEv2, ESP, PFS or L2TP/IPsec, without changing configuration."
---

# IPsec

IPsec is as strong as the weakest proposal both sides accept and as private as the key that admits a peer. The factory `default` profile and proposal still offer SHA-1, 3DES and MODP1024, and a new peer is IKEv1 `main` unless told otherwise.

## Rules

Read-only: `print`, `get`, `export`, `monitor` only — never `set`, `add`, `remove`, `enable`, `disable`, `reboot`. Connect with a dedicated least-privilege audit account; do not assume the built-in `read` group is strictly read-only. Audit only the devices that were named. Secrets never enter the report (use `proplist` on areas that store credentials). Method, severity scale (CRITICAL / HIGH / MEDIUM / LOW — in doubt, the lower), output format and the collection order live in `audit-method`; factory values in `factory-defaults`. Read the RouterOS version first: v6 and v7 menus differ, and a command in the wrong menu returns empty.

## Checks

[references/checks.md](references/checks.md)

## Traps

- **Never `print detail` on `identity` or `installed-sa`.** Identities hold `secret`, `password` and keys; installed SAs hold `enc-key` and `auth-key` (measured: 64 hex characters each). Use the `proplist` in the checks and `count-only` for presence.
- **A peer or policy that names no profile or proposal uses `default`.** Reading only the custom entries hides the weak factory set that is actually negotiated.
- **`proposal-check=obey` lets the initiator pick the lifetime.** It is the default and only matters together with long lifetimes.
- **Masquerade without an exception swallows the tunnel traffic** before the policy sees it. The fix is an accept, or `ipsec-policy=out,none` on the masquerade, ahead of it — audited in `firewall`.
- **The factory IPv6 input chain accepts IKE, ESP and AH from anywhere** even with no IPsec configured. Count it on devices that never terminate IPsec.
- L2TP/IPsec servers are in `l2tp`; certificates used by identities are in `certificates`.
