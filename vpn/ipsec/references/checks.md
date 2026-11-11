# IPsec checks

Severity scale: CRITICAL / HIGH / MEDIUM / LOW. Every command is read-only. Method and output format: `audit-method`. Measured on RouterOS 7.25.1.

## 1. Algorithms

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Broken phase 1 algorithm | `/ip ipsec profile print proplist=name,hash-algorithm,enc-algorithm,dh-group,lifetime,proposal-check,dpd-interval` + `/ip ipsec peer print proplist=name,address,exchange-mode,passive,profile,disabled` | a profile used by an enabled peer offering `md5`, `des`, `3des` or `dh-group` `modp768`/`modp1024`. Measured: the factory `default` profile offers `aes-128,3des` and `modp2048,modp1024`, so a peer left on `profile=default` can be negotiated down to them | HIGH |
| SHA-1 in phase 1 | same commands | `hash-algorithm=sha1` with otherwise sound algorithms (the factory `default`). Legacy, not broken as an HMAC/PRF | MEDIUM |
| Broken phase 2 algorithm | `/ip ipsec proposal print proplist=name,auth-algorithms,enc-algorithms,pfs-group,lifetime,disabled` + `/ip ipsec policy print proplist=src-address,dst-address,peer,proposal,tunnel,action,level,template,disabled where !dynamic` | a proposal used by a policy or template allowing `md5` auth or `des`/`3des` encryption, or `null` where confidentiality is required | HIGH |
| SHA-1 or weak PFS in phase 2 | same proposal command | `auth-algorithms=sha1`, `pfs-group=modp1024` (both in the factory `default` proposal) or `pfs-group=none`. `none` can be a documented interoperability choice for some IKEv2 clients | MEDIUM |
| Lifetime set by the other side | the profile command | `proposal-check=obey` (the default) on a responder: the initiator's lifetime is accepted, however long. `claim` or `strict` keep the local one | LOW |
| Dead peers never detected | the profile command | `dpd-interval=disable-dpd` on a profile used by a site-to-site peer: a dead tunnel keeps its policies and SAs | LOW |

## 2. Peers and identities

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| IKEv1 aggressive with PSK | `/ip ipsec peer print proplist=name,address,exchange-mode,passive,profile,disabled` + `/ip ipsec identity print proplist=peer,auth-method,generate-policy,policy-template-group,mode-config,my-id,remote-id,match-by,disabled` | `exchange-mode=aggressive` on a peer whose identity is `pre-shared-key`: the PSK hash is sent before authentication and can be cracked offline | HIGH |
| IKEv1 where IKEv2 fits | same peer command | `exchange-mode=main` between two RouterOS devices or modern clients. Measured: a peer added with no options is `exchange-mode=main profile=default` | LOW |
| Responder open to anyone with a shared key | `/ip ipsec peer print proplist=name,address,passive,exchange-mode,profile,disabled where passive` + the identity command | a passive peer with no `address` (measured: `address=0.0.0.0/0` is stored empty, so filter by `passive`, not by address) whose identity is `pre-shared-key` with `generate-policy` other than `no`: one leaked key admits any host and lets it create policies. Certificates or EAP fix it | HIGH |
| Template wider than the clients need | `/ip ipsec policy print proplist=src-address,dst-address,group,template,proposal,disabled where template` | a template in the group an identity with `generate-policy` uses, with destination `0.0.0.0/0` or `::/0`, when the clients are not meant to reach everything. Full tunnel is a legitimate design; the factory template `::/0`→`::/0` in group `default` only counts when such an identity uses that group | MEDIUM |
| Certificate identity accepting any certificate | the identity command | `auth-method=digital-signature` with no `remote-certificate` and `match-by=remote-id` with an empty `remote-id`: any certificate the trusted CA signed is accepted, not only the expected peer's | MEDIUM |

## 3. Policies and traffic

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Policy that falls back to cleartext | `/ip ipsec policy print count-only where level=use and !template and !dynamic` | `level=use` sends the traffic unencrypted while no SA exists (`require` and `unique` do not) | MEDIUM |
| Mode-config pushed to clients | `/ip ipsec mode-config print proplist=name,responder,address-pool,split-include,static-dns,system-dns` | context for the firewall review: a responder with `split-include` `0.0.0.0/0` sends all client traffic through the concentrator | LOW |
| Masquerade or FastTrack in the tunnel path | `/ip firewall nat print stats where action=masquerade or action=src-nat` + `/ip firewall filter print stats where chain=forward` + the policy command | a masquerade whose selectors cross a tunnel policy with no earlier accept and no `ipsec-policy=out,none`; or a `fasttrack-connection` with no earlier accept for `ipsec-policy=in,ipsec` and `out,ipsec`. Either breaks the tunnel or leaks its traffic outside it | MEDIUM |
| IKE open where nothing terminates it | `/ip firewall filter print stats where chain=input` + `/ipv6 firewall filter print stats where chain=input` + `/ip ipsec peer print count-only where !disabled` | udp/500, udp/4500, ESP or AH reachable from the WAN on either stack, in rule order, with zero enabled peers. Measured: the factory IPv6 input accepts all three from anywhere | LOW |

## 4. Evidence

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Live peers | `/ip ipsec active-peers print proplist=remote-address,state,side,ph2-total,uptime` | an established peer that no enabled peer entry explains: not a configured address and not served by a passive peer | MEDIUM |
| Algorithms actually negotiated | `/ip ipsec installed-sa print proplist=src-address,dst-address,state,enc-algorithm,auth-algorithm` | an SA using `md5`, `des`, `3des` or `null` that the profiles above should have excluded | HIGH |
