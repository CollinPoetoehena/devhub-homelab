# Storage and Backups

Design data ownership and acceptable loss/downtime before choosing disks or storage platforms.

DevHub's [Storage Architecture reference](https://github.com/CollinPoetoehena/DevHub/blob/main/reference/storage/Storage_Architecture.md) explains general media and backup concepts. This guide applies them to homelab choices. Before buying a device, also use the central [cost and scope guidance](../../1_Goals/Cost_and_Scope.md): start with the least complex design that meets a real need.

The HLD [Systems Storage guide](../../2_HLD/Systems/Storage.md) covers the physical storage roles and broad architecture; this LLD focuses on backup, recovery and their operational trade-offs.

---

## Table of Contents

- [Design Options](#design-options)
- [Cost and Recovery Comparison](#cost-and-recovery-comparison)
- [Backup and Recovery Choices](#backup-and-recovery-choices)
  - [Recovery Requirements and Backup Scope](#recovery-requirements-and-backup-scope)
  - [Independent Copies and Retention](#independent-copies-and-retention)
- [Implementation and Verification](#implementation-and-verification)

---

## Design Options

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| Local disks | Host-attached workload storage | Simple I/O and few network dependencies | Host loss affects access; volumes may pin workloads; independent recovery copies required. **Maintenance:** Monitor wear, errors and capacity; plan replacement and independent restore rather than assuming a host restart recovers failed storage. | Disposable VMs or outage-tolerant services |
| [ZFS](https://openzfs.github.io/openzfs-docs/) | Filesystem/storage management with checksums, snapshots and configurable redundancy | Integrity detection and local rollback/redundancy features | Capacity, redundancy, monitoring and independent backups; redundancy is not a backup. **Maintenance:** Monitor pool health, scrubs and capacity; replace failed devices through supported procedures and test independent backup restoration. | Supported hosts needing its integrity/storage features |
| NAS / shared file storage | Network appliance/server exposing files, commonly through NFS or SMB | Shared access and separate storage/compute capacity | Appliance/network failure domain, permissions and protocol/locking management. **Maintenance:** Patch the storage appliance, manage permissions and monitor disks/network; test exports and recovery before compute depends on them. | Shared files or supported shared VM storage |
| iSCSI / shared block storage | Network-delivered block devices | Block-storage integration for supported consumers | Correct ownership/locking/filesystem integration; not interchangeable with shared file access. **Maintenance:** Maintain initiator/target compatibility, access controls and path health; validate safe ownership and recovery after network/storage changes. | A platform explicitly supporting the chosen block-storage design |
| [Ceph](https://docs.ceph.com/en/latest/) / distributed storage | Replicated storage spread across nodes/devices | Distributed placement and availability within designed failure domains | Disk, network, quorum, replication and rebuild overhead; budget usable/surviving capacity. **Maintenance:** substantial cluster-health, disk-replacement, rebalance and version-coordination work; watch capacity before failures force rebuilds | Sufficient independent hardware and deliberate distributed-storage needs |
| [Kubernetes CSI](https://kubernetes.io/docs/concepts/storage/volumes/#csi) integrations | Driver-mediated volume lifecycle over a backend | Attachment, supported expansion, snapshots and application integration | Durability comes from backend design; verify access modes, node loss and backup coverage. **Maintenance:** Keep drivers, node components and backend credentials compatible; monitor volume lifecycle and test application-consistent restores. | Persistent Kubernetes workloads with a supported recovery model |

Size capacity, endurance, I/O, replication overhead and surviving/rebuild capacity. Owning several hosts is not by itself a reason to run distributed storage.

### Cost and Recovery Comparison

This comparison is intentionally about the **whole recovery design**, not just the price per terabyte. Relative cost is qualitative because device prices, capacities, used-market quality, electricity rates, and cloud fees vary by location and time. Include purchase cost, additional power, replacement media, ongoing administration and restore testing. A low-cost drive that is never checked may be less useful than a smaller backup that is known to restore.

| Setup | Primary data | Automated backup | Protection if a backup job/drive fails | Offline / off-site copy | Approximate cost and ongoing effort | Recovery limits | When it fits |
|---|---|---|---|---|---|---|---|
| No backup | Local SSD in a host | No | None; host/disk loss can lose data | None | Lowest purchase cost; least backup work, highest data-loss exposure | Rebuild only if data and configuration are reproducible; unique state may be lost | Disposable experiments with no valuable state |
| One external HDD | Local SSD; HDD holds backup | Yes, while connected | Job alerts, verification and restore tests can catch some failures; the sole backup drive can still fail or be compromised | Not offline while connected; no off-site copy | Low additional hardware cost; modest job/health-check effort and one extra device | SSD and HDD can still share host, power, location and online deletion risk | Small lab protecting data against a primary disk/host failure |
| Two external HDDs | Local SSD plus a regularly connected backup HDD | Routine automated backup to primary HDD | A second HDD can hold a periodic copy, if the copy succeeds and is checked | One drive can be disconnected and stored separately; still same-site unless moved off-site | Moderate one-time cost; manual/automated rotation and periodic verification | Not full 3-2-1 without an off-site copy; offline copy may be stale or damaged | Cost-conscious lab wanting protection from some online deletion and drive failures |
| Two-bay NAS with two HDDs | NAS volume, often mirrored, plus client-local data as needed | NAS backup/snapshot jobs can be scheduled | RAID/mirroring supports availability for one-disk failure if configured; separate backup remains required | Not automatically offline or off-site | Higher initial cost (appliance, two drives and perhaps extra network gear), plus continuous electricity and appliance maintenance | NAS, credentials, network, power and site may be shared failure domains; RAID is not backup | Several clients need shared files or a genuine storage service |
| Ceph/distributed storage | Data distributed across multiple nodes/disks | Replication/placement is usually automatic | Can remain available through designed node/disk failures when quorum, replicas and spare capacity are sufficient | Replicas are online and may share a site; independent backup still required | Highest equipment, power, network and operational complexity among these examples | Misconfiguration, correlated failure, full cluster/site loss, deletion or corruption can affect all replicas | Sustained distributed-storage learning or workload availability requirements justify the hardware and operations |

The table is not a maturity ladder. A small lab can deliberately select local SSDs plus independent backups rather than progress to a NAS or Ceph. A backup job's own failure handling needs alerts, enough retention and a restore test; having two backup drives does not automatically make the copy process safe.

For the value of data, define RPO (acceptable data-loss interval) and RTO (acceptable restoration time) before choosing backup frequency and media. The common 3-2-1 rule means three copies, two media types, one off-site. An SSD plus two HDDs can mimic the copy/media/offline parts, but if every drive stays in the same home it does **not** provide an off-site copy and should not be described as complete 3-2-1 protection.

### Start Simple and Expand Only for a Need

For many learning homelabs, local SSDs for active workloads plus one external HDD for automated backups is enough to learn hot versus cold storage, backup scheduling, integrity checking and restore procedures. A second HDD can provide an offline periodic copy. That protects against more risks than having no backup without requiring a separate storage appliance.

A NAS is useful when multiple clients need shared files, a service needs supported network storage, or the owner wants to learn NAS operation as an ongoing project. It is not automatically required because multiple compute nodes exist. A NAS adds another host/appliance, drives, power, software, accounts, network dependencies and maintenance, and costs more than local SSDs and a simple external backup. Likewise, Ceph is valuable for explicit distributed-storage goals, but needs enough independent nodes and time for failure/rebuild operations.

Do not buy advanced hardware simply because it is the most capable option or because one short experiment sounds interesting. Explore the architecture in theory or create a temporary VM/experiment on equipment already available, then remove it when the learning objective is complete. The [central cost and scope guide](../../1_Goals/Cost_and_Scope.md) explains this budget principle in more detail.

### Enterprise Designs (Illustration Only)

> **Illustration only:** The following describes common enterprise patterns to explain the concepts. It is not a recommendation or requirement for a homelab; the simpler options above may be entirely sufficient.

Larger organizations commonly separate compute from shared storage, use redundant controllers/paths, protect data through policy-controlled snapshots and off-site/immutable backup, monitor capacity/health centrally, and document tested recovery objectives. Virtualization platforms may consume shared block or distributed storage; applications may use their own databases and object stores. A separate backup platform/account and an independent recovery location help limit shared failure and credential risks.

This brings cost, power, network, staffing and operational responsibilities. "Enterprise" is not one design and not every organization has every component. Replicating the visible architecture without its failure domains, capacity planning, operations, security and recovery processes does not provide equivalent resilience. Learn the pieces that support an actual homelab goal and state clearly which failures the small setup accepts.

Recreating a container elsewhere does not recreate its local data. Choose supported file/block integrations rather than treating all shared storage as interchangeable. Small or heterogeneous distributed-storage labs can teach the platform without meeting production-like resilience; replicas on guests sharing one disk are not independent copies. A CSI driver's installation does not establish data protection: understand physical placement and application lifecycle first.

---

## Backup and Recovery Choices

Redundancy supports availability; snapshots aid rollback; backups provide independent recovery. None automatically replaces the others. Git records intent but not all data or device/runtime state.

For network equipment, use the [gateway availability, backup and recovery guidance](../../2_HLD/Network/README.md#availability-backups-and-recovery): retain protected local offline and cloud/off-site configuration copies accessible without the gateway or lab. A cold spare, temporary router VM and automatic HA have different costs and recovery guarantees; none is established by backing up workloads alone. This follows the central [lab-scale availability principle](../../1_Goals/README.md#learn-enterprise-patterns-without-enterprise-availability).

Choose backup tools by scope. Check application consistency, encryption, retention, keys and restore workflow rather than only archive size.

A 3-2-1 approach is a useful baseline, not proof of resilience. Choose independent/off-site copies and deletion/ransomware protection against the risks that matter. Keep recovery keys separately protected but accessible during failure.

### Recovery Requirements and Backup Scope

Define **RPO** as the acceptable data-loss window and **RTO** as the acceptable recovery time. They determine job frequency, independent copies and how much restoration work is acceptable. Different data can have different requirements: disposable images may be rebuilt, while personal files or a database need a real recovery copy.

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| [Proxmox Backup Server](https://pbs.proxmox.com/docs/) | Backup platform for supported VM/container/host scopes | Incremental deduplication, verification and platform integration | Backup-server/storage/key recovery and supported application consistency; restored disks still need dependencies. **Maintenance:** review failed jobs, verification, prune/garbage-collection schedules and capacity; periodically restore a representative guest | Proxmox-oriented system recovery |
| Built-in platform VM backups | Hypervisor-supported backup/export workflow | Fewer extra components and supported guest recovery | Features/retention/off-site integration vary; disk copies alone may not be application-consistent. **Maintenance:** Monitor schedules, retention and target capacity; test representative restores and recheck application consistency after guest changes. | Small installations with verified platform backup coverage |
| [Restic](https://restic.readthedocs.io/en/stable/) | Encrypted file/snapshot backup tool with repository backends | Portable file recovery and configurable storage targets | Permissions, repository credentials, retention and recovery keys; coordinate live application data. **Maintenance:** schedule/check jobs and repository integrity, manage retention and test recovery with independently available keys | File-oriented protected backups |
| [Borg](https://borgbackup.readthedocs.io/en/stable/) | Deduplicating archive backup tool with encryption support | Efficient file/archive workflows on supported targets | Repository/access/key management and restore validation; not automatic database consistency. **Maintenance:** Schedule/check archives and repository integrity, manage retention and preserve keys; test permissions/data recovery onto an isolated target. | File backup fitting its platform and target model |
| Native database backups | Supported dumps or physical/log-based database recovery | Application-aware recovery and possible point-in-time workflows | Database-specific consistency, log retention, credentials and restore compatibility. **Maintenance:** Monitor jobs/log retention and validate backups with the actual database version; test application recovery and point-in-time paths where required. | Important stateful databases; not merely a live disk copy |
| [Velero](https://velero.io/docs/) and compatible data integrations | Kubernetes resource backup plus supported volume/data workflows | Cluster-aware backup/restore composition | Verify plugins/backend/application coverage; Git alone omits persistent data. **Maintenance:** Coordinate plugins, backend access and cluster APIs; check backup coverage and test resource plus persistent-data restoration. | Kubernetes recovery with explicit tested data coverage |
| Protected device exports | Supported network/device configuration backup | Restores settings outside VM backup scope | Exports can include secrets; keep accessible without the lab and test supported restore/re-adoption. **Maintenance:** Refresh exports after significant changes, protect secrets and retain compatible restore instructions; test recovery without the lab/cloud dependency. | Gateway/switch/controller recovery |
| Managed/commercial backup systems | Provider/vendor backup tooling for supported scopes | May offload operations or add supported integrations | Cost, privacy, provider availability, export and key/recovery responsibility. **Maintenance:** Review job failures, agent/provider updates, billing and retention; test retrieval and restoration rather than assuming vendor status proves recoverability. | Required support/scope justifies another service/tool |

### Independent Copies and Retention

Keep copies outside the original failure domain. Another directory on the same disk is not an independent copy; replicas can propagate deletion or corruption. Off-site storage reduces a different risk, while offline/immutable copies can reduce deletion/ransomware exposure if credentials and retention controls are suitably separated.

| Option | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| Independent local copy | Backup on storage outside the original disk/failure domain | Faster local recovery | Can still share building, power or credentials. **Maintenance:** Monitor target health/capacity and successful jobs; verify restore access even when the original host is unavailable. | Routine restores alongside additional protection where needed |
| Off-site copy | Recovery data stored outside the primary location | Reduces location-disaster risk | Transfer cost/time, privacy, key management and retrieval dependencies. **Maintenance:** Check transfer completion, storage costs and key/access recovery; periodically retrieve data to validate realistic restoration time. | Important data must survive loss of the local site |
| Offline copy | Backup disconnected between controlled updates | Reduces online deletion/ransomware paths | Manual handling, stale-copy risk and secure storage. **Maintenance:** Requires deliberate update/rotation and secure handling; test media readability and track the age of the newest usable copy. | Deliberate periodic independent recovery |
| Immutable / retention-protected copy | Supported retention prevents ordinary deletion/modification | Limits damage from compromised routine credentials | Verify real enforcement, administrative bypass, retention capacity and separate privileges. **Maintenance:** Review enforced retention, capacity and privileged bypass paths; test restoration and keep deletion-capable identities separate. | Deletion protection with tested provider/storage controls |

Retention must fit capacity and restore needs. Deduplication/compression depend on the data, so do not budget from optimistic ratios alone. Encryption protects stored copies, but losing the key can also make a good backup unusable; record protected recovery access without committing keys or actual storage endpoints.

---

## Implementation and Verification

After choosing storage and recovery targets, configure data placement, backup schedules, protected copies and retention. Verify backups and restore into an isolated target; check usable applications/data, not only a readable file.

Record measured restore results and actual locations privately. A downloaded device backup, replicated volume or committed manifest is not evidence of successful recovery.

Exercise recovery in an isolated target, including application startup, permissions and representative data checks. Measure restoration time against RTO and verify the newest usable recovery point against RPO. Never restore onto the only live copy merely to test a script.

Plan recovery order: protected keys and basic access, storage/platform dependencies, data, then application validation. If the backup server runs inside the environment it protects, document how to recover it first. Automate scheduling, integrity checks and failure notifications, but retain a readable manual recovery path.
