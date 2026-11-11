# ZeroTier checks

Severity scale: CRITICAL / HIGH / MEDIUM / LOW. Every command is read-only. Method and output format: `audit-method`. Measured on RouterOS 7.25.1.

## 1. Instance and joined networks

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Instance on without a role | `/zerotier print proplist=name,port,disabled,interfaces,route-distance` | an enabled instance nobody declared. `interfaces=all` is the default and only matters as context for the checks below | MEDIUM |
| Routes into internal prefixes | `/ip route print proplist=dst-address,gateway,distance where gateway~"zerotier"` + `/ipv6 route print proplist=dst-address,gateway,distance where gateway~"zerotier"` | two kinds, read both on both stacks: the connected route of the assigned subnet (measured: present with `allow-managed` empty, distance 0) and the `routes` the controller pushes. Either overlapping internal or customer prefixes sends their traffic into the overlay | MEDIUM |
| Network allowed to push a default route | `/zerotier interface print proplist=name,instance,network,network-name,status,allow-default,allow-global,allow-managed,disabled` | `allow-default=yes`: the network controller can steer the router's Internet traffic into the overlay | HIGH |
| Network allowed to push global routes | same command | `allow-global=yes`: the controller can announce routes for public addresses and capture traffic to them | HIGH |
| Network that never joined | same command | `status` other than `OK` (`REQUESTING_CONFIGURATION`, `ACCESS_DENIED`, `NOT_FOUND`) for a long time: a stale or mistyped network ID | LOW |
| Overlay bridged into the LAN | `/interface bridge port print proplist=bridge,interface,disabled where interface~"zerotier"` | a ZeroTier interface as bridge port: every member of the network is on the LAN at layer 2 | HIGH |
| Overlay trusted as LAN | `/interface list member print proplist=list,interface,disabled where interface~"zerotier"` + `/ip firewall filter print stats where chain=input` + `/ipv6 firewall filter print stats where chain=input` | the ZeroTier interface in the `LAN` list (or any list the input chain accepts wholesale) on either stack: every member gets what the LAN gets. HIGH when that includes the router's management services | MEDIUM |
| IPv6 from the overlay unfiltered | `/ipv6 address print proplist=address,interface where interface~"zerotier"` + `/ipv6 firewall filter print stats where chain=input` | a ZeroTier interface holding a ULA or 6PLANE address (measured: `ip6-rfc4193=yes` gives members a `/88` flagged global) on a device whose IPv6 input chain does not cover it: members reach the router over IPv6 even when IPv4 is filtered | HIGH |

## 2. Networks this router controls

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Public network | `/zerotier controller print proplist=name,instance,network,private,ip-range,ip6-rfc4193,ip6-6plane,ip6-range,routes,broadcast,disabled` | `private=no`: any node that knows the network ID is authorized and gets an address from `ip-range`, and an IPv6 one when `ip6-rfc4193`, `ip6-6plane` or `ip6-range` is set (measured: a node joined and received both without approval) | HIGH |
| Members nobody recognises | `/zerotier controller member print proplist=name,network,zt-address,authorized,ip-address,bridge,disabled` | `authorized=yes` members with no name or comment that maps to a known device | MEDIUM |
| Member allowed to bridge | same command | `bridge=yes` on a member that is not a planned bridge: it can inject traffic from a whole network behind it | MEDIUM |

## 3. Evidence

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Direct peers | `/zerotier peer print proplist=instance,zt-address,role,latency,path` | `LEAF` peers with direct paths nobody recognises; `PLANET` peers are ZeroTier's roots | LOW |
</content>
