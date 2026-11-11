# OVPN checks

Severity scale: CRITICAL / HIGH / MEDIUM / LOW. Every command is read-only. Method and output format: `audit-method`. Measured on RouterOS 7.25.1.

## 1. Server

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Server without a certificate | `/interface ovpn-server server print proplist=name,disabled,port,protocol,mode,certificate,require-client-certificate,tls-version,auth,cipher,user-auth-method,default-profile,redirect-gateway,push-routes,push-routes-ipv6,enable-tun-ipv6,reneg-sec` | an entry with `disabled=no` and no `certificate`: no client can verify it. A referenced certificate that is missing, expired or without its private key counts the same — checked in `certificates` | HIGH |
| Broken algorithms accepted | same command | `cipher` containing `blowfish128` (64-bit block, Sweet32), or `auth` containing `md5` while a non-GCM cipher is allowed — both in the factory list. The fields are lists of allowed values: read "contains", not "equals". `md5` with GCM-only ciphers: MEDIUM. `sha1` or `aes*-cbc` without broken entries: LOW | HIGH |
| Old TLS accepted | same command | `tls-version=any` (default) | MEDIUM |
| Password as the only factor | same command | `require-client-certificate=no` (default): anyone with a PPP login connects | MEDIUM |
| Cleartext user authentication method | same command | `user-auth-method=pap` (default): inside TLS, but a client that skips server verification hands the password to an impostor | LOW |
| Clients bridged or routed wider than needed | same command | context: `mode=ethernet` puts every client on a layer 2 segment; `redirect-gateway` and `push-routes`/`push-routes-ipv6` decide which networks clients reach | LOW |

## 2. Client

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Client trusting any server | `/interface ovpn-client print proplist=name,connect-to,port,protocol,mode,auth,cipher,tls-version,verify-server-certificate,certificate,route-nopull,add-default-route,use-peer-dns,disabled,running` | `verify-server-certificate=no` (default): anyone on the path can pose as the server | HIGH |
| Client on broken algorithms | same command | `cipher=blowfish128` or `auth=md5`; the factory client is `blowfish128` with `sha1` | HIGH |
| Client allowing old TLS | same command | `tls-version=any` (default) | MEDIUM |
| DNS taken from the server | same command | `use-peer-dns=yes` (default) or `add-default-route=yes` on a link meant for specific networks | LOW |

## 3. Exposure and evidence

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Server reachable from the WAN | `/ip service print proplist=name,port,proto,disabled where name=ovpn-server` + `/ip firewall filter print stats where chain=input` + `/ipv6 firewall filter print stats where chain=input` | context for the checks above: one dynamic entry per server with its real port and protocol, reachable from the WAN on either stack. A server with `vrf` other than `main` listens in that VRF's interfaces | LOW |
| Tunnel IPv6 without forward policy | the server command above + `/ipv6 firewall filter print stats where chain=forward` | `enable-tun-ipv6=yes` with no IPv6 forward rule scoping traffic from or to the tunnel interfaces: clients and the networks behind them meet with no filter | MEDIUM |
| Sessions in use | `/ppp active print count-only where service=ovpn`, then `/ppp active print proplist=name,service,caller-id,address,encoding,uptime where service=ovpn` | context: `caller-id` is the client address (IPv4 or IPv6); `encoding` shows the negotiated cipher (measured: `AES-256-GCM`) | LOW |
