# EoIP checks

Severity scale: CRITICAL / HIGH / MEDIUM / LOW. Every command is read-only. Method and output format: `audit-method`. Measured on RouterOS 7.25.1.

## 1. Tunnels

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Tunnel nobody declared | `/interface eoip print proplist=name,local-address,remote-address,tunnel-id,keepalive,allow-fast-path,disabled,running` + `/interface eoipv6 print proplist=name,local-address,remote-address,tunnel-id,keepalive,disabled,running` | an enabled tunnel with no comment or known purpose, or a `remote-address` outside the inventory | LOW |
| Tunnel without cipher | `/interface eoip print count-only where ipsec-secret=""` + `/interface eoipv6 print count-only where ipsec-secret=""` + `/ip ipsec policy print proplist=src-address,dst-address,protocol,ph2-state,dynamic where dynamic and (protocol=gre or protocol=etherip)` | tunnel over the Internet or another untrusted path with no `ipsec-secret` (count above zero) and no dynamic policy for it (`protocol` `gre` for EoIP, `etherip` for EoIPv6), and no other protected transport under it: an L2 bridge open to whoever forges the endpoint and guesses the `tunnel-id`. The secret is never printed | HIGH |
| Weak cipher on the dynamic peer | `/ip ipsec proposal print proplist=name,auth-algorithms,enc-algorithms,pfs-group` | an encrypted tunnel relies on the `default` proposal still at factory values (`sha1`, CBC, `modp1024`); detail and severity of the algorithms in `vpn/ipsec` | MEDIUM |
| Keepalive turned off | `/interface eoip print count-only where !keepalive` + `/interface eoipv6 print count-only where !keepalive` | tunnels without keepalive: the running flag stays up with the peer dead or a mismatched `tunnel-id`, so monitoring and failover trust a dead link | LOW |
| IPsec left after the secret | `/ip ipsec policy print proplist=src-address,dst-address,protocol,ph2-state,dynamic where dynamic and (protocol=gre or protocol=etherip)` + the `count-only where ipsec-secret=""` above | a dynamic `gre`/`etherip` policy whose tunnel has an empty `ipsec-secret`: the configuration no longer shows the IPsec the router still runs | LOW |

## 2. Layer 2 reach

Tunnel names come from `/interface eoip print proplist=name` and `/interface eoipv6 print proplist=name`; a filter on the string `eoip` misses renamed tunnels and VLANs built on top of them.

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Tunnel bridged into the LAN | `/interface bridge port print proplist=bridge,interface,horizon,disabled` + `/interface vlan print proplist=name,interface,vlan-id` + `/interface bridge print proplist=name,protocol-mode,dhcp-snooping` | a tunnel, or a VLAN on it, in the LAN bridge carries the remote site's broadcasts, DHCP servers and RAs into the local segment. MEDIUM when the remote end is a third party or a client, or when `protocol-mode=none` leaves loops between several bridged tunnels unbroken | LOW |
| Tunnel trusted as LAN | `/interface list member print proplist=list,interface,disabled` + `/ip firewall filter print stats where chain=input` + `/ipv6 firewall filter print stats where chain=input` | the tunnel, or the bridge it is a port of, in the `LAN` list or any list the input chain accepts wholesale (`accept in-interface-list=LAN`, `drop in-interface-list=!LAN`): the remote site reaches the router's management services | MEDIUM |
| Duplicate tunnel MAC | `/interface eoip print proplist=name,mac-address` + `/interface eoipv6 print proplist=name,mac-address` + the bridge ports above | two tunnels in the same bridge with the same MAC: MAC learning and spanning tree misbehave (flapping, duplicates). A configuration copied between routers repeats the MAC at the far end too; compare both ends when the inventory has them | LOW |

## 3. Firewall

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Transport accepted from any source | `/ip firewall filter print stats where chain=input and protocol=gre` + `/ipv6 firewall filter print stats where chain=input and protocol=etherip` + `/ip firewall filter print stats where chain=input` | following the rule order, the first rule that matches GRE (EoIP) or `etherip` (EoIPv6) arriving from the WAN accepts it from any source address, on a router with a tunnel without cipher: frames from anyone who guesses the `tunnel-id` enter the bridge. Rules with no `protocol` count, and so does an input chain with no final drop | MEDIUM |
