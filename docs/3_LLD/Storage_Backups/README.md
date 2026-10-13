# Storage and Backups

Design data ownership and acceptable loss/downtime before choosing disks or storage platforms.

---

## Table of Contents

- [Design Options](#design-options)
- [Backup and Recovery Choices](#backup-and-recovery-choices)
- [Implementation and Verification](#implementation-and-verification)

---

## Design Options

| Option | Benefits | Drawbacks / fit |
|---|---|---|
| Local disks / ZFS | Simple local I/O and integrity/snapshot features where configured | Host loss affects access; independent recovery copies required |
| NAS / NFS / shared storage | Shared data and guest access | Network/appliance dependency; useful when shared access is required |
| Distributed storage, e.g. Ceph | Replication and distributed placement | Resource/network/quorum/rebuild cost; choose sufficient independent hardware and a deliberate need |
| Kubernetes CSI integrations | Workload volume lifecycle | Durability comes from underlying design, not the driver name |

Size capacity, endurance, I/O, replication overhead and surviving/rebuild capacity. Owning several hosts is not by itself a reason to run distributed storage.

---

## Backup and Recovery Choices

Redundancy supports availability; snapshots aid rollback; backups provide independent recovery. None automatically replaces the others. Git records intent but not all data or device/runtime state.

Proxmox Backup Server and built-in VM backup tools suit virtualization; file/database tools and managed/commercial systems cover other scopes. Check application consistency, encryption, retention, keys and restore workflow rather than only archive size.

A 3-2-1 approach is a useful baseline, not proof of resilience. Choose independent/off-site copies and deletion/ransomware protection against the risks that matter. Keep recovery keys separately protected but accessible during failure.

---

## Implementation and Verification

After choosing storage and recovery targets, configure data placement, backup schedules, protected copies and retention. Verify backups and restore into an isolated target; check usable applications/data, not only a readable file.

Record measured restore results and actual locations privately. A downloaded device backup, replicated volume or committed manifest is not evidence of successful recovery.
