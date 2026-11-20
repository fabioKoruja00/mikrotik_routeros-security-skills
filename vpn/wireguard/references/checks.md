# WireGuard checks

Severity scale: CRITICAL / HIGH / MEDIUM / LOW. Every command is read-only. Method and output format: `audit-method`. Measured on RouterOS 7.25.1.

## 1. Interfaces

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Interface nobody declared | `/interface wireguard print proplist=name,listen-port,mtu,vrf,disabled,running` | an enabled interface with no comment or known purpose | LOW |
| Tunnel trusted as LAN | `/interface list member print proplist=list,interface,disabled where interface~"wg"` + `/ip firewall filter print stats where chain=input` + `/ipv6 firewall filter print stats where chain=input` | a WireGuard interface in the `LAN` list or any list the input chain accepts wholesale, on either stack: every peer gets what the LAN gets. With IPv6 in `allowed-address` (`::/0` or a ULA), a permissive IPv6 input chain exposes the router even when the IPv4 one is tight. HIGH when that includes the router's management services and peers are road-warrior clients | MEDIUM |

## 2. Peers

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Peer trusted for any source | `/interface wireguard peers print proplist=interface,name,endpoint-address,allowed-address,disabled` | on a concentrator, `allowed-address` `0.0.0.0/0`, `::/0` or a wide prefix on a peer that is a client or a branch: it can send traffic as any address and the router accepts it. On a client pointing at its upstream, full-tunnel is legitimate | HIGH |
| Peers reaching internal networks | `/ip firewall filter print stats where chain=forward` + `/ipv6 firewall filter print stats where chain=forward` | following the rule order, traffic from the WireGuard interface forwarded to internal networks or to other peers with no rule scoping it, on either stack | MEDIUM |
| Overlapping allowed-address | same command, across all interfaces | the same prefix or address under two peers: traffic to it goes to whichever peer wins, silently | MEDIUM |
| Peer without preshared key | `/interface wireguard peers print count-only where preshared-key=""` | count of peers relying only on the key pair, for policy comparison (the preshared key is the post-quantum hedge) | LOW |
| Client private key kept on the router | `/interface wireguard peers print count-only where private-key!=""` | peers whose private key the router generated and still stores: whoever reads the configuration impersonates those clients | MEDIUM |
| Stale peer | `/interface wireguard peers print proplist=interface,name,endpoint-address,last-handshake,disabled` | enabled peers with no `last-handshake` for weeks: a lost device whose key still opens the tunnel | LOW |
</content>
