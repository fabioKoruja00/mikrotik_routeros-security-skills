# Tunnels and cryptography checks

Severity scale: CRITICAL / HIGH / MEDIUM / LOW. Every command is read-only. Method and output format: `audit-method`.

## 1. Cryptography, certificates and tunnels

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Cleartext PPP authentication | `/interface <type>-server server print proplist=enabled,authentication` and `/ppp aaa print` | `authentication` accepting `pap`, or `mschap1` without need, on a tunnel whose transport is not protected (PAP inside `use-ipsec=required` is protected) | HIGH |
| PPP session without cipher | `/ppp active print count-only` first, then a narrow `where` (never a full listing on a concentrator with thousands of sessions) | **empty** encoding on a PPTP/L2TP/SSTP session that should be encrypted. Empty encoding on PPPoE is normal | HIGH |
| PPP profile not requiring cipher | `/ppp profile print proplist=name,use-encryption,only-one,local-address,remote-address,disabled` | on profiles used by tunnels across an untrusted network: effective `use-encryption` other than `required` (`default` inherits — resolve it). Not applicable to PPPoE access | HIGH |
| Remote access created by the wizard | `/interface l2tp-server server print proplist=enabled,use-ipsec,authentication,default-profile` + `/ppp secret print proplist=name,profile,service` + named `/ip cloud get` fields only (settings menu: no `proplist`; no secret-bearing field) | VPN raised by Quick Set: generic user, and the wizard itself leaves **`Firewall Router` unchecked** | CRITICAL |
| Home tunnel with the admin credential | `:put [/ip cloud get back-to-home-vpn]`, `/interface wireguard peers print proplist=interface,public-key,allowed-address,endpoint-address,endpoint-port,disabled`, `/user print proplist=name,group` | Back To Home active: the app asks for **the administrative credential** to build the tunnel, and offers file access in full mode. Unknown peer in the list = remote access nobody registered | HIGH |
| Tunnel credential handling | inspect only non-secret metadata with explicit `proplist` fields | assess whether the tunnel uses an approved authentication mechanism and protected transport. Secret strength is intentionally not assessed; never retrieve PSKs/passwords | MEDIUM |
| VXLAN without cipher | `/interface vxlan print proplist=name,vni,port,local-address,disabled`, `/interface vxlan vteps print` and `/interface vxlan fdb print` | VXLAN crossing an untrusted underlay without IPsec/WireGuard underneath (customer L2 in cleartext), VTEP/VNI outside the inventory, or udp/4789 reachable from any source | HIGH |
| EoIP protection | `/interface eoip print proplist=name,remote-address,tunnel-id,disabled` + `/interface eoip print count-only where ipsec-secret=""` | tunnel over the Internet with no `ipsec-secret` (count above zero) and no other protected transport under it: an L2 bridge open to whoever forges the endpoint. The secret is never printed | HIGH |
| Badly built port knocking | `/ip firewall filter print detail` and `/ip firewall address-list print` | knock sequence without `address-list-timeout`, or the final port drop missing — the knock protects nothing | MEDIUM |
| Certificate | `/certificate print proplist=name,common-name,subject-alt-name,issuer,serial-number,fingerprint,invalid-before,invalid-after,expired,revoked,trusted,private-key,key-type,key-size,digest-algorithm` | detailed in `certificates` | HIGH |
