# L2TP checks

Severity scale: CRITICAL / HIGH / MEDIUM / LOW. Every command is read-only. Method and output format: `audit-method`. Measured on RouterOS 7.25.1.

## 1. Server

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| L2TP without mandatory IPsec | `/interface l2tp-server server print proplist=enabled,use-ipsec,authentication,default-profile,one-session-per-host,max-sessions,accept-proto-version,caller-id-type` + `:put [:len [/interface l2tp-server server get ipsec-secret]]` | `enabled=yes` with `use-ipsec` other than `required`: `no` is cleartext PPP, `yes` still accepts sessions without IPsec. Lab templates use `use-ipsec=yes` | HIGH |
| Cleartext PPP authentication | same command | `authentication` containing `pap`, `chap` or `mschap1` (the default is all four plus `mschap2`). HIGH when IPsec is not `required` | MEDIUM |
| L2TP/IPsec on the factory IPsec set | `/ip ipsec peer print proplist=name,exchange-mode,passive,profile,dynamic where dynamic` + `/ip ipsec profile print proplist=name,hash-algorithm,enc-algorithm,dh-group` + `/ip ipsec proposal print proplist=name,auth-algorithms,enc-algorithms,pfs-group` + `/ip ipsec installed-sa print proplist=src-address,dst-address,enc-algorithm,auth-algorithm` | the dynamic peer (`l2tp-in-server` on the server, one per `l2tp-client` on the client — measured on both) on `profile=default`, and the `default` profile or proposal still offering `3des` or `modp1024` (factory). With only `sha1` left in the IKE hash or ESP auth: MEDIUM. The installed SAs show what was actually negotiated | HIGH |
| One key for every client | the server command above and `/ip ipsec identity print proplist=peer,auth-method,generate-policy,dynamic where dynamic` | the server's IPsec is a single PSK for every client with `generate-policy=port-strict`: anyone who holds it reaches the PPP login. Inherent to L2TP/IPsec PSK; IKEv2 with certificates removes it | MEDIUM |
| Unlimited sessions per credential | the server command above | `one-session-per-host=no` with `max-sessions=unlimited` (both default) on a server whose credentials are personal: one leaked credential opens many sessions unnoticed | LOW |
| L2TPv3 accepted without need | the server command above + `/interface l2tp-ether print proplist=name,connect-to,local-address,use-ipsec,digest-hash,cookie-length,disabled` | `accept-proto-version=all` (default) on a server that only serves L2TPv2 clients; or an `l2tp-ether` tunnel with `use-ipsec=no` crossing an untrusted network (layer 2 in cleartext). `digest-hash=md5` is the default | MEDIUM |

## 2. Client

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Client without IPsec | `/interface l2tp-client print proplist=name,connect-to,user,use-ipsec,allow,profile,add-default-route,use-peer-dns,disabled,running` | `use-ipsec=no` to a server across an untrusted network | HIGH |
| Client accepting weak PPP authentication | same command | `allow` containing `pap` (the default is `pap,chap,mschap1,mschap2`): a rogue or misconfigured server gets the password in clear. `chap` or `mschap1` alone, or any of them inside `use-ipsec=yes` to a server that requires IPsec: LOW | MEDIUM |
| Route and DNS taken from the tunnel | same command | `add-default-route=yes` or `use-peer-dns=yes` on a link that should only reach specific networks | LOW |

## 3. Exposure and evidence

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| udp/1701 open without IPsec | `/ip firewall filter print stats where chain=input` + `/ipv6 firewall filter print stats where chain=input` | udp/1701 accepted from the WAN on either stack without `ipsec-policy=in,ipsec`, on a server whose `use-ipsec` is not `required` | MEDIUM |
| Sessions in use | `/ppp active print count-only where service=l2tp`, then `/ppp active print proplist=name,service,caller-id,address,encoding,uptime where service=l2tp` | context: `caller-id` is the client address, IPv4 or IPv6, and can change or sit behind NAT, so an unfamiliar one is a lead to check against the secret's owner, not a finding by itself; `encoding` shows the ESP algorithms in use (measured: `cbc(aes) + hmac(sha1)`) | LOW |
