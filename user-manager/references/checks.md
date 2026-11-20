# User Manager checks

Severity scale: CRITICAL / HIGH / MEDIUM / LOW. Every command is read-only. Method and output format: `audit-method`. Measured on RouterOS 7.25.1.

## 1. Server and exposure

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Server on without a role | `:put [/user-manager get enabled]` | `true` on a device nobody declared as RADIUS server: UDP 1812/1813 open for nothing | MEDIUM |
| RADIUS ports reachable from untrusted networks | `/ip service print where port=1812 or port=1813` (dynamic `manager` entries) + `/ip firewall filter print stats where chain=input` + `/ip firewall raw print stats` + `/ipv6 firewall filter print stats where chain=input` | UDP 1812/1813 accepted from a WAN or customer interface, on IPv4 or IPv6: any host there can try credentials and forged packets. No NAS-specific rule while the final drop already closes the WAN is LOW | HIGH |
| Message-Authenticator not required | `:put [/user-manager get require-message-auth]` | `no` (values: `no`, `yes-access-request`; 7.25.1 ships `yes-access-request`): Access-Request without Message-Authenticator is accepted, the opening for BlastRADIUS forgery. MEDIUM when every NAS sits on the local or management network, since the attack needs a position on the UDP path | HIGH |
| RadSec NAS without a server certificate | `:put [/user-manager get radsec-certificate]` + the NAS list below | a NAS with `protocol=radsec` while `radsec-certificate=none`: the TLS transport the operator chose cannot work as intended | MEDIUM |
| EAP without a server certificate | `:put [/user-manager get certificate]` + the group list below | `certificate=none` while a group in use has `eap-tls`, `eap-ttls` or `eap-peap` in `outer-auths`: EAP fails or clients are taught to accept any certificate | LOW |

## 2. NAS list

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| NAS address too wide | `/user-manager router print proplist=name,address,protocol,coa-port,disabled` | `address` covering a range or `0.0.0.0/0` instead of the NAS itself: every host in it may talk RADIUS with that secret | HIGH |
| NAS over UDP across an untrusted path | same command | `protocol=udp` (values: `udp`, `radsec`) for a NAS outside the local or management network | MEDIUM |
| Stale NAS | same command | `disabled=no` entry for a device that no longer exists: its secret still opens the server | LOW |

## 3. Users and groups

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| MS-CHAPv1 accepted | `/user-manager user group print proplist=name,outer-auths,inner-auths` | `mschap1` in `outer-auths` of a group in use: a captured exchange gives the password back. The factory `default` group has it | MEDIUM |
| PAP/CHAP accepted | same command | `pap` or `chap` in a group in use without OTP or hotspot HTTP-PAP needing it. Both are in the factory `default` group, so report once, not per device | LOW |
| Active user without password | `/user-manager user print count-only where password="" and disabled=no` | count above 0: the account authenticates with an empty password | MEDIUM |
| Unlimited shared users | `/user-manager user print proplist=name,group,shared-users,caller-id,disabled` | `shared-users=unlimited` or a high number on subscriber accounts: one credential serves many concurrent sessions | LOW |
| No caller binding on fixed subscribers | same command | `caller-id` empty where the plan is one fixed CPE: a leaked credential works from any device | LOW |
| OTP coverage | `/user-manager user print count-only where otp-secret=""` | count of users without OTP, for policy comparison only | LOW |

## 4. Data and extras

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Database on internal flash | `:put [/user-manager database get db-path]` + `/system resource print` (`free-hdd-space`) | path on the internal storage (7.25.1 default `/user-manager5`) on a device with a large subscriber base: session and accounting writes wear the flash and fill it | MEDIUM |
| Private web without password | `:put [/user-manager advanced get web-private-username]` + `:put [:len [/user-manager advanced get web-private-password]]` | username set with password length 0 | MEDIUM |
| PayPal configured without use | `:put [:len [/user-manager advanced get paypal-user]]` | non-empty where nobody takes payments: credentials stored for nothing | LOW |
| Session evidence | `/user-manager session print proplist=user,nas-ip-address,calling-station-id,user-address,started,ended,terminate-cause where active` | session from a NAS address not in the router list, or one user active from several `calling-station-id`: shared or stolen credential. Without `where active` the table can be huge on a small device | MEDIUM |
</content>
