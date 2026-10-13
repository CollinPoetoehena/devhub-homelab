# Host and Node Roles

Assign responsibilities before VM counts or resource values. Possible roles are a hypervisor host, a management workstation/VM, utility guests, Kubernetes control-plane/worker nodes, and storage or backup services. A separate VM for every tool is not inherently simpler.

Keep a management path usable when the cluster fails. Management VMs can centralize tools but add a dependency; essential network recovery should not require them. General-purpose Linux guests use supported OS configuration mechanisms; immutable/API-managed Kubernetes nodes use their own lifecycle API, not conventional SSH roles.

## Capacity and failure domains

Estimate physical and virtual CPU, RAM, disk capacity, I/O, network and headroom from active workloads. A starting VM size is not a guarantee: verify current platform minimums and monitor before adjusting.

- Start with owned hardware or one suitable host where outages are acceptable.
- Add a second host for capacity or physical-node practice, not a presumed automatic HA benefit.
- Spread replicas/voting members across independent failure domains if hardware-failure tolerance matters.
- Separate recovery copies from the disk/host they protect; sharing one enclosure or power source can defeat redundancy.

For majority-based systems, quorum is `floor(n/2) + 1`. Three voters tolerate one unavailable voter; four still tolerate one. Worker nodes are not etcd voters. Proxmox voting/witness rules and Kubernetes etcd membership differ: follow each platform's documented design rather than applying one witness recipe everywhere.

Multiple VMs on one host are useful for learning elections and node failures, but host failure takes all of them down. Odd numbers improve the efficiency of majority voting; they are not a requirement for an odd total number of machines.

Record the actual role/resource/placement table privately. Compare [hardware](./1_Design_Hardware_Prerequisites.md), [hypervisor architecture](./4_Design_Hypervisor.md), and [cluster design](./6_Design_Cluster.md) before allocating capacity.
