# Tunnels and cryptography checks

Severity scale: CRITICAL / HIGH / MEDIUM / LOW. Every command is read-only. Method and output format: `audit-method`.

## 1. Cryptography, certificates and tunnels

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Remote access created by the wizard | `/interface l2tp-server server print proplist=enabled,use-ipsec,authentication,default-profile` + `/ppp secret print proplist=name,profile,service` + named `/ip cloud get` fields only (settings menu: no `proplist`; no secret-bearing field) | VPN raised by Quick Set: generic user, and the wizard itself leaves **`Firewall Router` unchecked** | CRITICAL |
| Home tunnel with the admin credential | `:put [/ip cloud get back-to-home-vpn]`, `/interface wireguard peers print proplist=interface,public-key,allowed-address,endpoint-address,endpoint-port,disabled`, `/user print proplist=name,group` | Back To Home active: the app asks for **the administrative credential** to build the tunnel, and offers file access in full mode. Unknown peer in the list = remote access nobody registered | HIGH |
| Tunnel credential handling | inspect only non-secret metadata with explicit `proplist` fields | assess whether the tunnel uses an approved authentication mechanism and protected transport. Secret strength is intentionally not assessed; never retrieve PSKs/passwords | MEDIUM |
| Certificate | `/certificate print proplist=name,common-name,subject-alt-name,issuer,serial-number,fingerprint,invalid-before,invalid-after,expired,revoked,trusted,private-key,key-type,key-size,digest-algorithm` | detailed in `certificates` | HIGH |
