# High-Level Design (HLD): Infrastructure

Define boundaries, dependencies and capacity before selecting the platforms that run on them.

---

## Table of Contents

- [Network Foundation](#network-foundation)
- [Compute Foundation](#compute-foundation)
- [Connecting the Layers](#connecting-the-layers)

---

## Network Foundation

[Network](./Network/README.md) covers household/lab boundaries, layered responsibilities, segmentation, gateway/switch choices and safe implementation considerations. [Wi-Fi](./Network/WiFi.md) covers coverage and backhaul as part of that network, not as a second independent routing stack.

Choose which connectivity must remain available independently of experiments, then locate routing, switching, wireless access and policy enforcement. A small network can collapse several logical layers into one switch while still having clear ownership. Segmentation, equipment capabilities and recovery access should be decided together; an attractive gateway cannot implement a policy its switch/AP paths do not support.

---

## Compute Foundation

[Hosts and nodes](./Hosts/README.md) covers purchasing, roles, electricity, capacity, failure domains and virtualization. A hypervisor host cluster belongs here; the [container-orchestration cluster](../3_LLD/Cluster/README.md) is a separate platform concern.

[Provisioning and configuration](./Hosts/Automation.md) describes IaC and configuration-management concepts and tool choices within host lifecycle. Terraform and Ansible are options, not top-level documentation categories.

Physical capacity and failure domains constrain everything in the LLD. Decide reuse versus purchases, consolidated versus independent hosts, and bare metal versus virtualization before allocating logical nodes. Reserve maintenance/recovery capacity and count whole-system power. A hypervisor cluster is a management/availability choice with its own quorum/storage obligations, not an automatic consequence of owning two PCs.

---

## Connecting the Layers

Check which services must survive lab shutdown, which links carry administration/workloads, and which failure domains are shared. Use these constraints to evaluate the [low-level platform and services](../3_LLD/README.md); revisit the foundation if they exceed its resources or availability.

A useful HLD output is a role-based view of dependencies, capacity assumptions, trust boundaries and accepted single points of failure. It does not need every DHCP reservation or application manifest. The private record supplies the chosen equipment and actual mappings; the public reference explains how to reason about them. Use the [project-structure guide](../Project_Structure.md) to keep implementation ownership aligned with that design.
