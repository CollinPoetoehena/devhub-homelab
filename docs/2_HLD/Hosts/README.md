# Hosts and Nodes

Choose physical capacity and failure domains before allocating VMs or installing an orchestration platform. This category combines host roles, buying decisions, power costs and virtualization; there is no separate generic hardware/software catalog.

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

Possible roles include hypervisor hosts, management clients/VMs, utility guests, Kubernetes nodes and storage/backup services. A VM per tool is not inherently simpler. Keep administration/recovery usable when the workload cluster fails.

Estimate active CPU, memory, storage/I/O, network and headroom. One host is often enough for learning; independent hosts add capacity without cluster coupling, while a hypervisor cluster introduces quorum, fencing, migration/storage and surviving-capacity requirements. Kubernetes orchestration is a [separate LLD concern](../../3_LLD/Cluster/README.md).

Separate **physical hosts** from **logical nodes**. A host supplies real CPU, RAM, disks and NICs; a Kubernetes node can be a VM consuming those resources. Running several node VMs is a useful way to learn upgrades and scheduling, but their memory requirements still add up and a physical-host outage can remove several nodes together.

Start a capacity budget with host overhead, always-on management, required workloads and temporary experiments. CPU can often be shared when guests are idle; memory pressure and storage latency are harder to hide. Leave room for updates, snapshots, backup jobs and moving workloads off a host for maintenance. If failover matters, size the remaining hosts for the workloads they must inherit, not merely normal balanced operation.

---

## Buying and Sizing Nodes

| Option | Benefits | Drawbacks / fit |
|---|---|---|
| Owned laptop/PC | Little purchase cost, quick start | Check battery/health, sustained cooling and expansion |
| Refurbished business mini PC | Compact, often upgradeable RAM/SSD and good value | Few bays/NICs/PCIe slots; seller testing/terms vary |
| New low-power mini PC | Compact, current hardware | Firmware/support, compatibility and upgradeability vary |
| Tower/workstation/rack server | More storage/expansion/capacity | Space, noise and idle power; choose for a demonstrated requirement |
| ARM single-board computer | Embedded/ARM learning and small services | Include storage, power/case/cooling and image compatibility in total cost |

Compare exact CPU generation/model, sustained performance, virtualization/IOMMU support, firmware, memory limits and SSD interfaces/health. CPU tier names and power ratings are not whole-system measurements. RAM/SSD headroom can matter more than a small CPU-tier difference.

