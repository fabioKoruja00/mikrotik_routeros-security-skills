# Logging, time and SNMP checks

Severity scale: CRITICAL / HIGH / MEDIUM / LOW. Every command is read-only. Method and output format: `audit-method`.

## 1. Log and time

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Log only in memory | `/system logging print detail` and `/system logging action print detail` | no `remote` action: a reboot erases everything, and whoever breaks in reboots | HIGH |
| Critical topic not logged | `/system logging print detail` | absence of `account`, `critical`, `error`, `warning` (and `firewall` where a rule has `log=yes`) | HIGH |
| Remote log not arriving | `/system logging action print detail` + count on the collector | `remote` action configured pointing to an IP that no longer receives — **the system lies about its own state** | CRITICAL |
| NTP off | `/system ntp client print` and `/system clock print` | `enabled=no` or clock out of time: invalidates TLS certificates, breaks RPKI/DNSSEC and makes the log useless for forensics | HIGH |
| Open NTP server | `/system ntp server print` and `/ip firewall filter print detail` | `enabled=yes` reachable from the WAN — amplification vector | HIGH |
| Debug on permanently | `/system logging print detail where topics~"debug" or topics~"packet" or topics~"raw"` | debug/packet/raw topic logging non-stop: leaks traffic content and fills the disk | HIGH |
| E-mail without TLS | `:put [/tool e-mail get server]`, `:put [/tool e-mail get port]`, `:put [/tool e-mail get tls]` | `tls=no` on an untrusted path. SMTP password presence/strength is intentionally not assessed by the agent because the password field is sensitive | HIGH |
| Evidence never handled | `/log print without-paging where topics~"account" or topics~"critical" or topics~"error" or topics~"warning"` | serial failed logins, unexpected reboot or configuration change recorded and never looked at | HIGH |
| Netwatch actions present | v7.12+: `/tool netwatch print proplist=name,host,type,interval,timeout,status,disabled`; v6 and early v7 have no `name`/`type`/`test-script` — use `proplist=host,interval,timeout,status,disabled`. Plus presence-only counts for `up-script`, `down-script` and (v7.12+) `test-script` | action scripts exist but their bodies must never enter the agent. Assess only presence, target and schedule unless a trusted local scanner returns sanitized classifications | MEDIUM |
