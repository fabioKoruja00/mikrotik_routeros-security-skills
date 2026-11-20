# SNMP checks

Severity scale: CRITICAL / HIGH / MEDIUM / LOW. Every command is read-only. Method and output format: `audit-method`. Measured on RouterOS 7.25.1.

## 1. Communities

Exposure needs three things at once: `/snmp` enabled, the entry enabled, and UDP 161 reachable from that network in the input chain (`/ip firewall filter print detail where chain=input` and the IPv6 equivalent). Judge each row with them.

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Well-known community | `:put [/snmp get enabled]` + `/snmp community print count-only where name=public and !disabled` and the same for `private` + the `proplist` below | an enabled entry with a well-known name; the value itself is never printed. Only `public` ships with the factory configuration. CRITICAL when that entry also has `write-access=yes` and is exposed | HIGH |
| No source restriction | `/snmp community print proplist=disabled,addresses,security,read-access,write-access,authentication-protocol,encryption-protocol` | `addresses=0.0.0.0/0` or `::/0` (the factory value) on an exposed entry: the MIB answers interfaces, addresses, clients and topology | HIGH |
| Write enabled | the same command | `write-access=yes` on an exposed entry: every writable OID can be set over SNMP. On an entry that is disabled, or with `/snmp` off, it is a dormant risk | CRITICAL |
| v1/v2c on an untrusted network | the same command | `security=none` on an entry exposed outside an isolated management network: the community crosses the network in clear text | HIGH |
| Weak SNMPv3 algorithms | the same command | `authentication-protocol=MD5` on `security=authorized` or `private`, or `encryption-protocol=DES` on `security=private`. 7.25.1 accepts SHA-2 (`SHA256`). The values shown on `security=none` entries mean nothing | LOW |
| Sensitive OIDs readable | `/snmp community print count-only where read-access and !denied-oids and !disabled` | read access with no `denied-oids`, on an entry exposed to a network outside management: the full MIB answers | LOW |

## 2. Traps

A trap leaves only with `/snmp` enabled and a non-empty `trap-target`; on 7.25.1 the factory has `trap-version=1`, `trap-target` empty and `trap-generators=temp-exception`.

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| Unprotected trap | `:put [/snmp get enabled]` + `:put [/snmp get trap-version]` + `:put [/snmp get trap-target]` + `:put [/snmp get trap-generators]` | traps leaving with `trap-version=1` or `2` to a collector outside the management network, without a protected transport: the trap community crosses in clear text. Never retrieve `trap-community` | MEDIUM |
| Trap to the wrong destination | `:put [/snmp get trap-target]` | a non-empty `trap-target` that is not the current collector: traps, and the community with them, go to a third party | MEDIUM |