Compare new/refurbished/second-hand by delivered cost, inspection, warranty, returns and support. A marketplace name does not prove refurbishment quality. Employer equipment requires explicit transfer and approved data sanitization. Use [DevHub hardware inspection](https://github.com/CollinPoetoehena/DevHub/blob/main/reference/os_hardware/Hardware_Specs.md#how-to-check-your-hardware-specs) for commands.

### Choosing a Form Factor

**Business mini PCs**, such as Dell OptiPlex Micro, Lenovo ThinkCentre Tiny or HP EliteDesk Mini families, are often attractive for quiet VM labs. Retired office machines can provide service manuals, replaceable memory/storage and an established firmware ecosystem. Check the exact generation: soldered components, expansion bays, memory ceilings and vendor support differ. A business label is not a guarantee of health or enterprise-server features such as ECC or remote out-of-band management.

**New consumer mini PCs** can offer modern efficiency, integrated graphics and faster interfaces, but some have soldered RAM, limited firmware updates or less accessible spare parts. Compare the complete configuration rather than assuming a new low-end CPU outperforms an older business CPU. For either option, include the correct power adapter, cooling and any Ethernet/storage adapters in the price.

**Towers and workstations** suit several drives, PCIe NICs, GPUs or storage-controller passthrough. **Rack servers** can add management and redundant-component features useful for enterprise learning, but old cheap servers may cost more in electricity, fans, rails and replacement disks than the purchase price suggests. Choose them for an actual expansion or management requirement, not because a rack makes a lab more professional.

**Laptops** are a good reuse option, but verify battery safety, cooling, wired networking and operation with the lid closed; prevent unexpected sleep. **ARM boards** are useful for architecture/embedded learning and small dedicated services. Count the power supply, case, cooling and reliable storage, and verify multi-architecture images and virtualization support. They are not automatically the cheapest complete VM host.

### Purchasing and Inspection

| Channel | Why it can fit | What to verify |
|---|---|---|
| New retail / vendor | Predictable supply, current support and straightforward returns | Exact specifications, warranty jurisdiction and delivered price |
| Professional refurbisher | Tested used business machines and potential cost savings | Test scope, SSD health, RAM configuration, condition grading and return terms |
| Private second-hand | Low price and local inspection opportunities | Ownership, missing parts, firmware locks, wear and limited recourse |
| Marketplace seller | Broad selection, including specialist appliances | Seller-specific warranty, import/VAT costs, shipping/returns and authentic specification |
| Approved employer retirement | Reuse equipment that would otherwise leave service | Written transfer, permitted disposal and verified data sanitization |

Refurbed, Back Market and local refurbishers are examples of channels to compare, not blanket endorsements. Amazon, eBay or AliExpress are marketplaces, not a single quality-control process. A specialist multi-NIC appliance listing and a refurbished office PC are different buying decisions; inspect the actual seller and product.

Before relying on a used host, check firmware passwords, secure-boot/virtualization options, RAM stability, SSD wear, cooling, NIC operation and firmware availability. Ask whether memory is one module or fills all slots: reaching the next capacity tier may require replacing modules. Confirm a working return path before investing time in installation.

### CPU, Memory, Storage and Networking

CPU labels such as i5/i7 or Ryzen 5/7 are tiers **within generations**, not reliable cross-generation rankings. Compare core/thread count, single-thread responsiveness, sustained multi-core performance and virtualization features. Intel N-series can fit light services and efficiency-focused hosts, while mainstream Core/Ryzen systems often offer more VM headroom; measure the workload rather than using the badge as a recommendation.

Choose RAM from the sum of concurrent guest needs and host/storage overhead. Check the manufacturer's supported capacity and module type. GPU or PCIe passthrough also depends on firmware, IOMMU groups and device compatibility; a CPU feature alone does not guarantee a usable passthrough setup.

SSD capacity is not its only property: latency, endurance, sustained writes, health and replacement cost matter for VM disks. Separate working data from recovery copies according to [storage and backup design](../../3_LLD/Storage_Backups/README.md). A host with one drive can be a useful learning machine if downtime/data loss are acceptable and important data has independent backups.

Gigabit Ethernet can be sufficient for small VM labs. Faster NICs become useful for measured storage, migration or backup traffic, but require suitable switches, cabling and disks to benefit. Extra NICs can separate traffic physically; VLANs can separate it logically on a shared link. Neither choice removes the need to protect hypervisor administration.

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

| Choice | Benefits | Costs / fit |
|---|---|---|
| Proxmox VE | Integrated KVM/LXC, UI/API and lifecycle tooling | Maintain hosts, storage and any cluster/quorum |
| XCP-ng/Xen Orchestra | Xen-based management and backup ecosystem | Different tooling/support/build choices |
| VMware ESXi/vSphere | Enterprise-platform learning | Check current licensing, HCL and management/API access |
| KVM/libvirt or Hyper-V | Linux- or Windows-native virtualization | More assembly or platform-specific requirements |
| Bare-metal services/containers | Fewer layers | Less guest isolation/lifecycle flexibility |

Use the platform's supported host OS. Ubuntu/Debian or RHEL-family utility guests fit different learning/ecosystem needs; immutable Kubernetes nodes are chosen with the [cluster platform](../../3_LLD/Cluster/README.md).

### Proxmox VE

Proxmox VE combines KVM virtual machines, LXC system containers, storage/network management and a web UI/API on a supported Debian-based platform. It suits a homelab that wants several operating systems and API-driven provisioning without assembling every management component. It can run as a single host; creating a cluster is not a prerequisite for using it.

VMs provide their own kernels and are a natural fit for different operating systems or dedicated Kubernetes nodes. LXC shares the host kernel and can be lighter for supported Linux services, but has different isolation and compatibility constraints; it is not the same thing as deploying application containers with Compose or Kubernetes.

Clustering adds shared management and supported migration/HA capabilities, not automatic redundancy for every guest. Storage accessibility or replication, quorum, fencing, spare capacity and suitable networks still determine recovery behavior. Evaluate backup integration, repository/subscription choices and upgrade practices against the official [Proxmox documentation](https://pve.proxmox.com/pve-docs/).

### XCP-ng, VMware and Native Hypervisors

**XCP-ng with Xen Orchestra** is a Xen-based alternative for VM-centric learning, centralized management and backups. Its storage/network concepts and automation ecosystem differ from Proxmox; assess supported hardware, management-appliance deployment, backup scope and available tooling rather than treating it as the same platform with another UI.

**VMware ESXi/vSphere** can be useful when learning that enterprise ecosystem is the goal. Verify current licensing, hardware compatibility and access to the required management/API features before buying hosts. Do not assume historical free-license or consumer-NIC support applies to a current release.

**KVM/libvirt** gives Linux-native VM control with fewer integrated management assumptions, but requires more assembly for fleet management, backup and HA. **Hyper-V** fits Windows-oriented administration and guest requirements; check the chosen Windows edition and its management/licensing constraints. Favor the ecosystem you want to learn and can maintain, not simply the longest feature list.

### Bare Metal and Guest Operating Systems

Bare-metal Linux with services or containers removes the hypervisor layer and can suit one-purpose hosts or limited resources. It couples host maintenance more closely to workloads, and rebuilding experiments may be less convenient than replacing guests. Virtualization adds resource overhead but makes isolation, templates and different OS experiments easier.

Keep the hypervisor's supported base rather than replacing its distribution underneath it. For utility VMs, Ubuntu/Debian offer familiar packaging and broad cloud-init support; RHEL-family guests suit RPM/SELinux and enterprise-Linux practice. Minimal or immutable systems reduce mutable surface but change diagnostics and extension workflows. Choose each guest OS for its role, with a repeatable image/update process rather than one default for every machine.

---

## Implementation and Verification

Inspect hardware, select supported images, plan management/guest boundaries, prepare storage/templates and allocate resources only after the design. CPU overcommit does not create capacity; host CPU passthrough can limit migration across unlike hardware.

Use [IaC and configuration management](./Automation.md) where useful. Verify guest readiness, resource pressure, restricted management and recoverability before expanding. Start small; add capacity or independent failure domains for a concrete need, not unsupported percentages of learning value.

Define which parts a template, provisioning tool and configuration tool own. Prepare supported guest images, verify image integrity, separate credentials from configuration and pin tested versions of providers/components. Record expected networking and storage dependencies before cloning guests.

Validation should include cold boot, guest readiness, load/temperature behavior, backup and an isolated restore. If migration or HA is required, test that specific behavior with compatible CPU settings and surviving storage/capacity. Successful VM creation alone does not prove that its service is healthy or recoverable.
