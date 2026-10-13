# Storage and Backup Design

Choose data placement separately from backup. Redundancy supports availability; snapshots support short-term rollback; backups support recovery from loss, corruption or compromise. None replaces the others.

## Storage options

| Option | Benefits | Drawbacks | Fit |
|---|---|---|---|
| Local disks / ZFS | Simple, fast local I/O; integrity/snapshot features where configured | Host failure limits access; replicas/backups needed separately | Small outage-tolerant labs |
| NAS / NFS / other shared storage | Shared data and VM/volume access | Network/storage appliance is a dependency and possible single point of failure | Deliberate shared-data needs |
| Distributed storage (e.g. Ceph) | Replication and distributed placement | RAM, disks, network, quorum and recovery traffic; complex on small hosts | Sufficient independent hardware and a reason to learn/operate it |
| Kubernetes storage systems / CSI integrations | Workload-oriented volume lifecycle | Durability depends on underlying storage and selected driver | Persistent cluster workloads with a documented failure model |

Do not choose distributed storage merely because several nodes exist. Work out surviving capacity, I/O, replication overhead and rebuild behavior. A CSI driver is not automatically replicated storage or backup.

## Backups and restoration

Set recovery point/time targets and data ownership. Proxmox Backup Server, built-in VM backup tools, file/database backup tools and managed/commercial products solve different scopes. Ensure application consistency; a live filesystem copy is not necessarily a consistent database backup.

Keep independent copies, including a protected off-site copy where loss of the home location matters. A 3-2-1 approach is a useful planning baseline, not proof of resilience. Protect against deletion/ransomware, encrypt where appropriate, and store decryption/recovery material separately but recoverably.

Back up VM data, application volumes/databases, cluster state where needed, and device configuration. Git holds declared intent but not all runtime data, private keys, Terraform state or raw device settings. An encrypted backup without its key is not a recovery plan.

Test restore into a safe isolated target and record the results privately. Verification of a backup file is valuable but is not proof that applications can be recovered. See [setup and operations](../2_Setup/Services_Operations.md#backup-and-recovery).
