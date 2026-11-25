# Back To Home and Quick Set checks

Severity scale: CRITICAL / HIGH / MEDIUM / LOW. Every command is read-only. Method and output format: `audit-method`. Measured on RouterOS 7.25.2.

## 1. Back To Home

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| BTH enabled without approval | `:put [/ip cloud get back-to-home-vpn]` + `:put [/ip cloud get vpn-status]` + `:put [/ip cloud get vpn-port]` + `:put [/ip cloud get vpn-interface]` | `back-to-home-vpn=enabled` on a device where nobody approved remote access: a WireGuard server reachable through MikroTik's relay, with its port accepted from any source | HIGH |
| BTH user or peer nobody registered | `/ip cloud back-to-home-user print proplist=name,allow-lan,client-allowed-address,file-access,file-access-path,expires,disabled,comment` + `/interface wireguard peers print proplist=interface,name,allowed-address,disabled where interface=back-to-home-vpn` | a user outside the inventory, or a peer on the BTH interface beyond the router-level one and the users listed: remote access nobody registered | HIGH |
| BTH user without expiry | `/ip cloud back-to-home-user print count-only where !expires` | permanent users: a lost phone keeps its tunnel | LOW |
| BTH user with LAN access | `/ip cloud back-to-home-user print count-only where allow-lan=yes` | users with `allow-lan=yes` reach every network the router reaches. With `allow-lan=no`, check the input chain and the networks outside the `LAN` list before calling it contained | MEDIUM |
| Files shared over BTH | `/ip cloud back-to-home-user print count-only where file-access!=disabled` + `/ip cloud back-to-home-file print proplist=path,allow-uploads,expires,disabled` | file access on a user, or a shared path with `allow-uploads=yes` or no expiry: the router's storage reachable from the tunnel | MEDIUM |
| Rules injected by BTH | `/ip firewall filter print detail where dynamic` + `/ip firewall nat print detail where dynamic` + `/ipv6 firewall filter print detail where dynamic` | dynamic rules with comment `back-to-home-vpn` on a device where BTH should be off: the service is running whatever the static rules say | HIGH |

## 2. Quick Set and tunnel credentials

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Remote access created by the wizard | `:put [/interface l2tp-server server get <field>]` for `enabled`, `use-ipsec`, `authentication`, `default-profile` + the same `get enabled` on `pptp-server` and `sstp-server` + `/ppp secret print proplist=name,profile,service,disabled` | VPN raised by Quick Set: a server enabled with a generic user (the wizard names it `vpn`), and the wizard itself leaves **`Firewall Router` unchecked** | CRITICAL |
| Tunnel credential handling | non-secret metadata only, with explicit `proplist` fields or `get` on named fields | assess whether the tunnel uses an approved authentication mechanism and protected transport. Secret strength is intentionally not assessed; never retrieve PSKs or passwords | MEDIUM |
