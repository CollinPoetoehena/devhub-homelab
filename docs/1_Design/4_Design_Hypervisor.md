# Hypervisor Design

Decide whether virtualization is needed before choosing a platform. VMs provide separate OS lifecycles, snapshots and rebuildable environments; bare-metal containers can be simpler. See [software choices](./2_Design_Software_Prerequisites.md#hypervisor-and-host-os) for Proxmox, XCP-ng, VMware, libvirt and Hyper-V.

## Architecture choices

| Pattern | Benefit | Cost / limitation | Fit |
|---|---|---|---|
| One host | Simple, low idle cost, sufficient for many learning workloads | All guests share one physical failure domain | Starting small or accepting outages |
| Independent hosts | Extra capacity without cluster coupling | Manual placement/migration workflows | Modest expansion without HA requirements |
| Hypervisor cluster | Central management, migration and possible HA | Quorum, fencing, storage, network and surviving capacity requirements | Deliberate cluster lifecycle/availability needs |

A hypervisor cluster and a Kubernetes cluster are separate systems. Several Kubernetes VMs on one host do not provide physical HA. Two clustered hosts alone do not guarantee quorum or safe failover; use platform-supported witness designs only when appropriate.

Separate host administration from guest traffic using VLAN-aware bridges or appropriately isolated interfaces. Do not expose host management merely because a guest needs a workload network. Decide storage placement, template ownership, disk headroom, CPU model/passthrough, update windows and recovery before automation.

Use representative measurements for resource sizing. CPU overcommit can be useful but does not create capacity; memory overcommit and ballooning require workload support and monitoring. CPU passthrough may improve performance but restrict migration between unlike CPUs.

Back up guests and host configuration with an explicit restore plan. Snapshots are rollback aids on the same storage, not independent backups. See [storage and backups](./8_Design_Storage_Backups.md) and [compute setup](../2_Setup/Compute_Cluster.md).

## VM CPU settings

| Setting | Meaning | Homelab selection criterion |
|---|---|---|
| Sockets and cores | Virtual CPU topology; total vCPUs commonly derives from sockets x cores | One virtual socket is often a simple starting point; size cores against workload and platform guidance |
| CPU model | Features exposed to the guest | Host passthrough can improve feature access, but restrict migration between unlike CPUs; choose a supported common model when portability matters |
| CPU units / weight | Relative scheduling share under contention | Use the current platform default unless a deliberate priority is needed; defaults/scales vary by version |
| CPU limit | Cap on CPU consumption | Set only with an understood workload/SLO; more guest vCPUs do not override a cap |
| NUMA | Exposes memory/CPU locality | Useful when virtual/physical topology and large workloads justify it; not automatically useful or harmful to all Kubernetes nodes |

Avoid treating old numeric defaults or sample VM allocations as universal recommendations. Measure contention and latency, reserve capacity for the host, and distinguish a lightweight learning node from the current platform's production requirements.
