# Compute

> **Part 2 · Architecture** — [Part overview](./README.md) · Previous: [Network](./network.md) · Next: [Storage](./storage.md)

Compute is where the lab's workloads actually run: the physical hosts, their capacity and power use, and the virtualization layer that turns one or more machines into many guests. It sits directly on the [physical environment](./physical.md) and the [network](./network.md), and everything above it—[storage](./storage.md) on local disks, the [platform](./platform.md), [platform services](../03-platform-services/README.md) and [services](../05-services/README.md)—consumes its capacity and shares its failure domains.

Choose physical capacity and failure domains before allocating VMs or installing an orchestration platform. This chapter combines host roles, buying decisions, power costs and virtualization; there is no separate generic hardware/software catalog.

> **Scope:** Use DevHub's [compute hardware fundamentals](https://github.com/CollinPoetoehena/DevHub/tree/main/reference/compute) and [operating-system references](https://github.com/CollinPoetoehena/DevHub/tree/main/reference/os) for shared fundamentals and inspection commands. This guide applies them to lab purchasing, capacity, power, and recovery; actual inventory and allocations remain private. See the [repository relationship](../../README.md#how-devhub-and-the-homelab-repositories-fit-together) rather than treating this as another general hardware reference.

---

## Table of Contents

- [Design Roles and Capacity](#design-roles-and-capacity)
- [Buying and Sizing Nodes](#buying-and-sizing-nodes)
  - [Choosing a Form Factor](#choosing-a-form-factor)
  - [Purchasing and Inspection](#purchasing-and-inspection)
  - [CPU, Memory, Storage and Networking](#cpu-memory-storage-and-networking)
- [Electricity and Consolidation](#electricity-and-consolidation)
- [Quorum and Failure Domains](#quorum-and-failure-domains)
- [Virtualization and Operating Systems](#virtualization-and-operating-systems)
  - [Proxmox VE](#proxmox-ve)
  - [XCP-ng, VMware and Native Hypervisors](#xcp-ng-vmware-and-native-hypervisors)
  - [Bare Metal and Guest Operating Systems](#bare-metal-and-guest-operating-systems)
- [Implementation and Verification](#implementation-and-verification)

---

## Design Roles and Capacity

Possible roles include hypervisor hosts, management clients/VMs, utility guests and Kubernetes nodes. A VM per tool is not inherently simpler. Keep administration/recovery usable when the workload cluster fails. Storage capacity and backend choices are described separately in [Storage](./storage.md); local disks remain part of the compute host's capacity and failure domain.

Estimate active CPU, memory, storage/I/O, network and headroom. One host is often enough for learning; independent hosts add capacity without cluster coupling, while a hypervisor cluster introduces quorum, fencing, migration/storage and surviving-capacity requirements. Kubernetes orchestration is a [separate platform concern](./platform.md).

Separate **physical hosts** from **logical nodes**. A host supplies real CPU, RAM, disks and NICs; a Kubernetes node can be a VM consuming those resources. Running several node VMs is a useful way to learn upgrades and scheduling, but their memory requirements still add up and a physical-host outage can remove several nodes together.

Start a capacity budget with host overhead, always-on management, required workloads and temporary experiments. CPU can often be shared when guests are idle; memory pressure and storage latency are harder to hide. Leave room for updates, snapshots, backup jobs and moving workloads off a host for maintenance. If failover matters, size the remaining hosts for the workloads they must inherit, not merely normal balanced operation.

| Option | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| One consolidated host | Several guests/services share physical capacity | Lower equipment/lifecycle overhead; upgrades may avoid another always-on machine | Host failure stops its guests; no physical redundancy. **Maintenance:** One host lifecycle is simpler, but reboots affect all guests; monitor capacity/disks and keep independent recovery copies. | Capacity fits and outages are acceptable |
| Independent hosts | Separate hosts without a shared hypervisor-cluster lifecycle | Extra capacity and independent maintenance; limited cluster coupling | Placement/recovery remain explicit; no automatic failover. **Maintenance:** Patch and monitor each host and its guests; maintain explicit placement and recovery records without relying on cluster automation. | Several experiments or distinct roles without HA requirements |
| Hypervisor cluster | Hosts share supported management/migration/HA mechanisms | Central management and potential workload movement | Quorum, fencing, shared/replicated storage, network and surviving capacity must be designed. **Maintenance:** Coordinate member updates, quorum/fencing and storage compatibility; test migration, surviving capacity and recovery after topology changes. | Explicit migration or availability requirements justify coupling |

---

## Buying and Sizing Nodes

| Option | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| Owned laptop/PC | Repurpose an available general-purpose machine | Little purchase cost and quick start | Check health, cooling, expansion and power; laptops need safe batteries, lid/sleep behavior and wired networking. **Maintenance:** Inspect cooling, disks and firmware; laptops also need battery safety and sleep/power behavior checks during prolonged use. | First experiments or capacity already available |
| Refurbished business mini PC | Used compact office system, e.g. OptiPlex Micro, ThinkCentre Tiny or EliteDesk Mini families | Quiet VM lab; often replaceable RAM/SSD, service manuals and established firmware | Few bays/NICs/PCIe slots; exact generation, memory ceiling, health, seller testing and support vary; ECC/out-of-band management not guaranteed. **Maintenance:** Track firmware support, disk wear and cooling; retain compatible replacement RAM/storage/adapters and account for aging parts. | Value-oriented small VM hosts with sufficient expansion |
| New low-power mini PC | Current compact consumer/efficiency-focused hardware | Small footprint, modern interfaces and potentially efficient operation | Soldered components, firmware updates, parts access and compatibility vary; include adapters/cooling. **Maintenance:** Monitor vendor firmware/support availability and cooling; verify replacement parts and image/driver compatibility before upgrades. | New hardware/support or specific interface needs justify delivered cost |
| Tower/workstation | Larger general-purpose host with drive bays and PCIe expansion | Multiple disks, NICs, GPUs or controller passthrough | More space/power; firmware/IOMMU and device compatibility still matter. **Maintenance:** Maintain fans, disks and expansion-device drivers/firmware; document passthrough dependencies and inspect cooling under sustained load. | Demonstrated storage or expansion requirements |
| Rack server | Server chassis with model-dependent management/redundancy features | Enterprise management and hardware-failure practice | Noise, idle power, rails, fans and replacement disks can outweigh a low purchase price. **Maintenance:** More fans, disks, controllers and management firmware need attention; retain suitable spares and monitor power/noise as well as hardware health. | Specific server-management/capacity goals and suitable space |
| ARM single-board computer | Small ARM system for supported services or experiments | Embedded/architecture learning and dedicated light workloads | Include reliable storage, power, case and cooling; verify images and virtualization; not automatically the cheapest complete VM host. **Maintenance:** Keep supported OS/images current and monitor storage wear, power stability and cooling; retest architecture-specific workloads after updates. | ARM/embedded learning or modest supported services |

Compare exact CPU generation/model, sustained performance, virtualization/IOMMU support, firmware, memory limits and SSD interfaces/health. CPU tier names and power ratings are not whole-system measurements. RAM/SSD headroom can matter more than a small CPU-tier difference.

Compare new/refurbished/second-hand by delivered cost, inspection, warranty, returns and support. A marketplace name does not prove refurbishment quality. Employer equipment requires explicit transfer and approved data sanitization. Use [DevHub hardware inspection](https://github.com/CollinPoetoehena/DevHub/blob/main/reference/compute/Hardware_Specs.md#how-to-check-your-hardware-specs) for commands.

### Choosing a Form Factor

The table compares complete form factors, not CPU badges. A new low-end CPU need not outperform an older business CPU; inspect the complete configuration. Include the correct power adapter, cooling and Ethernet/storage adapters in delivered cost. Choose expansion or server-management features for actual requirements, not because a rack makes a lab more professional.

### Purchasing and Inspection

| Option | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| New retail / vendor | Purchase new equipment through a retailer/manufacturer | Predictable supply, current support and clearer returns | Exact specifications, warranty jurisdiction and delivered price; higher purchase cost. **Maintenance:** The channel adds no separate runtime task; retain proof of purchase/support terms and monitor the device vendor’s firmware/support lifecycle. | Support/availability outweigh used-equipment savings |
| Professional refurbisher | Used machines sold under an explicit testing/condition process | Potential savings with testing and return terms | Verify test scope, SSD health, RAM layout and grading; refurbishment quality varies. **Maintenance:** Retain test/warranty records and check aging components; seller testing does not replace ongoing firmware and health monitoring. | Used business machines with documented condition/support |
| Private second-hand | Direct resale of used equipment | Low price and local inspection opportunities | Ownership, wear, missing parts, firmware locks and limited recourse. **Maintenance:** Expect more self-supported repairs; record exact components, check health early and establish a replacement path without relying on a warranty. | You can inspect it and accept the risk |
| Marketplace seller | Seller-specific listings on a broad marketplace | Wide selection, including specialist appliances | Verify authentic specifications, seller warranty, import/VAT, shipping and returns. **Maintenance:** Preserve exact device/revision and seller support details; verify firmware provenance and spare-part availability rather than assuming marketplace-wide support. | A suitable product is available with acceptable seller terms |
| Approved employer retirement | Formally transferred equipment leaving service | Reuse without another retail purchase | Written ownership transfer, permitted disposal and verified sanitization. **Maintenance:** Retain transfer/sanitization evidence; ongoing firmware, wear and replacement responsibility transfers with the machine. | Explicit authorization and safe data removal are established |

Refurbed, Back Market and local refurbishers are examples of channels to compare, not blanket endorsements. Amazon, eBay or AliExpress are marketplaces, not a single quality-control process. A specialist multi-NIC appliance listing and a refurbished office PC are different buying decisions; inspect the actual seller and product.

Before relying on a used host, check firmware passwords, secure-boot/virtualization options, RAM stability, SSD wear, cooling, NIC operation and firmware availability. Ask whether memory is one module or fills all slots: reaching the next capacity tier may require replacing modules. Confirm a working return path before investing time in installation.

### CPU, Memory, Storage and Networking

CPU labels such as i5/i7 or Ryzen 5/7 are tiers **within generations**, not reliable cross-generation rankings. Compare core/thread count, single-thread responsiveness, sustained multi-core performance and virtualization features. Measure the workload rather than using the badge as a recommendation.

| Option | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| Efficiency-focused CPU/platform | Low-power class of host, including Intel N-series examples | Can fit light services in compact systems | Exact performance, memory limits and interfaces vary; low CPU power is not measured whole-system consumption. **Maintenance:** Keep firmware and cooling healthy and remeasure capacity/power as workloads grow; a low-power label does not guarantee sufficient headroom. | Light always-on services with verified capacity |
| Mainstream desktop CPU/platform | Core/Ryzen-class general-purpose host | Often more sustained VM/workload headroom | Generation, cooling and full-system idle/load draw matter more than tier labels. **Maintenance:** Review cooling, firmware and sustained load; reassess whole-system consumption and guest allocations as capacity is used. | Several concurrent guests or demonstrated CPU/memory needs |

Choose RAM from the sum of concurrent guest needs and host/storage overhead. Check the manufacturer's supported capacity and module type. GPU or PCIe passthrough also depends on firmware, IOMMU groups and device compatibility; a CPU feature alone does not guarantee a usable passthrough setup.

SSD capacity is not its only property: latency, endurance, sustained writes, health and replacement cost matter for VM disks. Separate working data from recovery copies according to [storage and backup design](../06-operations/backups.md). A host with one drive can be a useful learning machine if downtime/data loss are acceptable and important data has independent backups.

| Option | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| Gigabit host uplink | Conventional 1 GbE network connection | Often sufficient with lower equipment cost | Shared storage/migration/backup traffic can saturate it. **Maintenance:** Low configuration overhead; monitor errors and saturation and recheck cable/driver health before assuming more bandwidth is needed. | Small VM labs with acceptable measured throughput |
| Faster host uplink | Higher-capacity NIC/link and matching network path | Can reduce measured transfer bottlenecks | Requires suitable switch ports, cable and disks; bandwidth does not fix every latency problem. **Maintenance:** Coordinate NIC drivers/firmware, switch compatibility and link settings; monitor thermals, errors and end-to-end throughput. | Storage/migration/backup demand justifies capacity |
| Extra physical NICs | Separate physical interfaces for selected traffic roles | Distinct links and potential bandwidth separation | Ports, cabling, drivers and configuration; shared host/power remain. **Maintenance:** Track interface identity, cabling and drivers; verify bridge/routing roles after replacement or host updates. | Physical path separation or link capacity is needed |
| VLANs on a shared NIC | Logical traffic roles carried by a controlled trunk | Segmentation without extra physical interfaces | Shared bandwidth/failure domain and correct tagging/policy required. **Maintenance:** Maintain explicit trunk membership and tagging across host/switch changes; test administration and denied cross-segment paths. | Logical separation is sufficient |

Neither extra NICs nor VLANs removes the need to protect hypervisor administration. Use [DevHub's segmentation and access principles](https://github.com/CollinPoetoehena/DevHub/blob/main/reference/network/Network_Design.md#segmentation-and-access) for the general policy, then select host NIC/bridge roles that support the [home-and-lab boundary](./network.md#segmentation-and-access).

---

## Electricity and Consolidation

For the **same useful capacity**, fewer suitably sized hosts can be cheaper to run than many small hosts: each physical machine adds motherboard, cooling, storage and power-supply overhead, plus cabling and administration. Upgrading RAM in a compatible host may avoid buying another always-on machine.

This is not a guarantee that any large server is more efficient. Compare measured whole-system average draw under representative workloads; CPU type, memory configuration, disks and idle behavior matter. Do not assume RAM is free or that a rack server beats several efficient mini PCs.

**Illustrative assumptions only:** each otherwise comparable host averages 20 W, and electricity costs EUR 0.30/kWh.

| Capacity arrangement | Total RAM | Assumed total draw | Annual energy | Annual cost |
|---|---|---|---|---|
| 1 x 64 GB host | 64 GB | 20 W | 175.2 kWh | EUR 52.56 |
| 2 x 32 GB hosts | 64 GB | 40 W | 350.4 kWh | EUR 105.12 |
| 4 x 16 GB hosts | 64 GB | 80 W | 700.8 kWh | EUR 210.24 |

Calculation: average watts x 24 x 365 / 1000 x local tariff. These are not measured device figures or a current regional price claim. Use your own measurements, active hours and tariff.

Consolidation trades independence for efficiency: one host failure stops its guests. Several hosts only improve availability when the platform/data design supports failover and survivors have sufficient resources. Choose against actual outage tolerance, not node count alone.

Include switches, disk enclosures, backup devices and UPS losses in the energy budget, not just compute. Measure idle and representative loaded draw at the wall over time; CPU TDP or an adapter's maximum wattage is not average consumption. Turning disposable experiments off can save more than optimizing a host that must stay on anyway. Compare purchase plus multi-year operating cost before replacing a working machine solely for efficiency.

---

## Quorum and Failure Domains

For majority-based voting, quorum is `floor(n/2) + 1`. Without a majority, the system cannot safely make the decisions/writes requiring consensus; it prevents conflicting split-brain operation.

| Voting members | Majority | Unavailable members tolerated |
|---|---|---|
| 1 | 1 | 0 |
| 2 | 2 | 0 |
| 3 | 2 | 1 |
| 4 | 3 | 1 |
| 5 | 3 | 2 |

Odd voting counts are usually more efficient: moving from three to four voters does not add majority-failure tolerance. **The total machine count need not be odd.** Kubernetes etcd members vote, not workers; hypervisor clusters have their own membership and supported witness/fencing rules. A witness approach supported by a hypervisor is not an etcd recipe.

Two hosts do not automatically provide HA. Distinct hosts, surviving capacity, suitable storage/network, quorum and tested recovery all matter; shared power/storage can still defeat them. Three VMs on one host can teach elections and node loss without surviving that host's failure.

---

## Virtualization and Operating Systems

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| [Proxmox VE](https://pve.proxmox.com/pve-docs/) | Supported Debian-based platform integrating KVM VMs, LXC, storage/networking and web UI/API | Multiple OSes, templates and automation without assembling every management component; works on one host | Host/storage updates, repository/subscription choices and backup integration; clustering needs quorum, fencing and surviving capacity. **Maintenance:** review host releases, coordinate reboots/guest movement, monitor disks and test guest restores; the UI does not make HA or recovery automatic | Integrated virtualization and API-driven provisioning at homelab scale |
| [XCP-ng](https://docs.xcp-ng.org/) with [Xen Orchestra](https://docs.xen-orchestra.com/) | Xen hypervisor plus management/backup ecosystem | VM-centric centralized management and backup workflows | Different storage/network concepts, appliance deployment, support/build and automation choices. **Maintenance:** Coordinate hypervisor and management-appliance updates, backup jobs and restore tests; protect appliance configuration and credentials. | Xen-based learning or a suitable VM-management ecosystem |
| [VMware ESXi/vSphere](https://techdocs.broadcom.com/us/en/vmware-cis/vsphere.html) | Enterprise virtualization and management platform | Practice with that specific enterprise ecosystem | Verify current licensing, HCL, consumer-NIC support and required API/management access; historical free-license assumptions may not apply. **Maintenance:** Track supported host/management versions, hardware drivers and entitlement changes; test upgrades and restore access before relying on the platform. | VMware learning is an explicit goal |
| [KVM/libvirt](https://libvirt.org/docs.html) | Linux-native VM control and management APIs | Flexible VM configuration with fewer integrated-platform assumptions | Assemble fleet management, backups and HA yourself. **Maintenance:** more custom integration for host/network/storage updates and recovery than an integrated management platform | Linux virtualization fundamentals or a custom supported management stack |
| [Hyper-V](https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/overview) | Windows-oriented virtualization | Fits Windows administration and guest requirements | Edition, licensing and management constraints; platform-specific lifecycle. **Maintenance:** Coordinate Windows/host updates and guest maintenance; protect virtual-switch settings and validate checkpoints, backups and supported recovery. | Windows ecosystem learning/integration |
| Bare-metal services/containers | Workloads run directly on the host without a hypervisor | Fewer layers and less virtualization overhead | Host maintenance affects workloads directly; less guest isolation and rebuild flexibility. **Maintenance:** fewer platform components, but OS/runtime updates and service recovery are tightly coupled | One-purpose hosts or tight resource limits |

Use the platform's supported host OS. Ubuntu/Debian or RHEL-family utility guests fit different learning/ecosystem needs; immutable Kubernetes nodes are chosen with the [cluster platform](./platform.md).

### Proxmox VE

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| KVM virtual machine | Guest with its own kernel and allocated virtual resources | Different OSes and dedicated node isolation; replaceable guest lifecycle | Guest overhead/patching and resource budgeting; migration compatibility. **Maintenance:** Each guest needs its own OS updates and backups; maintain templates and test restore/boot and migration compatibility where required. | OS experiments and dedicated Kubernetes nodes |
| [LXC system container](https://linuxcontainers.org/lxc/documentation/) | Linux system container sharing the host kernel | Lower overhead for supported Linux services | Different isolation and kernel/compatibility limits; not Compose/Kubernetes application packaging. **Maintenance:** Update guest userspace and the shared host kernel; verify compatibility and isolation after host changes and test container-data recovery. | Lightweight supported utility services |

Clustering adds shared management and supported migration/HA capabilities, not automatic redundancy for every guest. Storage accessibility or replication, quorum, fencing, spare capacity and suitable networks still determine recovery behavior. Evaluate backup integration, repository/subscription choices and upgrade practices against the official [Proxmox documentation](https://pve.proxmox.com/pve-docs/).

### XCP-ng, VMware and Native Hypervisors

Use the platform table to compare hardware support, management-appliance deployment, backup scope, licensing and automation before buying hosts. These ecosystems are not interchangeable UIs. Favor what you want to learn and can maintain, not simply the longest feature list.

### Bare Metal and Guest Operating Systems

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| [Ubuntu](https://documentation.ubuntu.com/server/) / [Debian](https://www.debian.org/doc/) utility guest | General-purpose Linux VM | Familiar packages and broad supported cloud-init workflows; useful Linux practice | OS/users/services/SSH maintenance and configuration drift. **Maintenance:** Patch packages, maintain users/SSH/services and review drift; refresh images and test the management/recovery path after changes. | Management and utility roles needing mutable Linux |
| [RHEL](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux) and compatible family guests | Enterprise-oriented RPM/SELinux Linux environment | Practice with enterprise-Linux administration | Distribution-specific support, licensing/repository and compatibility choices. **Maintenance:** Maintain supported repositories/releases, security policy and service configuration; test major-version and SELinux behavior changes. | RPM/SELinux or enterprise ecosystem learning |
| Minimal / immutable node system | Purpose-limited OS with controlled configuration/update paths | Smaller mutable surface and repeatable node lifecycle | Different diagnostics, extensions and application support; not a general utility host. **Maintenance:** Use supported image/API upgrades rather than ad-hoc repairs; retain declarative configuration, extension compatibility and recovery access. | Dedicated roles, with [Kubernetes node choices](./platform.md#kubernetes-node-lifecycle) assessed separately |

Keep the hypervisor's supported base rather than replacing its distribution underneath it. Choose each guest OS for its role, with a repeatable image/update process rather than one default for every machine.

---

## Implementation and Verification

Inspect hardware, select supported images, plan management/guest boundaries, prepare storage/templates and allocate resources only after the design. CPU overcommit does not create capacity; host CPU passthrough can limit migration across unlike hardware.

Use the [automation part](../04-automation/README.md) where useful. Verify guest readiness, resource pressure, restricted management and recoverability before expanding. Start small; add capacity or independent failure domains for a concrete need, not unsupported percentages of learning value.

Define which parts a template, provisioning tool and configuration tool own. Prepare supported guest images, verify image integrity, separate credentials from configuration and pin tested versions of providers/components. Record expected networking and storage dependencies before cloning guests.

Validation should include cold boot, guest readiness, load/temperature behavior, backup and an isolated restore. If migration or HA is required, test that specific behavior with compatible CPU settings and surviving storage/capacity. Successful VM creation alone does not prove that its service is healthy or recoverable.
