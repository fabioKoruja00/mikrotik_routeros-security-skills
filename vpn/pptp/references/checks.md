# PPTP checks

Severity scale: CRITICAL / HIGH / MEDIUM / LOW. Every command is read-only. Method and output format: `audit-method`. Measured on RouterOS 7.25.1.

## 1. Server and client

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| PPTP server reachable from an untrusted network | `/interface pptp-server server print proplist=enabled,authentication,default-profile,keepalive-timeout` + `/ip firewall filter print stats where chain=input` | `enabled=yes` and tcp/1723 with GRE accepted from the WAN in rule order. `authentication` keeping `mschap1` (default) or adding `pap`/`chap` makes it worse but changes nothing: PPTP is broken either way. Enabled but reachable only from a closed internal segment: MEDIUM | CRITICAL |
| Sessions without MPPE | `/ppp profile print proplist=name,use-encryption,only-one` | the server's `default-profile` (or the profile a PPTP secret uses) with `use-encryption` other than `required` or `yes`: the PPP payload crosses in clear | HIGH |
| PPTP client across an untrusted network | `/interface pptp-client print proplist=name,connect-to,user,allow,profile,add-default-route,use-peer-dns,disabled,running` | an enabled client whose `connect-to` is across the Internet: its MS-CHAPv2 exchange can be captured and cracked | HIGH |
| Client accepting PAP | same command | `allow` containing `pap` (the default is `pap,chap,mschap1,mschap2`): a rogue server gets the password in clear | MEDIUM |
| PPTP still allowed by device-mode | `/system device-mode print` | hardening only: `pptp: yes` on a device that serves no PPTP. device-mode can switch it off, and turning it back on needs physical access | LOW |

## 2. Traffic and evidence

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Sessions in use | `/ppp active print count-only where service=pptp`, then `/ppp active print proplist=name,service,caller-id,address,encoding,uptime where service=pptp` | a live session whose `caller-id` is outside the trusted segments: that credential and key are exposed to anyone on the path. `encoding` shows MPPE (measured: `MPPE128 stateless`); empty encoding is cleartext | MEDIUM |
| NAT helper for PPTP | `/ip firewall service-port print proplist=name,disabled where name=pptp` | context only, when PPTP is seen crossing the router: the helper is enabled by default and lets clients behind NAT reach PPTP servers | LOW |
