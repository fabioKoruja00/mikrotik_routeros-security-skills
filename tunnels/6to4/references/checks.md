# 6to4 checks

Severity scale: CRITICAL / HIGH / MEDIUM / LOW. Every command is read-only. Method and output format: `audit-method`. Measured on RouterOS 7.25.2.

## 1. Tunnels

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Tunnel nobody declared | `/interface 6to4 print proplist=name,local-address,remote-address,disabled,running,comment` | an enabled tunnel with no comment or known purpose, or a `remote-address` outside the inventory | LOW |
| Tunnel open to any source | `/interface 6to4 print count-only where remote-address=unspecified and !disabled` + `/ip firewall raw print stats where chain=prerouting` + `/ip firewall filter print stats where chain=input` | an enabled tunnel with `remote-address` unspecified and no rule in `raw` or `filter` dropping protocol 41 from sources other than the expected peers before any accept, `established` included: any IPv4 host injects forged IPv6 through the router (measured: decapsulated from a host not configured as remote) | HIGH |
| 6to4 relay without need | the first command + `/ipv6 route print proplist=dst-address,gateway,active where dst-address=::/0 or dst-address=2002::/16` | `remote-address` unspecified and a route to `::/0` or `2002::/16` through that tunnel on a network with native IPv6 or a broker: traffic goes through third-party relays nobody controls. Reported with the check above, not twice | MEDIUM |
| Tunnel without cipher | `/interface 6to4 print count-only where ipsec-secret="" or !ipsec-secret` + `/ip ipsec policy print proplist=src-address,dst-address,protocol,ph2-state,dynamic` | a tunnel between own sites with no secret (count above zero), no policy, dynamic or static, covering its endpoints, and no other protected transport under it. Not a finding for a public broker (its `remote-address` is the broker's server), which offers no IPsec. The secret is never printed | MEDIUM |
| IPsec not required on input | `/interface 6to4 print proplist=name,remote-address` minus `/interface 6to4 print proplist=name,remote-address where ipsec-secret="" or !ipsec-secret` (the rest have a secret) + `/ip firewall raw print stats where chain=prerouting` + `/ip firewall filter print stats where chain=input` | tunnels with a secret and no rule dropping protocol 41 from the remote with `ipsec-policy=in,none` before any accept, `established` included: plaintext forged with the remote's source is decapsulated although the tunnel is encrypted (measured) | MEDIUM |
| Keepalive on one end only | `/interface 6to4 print proplist=name,remote-address,keepalive,running` on both ends | keepalive set on one end and not on the other: that end stays not running and the tunnel carries nothing. No keepalive on either end (the default): the running flag stays up with the remote dead, so monitoring and failover trust a dead link | LOW |
| IPsec broken or left behind | `/ip ipsec policy print proplist=src-address,dst-address,protocol,ph2-state,dynamic where dynamic and protocol=ipv6-encap` + the `count-only` of the secret above | a dynamic `ipv6-encap` policy from `0.0.0.0/0` to `0.0.0.0/0` (secret on a tunnel with `remote-address` unspecified), one stuck in `ready-to-send` (secret differs or is missing on the remote), or one whose tunnel has no secret any more. IPIPv6 policies also use `ipv6-encap`; match by endpoints | LOW |

## 2. Reach

Tunnel names come from `/interface 6to4 print proplist=name`; a filter on the string `6to4` misses renamed tunnels.

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Tunnel trusted as LAN | `/interface list member print proplist=list,interface,disabled` + `/ipv6 firewall filter print stats where chain=input` + `/ipv6 firewall filter print stats where chain=forward` | the tunnel in the `LAN` list or any list the IPv6 input or forward chain accepts wholesale: the remote, or whoever forges it, reaches the router and the LAN over IPv6 | MEDIUM |
| MSS and MTU | `/interface 6to4 print proplist=name,mtu,actual-mtu,clamp-tcp-mss` | `actual-mtu` below 1280 breaks IPv6 outright (`mtu=auto` is the default; 1480 is normal, 1438 with an established SA; 65516 with a secret and no SA proves nothing); `clamp-tcp-mss=no` with the reduced MTU hangs TCP sessions wherever PMTUD is blocked | LOW |
