# The Dude checks

Severity scale: CRITICAL / HIGH / MEDIUM / LOW. Every command is read-only. Method and output format: `audit-method`. Measured on RouterOS 7.25.1.

## 1. Server and exposure

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Server on without a role | `/dude print` | `enabled=yes`, `status=running` on a device nobody declared as the monitoring server | MEDIUM |
| Client access reachable | `/ip service print where name=winbox` + `/ip firewall filter print stats where chain=input` + `/user print proplist=name,group,address,disabled` | the Dude client logs in through the Winbox service. Winbox enabled with empty `available-from` and accepted by the input chain from a WAN or customer interface, or users of a write-capable group without `address`: a login there reaches the credentials of the whole monitored network. Winbox open only to management is not a finding here | HIGH |
| Syslog listener reachable | `/ip service print where name=dudes` + `/ip firewall filter print stats where chain=input` + `/ipv6 firewall filter print stats where chain=input` | following the rule order, UDP 514 accepted from an untrusted interface or interface list on IPv4 or IPv6: anyone can inject or flood log events the operator acts on | MEDIUM |

## 2. Data

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Database on internal flash | `:put [/dude get data-directory]` + `/file print proplist=name,size,last-modified where name~"dude.db"` + `/disk print proplist=slot,model,interface,size,free,mount-point` + `/system resource print` (`free-hdd-space`) | `data-directory` on the internal storage with no external disk, and `dude.db` with its `-wal` growing against little free space: the polling history wears and fills the flash | MEDIUM |
| Database copy left on disk | `/file print proplist=name,type,size,last-modified where name~"dude" and type!=directory and !(name~"^<data-directory>/")` (with the default `data-directory=dude`: `!(name~"^dude/")`) | files outside the data directory with `dude` in the name: possibly an export carrying every stored device credential. Judge by name, size and date only; a renamed export escapes this filter | MEDIUM |

## 3. Inventory

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Remote agents | `/dude agent print` | agents nobody recognises: a remote poller that receives probe tasks and credentials | MEDIUM |
| Monitored inventory | `/dude device print`, `/dude service print`, `/dude probe print`, `/dude notification print` | names only; objects nobody recognises in the map, or notifications nobody planned | LOW |
</content>
