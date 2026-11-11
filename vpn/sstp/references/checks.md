# SSTP checks

Severity scale: CRITICAL / HIGH / MEDIUM / LOW. Every command is read-only. Method and output format: `audit-method`. Measured on RouterOS 7.25.1.

## 1. Server

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Server without a certificate | `/interface sstp-server server print proplist=enabled,port,certificate,tls-version,ciphers,pfs,verify-client-certificate,authentication,default-profile` | `enabled=yes` with `certificate=none` (the default): no client can verify who it talks to | HIGH |
| Old TLS accepted | same command | `tls-version=any` (default) instead of `only-1.2` or `only-1.3` | MEDIUM |
| Weak cipher set or no PFS | same command | `ciphers` without a GCM suite, or `pfs=no` (default): captured sessions decrypt if the server key leaks | LOW |
| Cleartext PPP authentication inside | same command | `authentication` containing `pap` (the default is all four): the TLS layer protects it in transit, but a client that skips verification hands the password to an impostor | MEDIUM |
| Clients not authenticated by certificate | same command | `verify-client-certificate=no` (default) on a server whose only other factor is the PPP password | LOW |

## 2. Client

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Client trusting any server | `/interface sstp-client print proplist=name,connect-to,port,verify-server-certificate,verify-server-address-from-certificate,tls-version,ciphers,pfs,authentication,certificate,add-sni,http-proxy,profile,add-default-route,disabled,running` | `verify-server-certificate=no` (default): anyone on the path can pose as the server and collect the PPP login | HIGH |
| Client accepting PAP | same command | `authentication` containing `pap` together with `verify-server-certificate=no`: an impostor can ask for PAP and get the password in clear. The list proves the risk, not that PAP was negotiated. With verification on: LOW | MEDIUM |
| Client allowing old TLS | same command | `tls-version=any` (default) | MEDIUM |
| Client limited to a CBC suite | same command | `ciphers=aes256-sha` (default): the session settles on CBC with SHA-1 even against a server that offers GCM (measured: `encoding=AES256-CBC`) | LOW |

## 3. Exposure and evidence

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Port shared or exposed | `/ip service print proplist=name,port,disabled where name=sstp-server` + `/ip service print proplist=name,port,disabled where name=www-ssl or name=reverse-proxy` + `/ip firewall filter print stats where chain=input` + `/ipv6 firewall filter print stats where chain=input` | context: the dynamic `sstp-server` entry gives the real port (443 unless changed); whether it is reachable from the WAN on either stack, and whether `www-ssl` or `reverse-proxy` share it | LOW |
| Sessions in use | `/ppp active print count-only where service=sstp`, then `/ppp active print proplist=name,service,caller-id,address,encoding,uptime where service=sstp` | context: `caller-id` is the client address (IPv4 or IPv6); `encoding` shows the TLS cipher (measured: `AES256-CBC`) | LOW |
