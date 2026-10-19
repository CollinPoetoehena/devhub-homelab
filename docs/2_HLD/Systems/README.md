# Systems Foundation: Compute and Storage

Compute hosts and storage are closely related infrastructure decisions: most systems use local storage inside the compute host, while shared or distributed storage can add separate machines, network dependencies and power. Consider those responsibilities and failure domains together, while keeping the detailed design in two focused pages: [Compute](./Compute.md) and [Storage](./Storage.md).

---

## Table of Contents

- [Compute](#compute)
- [Storage](#storage)
- [Relationship to LLD](#relationship-to-lld)

---

## Compute

[Compute](./Compute.md) covers host roles, capacity, form factors, virtualization and whole-system power. It explains how to size physical hosts before assigning VMs or cluster nodes.

---

## Storage

[Storage](./Storage.md) covers the broad HLD choice between local storage in a compute host, external backup devices, NAS/shared storage and distributed storage. The local SSD is commonly part of the compute system, which is why these HLD concerns are grouped here; their requirements and failure domains should still be considered explicitly.

---

## Relationship to LLD

HLD establishes physical responsibilities and accepted failure domains. The [LLD Storage and Backups guide](../../3_LLD/Storage_Backups/README.md) covers backup/recovery options and operational considerations; it complements, rather than replaces, the HLD storage choice. Review the central [cost and scope guidance](../../1_Goals/Cost_and_Scope.md) before buying equipment.
