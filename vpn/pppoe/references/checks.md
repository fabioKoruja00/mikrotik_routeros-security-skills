# PPPoE checks

Severity scale: CRITICAL / HIGH / MEDIUM / LOW. Every command is read-only. Method and output format: `audit-method`. Measured on RouterOS 7.25.1.

## 1. Concentrator

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Several sessions from one host | `/interface pppoe-server server print detail` (no secret in this menu) | `one-session-per-host=no` (default): one MAC opens many sessions | MEDIUM |
| Same credential on several sessions | `/ppp profile print proplist=name,only-one,interface-list,use-encryption` | the server's `default-profile` and the profiles of PPPoE secrets with `only-one` other than `yes`: one credential is shared or cloned unnoticed. With RADIUS (`/ppp aaa`) the limit lives in the RADIUS server instead | MEDIUM |
| No ceiling on sessions | the server command above | `max-sessions=unlimited` (default) on a concentrator sized for a known subscriber count: a flood of sessions exhausts pool and CPU | LOW |
| Concentrator accepting empty service | same command | no `accept-empty-service=no` in the output (the field is hidden while `yes`, the default). Measured: a client with an empty `service-name` joined a server that announced `svc-teste` | MEDIUM |
| Weak PPP authentication | same command | `authentication` containing `pap`, `chap` or `mschap1` (the default is all four plus `mschap2`): PAP sends the password in clear on the access segment, the others are weak. A RADIUS backend that only takes PAP is a documented exception | MEDIUM |
| Concentrator with an IP on the access interface | `/ip address print proplist=address,interface` + `/ipv6 address print proplist=address,interface where !link-local` + `/interface bridge port print proplist=bridge,interface` | the interface where PPPoE listens, or the bridge it belongs to, holds an IPv4 or global IPv6 address: subscribers reach the router and each other at layer 3 outside PPP. No address there: no finding | MEDIUM |
| Subscriber IPv6 without policy | the profile command above + `/ipv6 firewall filter print stats where chain=forward` + `/ipv6 firewall filter print stats where chain=input` | sessions get IPv6 (DHCPv6-PD measured inside PPPoE) but no `interface-list` on the profile and no IPv6 rule scopes PPP interfaces: subscribers reach the router and each other over IPv6 | MEDIUM |
| VLAN scope wider than planned | the server command above | context: `accept-untagged=yes` (default) and `pppoe-over-vlan-range` decide which VLANs may dial | LOW |

## 2. Client

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Client without a fixed concentrator | `/interface pppoe-client print proplist=name,interface,user,service-name,ac-name,allow,use-peer-dns,add-default-route,default-route-distance,disabled,running` | `ac-name` empty: the client joins the first concentrator that answers (measured). A `service-name` does not help: a rogue can announce the same one | MEDIUM |
| Client accepting weak authentication | same command | `allow` containing `pap` (a new client is `pap,chap,mschap1,mschap2`): a rogue concentrator gets the password in clear. HIGH together with an empty `ac-name`. `chap`/`mschap1` only: LOW; only `mschap2` authenticates the server back | MEDIUM |
| Route and DNS dictated by the link | same command | `use-peer-dns=yes` and `add-default-route=yes` on a link that is not the approved Internet uplink | LOW |

## 3. Evidence

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Sessions in use | `/ppp active print count-only where service=pppoe` first, then a narrow `where` (never a full listing on a concentrator with thousands of sessions) | the same MAC (`caller-id`) or credential on more than one session within the same server and interface confirms the permissive settings above in use. The same MAC on different servers or VLANs can be legitimate | MEDIUM |
