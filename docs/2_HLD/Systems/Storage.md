# Storage

Storage is often physically part of the compute host: a mini PC's local SSD can hold its hypervisor, VM disks and frequently used data. A separate NAS or distributed-storage system changes the failure domains, cost, power and maintenance of the design. Decide what must be stored, where it is accessed, and how it can be recovered before selecting a device.

See the [systems overview](./README.md) for why compute and storage are grouped in the HLD, and the central [cost and scope guidance](../../1_Goals/Cost_and_Scope.md) before buying. DevHub's [Storage Architecture](https://github.com/CollinPoetoehena/DevHub/blob/main/reference/storage/Storage_Architecture.md) covers general storage concepts; this page applies them to home-and-lab choices.

---

## Table of Contents

- [Storage Roles](#storage-roles)
- [Match the Design to the Need](#match-the-design-to-the-need)
- [Backups and Recovery](#backups-and-recovery)

---

## Storage Roles

Separate the role of primary storage from the role of a recovery copy:

- **Local SSD in a compute host:** a simple, fast primary location for hypervisor, VM and container data. It avoids another always-on appliance, but access depends on that host and local data needs an independent recovery copy.
- **External HDD:** an inexpensive backup target for scheduled copies. It can be disconnected between runs to reduce exposure to host, software or accidental deletion failures. A USB HDD is not shared storage for active multi-host workloads.
- **NAS or network storage:** a separate appliance or server exposing supported file or block storage. It can serve multiple clients or workloads that require a shared storage service, but adds purchase, power, network, software, access-control and maintenance dependencies.
- **Distributed storage:** data placed across multiple hosts for a deliberate availability or learning requirement. It requires enough genuinely independent nodes/disks, network capacity, spare capacity and operational time; several VMs on one physical disk do not create independent failure domains.

---

## Match the Design to the Need

> The [cost and scope guide](../../1_Goals/Cost_and_Scope.md) is a central design constraint: start simple, count purchase and running costs, and add hardware only when an actual use case justifies the extra lifecycle.

For a small lab, local SSDs with a separate external-drive backup are often sufficient. A single mini PC can provide both compute and primary storage, with an external HDD providing a distinct backup device. Add a NAS only when shared access, capacity or a specific workload justifies owning another system. Add distributed storage only when its availability or learning goals justify the hardware, network and maintenance requirements.

Do not equate a separate device, RAID, snapshot or replication with a backup automatically. Assess what failures each copy can survive, who can delete it, how often it is refreshed and whether it can be restored. Storage is not safer merely because it is more elaborate.

---

## Backups and Recovery

Choose backup scope and frequency from how much data loss is acceptable, and choose recovery steps from how quickly services must return. Include application consistency, configuration, encryption keys and a tested restore—not just successful copy jobs. The LLD [Storage and Backups](../../3_LLD/Storage_Backups/README.md) guide compares backup approaches, retention, recovery and cost in more detail.

Start with the simplest storage and backup design that meets the workload and recovery need. Revisit it when measurements or a sustained requirement demonstrate a gap, not simply because a more complex option exists.
