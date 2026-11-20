# VXLAN checks

Severity scale: CRITICAL / HIGH / MEDIUM / LOW. Every command is read-only. Method and output format: `audit-method`. Measured on RouterOS 7.25.1.

## 1. Overlays

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| VNI or VTEP nobody declared | `/interface vxlan print proplist=name,vni,port,local-address,group,interface,vteps-ip-version,learning,max-fdb-size,vtep-vrf,bridge,disabled,running` + `/interface vxlan vteps print proplist=interface,remote-ip,disabled,dynamic` | an enabled interface, VNI or static VTEP with no comment or outside the inventory | LOW |
| VXLAN without cipher | the two commands above + `/ip ipsec policy print proplist=src-address,dst-address,protocol,dst-port,dynamic` + `/ip ipsec installed-sa print proplist=src-address,dst-address,enc-algorithm,current-bytes` + `/ip address print proplist=address,interface` + `/ipv6 address print proplist=address,interface` | VTEPs across the Internet or another untrusted path with no IPsec SA carrying the VXLAN port between them and no `local-address` on a WireGuard or IPsec-protected interface: the overlay's Layer 2 travels in clear text | HIGH |

## 2. Who can send

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| VXLAN port reachable from any source | `/ip firewall raw print stats where chain=prerouting` + `/ip firewall filter print stats where chain=input` + `/ipv6 firewall raw print stats where chain=prerouting` + `/ipv6 firewall filter print stats where chain=input` | following the rule order, the first rule that matches UDP to the VXLAN `port` from the WAN accepts it from any source address; rules with no `protocol` count, and so does an input chain with no final drop. Anyone who guesses the VNI injects frames. HIGH when the interface is in a bridge with the LAN | MEDIUM |
| MAC learned from a VTEP nobody declared | `/interface vxlan fdb print proplist=interface,remote-ip,mac-address` + the `vteps` command above | an FDB entry whose `remote-ip` is not a static VTEP of the same interface, with no multicast `group` or dynamic VTEP to explain it: a source outside the design is already sending frames | MEDIUM |
| FDB without a tight cap | `/interface vxlan print proplist=name,learning,max-fdb-size` | `learning=yes` with `max-fdb-size` left at 4096 on an interface whose port is reachable from untrusted sources: a flood of fake MACs fills the table | LOW |

## 3. Layer 2 reach

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| VXLAN bridged into the LAN | `/interface bridge port print proplist=bridge,interface,horizon,disabled` + `/interface vxlan print proplist=name,bridge` + `/interface bridge print proplist=name,protocol-mode,dhcp-snooping` | a VXLAN interface, as a bridge port or through its `bridge` property, in the LAN bridge: the remote segment's broadcasts, DHCP servers and RAs reach the local segment. MEDIUM when the remote end is a third party or a client | LOW |
| VXLAN trusted as LAN | `/interface list print proplist=name,include,exclude` + `/interface list member print proplist=list,interface,disabled` + `/ip firewall filter print stats where chain=input` + `/ipv6 firewall filter print stats where chain=input` | the VXLAN interface, or the bridge it is in, part of the `LAN` list (directly or through `include`) or any list the input chain accepts wholesale: the remote segment reaches the router's management services | MEDIUM |
