# IPIP checks

Severity scale: CRITICAL / HIGH / MEDIUM / LOW. Every command is read-only. Method and output format: `audit-method`. Measured on RouterOS 7.25.2.

## 1. Tunnels

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Tunnel nobody declared | `/interface ipip print proplist=name,local-address,remote-address,keepalive,allow-fast-path,disabled,running,comment` + `/interface ipipv6 print proplist=name,local-address,remote-address,keepalive,disabled,running,comment` | an enabled tunnel with no comment or known purpose, or a `remote-address` outside the inventory | LOW |
| Tunnel without cipher | `/interface ipip print count-only where ipsec-secret="" or !ipsec-secret` + `/interface ipipv6 print count-only where ipsec-secret="" or !ipsec-secret` + `/ip ipsec policy print proplist=src-address,dst-address,protocol,ph2-state,dynamic` | tunnel over the Internet or another untrusted path with no secret (count above zero), no policy, dynamic or static, covering its endpoints, and no other protected transport under it: anyone who forges the `remote-address` gets packets decapsulated and routed by the router. The secret is never printed | HIGH |
| Weak cipher on the dynamic peer | `/ip ipsec proposal print proplist=name,auth-algorithms,enc-algorithms,pfs-group` + `/ip ipsec profile print proplist=name,hash-algorithm,enc-algorithm,dh-group` | an encrypted tunnel relies on a `default` proposal and profile that still allow legacy parameters (`sha1`, CBC, `modp1024`); detail and severity of the algorithms in `vpn/ipsec` | MEDIUM |
| Keepalive turned off | `/interface ipip print count-only where !keepalive` + `/interface ipipv6 print count-only where !keepalive` | tunnels without keepalive: the running flag stays up with the peer dead or a secret on one end only, so monitoring and failover trust a dead link | LOW |
| IPsec left after the secret | `/ip ipsec policy print proplist=src-address,dst-address,protocol,ph2-state,dynamic where dynamic and (protocol=ipencap or protocol=ipv6-encap)` + `/interface ipip print count-only where ipsec-secret="" or !ipsec-secret` + the same on `ipipv6` | a dynamic `ipencap`/`ipv6-encap` policy whose tunnel has no secret: the peer and policy persist although the tunnel configuration shows no IPsec | LOW |

## 2. Reach

Tunnel names come from `/interface ipip print proplist=name` and `/interface ipipv6 print proplist=name`; a filter on the string `ipip` misses renamed tunnels.

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Tunnel trusted as LAN | `/interface list member print proplist=list,interface,disabled` + `/ip firewall filter print stats where chain=input` + `/ipv6 firewall filter print stats where chain=input` | the tunnel in the `LAN` list or any list the input chain accepts wholesale (`accept in-interface-list=LAN`, `drop in-interface-list=!LAN`): the remote site, or whoever forges it on a tunnel without cipher, reaches the router's management services | MEDIUM |
| MSS and MTU | `/interface ipip print proplist=name,mtu,actual-mtu,clamp-tcp-mss` + `/interface ipipv6 print proplist=name,mtu,actual-mtu,clamp-tcp-mss` | `clamp-tcp-mss=no` with the reduced MTU: TCP sessions through the tunnel hang on large packets wherever PMTUD is blocked. IPv6 carried in a tunnel with `actual-mtu` below 1280 breaks outright | LOW |

## 3. Firewall

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Transport accepted from any source | `/ip firewall filter print stats where chain=input and protocol=ipencap` + `/ipv6 firewall filter print stats where chain=input and (protocol=ipencap or protocol=ipv6-encap)` + `/ip firewall filter print stats where chain=input` + `/ipv6 firewall filter print stats where chain=input` | following the rule order, the first rule that matches IP-in-IP arriving from the WAN accepts it from any source address, on a router with a tunnel without cipher. Rules with no `protocol` count, and so does an input chain with no final drop | MEDIUM |
