# routeros-security-skills

Read-only security audit skills for MikroTik RouterOS (v6/v7), packaged for AI coding agents
(Claude Code, ChatGPT Skills and compatible skill loaders). One folder per domain; a category folder holds one skill per type (33 skills). Domain skills share two base dependencies: `audit-method` and `factory-defaults`.

[Leia em português](README.pt-BR.md)

## What is in here

| Skill | Domain |
|---|---|
| [`audit-method`](audit-method/) | How to run the audit: unbreakable rules, what to read before judging, collection order, severity scale, output format, documented traps, incident procedure. **Read first.** |
| [`factory-defaults`](factory-defaults/) | Factory values of `/ip settings`, `/ipv6 settings`, connection tracking, device-mode, the default firewall per chain, the official bogon lists, and the services the vendor tells you to disable. |
| [`access`](access/) | Administrative access: services and their source restriction, SSH, users/groups/keys, RouterBOOT, device-mode, AAA default group, physical button, supout, graphs, version. |
| [`services`](services/) | Auxiliary services: mac-server, MNDP, bandwidth test, DNS cache, DoH, proxy, SOCKS, UPnP, cloud/DDNS, RoMON. |
| [`firewall`](firewall/) | Filter/NAT/mangle chain policy: input/forward, bogons, anti-spoof, TCP flags, zones, counters, fasttrack, NAT, ALG, FQDN lists, drop logging. |
| [`flood-defense`](flood-defense/) | RAW table, SYN flood, DDoS detection, amplification, port scan, brute-force blacklists, conntrack exhaustion, ICMP by type. |
| [`layer2`](layer2/) | MNDP, neighbor table, DHCP snooping, RA Guard, horizon, BPDU guard, STP, bridge filters and hardware offload, VLAN filtering, PVID, MAC limits. |
| [`dhcp`](dhcp/) | DHCP server alerts, pool exhaustion, static leases, add-arp, lease scripts, DHCP client trust. |
| [`ip-settings`](ip-settings/) | `/ip settings` values that only show in print: rp-filter, redirects, source route, forwarding, ARP limits, ICMP rate. |
| [`vpn`](vpn/) | Category, one skill per type: `wireguard`, `zerotier`, `ipsec`, `l2tp`, `pptp`, `sstp`, `ovpn`, `pppoe`. PPP, Quick Set and Back To Home, VXLAN, EoIP and port knocking are still in the category skill. |
| [`qos`](qos/) | Queues with a security effect: management priority, guarantees, PCQ, dead queues, dynamic queues, bufferbloat. |
| [`ipv6`](ipv6/) | IPv6 stack, RA/ND, IPv6 firewall, DHCPv6/PD, IPv6 routing, transition, addressing, extension headers. |
| [`ospf`](ospf/) | OSPF neighborship, network types, areas, redistribution, PPPoE /32 flood, v6↔v7 menu map. |
| [`bgp`](bgp/) | BGP sessions, filters, attributes, leaks, reflection, confederation, RPKI, silent v6↔v7 traps. |
| [`mpls`](mpls/) | LDP, VPLS, L3VPN isolation, traffic engineering, control plane under saturation. |
| [`wireless`](wireless/) | Wireless interfaces (legacy and wifi-qcom): ciphers, PSK exposure, client admission, modes, regulatory, registration table. |
| [`capsman`](capsman/) | CAPsMAN: CAP–controller trust, discovery, provisioning, datapaths, versions, automatic CA. |
| [`scripts-storage`](scripts-storage/) | Scripts, scheduler, fetch, files and backups on disk, SMB, disks, containers. |
| [`logging`](logging/) | Logging, NTP, e-mail, Netwatch, SNMP. |
| [`aaa`](aaa/) | RADIUS client, CoA, AAA default group, PPP secrets. |
| [`user-manager`](user-manager/) | User Manager (built-in RADIUS server): exposure, NAS list, auth methods, stored credentials. |
| [`dude`](dude/) | The Dude server: client access, syslog listener, database, stored credentials. |
| [`rose-storage`](rose-storage/) | Storage served by the router: SMB, NFS, DLNA, iSCSI, NVMe/TCP, file disks, RAID. |
| [`iot`](iot/) | iot package: Modbus gateway, MQTT, LoRa, Bluetooth, Wiliot. |
| [`certificates`](certificates/) | Certificate store and its use by services. |

Every check is a table row with: the **read command**, **what characterises a failure**, and a
**suggested severity** (CRITICAL / HIGH / MEDIUM / LOW). Every file ends with the nuances that
generate false positives — what *looks* like a finding and is not.

## Principles

- **Read-only workflow.** `print`, `get`, `export`, `monitor`. Never `set`, `add`, `remove`, `enable`, `disable`, `reboot`. Use a dedicated least-privilege audit account; do not assume the built-in RouterOS `read` group is strictly read-only.
- **Read the version and the factory default before judging.** v6 and v7 menus differ; a command
  in the wrong menu returns empty, and empty looks like "not configured". A value equal to the
  factory default is not somebody's insecure choice.
- **Read the role of the device.** Edge, concentrator, switch, AP and multihomed core do not
  accept the same list.
- **Secrets never enter AI-assisted collection.** Use safe `proplist` selections on credential-bearing areas. Secret-strength checks are outside agent scope because secret values must never be retrieved. Never include secret values in prompts, context, logs, artifacts or reports. Never use `show-sensitive` in agent-assisted collection.
- **In doubt between two severities, the lower.** Inflated severity turns the report into noise.
- **Absence of a rule is not automatically a failure.** Check `disabled`, order in the chain, the
  interface list and the packet counter first.
- **The audit itself can take the target down.** Wireless `scan`/`snooper` without
  `background=yes`, Torch and `profile` on a saturated CPU. Read state, do not provoke it.

## Base dependencies

Install `audit-method` and `factory-defaults` together with any domain skill. The method defines collection/severity/output behavior; factory-defaults prevents false positives caused by comparing every platform to a home-router baseline.

## Install

Clone once, then expose the same skill folders to the agent you use:

```bash
git clone https://github.com/fabioKoruja00/routeros-security-skills
```

- **Claude Code**: copy or symlink the selected skill folders (each one has a `SKILL.md`) into `~/.claude/skills/` or `<repo>/.claude/skills/`.
- **Codex**: copy or symlink them into `~/.codex/skills/` or `<repo>/.codex/skills/`.
- **Google Antigravity**: copy or symlink them into `~/.gemini/config/skills/` globally or `<repo>/.agents/skills/` for a project.
- **ChatGPT Skills**: each skill already includes `agents/openai.yaml` UI metadata in addition to the common `SKILL.md`.

Do not maintain separate copies of the skill logic for each agent. Keep this repository as the single source of truth and use symlinks where practical.

See [COMPATIBILITY.md](COMPATIBILITY.md) for installation patterns and notes.

## Output the skills produce

One document per device **with findings**. Each finding: what is wrong, the read command that
proves it, what is at stake, and the possible ways to fix it with the risk of each — including
the risk of losing access to the device. A path, not a paste-ready recipe.

## Sources and disclaimer

Everything in this repository comes from official, public sources: the MikroTik RouterOS
documentation and public standards. Independent material, not affiliated with, endorsed by or
certified by MikroTik. Commands, menus and defaults change
between RouterOS releases — confirm on the installed version before acting on any finding.

## License

MIT — see [LICENSE](LICENSE).
