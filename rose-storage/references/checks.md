# ROSE storage checks

Severity scale: CRITICAL / HIGH / MEDIUM / LOW. Every command is read-only. Method and output format: `audit-method`. Measured on RouterOS 7.25.1.

The `/disk` fields without `server` (`iscsi-address`, `iscsi-iqn`, `nfs-address`, `nfs-share`, `nvme-tcp-address`, `smb-address`, `smb-user`) belong to *client* disks that mount a remote target (`type=iscsi`, `nfs`, `nvme-tcp`, `smb`). Exports have no source restriction of their own: the input chain, and for SMB the `/ip smb interfaces` list, are the only scoping.

## 1. Exports

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| iSCSI export reachable | `/disk print proplist=slot,type,iscsi-export,iscsi-server-iqn,iscsi-server-port where iscsi-export=yes` + `/ip service print where name=iscsi` + `/ip firewall filter print stats where chain=input` | an export with TCP 3260 accepted from anything beyond the storage hosts: the target has no authentication setting, so any host reaching it attaches the block device and reads or overwrites it | CRITICAL |
| NVMe/TCP export without host lock | `/disk print proplist=slot,nvme-tcp-export,nvme-tcp-server-allow-host-name,nvme-tcp-server-port where nvme-tcp-export=yes` + `/disk print count-only where nvme-tcp-export=yes and nvme-tcp-server-password=""` | export without `nvme-tcp-server-allow-host-name` or without a server password, reachable beyond the storage hosts | HIGH |
| NFS export reachable | `/disk print proplist=slot,nfs-sharing where nfs-sharing=yes` + `/ip service print where dynamic` + `/ip firewall filter print stats where chain=input` | NFS sharing on with its ports accepted from a WAN or customer interface: NFS has no user authentication and the export has no address list | HIGH |
| Disk shared to guest | `/disk print proplist=slot,type,smb-sharing,smb-server-user,smb-server-encryption,media-sharing,media-interface where smb-sharing=yes` | `smb-server-user=guest`: anyone reaching SMB opens the disk. 7.25.1 shares every new disk this way by default | HIGH |
| SMB without encryption | same command | `smb-server-encryption=no` on a disk shared beyond a trusted segment, with a named user (with `guest` there is nothing to protect in transit) | MEDIUM |
| DLNA media sharing on | same command | `media-sharing=yes` (default on a new disk): DLNA has no authentication and ignores the SMB user, so the disk's media is open to the whole `media-interface`. HIGH when that bridge carries guest, IoT or customer devices | MEDIUM |

## 2. SMB server

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| SMB on every interface | `/ip smb print` | `enabled=yes` (or `auto` with a share active) and `interfaces=all`, the default. HIGH only when the ports check below proves reach from an untrusted interface | MEDIUM |
| Ports reachable | `/ip service print where name~"smb"` + `/ip firewall filter print stats where chain=input` + `/ipv6 firewall filter print stats where chain=input` | following the rule order, TCP 445 or UDP 137/138 accepted from a WAN or customer interface | HIGH |
| Share pointing at the router's own files | `/ip smb shares print proplist=name,directory,valid-users,read-only,disabled` | enabled share with `directory` empty, `/` or a path on the internal flash root: exposes `.backup`, `.rsc`, certificate keys and databases | CRITICAL |
| Share open to everyone | `/ip smb shares print proplist=name,directory,valid-users,invalid-users,read-only,require-encryption,disabled` | enabled share with `valid-users` empty and `read-only=no`, or `require-encryption=no` across an untrusted segment | MEDIUM |
| Guest user active | `/ip smb users print proplist=name,read-only,disabled` | the default `guest` user with `disabled=no` while a share is enabled | MEDIUM |
| Named user with empty password | `/ip smb users print count-only where password="" and disabled=no and name!="guest"` | count above 0 | HIGH |

## 3. Disks

| Check | Read command | Characterises a failure | Sev. |
|---|---|---|---|
| File-backed disk on internal flash | `/disk print proplist=slot,type,file-path,file-size where type=file` + `/disk print proplist=slot,type,model,size` | a file disk whose `file-path` is on the internal flash rather than an SD or USB disk: share writes wear it, and a full flash takes the router configuration down with it | MEDIUM |
| Data at rest without encryption | `/disk print proplist=slot,type,model,crypted-backend,self-encryption` + `/disk print count-only where encryption-key=""` | removable disk holding shared data with no encryption: physical theft gives the data | LOW |
| Degraded RAID | `/disk print proplist=slot,type,raid-type,raid-master,raid-role,raid-member-failed` | `raid-member-failed=yes` or a member missing: the next failure loses the array | MEDIUM |
</content>
