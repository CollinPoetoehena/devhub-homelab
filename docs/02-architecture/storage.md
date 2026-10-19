# Storage

> **Part 2 · Architecture** — [Part overview](./README.md) · Previous: [Compute](./compute.md) · Next: [Platform](./platform.md)

Storage is often physically part of the compute host: a mini PC's local SSD can hold its hypervisor, VM disks and frequently used data. A separate NAS or distributed-storage system changes the failure domains, cost, power and maintenance of the design. Decide what must be stored, where it is accessed, and how it can be recovered before selecting a device.

This chapter covers *where data lives while it is being used*: storage roles, storage platforms and how to match them to real needs. Keeping additional, independent copies is a different responsibility—storage provides capacity and availability, backups provide recovery—so backup tools, retention and the cost of complete backup setups are described in [Backups](../06-operations/backups.md), and recovery planning in [Disaster Recovery](../06-operations/disaster-recovery.md). This chapter links to them wherever the two meet.

See the [architecture overview](./README.md#compute-and-storage-together) for why compute and storage are designed together, and the central [cost and scope guidance](../01-foundations/cost.md) before buying. DevHub's [Storage Architecture](https://github.com/CollinPoetoehena/DevHub/blob/main/reference/storage/Storage_Architecture.md) covers general storage concepts; this chapter applies them to home-and-lab choices.

---

## Table of Contents

- [Storage Roles](#storage-roles)
- [Design Options](#design-options)
- [Match the Design to the Need](#match-the-design-to-the-need)
  - [Start Simple and Expand Only for a Need](#start-simple-and-expand-only-for-a-need)
  - [Enterprise Designs (Illustration Only)](#enterprise-designs-illustration-only)
- [Backups and Recovery](#backups-and-recovery)
- [Implementation and Verification](#implementation-and-verification)

---

## Storage Roles

Separate the role of primary storage from the role of a recovery copy:

- **Local SSD in a compute host:** a simple, fast primary location for hypervisor, VM and container data. It avoids another always-on appliance, but access depends on that host and local data needs an independent recovery copy.
- **External HDD:** an inexpensive backup target for scheduled copies. It can be disconnected between runs to reduce exposure to host, software or accidental deletion failures. A USB HDD is not shared storage for active multi-host workloads.
- **NAS or network storage:** a separate appliance or server exposing supported file or block storage. It can serve multiple clients or workloads that require a shared storage service, but adds purchase, power, network, software, access-control and maintenance dependencies.
- **Distributed storage:** data placed across multiple hosts for a deliberate availability or learning requirement. It requires enough genuinely independent nodes/disks, network capacity, spare capacity and operational time; several VMs on one physical disk do not create independent failure domains.

---

## Design Options

Design data ownership and acceptable loss/downtime before choosing disks or storage platforms. The roles above describe *why* a piece of storage exists; the options below describe *how* it is provided. Several can coexist—for example, ZFS on local disks for VM storage, with a Kubernetes CSI integration exposing part of it to workloads.

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| Local disks | Host-attached workload storage | Simple I/O and few network dependencies | Host loss affects access; volumes may pin workloads; independent recovery copies required. **Maintenance:** Monitor wear, errors and capacity; plan replacement and independent restore rather than assuming a host restart recovers failed storage. | Disposable VMs or outage-tolerant services |
| [ZFS](https://openzfs.github.io/openzfs-docs/) | Filesystem/storage management with checksums, snapshots and configurable redundancy | Integrity detection and local rollback/redundancy features | Capacity, redundancy, monitoring and independent backups; redundancy is not a backup. **Maintenance:** Monitor pool health, scrubs and capacity; replace failed devices through supported procedures and test independent backup restoration. | Supported hosts needing its integrity/storage features |
| NAS / shared file storage | Network appliance/server exposing files, commonly through NFS or SMB | Shared access and separate storage/compute capacity | Appliance/network failure domain, permissions and protocol/locking management. **Maintenance:** Patch the storage appliance, manage permissions and monitor disks/network; test exports and recovery before compute depends on them. | Shared files or supported shared VM storage |
| iSCSI / shared block storage | Network-delivered block devices | Block-storage integration for supported consumers | Correct ownership/locking/filesystem integration; not interchangeable with shared file access. **Maintenance:** Maintain initiator/target compatibility, access controls and path health; validate safe ownership and recovery after network/storage changes. | A platform explicitly supporting the chosen block-storage design |
| [Ceph](https://docs.ceph.com/en/latest/) / distributed storage | Replicated storage spread across nodes/devices | Distributed placement and availability within designed failure domains | Disk, network, quorum, replication and rebuild overhead; budget usable/surviving capacity. **Maintenance:** substantial cluster-health, disk-replacement, rebalance and version-coordination work; watch capacity before failures force rebuilds | Sufficient independent hardware and deliberate distributed-storage needs |
| [Kubernetes CSI](https://kubernetes.io/docs/concepts/storage/volumes/#csi) integrations | Driver-mediated volume lifecycle over a backend | Attachment, supported expansion, snapshots and application integration | Durability comes from backend design; verify access modes, node loss and backup coverage. **Maintenance:** Keep drivers, node components and backend credentials compatible; monitor volume lifecycle and test application-consistent restores. | Persistent Kubernetes workloads with a supported recovery model |

Size capacity, endurance, I/O, replication overhead and surviving/rebuild capacity. Owning several hosts is not by itself a reason to run distributed storage.

---

## Match the Design to the Need

> The [cost and scope guide](../01-foundations/cost.md) is a central design constraint: start simple, count purchase and running costs, and add hardware only when an actual use case justifies the extra lifecycle.

For a small lab, local SSDs with a separate external-drive backup are often sufficient. A single mini PC can provide both compute and primary storage, with an external HDD providing a distinct backup device. Add a NAS only when shared access, capacity or a specific workload justifies owning another system. Add distributed storage only when its availability or learning goals justify the hardware, network and maintenance requirements.

Do not equate a separate device, RAID, snapshot or replication with a backup automatically. Assess what failures each copy can survive, who can delete it, how often it is refreshed and whether it can be restored. Storage is not safer merely because it is more elaborate.

### Start Simple and Expand Only for a Need

For many learning homelabs, local SSDs for active workloads plus one external HDD for automated backups is enough to learn hot versus cold storage, backup scheduling, integrity checking and restore procedures. A second HDD can provide an offline periodic copy. That protects against more risks than having no backup without requiring a separate storage appliance.

A NAS is useful when multiple clients need shared files, a service needs supported network storage, or the owner wants to learn NAS operation as an ongoing project. It is not automatically required because multiple compute nodes exist. A NAS adds another host/appliance, drives, power, software, accounts, network dependencies and maintenance, and costs more than local SSDs and a simple external backup. Likewise, Ceph is valuable for explicit distributed-storage goals, but needs enough independent nodes and time for failure/rebuild operations.

Do not buy advanced hardware simply because it is the most capable option or because one short experiment sounds interesting. Explore the architecture in theory or create a temporary VM/experiment on equipment already available, then remove it when the learning objective is complete. The [central cost and scope guide](../01-foundations/cost.md) explains this budget principle in more detail.

### Enterprise Designs (Illustration Only)

> **Illustration only:** The following describes common enterprise patterns to explain the concepts. It is not a recommendation or requirement for a homelab; the simpler options above may be entirely sufficient.

Larger organizations commonly separate compute from shared storage, use redundant controllers/paths, protect data through policy-controlled snapshots and off-site/immutable backup, monitor capacity/health centrally, and document tested recovery objectives. Virtualization platforms may consume shared block or distributed storage; applications may use their own databases and object stores. A separate backup platform/account and an independent recovery location help limit shared failure and credential risks.

This brings cost, power, network, staffing and operational responsibilities. "Enterprise" is not one design and not every organization has every component. Replicating the visible architecture without its failure domains, capacity planning, operations, security and recovery processes does not provide equivalent resilience. Learn the pieces that support an actual homelab goal and state clearly which failures the small setup accepts.

Recreating a container elsewhere does not recreate its local data. Choose supported file/block integrations rather than treating all shared storage as interchangeable. Small or heterogeneous distributed-storage labs can teach the platform without meeting production-like resilience; replicas on guests sharing one disk are not independent copies. A CSI driver's installation does not establish data protection: understand physical placement and application lifecycle first.

---

## Backups and Recovery

This section is the hand-off from storage architecture to the operations part. Choose backup scope and frequency from how much data loss is acceptable, and choose recovery steps from how quickly services must return. Include application consistency, configuration, encryption keys and a tested restore—not just successful copy jobs. The [Backups](../06-operations/backups.md) chapter compares complete backup setups by [cost and recovery value](../06-operations/backups.md#cost-and-recovery-comparison), backup tools, retention and verification in detail; [Disaster Recovery](../06-operations/disaster-recovery.md) covers recovery order and rehearsal.

Start with the simplest storage and backup design that meets the workload and recovery need. Revisit it when measurements or a sustained requirement demonstrate a gap, not simply because a more complex option exists.

---

## Implementation and Verification

After choosing the storage roles and platform, prepare and verify the storage before workloads depend on it. Check new and used disks for health and wear, choose filesystems and redundancy deliberately, and record which workloads and data live on which device. Keep capacity headroom for snapshots, backup staging and growth; a nearly full disk or pool is harder to recover and can slow every guest on the host.

Monitor wear, errors, pool or array health and free capacity through [observability](../06-operations/observability.md), and plan disk replacement before a failure forces it. Test that a workload can start again after its host or volume is restarted, and—if shared or distributed storage is used—what happens when the storage service or a node is unavailable. Storage redundancy is verified separately from backups: a degraded-array test proves availability behavior, while only a [restore test](../06-operations/backups.md#implementation-and-verification) proves recovery.

Actual devices, capacities, pool layouts and measured results belong in the private record; this chapter explains the reasoning and the checks.
