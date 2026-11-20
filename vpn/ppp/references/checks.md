# PPP checks

Severity scale: CRITICAL / HIGH / MEDIUM / LOW. Every command is read-only. Method and output format: `audit-method`. Measured on RouterOS 7.25.1.

## 1. Profiles

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Tunnel profile not requiring encryption | `/ppp profile print proplist=name,use-encryption,only-one,interface-list,local-address,remote-address,remote-ipv6-prefix-pool,dhcpv6-pd-pool,on-up,on-down` | the effective profile (the secret's `profile`, else the server's `default-profile`, or the one RADIUS returns) of PPTP or L2TP sessions with `use-encryption` other than `required` (`default` inherits `no` on the factory `default` profile): the PPP payload can cross in clear. `required` is not a mitigation for PPTP itself (see `pptp`). Not a finding for PPPoE, SSTP, OVPN, or L2TP whose server has `use-ipsec=required` | MEDIUM |
| Credential usable on several sessions | same command | `only-one` other than `yes` on a profile used by personal credentials (both factory profiles are `default`) | MEDIUM |
| PPP interfaces outside the firewall lists | same command | context: `interface-list` empty, so dynamic PPP interfaces fall into no list and the firewall cannot scope them except by `all-ppp` | LOW |
| Scripts on session start or stop | same command | `on-up` or `on-down` set: a script runs on every session, a persistence point — trace its origin in `scripts-storage` | MEDIUM |

## 2. Secrets

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Secret valid on every PPP service | `/ppp secret print count-only where service=any` + `/ppp secret print proplist=name,service,profile,local-address,remote-address,routes,disabled where service=any` | secrets on `service=any` (the default): a credential meant for PPPoE or L2TP/IPsec also opens PPTP or any other PPP server enabled later | MEDIUM |
| Routes handed out per secret | `/ppp secret print proplist=name,service,profile,routes,ipv6-routes,disabled where routes!="" or ipv6-routes!=""` | context: secrets that install routes on the router at login; a wrong one steers internal traffic to a client | LOW |
| Local secrets beside RADIUS | `/ppp aaa print` + `/ppp secret print count-only where !disabled` | `use-radius=yes` and enabled local secrets: local secrets are checked before RADIUS, so a forgotten local credential bypasses the RADIUS policy | MEDIUM |

## 3. Accounting and evidence

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| No interim accounting | `/ppp aaa print` | `use-radius=yes` with `interim-update=0s` (default): sessions leave no periodic trail for identifying a subscriber by address and time. `enable-ipv6-accounting=no` counts only when sessions get IPv6 (`dhcpv6-pd-pool` or `remote-ipv6-prefix-pool` on a profile) | LOW |
| Sessions by service | `/ppp active print count-only`, then `/ppp active print count-only where service=pptp` and the same for `l2tp`, `sstp`, `ovpn`, `pppoe` | context: which PPP services actually carry sessions, to weigh the service skills' findings | LOW |
