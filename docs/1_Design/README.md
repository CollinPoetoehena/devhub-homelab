# Homelab Design Reference

These documents explain **how to choose**, not what is installed in my home. Start with goals, compare options against constraints, and keep the actual decision record in the private repository under the [repository guidelines](../Repository_Guidelines.md#writing-design-and-setup-documentation).

| Area | Reference |
|---|---|
| Purpose and constraints | [Goals](./0_Design_Goals.md) |
| Buying devices and sizing capacity | [Hardware](./1_Design_Hardware_Prerequisites.md) |
| Platforms, operating systems and tooling | [Software options](./2_Design_Software_Prerequisites.md) |
| Gateway boundaries, network layers and segmentation | [Network](./3_Design_Network.md) |
| Wireless coverage and backhaul | [Wi-Fi](./WiFi.md) |
| Virtual machines and host lifecycle | [Hypervisor](./4_Design_Hypervisor.md) |
| Node roles, capacity and failure domains | [Hosts and nodes](./5_Design_Hosts_Nodes.md) |
| Kubernetes, networking and lifecycle | [Cluster](./6_Design_Cluster.md) |
| Signals, placement and alerting | [Monitoring](./7_Design_Monitoring.md) |
| Persistent data and recovery | [Storage and backups](./8_Design_Storage_Backups.md) |

Tool selection is centralized in the software reference where useful; domain documents apply those choices to homelab architecture. Broad [network fundamentals](https://github.com/CollinPoetoehena/DevHub/tree/main/reference/network) and [hardware inspection](https://github.com/CollinPoetoehena/DevHub/blob/main/reference/os_hardware/Hardware_Specs.md#how-to-check-your-hardware-specs) stay in DevHub.

After selecting a design, use the [setup reference](../2_Setup/README.md). Do not treat an example diagram as an inventory or a planning checklist as live verification.
