# Backups

> **Part 6 · Operations** — [Part overview](./README.md) · Previous: [Observability](./observability.md) · Next: [Disaster Recovery](./disaster-recovery.md)

Design data ownership and acceptable loss before choosing backup tools or media. Storage keeps data available while it is used; **backups keep independent copies so data can be recovered after deletion, corruption, compromise or the loss of the original device.** Redundancy, snapshots and replication help, but none of them replaces a backup that lives outside the original failure domain and has been restored successfully.

This chapter answers part of the storyline's last question: **how is the homelab recovered?** It compares complete backup setups by cost and recovery value, backup tools by scope, independent copies and retention, and how to verify backups. Storage roles and platforms are covered in [Storage](../02-architecture/storage.md); the order and planning of a full recovery are covered in [Disaster Recovery](./disaster-recovery.md). The recovery targets themselves—[RPO and RTO](../01-foundations/requirements.md#availability-and-recovery-requirements)—are requirements defined in the foundations.

For a backup to help, it must be independent of the failure it is meant to survive and successfully restore the required data. Copies, snapshots and redundancy have different failure coverage; this chapter compares complete home-lab recovery choices. Before buying a device, use the central [cost and scope guidance](../01-foundations/cost.md): start with the least complex design that meets a real need.

---

## Table of Contents

- [Cost and Recovery Comparison](#cost-and-recovery-comparison)
- [Backup and Recovery Choices](#backup-and-recovery-choices)
  - [Recovery Requirements and Backup Scope](#recovery-requirements-and-backup-scope)
  - [Independent Copies and Retention](#independent-copies-and-retention)
- [Implementation and Verification](#implementation-and-verification)
- [Further Reading](#further-reading)

---

## Cost and Recovery Comparison

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

Choosing the primary storage itself—local disks, ZFS, NAS, iSCSI, Ceph or CSI—is described in the [storage design options](../02-architecture/storage.md#design-options), including when a NAS or distributed storage is worth its lifecycle ([start simple and expand only for a need](../02-architecture/storage.md#start-simple-and-expand-only-for-a-need)).

---

## Backup and Recovery Choices

Redundancy supports availability; snapshots aid rollback; backups provide independent recovery. None automatically replaces the others. Git records intent but not all data or device/runtime state.

For network equipment, use the [gateway availability, backup and recovery guidance](../02-architecture/network.md#availability-backups-and-recovery): retain protected local offline and cloud/off-site configuration copies accessible without the gateway or lab. A cold spare, temporary router VM and automatic HA have different costs and recovery guarantees; none is established by backing up workloads alone. This follows the central [lab-scale availability principle](../01-foundations/design-principles.md#learn-enterprise-patterns-without-enterprise-availability).

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

## Further Reading

For general contingency-planning practices, see NIST [SP 800-34 Rev. 1](https://csrc.nist.gov/pubs/sp/800/34/r1/upd1/final). For ransomware-specific backup and recovery recommendations, see CISA's [StopRansomware Guide](https://www.cisa.gov/stopransomware/ransomware-guide). Product-specific backup and restore behavior is linked in the options table above; this chapter focuses on selecting and testing copies for home-lab failure scenarios. See [Goals and Scope](../01-foundations/goals-and-scope.md#define-what-is-out-of-scope).
