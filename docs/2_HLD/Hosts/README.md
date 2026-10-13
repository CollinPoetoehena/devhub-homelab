# Hosts and Nodes

Choose physical capacity and failure domains before allocating VMs or installing an orchestration platform. This category combines host roles, buying decisions, power costs and virtualization; there is no separate generic hardware/software catalog.

---

## Table of Contents

- [Design Roles and Capacity](#design-roles-and-capacity)
- [Buying and Sizing Nodes](#buying-and-sizing-nodes)
- [Electricity and Consolidation](#electricity-and-consolidation)
- [Quorum and Failure Domains](#quorum-and-failure-domains)
- [Virtualization and Operating Systems](#virtualization-and-operating-systems)
- [Implementation and Verification](#implementation-and-verification)

---

## Design Roles and Capacity

Possible roles include hypervisor hosts, management clients/VMs, utility guests, Kubernetes nodes and storage/backup services. A VM per tool is not inherently simpler. Keep administration/recovery usable when the workload cluster fails.

Estimate active CPU, memory, storage/I/O, network and headroom. One host is often enough for learning; independent hosts add capacity without cluster coupling, while a hypervisor cluster introduces quorum, fencing, migration/storage and surviving-capacity requirements. Kubernetes orchestration is a [separate LLD concern](../../3_LLD/Cluster/README.md).

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

---

## Implementation and Verification

Inspect hardware, select supported images, plan management/guest boundaries, prepare storage/templates and allocate resources only after the design. CPU overcommit does not create capacity; host CPU passthrough can limit migration across unlike hardware.

Use [IaC and configuration management](./Automation.md) where useful. Verify guest readiness, resource pressure, restricted management and recoverability before expanding. Start small; add capacity or independent failure domains for a concrete need, not unsupported percentages of learning value.
