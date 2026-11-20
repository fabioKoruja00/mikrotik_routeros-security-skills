---
name: rose-storage
description: "Read-only audit of RouterOS v7 storage services (rose-storage package and the built-in SMB server): disks shared over SMB, NFS or DLNA, block devices exported over iSCSI or NVMe/TCP, the SMB server, its users and shares, file-backed disks, encryption and RAID. Use this skill whenever a MikroTik device has rose-storage installed, has disks attached, serves files, or the audit mentions SMB, NFS, iSCSI, NVMe-TCP, DLNA, /disk or /ip smb, without changing configuration."
---

# ROSE storage

A router that serves storage hands its data to whoever reaches TCP 445, 3260 or the NFS ports. RouterOS makes this easy to do by accident: on 7.25.1 a newly added disk is shared over SMB to `guest` and over DLNA on the bridge by default, and an iSCSI export has no authentication to set.

## Rules

Read-only: `print`, `get`, `export`, `monitor` only — never `set`, `add`, `remove`, `enable`, `disable`, `reboot`. Connect with a dedicated least-privilege audit account; do not assume the built-in `read` group is strictly read-only. Audit only the devices that were named. Secrets never enter the report (use `proplist` on areas that store credentials). Method, severity scale (CRITICAL / HIGH / MEDIUM / LOW — in doubt, the lower), output format and the collection order live in `audit-method`; factory values in `factory-defaults`. Read the RouterOS version first: v6 and v7 menus differ, and a command in the wrong menu returns empty.

## Checks

[references/checks.md](references/checks.md)

## Traps

- **Never `/disk print detail` or a bare `/ip smb users print`.** `/disk` holds `encryption-key`, `smb-password`, `smb-server-password`, `nvme-tcp-password` and `nvme-tcp-server-password`; SMB users hold `password`. Use the `proplist` in the checks.
- **Never browse a share or open a disk's files** to judge what is exposed. Report the sharing setting, the audience and the reachability.
- **The services show in `/ip service` as dynamic entries** (`smb` 445, `smb-netbios` 137/138, `iscsi` 3260) with no `available-from`. `/ip smb interfaces` is the only built-in scoping; the per-disk `*-address` fields belong to client disks, not exports. The input chain does the rest.
- **Block export and file sharing are exclusive per disk.** A disk with `iscsi-export=yes` cannot have SMB, NFS or DLNA at the same time, so a disk showing both is impossible — re-read before reporting.
- Unplanned disks and the package itself are in `scripts-storage`.
</content>
