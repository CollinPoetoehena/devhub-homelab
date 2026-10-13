# Hardware Selection and Buying

Select devices from [goals](./0_Design_Goals.md), workload estimates, [network requirements](./3_Design_Network.md), and [Wi-Fi coverage needs](./WiFi.md). Compare exact models and delivered cost rather than treating a brand, CPU label or advertised link speed as a guarantee.

## Buying channels

| Option | Benefits | Drawbacks / checks | Choose when |
|---|---|---|---|
| Reuse an owned PC or laptop | Lowest purchase cost, quick start | Wear, battery safety, cooling, limited expansion and sustained load | Learning first; existing hardware has sufficient capacity |
| New retail or official store | Clear product/support path, current hardware | Higher purchase price; warranty and return terms still vary | Essential infrastructure or a specific support/feature need |
| Professional refurbished | Often good business-PC value, testing and warranty may be included | Grade, inspection process, components and seller terms vary | Budget compute with a credible refurbishment process |
| Private second-hand | Low prices and unusual equipment | Unknown wear, limited recourse, testing/replacement effort | You can inspect/test and accept the risk |
| Employer-decommissioned hardware | Reuse equipment at low cost | Requires formal transfer, policy compliance and approved data sanitization | The employer explicitly releases it for personal use |

For refurbished purchases, compare the actual seller, not only the marketplace. Refurbed, Back Market, and regional business-IT refurbishers are possible sources, not endorsements or guarantees. Check exact CPU/RAM/disk, firmware locks, included power supply, memory slots, storage interfaces, condition grade, support horizon, warranty, return shipping and tax. Refurbished and private second-hand are not equivalent, but a label alone proves no testing quality.

Follow the employer's approved sanitization process for released devices; reinstalling an OS alone does not guarantee old data has been securely erased. Do not use equipment still owned or managed by an employer without permission.

## Compute form factors

| Type | Strengths | Limits | Typical fit |
|---|---|---|---|
| Business mini PC (e.g. ThinkCentre Tiny, OptiPlex Micro, EliteDesk Mini) | Compact, widely available refurbished, often serviceable RAM/SSD | Few drive bays/NICs/PCIe slots; capabilities vary by generation | Quiet VM and Kubernetes learning hosts |
| New low-power mini PC | Efficient, compact, current hardware | Firmware/support, cooling and upgradeability vary | Light always-on services after exact-model checks |
| Laptop | Reuse, built-in screen and short battery ride-through | Battery health, thermals, limited expansion | Starting or temporary extra host |
| Small-form-factor desktop / tower | Drive bays, PCIe expansion, flexible upgrades | Space and power consumption | Storage, passthrough or larger workload needs |
| Rack server / workstation | Capacity, remote management, ECC options | Noise, idle power, size and accessory cost | Explicit hardware/enterprise learning requirements |
| ARM single-board computer | GPIO, small embedded projects, ARM learning | Architecture compatibility, RAM limits, storage/accessory costs | Hardware projects and supported lightweight workloads |

ARM is used in production too; x86 is not inherently more professional. Verify images and drivers for the intended architecture. A single-board computer's complete cost includes power, cooling, case, reliable storage and any network adapters.

## CPU, memory, storage and electricity

- Compare exact CPU generation, cores, sustained benchmarks and cooling. Intel N-series can suit light services; Core/Ryzen tiers can offer more VM/build headroom. Tier names and T/U/H suffixes are not fixed performance or wall-power guarantees.
- Confirm virtualization extensions and firmware support; verify IOMMU/device passthrough separately if required.
- Estimate RAM for the hypervisor, every active VM and cache/headroom. Upgradeable 32-64 GB machines can suit several VMs, but this is not a minimum for every lab.
- SSDs improve VM responsiveness. Check SATA/NVMe compatibility, endurance, health, available bays and real workload I/O; peak sequential speed does not predict every workload.
- Measure whole-system idle and representative load. Annual energy is average watts × 24 × 365 / 1000 kWh. An **illustrative** 10 W average is 87.6 kWh/year; multiply by the local tariff. CPU power ratings are not wall measurements.
- Fewer larger hosts may reduce idle draw and administration; more hosts can provide distinct failure domains only with sufficient surviving capacity and appropriate storage/quorum.

