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

---

## Compute Foundation

[Hosts and nodes](./Hosts/README.md) covers purchasing, roles, electricity, capacity, failure domains and virtualization. A hypervisor host cluster belongs here; the [container-orchestration cluster](../3_LLD/Cluster/README.md) is a separate platform concern.

[Provisioning and configuration](./Hosts/Automation.md) describes IaC and configuration-management concepts and tool choices within host lifecycle. Terraform and Ansible are options, not top-level documentation categories.

---

## Connecting the Layers

Check which services must survive lab shutdown, which links carry administration/workloads, and which failure domains are shared. Use these constraints to evaluate the [low-level platform and services](../3_LLD/README.md); revisit the foundation if they exceed its resources or availability.
