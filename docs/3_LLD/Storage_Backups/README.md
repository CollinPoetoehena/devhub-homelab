# Storage and Backups

Design data ownership and acceptable loss/downtime before choosing disks or storage platforms.

---

## Table of Contents

- [Design Options](#design-options)
- [Backup and Recovery Choices](#backup-and-recovery-choices)
  - [Recovery Requirements and Backup Scope](#recovery-requirements-and-backup-scope)
  - [Independent Copies and Retention](#independent-copies-and-retention)
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

**Local disks** minimize dependencies and can be enough for disposable VMs or outage-tolerant services. ZFS can add checksums, snapshots and chosen redundancy, but still needs suitable capacity, monitoring and independent backups. A local volume may pin a workload to its host; recreating its container elsewhere does not recreate the data.

**NAS/shared storage** makes files or VM disks available across hosts and can separate storage capacity from compute purchases. It introduces a storage-appliance/network failure domain and permissions/protocol management. NFS file access and block storage such as iSCSI have different consistency, locking and administration requirements; choose supported integrations rather than treating all shared storage as interchangeable.

**Distributed storage**, such as Ceph, trades extra disks, replication, network traffic and operational complexity for distributed placement/availability. Plan usable rather than raw capacity, surviving replicas/quorum and rebuild load. Very small or heterogeneous setups can teach the platform without meeting production-like performance/resilience. Do not place all replicas on guests sharing one underlying disk and call that independent storage.

**CSI-backed Kubernetes volumes** connect workloads to a storage backend. Compare attachment/access modes, expansion, snapshots, node-loss behavior and backup integration. A driver's installation does not establish data protection; understand the physical backend and application lifecycle first.

---

## Backup and Recovery Choices

Redundancy supports availability; snapshots aid rollback; backups provide independent recovery. None automatically replaces the others. Git records intent but not all data or device/runtime state.

Proxmox Backup Server and built-in VM backup tools suit virtualization; file/database tools and managed/commercial systems cover other scopes. Check application consistency, encryption, retention, keys and restore workflow rather than only archive size.

A 3-2-1 approach is a useful baseline, not proof of resilience. Choose independent/off-site copies and deletion/ransomware protection against the risks that matter. Keep recovery keys separately protected but accessible during failure.

### Recovery Requirements and Backup Scope

Define **RPO** as the acceptable data-loss window and **RTO** as the acceptable recovery time. They determine job frequency, independent copies and how much restoration work is acceptable. Different data can have different requirements: disposable images may be rebuilt, while personal files or a database need a real recovery copy.

| Scope | Options | Important limit |
|---|---|---|
| VM/system backup | Proxmox Backup Server, platform backup tools, suitable commercial tooling | Restored disks may still need external dependencies and application consistency |
| Files | Restic, Borg and other file-backup workflows | Include permissions, encryption keys and actual restore tests |
| Databases | Native dumps or supported physical/log backup workflows | A live disk copy is not automatically a consistent database backup |
| Kubernetes | Manifests plus volume/application backups, e.g. compatible Velero integrations | Git alone omits persistent data; plugins/backend coverage must be verified |
| Devices/configuration | Protected exports and documented supported restore | Raw exports can include secrets and are not ordinary Git artifacts |

### Independent Copies and Retention

Keep copies outside the original failure domain. Another directory on the same disk is not an independent copy; replicas can propagate deletion or corruption. Off-site storage reduces a different risk, while offline/immutable copies can reduce deletion/ransomware exposure if credentials and retention controls are suitably separated.

Retention must fit capacity and restore needs. Deduplication/compression depend on the data, so do not budget from optimistic ratios alone. Encryption protects stored copies, but losing the key can also make a good backup unusable; record protected recovery access without committing keys or actual storage endpoints.

---

## Implementation and Verification

After choosing storage and recovery targets, configure data placement, backup schedules, protected copies and retention. Verify backups and restore into an isolated target; check usable applications/data, not only a readable file.

Record measured restore results and actual locations privately. A downloaded device backup, replicated volume or committed manifest is not evidence of successful recovery.

Exercise recovery in an isolated target, including application startup, permissions and representative data checks. Measure restoration time against RTO and verify the newest usable recovery point against RPO. Never restore onto the only live copy merely to test a script.

Plan recovery order: protected keys and basic access, storage/platform dependencies, data, then application validation. If the backup server runs inside the environment it protects, document how to recover it first. Automate scheduling, integrity checks and failure notifications, but retain a readable manual recovery path.