Use the [DevHub hardware reference](https://github.com/CollinPoetoehena/DevHub/blob/main/reference/os_hardware/Hardware_Specs.md#how-to-check-your-hardware-specs) for inspection commands. Size against [node roles](./5_Design_Hosts_Nodes.md), not a universal number of nodes or an unsupported percentage of learning value.

## Network equipment

Choose gateway capacity with firewall/inspection/VPN features enabled, not port speed alone. All-in-one routers simplify small installations; separate gateways, switches and APs allow independent placement and replacement at a higher cost. Integrated systems such as UniFi or Omada reduce management surfaces; mixed vendors offer flexibility but require interoperability and multiple update/recovery workflows.

For switches, verify 802.1Q access/trunk configuration, management isolation, port count, uplink capacity and per-port **and total** PoE budget. An unmanaged switch passing VLAN tags is not enforcing VLAN membership. A basic smart switch can be sufficient if its exact controls meet requirements; a Layer 3 enterprise switch is not a default home requirement.

For APs, check wired backhaul, SSID-to-VLAN mapping, client compatibility, PoE standard and placement. New Wi-Fi generations are valuable only with suitable clients, spectrum availability and uplinks. See the [wireless comparison](./WiFi.md).

Use installation-rated copper Ethernet cable, not copper-clad aluminium for PoE runs. Good existing Cat5e may be adequate for gigabit; choose Cat6/Cat6A according to length, target speed and installation conditions. Fibre suits longer/faster links but does not supply AP power. Powerline and wireless links avoid cabling but have more variable performance. Include UPS needs, ventilation, adapters, mounting, cables and replacement cost in the budget.

### Gateway classes and examples

| Class / examples | Benefits | Drawbacks / selection checks |
|---|---|---|
| Compact all-in-one (consumer routers, UniFi Dream Router/Express families) | Routing and wireless in fewer boxes | Radio placement follows the gateway unless extended; ports, VLANs, PoE and capacity vary by exact generation/model |
| Compact integrated gateway (e.g. UniFi Cloud Gateway Ultra / Max, Omada gateways) | Purpose-built routing with coordinated management options | Compare local-controller requirements, routed/inspection performance, port rates and external AP/switch needs |
| Standalone firewall PC (e.g. Protectli, Qotom, CWWK/Topton-style appliances) | Router-OS freedom, native NIC choices, possible RAM/storage upgrades | Verify NIC drivers, cooling, support and total cost; OS/plugins plus switch/AP integration remain your responsibility |
| Larger/rack gateway | More ports/capacity, rack integration or optional application functions | Size, noise, power and cost may exceed home requirements; buy for an actual throughput/function need |
| Virtual router | Snapshots and isolated experiments with no extra appliance | Its hypervisor/storage/power can become a network/recovery dependency; deliberate essential-network design required |

For example, Cloud Gateway Ultra and Max offer different LAN port-rate headroom; paying for faster gateway ports does not accelerate a gigabit switch or AP. Check current specifications and enabled-feature throughput before purchase. Optional camera-recording storage on suitable gateways is separate from OS storage and has no automatic VM/NAS use.

### Switch and AP classes

| Option | Benefit | Cost / when to choose |
|---|---|---|
| Small non-PoE smart VLAN switch | Low-cost basic segmentation | Check exact access/trunk/management controls; add injectors if APs need power |
| Small managed PoE switch | VLANs plus convenient AP power and management | Verify usable client ports after uplinks and total power budget |
| Larger PoE switch | Expansion ports and more power capacity | More cost/space; useful for demonstrated endpoint growth |
| Multi-gigabit switch | Faster NAS/host/AP uplinks | Clients, cabling and gateway paths must also support the benefit; inspect PoE variants carefully |
| Entry managed Wi-Fi 6 AP (e.g. UniFi U6+, Omada EAP families) | Often sufficient coverage and VLAN-capable access | Check exact radios/uplink/PoE; more APs are not automatically needed |
| Higher-capacity / newer-band AP (e.g. U6 Pro/Enterprise or U7 families) | Additional capacity, bands or uplink features depending on model | Benefits depend on clients, spectrum rules, placement and switch/power support |
| Wall/outdoor AP | Installation-specific coverage and mounting | Choose for the environment; check weather rating, cabling and power |

These are comparison examples, not a recommended bundle or the actual inventory. The [Wi-Fi reference](./WiFi.md) explains when managed APs, consumer mesh and ISP equipment fit.
