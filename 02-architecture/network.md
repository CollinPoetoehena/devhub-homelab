# Network

> **Part 2 · Architecture** — [Part overview](./README.md) · Previous: [Physical](./physical.md) · Next: [Compute](./compute.md)

> **Scope:** This chapter gives the networking context needed for a homelab: household and lab boundaries, traffic paths, equipment choices, Wi-Fi, placement and recovery. The examples are home-scale design options, not a map of the author's network; see the [publication boundary](../GUIDELINES.md#publication-boundary).

Design household/lab boundaries and recovery first, then select networking gear and its management software together. This chapter explains the terms and trade-offs it uses; follow linked standards and device documentation for protocol details and model-specific configuration.

The network is one of the most consequential parts of a homelab: it connects every platform and may also support household work, communication and entertainment. A failed experiment should not unnecessarily remove those services or the management path needed to fix it. Spend effort on understandable traffic paths, safe changes and recovery before adding features.

This is deliberately **one long chapter**. It starts with the requirements and the overall arrangement of routers and segments, zooms in on trust boundaries, networks inside the lab and equipment, then Wi-Fi, and finishes with recovery and verification. Everything about the network—from the high-level topology choice down to switch, cabling and SSID considerations—lives here, so a reader can follow one decision from its reason to its implementation checks without jumping between separate design and setup documents.

---

## Table of Contents

- [Purpose and Requirements](#purpose-and-requirements)
- [Design Options](#design-options)
  - [Flat or ISP-Managed Networks](#flat-or-isp-managed-networks)
  - [One Shared Segmented Gateway](#one-shared-segmented-gateway)
  - [A Separate Lab Router: Physical or Virtual](#a-separate-lab-router-physical-or-virtual)
  - [ISP Handoff, NAT and Remote Access](#isp-handoff-nat-and-remote-access)
- [Layered Design](#layered-design)
- [Segmentation and Access](#segmentation-and-access)
  - [Trust Roles and Minimum Access](#trust-roles-and-minimum-access)
- [Trust Zones and Traffic Flow](#trust-zones-and-traffic-flow)
- [Networks Inside the Lab](#networks-inside-the-lab)
  - [VM and Container Network Attachment](#vm-and-container-network-attachment)
  - [Text Diagrams for Lab Attachment Choices](#text-diagrams-for-lab-attachment-choices)
  - [Kubernetes Nodes, Pods and Services](#kubernetes-nodes-pods-and-services)
  - [Reaching Applications from Home](#reaching-applications-from-home)
  - [Public Services and the DMZ Boundary](#public-services-and-the-dmz-boundary)
- [Equipment and Management Choices](#equipment-and-management-choices)
  - [Router Platform Trade-offs](#router-platform-trade-offs)
  - [Should You Build Your Own Firewall or Buy an Appliance?](#should-you-build-your-own-firewall-or-buy-an-appliance)
  - [Switching, Power and Cabling](#switching-power-and-cabling)
    - [Unmanaged, Managed, and Smart](#unmanaged-managed-and-smart)
- [Wi-Fi](#wi-fi)
  - [Wi-Fi Design Options](#wi-fi-design-options)
  - [Wi-Fi Backhaul and Equipment](#wi-fi-backhaul-and-equipment)
  - [Wi-Fi Implementation and Verification](#wi-fi-implementation-and-verification)
- [Equipment Placement and Accessories](#equipment-placement-and-accessories)
- [Availability, Backups and Recovery](#availability-backups-and-recovery)
  - [Backups That Survive the Outage](#backups-that-survive-the-outage)
  - [Temporary Virtual-Gateway Recovery](#temporary-virtual-gateway-recovery)
- [Implementation and Verification](#implementation-and-verification)
- [Further Reading](#further-reading)

---

## Purpose and Requirements

The network turns the [foundation requirements](../01-foundations/requirements.md) into connectivity and boundaries. Before comparing routers or switches, write down what the network must achieve; every later section in this chapter answers one of these needs:

| Requirement | Why it matters in a homelab | Where this chapter answers it |
|---|---|---|
| Household connectivity survives lab work | Family members, work calls and entertainment should not stop because a VM, cluster or experiment fails | [Design options](#design-options), [availability and recovery](#availability-backups-and-recovery) |
| Experiments are contained | Disposable, misconfigured or compromised lab workloads should not freely reach household clients or management | [Segmentation and access](#segmentation-and-access), [trust zones](#trust-zones-and-traffic-flow) |
| Administration is protected and recoverable | Gateway, switch, hypervisor and controller management has the highest impact if misused, and must remain reachable when something breaks | [Trust roles](#trust-roles-and-minimum-access), [implementation checks](#implementation-and-verification) |
| Applications are reachable deliberately | Household clients may need selected lab applications; public exposure needs a separate, narrower decision | [Networks inside the lab](#networks-inside-the-lab), [ISP handoff and remote access](#isp-handoff-nat-and-remote-access) |
| Coverage and capacity fit actual use | Wi-Fi reach, uplinks, PoE and backup/storage traffic determine whether the design works in practice | [Equipment](#equipment-and-management-choices), [Wi-Fi](#wi-fi) |
| Cost and maintenance stay proportional | Every extra appliance, segment or plugin adds purchase, power, update and recovery work | [Design options](#design-options) and the central [cost guide](../01-foundations/cost.md) |

The [design principles](../01-foundations/design-principles.md) apply directly here: keep the household foundation independent of disposable compute, learn enterprise segmentation patterns without paying for enterprise availability, and prefer an understandable, recoverable design over a feature-complete diagram. Name resolution and certificates build on this network but are described as [platform services](../03-platform-services/README.md); the physical location of the equipment is part of the [physical chapter](./physical.md).

---

## Design Options

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| Flat LAN | Clients share a broadcast domain with ordinary gateway addressing/internet access | Few settings, low initial cost and easy first connectivity | Broad lateral reachability, including management; a different SSID name alone does not isolate clients. **Maintenance:** low configuration effort, but endpoint patching and gateway updates remain; limited isolation can make incident containment harder | Temporary isolated learning without sensitive household dependencies |
| Segmentation on ISP equipment | The provider gateway supplies separate subnets/guest networks and access policy | Fewer appliances; retains familiar provider support | VLAN/routing/firewall features, export and provider control vary; verify actual isolation rather than trusting a guest button. **Maintenance:** relatively low assembly effort; recheck policy after firmware changes and retain a recovery method when exports are limited | The exact device satisfies the required policy and recovery needs |
| Shared gateway with VLANs | One gateway routes multiple trust networks over VLAN-aware switches/APs | One routing, DHCP/DNS, update and recovery lifecycle; explicit cross-segment access; household IoT/guest separation | Shared outage, compromise and bandwidth domain; a policy mistake can affect several segments; no physical independence. **Maintenance:** moderate initial policy/trunk design, then one routing lifecycle; review allowed flows, back up settings and stage updates because mistakes affect household and lab together | Logical isolation and consistent management are sufficient for a maintainable small network |
| Separate lab router: physical or virtual | A dedicated physical router or router VM, for example [OPNsense](https://docs.opnsense.org/) on [Proxmox VE](https://pve.proxmox.com/pve-docs/), supplies a separate lab routing boundary, usually downstream of the home gateway; dedicated switching/cabling can extend physical separation | Dedicated router-OS practice and separate administration; a VM supports disposable exercises without another appliance, while dedicated hardware can reduce shared compute or cabling dependencies | Another address/policy lifecycle; downstream use needs coordinated routes, inbound access, VPN and IPv6, with possible double NAT. Physical separation adds hardware/power cost; a VM consumes resources and depends on host/storage/boot order. Neither guarantees independent ISP or power. **Maintenance:** maintain router updates, backups and tested recovery; coordinate upstream policy and any dedicated switches/cabling or virtual bridges/NICs; contain experimental DHCP/RA and keep management reachable without the router VM | Routing experiments or separate administration are explicit goals; choose physical or virtual placement and the degree of separation according to the required failure/trust boundary |

A second router is not automatically safer; a shared gateway is not automatically suitable for every experiment. Household DHCP/DNS and local recovery should not depend on disposable lab compute when household availability matters.

Maintenance descriptions are qualitative comparisons of ownership, not measured hours or promises that a product is maintenance-free. More segments, plugins, public services and independent devices increase work; operator familiarity, supported updates and tested recovery can matter more than initial installation speed.

Apply the central [cost and scope test](../01-foundations/cost.md) before buying networking equipment. For example, test firewall rules, routing and VLAN policy using an existing supported gateway/switch or isolated virtual networks first. A separate physical firewall, switch or redundant gateway pair is justified when the goal requires a distinct physical failure boundary, sustained capacity or approved hands-on hardware practice—not just because the network diagram looks more complete. Virtual tests teach policy and traffic behavior but do not prove physical failover, radio coverage or cabling resilience.

**Additional explanation below the table:** routing boundaries, shared dependencies and virtual placement are easier to misunderstand than a product comparison. The role-based text diagrams below show the traffic paths for each option. They are generic teaching designs, not a map of an actual household; arrows show connectivity, not permission to access everything along the path.

### Flat or ISP-Managed Networks

For the flat/ISP options in the table, test guest-to-LAN and guest-to-guest behavior separately. Check whether the provider gateway can route distinct subnets, enforce chosen cross-network rules and restrict administration; some devices offer only a fixed guest feature.

**Flat LAN:**

```text
Internet -> ISP/home router -> switch or Wi-Fi -> household + lab clients
                                                (one shared trust network)
```

The switch expands connectivity; it does not create a trust boundary between those clients.

**Segmentation on ISP equipment:**

```text
Internet -> ISP gateway with verified isolation
              +-> household network -> ordinary clients
              +-> guest/lab network -> restricted clients
```

The ISP gateway must enforce the boundary. Confirm whether its guest network also supports wired clients and the access exceptions required by the lab.

### One Shared Segmented Gateway

For a shared gateway, define allowed flows before configuring VLANs. Restrict administration, keep essential services off disposable compute and test policy. Logical segmentation is not a safe place for uncontrolled DHCP, RA or bridging experiments.

```text
Internet -> provider handoff -> shared gateway/firewall
                                  |
                              VLAN-aware switch
                                +-> restricted administration
                                +-> household clients
                                +-> lab hosts and VMs
                                +-> IoT and guest access via VLAN-aware APs
                                +-> optional DMZ services
```

Traffic between VLANs is routed and filtered at the gateway in this design. Required tags must survive every switch/AP/host link; a VLAN-capable router alone is insufficient.

**Personal preference, not a universal recommendation:** I prefer a shared gateway with VLANs. The home network needs managing anyway; one central routing and network-management foundation avoids a second permanent system. Lab VLANs and narrow firewall exceptions let ordinary workloads remain separate from household clients while learning addressing, VLANs, routing and policy. That boundary reduces accidental reachability, not shared hardware failures or the consequences of a mistaken gateway rule.

This follows **KISS: Keep It Simple**. A homelab is fun, but it shares a finite day with work and other commitments. Every additional permanent router or management system consumes some of that limited time in updates, troubleshooting and recovery. Keeping the foundation small saves time for useful experiments, other parts of the lab and life outside it. Simplicity means removing unnecessary moving parts, not skipping segmentation, backups or verification.

For me, the important distinction is **the network as a dependable foundation versus networking as an experiment**. I want ordinary connectivity to be predictable so that an evening intended for a VM, Kubernetes or an application is not first spent repairing the infrastructure underneath it. Learning and maintenance can overlap, but repeating routine upkeep on a second permanent gateway is not automatically a new learning opportunity. A small, well-understood setup is easier to return to after a busy period without having to reconstruct how several routing layers fit together.

One central policy point also makes the intended traffic path easier to reason about: which network a client belongs to, which gateway routes its traffic and where an exception must be reviewed. For example, permitting a household client to reach one lab application should involve a deliberate endpoint/port rule, not coordinating permissions and possibly translations across two routers. A coordinated gateway/switch/AP platform can simplify the management workflow further, although those devices still have their own updates and recovery requirements.

I can still learn VLAN membership, DHCP, DNS, routing and least-privilege firewall policy on that shared foundation. When I want to explore something disruptive, I would rather add a bounded, isolated exercise than make the essential network permanently more complicated. That keeps complexity where it serves the learning goal and allows the experiment to be removed afterward. It does **not** make arbitrary experiments on the shared gateway safe: changes there still need a backup, a maintenance window where appropriate and a tested administration/rollback path.

The compromise is deliberate. One shared gateway remains a shared outage and configuration-risk domain, and a mistaken central rule can affect several segments. I accept that home-scale trade-off when the recovery target allows it; I would revisit it for a genuine independent-perimeter, administration or availability requirement. KISS is a reason to avoid unnecessary systems, not a reason to dismiss a second router when it solves a concrete problem.

### A Separate Lab Router: Physical or Virtual

This is one design family with different placements: run the lab router on dedicated hardware or as a VM, and choose shared or dedicated downstream switching/cabling. The usual arrangement is behind the home gateway; a separately supported provider handoff is a different upstream variant, not an automatic consequence of using a physical box. Define which trust and failure dependencies must be separate before choosing the placement.

The separate-router pattern still needs a deliberate upstream boundary. Keep management/recovery reachable and avoid making household Wi-Fi or DNS depend on the experimental router. NAT versus explicit upstream routing is compared below; neither removes the need for policy.

**Typical downstream arrangement, using a physical router or VM:**

```text
Internet -> home gateway -> household network -> ordinary home clients
                              |
                         lab router WAN
                         (physical box OR Proxmox router VM)
                              |
                         downstream lab subnet -> lab clients
```

The lab router's upstream link can reach the home network unless deliberately restricted. Enforce that boundary and protect the router's own management services; downstream NAT is not a household-access policy.

**Why I generally avoid two permanent routers:** the home gateway still needs updates, backups and recovery, and the lab gateway adds another set. A physical box adds purchase cost, cabling and power; a VM replaces that cost with CPU/RAM/storage consumption, virtual-NIC/bridge configuration and host dependencies. Both add DHCP/DNS ownership, routes, firewall rules, credentials and fault tracing. Double NAT can complicate inbound access and VPNs; routed subnets avoid the second translation but still need coordinated routes and policy. The two routers still share the upstream home gateway and ISP. This can be worthwhile for a routing-specific goal, but it is extra setup and maintenance, not a prerequisite for learning networking.

The household already needs routing, switching and Wi-Fi, whether or not the lab runs. Reusing that foundation for segmented lab workloads avoids a second essential routing lifecycle without falling back to a flat, unrestricted LAN. A second router also does not remove the household switch/AP maintenance. Each gateway brings its own administrator identity, configuration backup and recovery procedure; IPv6 routing or prefix delegation and filtering need deliberate coordination with or without downstream IPv4 NAT. Two routers do not necessarily mean double NAT, but avoiding translation does not remove the extra operational boundary.

The extra work is not limited to installation. When an application becomes unreachable, I may need to check the home gateway, the downstream router, their addressing/routes and both sets of policy before even reaching the application itself. A later change to remote access, DNS or an upstream subnet can require revisiting both systems. Each configuration also needs a usable backup and a recovery process that still works when another dependency is down. That ongoing coordination is what I prefer to avoid, rather than the mere presence of another box.

I also distinguish **administrative separation from actual independence**. A downstream router can be useful for owning a different router OS or containing a specifically designed test network, but it still depends on the home uplink and does not automatically prevent access to upstream household clients. If I do not need that separate router lifecycle, VLANs with explicit policy already provide the ordinary machine/network boundaries I want. Adding another router solely because the lab has a separate name would spend time and resources without meeting an additional requirement.

For deeper routing practice, my preference is therefore an isolated virtual-router exercise, illustrated below: create it for a defined question, connect only test clients, observe what happens, then reset or remove it. If a long-lived downstream router later becomes the actual learning goal, keeping one can be entirely reasonable. I simply do not want every future lab session to inherit its maintenance obligations by default.

**Variant with dedicated physical paths and a supported provider handoff:**

```text
Provider handoff(s), as supported
  +-> household gateway -> household switch/APs -> household clients
  +-> lab gateway       -> separate lab switch -> lab clients
```

A single provider handoff may not support two gateways or public leases. Separate cabling is not a separate ISP or power supply; document those remaining shared failure domains. A lab gateway connected behind the home gateway is the downstream-router pattern instead.

**Variant for an isolated virtual routing exercise:**

```text
Existing gateway -> restricted lab uplink -> hypervisor
                                             |
                                          OPNsense VM WAN
                                          OPNsense VM LAN
                                             |
                                          isolated virtual switch -> test VMs
```

For deeper routing practice, I recommend an optional disposable OPNsense VM on the hypervisor rather than experimenting on the essential gateway. It can teach firewall rules, routing, VPNs or recovery without becoming the household router; skip it if those are not current learning goals. Give it an upstream interface in the Lab segment and downstream interfaces on isolated virtual networks used only by test clients. If internet access is unnecessary, an entirely isolated virtual topology is sufficient. This is a learning pattern, not a reason to route ordinary clients through another permanent service.

Do not bridge those test networks into Home, Management, IoT or Guest, trunk household segments into the router VM, or let its DHCP/IPv6 Router Advertisements escape onto shared networks. Keep upstream access narrowly controlled by the physical gateway's Lab policy: an uplink is not permission to reach household administration. Keep the hypervisor's recovery access outside the VM, and design/review any different boundary explicitly rather than weakening normal policy to make an exercise work.

The exercise should be breakable, restorable and removable without taking away household routing, DHCP, DNS, Wi-Fi or the management path used to recover the lab. A dedicated firewall distribution already provides substantial router-OS learning without requiring every underlying network service to be assembled by hand. This separates **learning networking** from **operating the essential network**.

Constrain the upstream link and prevent experimental DHCP/RA or bridges from reaching household segments. Router placement and router software remain separate choices; see the [platform comparison](#equipment-and-management-choices). If a VM becomes the essential edge rather than an isolated exercise, hypervisor/storage/boot failures can remove connectivity and the path needed to fix the host; that dependency requires explicit acceptance and offline recovery.

For a short-lived exercise, configuring the VM through its GUI is reasonable. An Infrastructure as Code (IaC) workflow, for example using Terraform, can be overkill when the goal is to explore a few settings and delete the VM afterward. Use automation when reproducibility or learning IaC is part of the goal, not as an entry requirement. Break, observe, reset and remove the test environment; it must not become an unnoticed dependency for ordinary lab or household services.

A downstream learning VM is not automatically a working replacement for a failed edge gateway. The separate [temporary virtual-gateway recovery exercise](#temporary-virtual-gateway-recovery) needs a working upstream handoff, a minimal recovery policy and tested takeover/rollback. For normal operation, keep essential gateway/DHCP/DNS/Wi-Fi independent of Proxmox/Kubernetes. A shared physical gateway still shares failure, compromise and bandwidth risks; VLANs do not create physical independence, and isolation must be validated over every enabled IP family.

### ISP Handoff, NAT and Remote Access

Provider access may use a modem, optical termination or another access device; replacing routing does not necessarily replace that termination. Bridge/passthrough can avoid duplicate IPv4 NAT, but provider authentication, VLAN tagging, TV/voice and IPv6 delegation may constrain the choice. With double NAT or carrier-grade NAT, do not assume a port forward on the lab gateway creates internet reachability.

| Option | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| Provider bridge/passthrough | Retain access termination while handing routing to the chosen gateway | Avoids a second provider IPv4 NAT layer; clearer routing ownership | Provider/model support, authentication, TV/voice and IPv6 delegation must be checked; modes differ. **Maintenance:** retain provider handoff/recovery details and revalidate after replacing access equipment or changing service | Supported handoff with a separately managed gateway |
| Downstream IPv4 NAT | A lab router translates addresses behind an existing router | Convenient outbound connectivity without adding an upstream route | Double NAT complicates inbound services, VPNs and diagnostics; does not solve carrier-grade NAT or IPv6 design. **Maintenance:** synchronize any forwards/address reservations and diagnose translation at both layers; outbound-only use is simpler | Outbound-focused experiments where limitations are acceptable |
| Routed downstream subnet | Upstream routes reach the lab without another translation layer | Clearer end-to-end addressing and fewer NAT layers | Requires supported upstream routes and explicit policy on both routers; ISP equipment may not support it. **Maintenance:** update routes and policy when ranges/interfaces change; test allowed/denied paths in both directions | Routing practice or services needing direct controlled reachability |
| Private/VPN application and admin access | Only authorized local/VPN clients reach services | Reduces public attack exposure; limits management reach | VPN identity, routing, updates and recovery still need maintenance; protect real endpoints. **Maintenance:** patch the entry point, revoke lost clients, manage keys and test a fallback administration path; VPN access does not justify broad internal permissions | Administration and services with no need for anonymous public reachability |
| Deliberate public application exposure | Selected services are reachable from the internet | Supports genuinely public workloads | Requires authentication where appropriate, TLS, updates, narrowly scoped firewall rules and recovery; management UIs should not become public by default. **Maintenance:** ongoing certificate renewal, security updates, exposure review and monitoring; public reach adds obligations beyond merely opening a port | Applications whose purpose requires external public access |

Use private/VPN access for administration where appropriate rather than exposing management UIs. For deliberately public applications, review authentication, updates, TLS, firewall scope and recovery separately. Store real internet-facing endpoints under the [sensitive-value rules](../GUIDELINES.md#secrets-and-sensitive-artifacts), not in public examples.

---

## Layered Design

At home scale, one gateway and shared switching can implement several responsibilities without a separate device for every logical layer. Add access or aggregation switches only when property size, port count, placement or storage traffic creates a real need; separate devices add power, cabling, management and recovery work.

Separate components let radio placement follow coverage instead of the ISP socket, and let a failed AP or undersized switch be replaced without replacing routing. They add power supplies, cables and update work. An all-in-one is a reasonable small-network choice when coverage, ports and controls suffice; it is not inherently less professional.

---

## Segmentation and Access

Segmentation separates devices into networks with different trust and access needs. A VLAN defines Layer 2 membership; the gateway routes and filters traffic between subnets, while devices on the same subnet may communicate without passing through it. VLAN tags must be supported along each switch, access-point and host link. Use private IPv4 space as defined by [RFC 1918](https://www.rfc-editor.org/rfc/rfc1918.html), check for overlap with home, VPN and cluster ranges, and consult the chosen equipment's documentation for its VLAN and firewall behavior.

Keep household DHCP/DNS and recovery independent of disposable compute. Restrict host/AP trunks to their intended home/lab roles, check subnet overlap with home/VPN and cluster ranges, and decide whether IoT discovery or access to a lab application is genuinely required. Those are explicit exceptions, not reasons to merge the networks. Test guest isolation on both wired and wireless paths.

### Trust Roles and Minimum Access

**Synthetic six-VLAN example:** the numbers illustrate role separation only, not the owner's configuration or a prescribed deployment.

| # | Role | Example VLAN ID | Why separate it | Typical policy question |
|---|---|---|---|---|
| 1 | Management | 10 | Network/hypervisor control has a high impact | Which approved administrator clients can reach which management services? |
| 2 | Home | 20 | Trusted household users need normal connectivity | Which specific lab applications, if any, should be reachable? |
| 3 | Lab | 30 | Experiments can be faulty or compromised | What dependencies are required without broad household access? |
| 4 | IoT / untrusted | 40 | Limited update/support and different trust | Does a device need internet, a local controller or discovery? |
| 5 | Guest | 50 | Visitors need internet without household trust | Can guests reach only necessary gateway services and WAN? |
| 6 | DMZ | 60 | Internet-exposed services face a different threat level | Which public service ports and narrow backend dependencies are required, while Home and Management remain blocked? |

These are homelab example roles, not a prescribed VLAN count or a map of the owner's network. Choose only the boundaries required for the learning and household goals. A small lab with no public workloads does not need a DMZ merely to complete the table. Record chosen mappings and the actual allowed-flow matrix privately; [implementation checks](#implementation-and-verification) should show that household connectivity and restricted administration survive lab shutdown.

---

## Trust Zones and Traffic Flow

Trust zones are groups of devices with similar access needs and risk; define the allowed paths between them rather than assuming that a separate subnet alone prevents communication.

At homelab scale, apply the model only where it solves a real problem. Restrict management, contain disposable experiments, and keep trusted household clients separate from less-trusted devices. Hosting a public application may justify a DMZ and a narrow backend path; it does not require recreating every zone or a multi-tier DMZ. The [home-to-application](#reaching-applications-from-home) and [public-service DMZ](#public-services-and-the-dmz-boundary) examples below apply those principles to a small lab without implying production-grade isolation.

---

## Networks Inside the Lab

Not every logical network inside a workload platform needs another physical VLAN. Use VLANs for meaningful machine/network trust boundaries; let the platform manage its internal networking. This keeps the physical network understandable without giving up workload-level controls. Docker bridge networks and isolated hypervisor test switches follow a similar distinction: they are not automatically extra gateway VLANs, and attaching them to a physical bridge changes their exposure.

### VM and Container Network Attachment

Before attaching a VM or container, decide whether it should join a trusted subnet, use a routed/NAT boundary with restricted access, or remain on an isolated network for a disposable exercise. Bridging is simple but inherits the attached network's reachability; NAT can restrict unsolicited inbound access but does not replace firewall policy; an isolated network is useful for tests but may not have required upstream services. The diagrams below compare these home-lab choices.

**Keep three meanings of bridge separate:** provider bridge/passthrough changes the ISP-to-gateway handoff; an uplinked hypervisor bridge attaches guests to a selected LAN/VLAN; a usual Docker bridge creates a host-local container subnet with managed outbound NAT. AP/bridge mode on Wi-Fi equipment is another device role, normally extending an existing network instead of adding routing. None of these labels alone proves isolation.

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| VM bridged to the Lab segment | Guest NIC connects through the hypervisor switch to the selected Lab network; [Proxmox VE](https://pve.proxmox.com/wiki/Network_Configuration) uses Linux bridges | Ordinary server/node addresses and direct, policy-controlled application access; no extra host NAT layer | Guest shares the selected segment, so experimental DHCP/RA can escape; same-segment traffic may bypass the gateway. **Maintenance:** preserve guest VLAN membership, address ownership and host/guest filtering; recheck after bridge/NIC changes | Stable server or Kubernetes node VMs that are intended Lab peers, not disruptive network experiments |
| VM NAT / shared NAT network | Hypervisor or router translates a private guest network's outbound traffic; [VirtualBox](https://www.virtualbox.org/manual/ch06.html) distinguishes per-VM NAT from a shared NAT Network | Convenient downloads and disposable environments without upstream route changes; a shared network lets test guests communicate | Inbound access needs selected forwarding; outbound access can still reach household services unless denied; another NAT layer complicates diagnosis. **Maintenance:** track guest ranges, DHCP/DNS, forwards and egress policy, and remove temporary entries | Short-lived development VMs or multi-VM tests that need outbound dependencies rather than LAN identities |
| Routed guest subnet without NAT | An explicit route reaches a guest subnet through a host or lab router | Preserves guest source addresses and supports controlled access without another translation layer | Requires return routes and filtering, often beyond ISP-gateway capabilities. **Maintenance:** keep upstream routes, host forwarding and IPv4/IPv6 rules consistent across reboots and address changes | Routing practice or a managed home/lab foundation where route ownership is clear |
| Host-only test network | Private guest switch with a host-side interface, normally without a physical uplink | Workstation can administer a contained development environment | Host remains reachable; host forwarding or another NIC can undermine the boundary. **Maintenance:** review host filtering, forwarding, membership and any DHCP service | Tests needing host access but no ordinary household/LAN path |
| Internal / isolated guest switch | Selected test VMs share a switch without a physical uplink or host-side IP interface | Contains DHCP, RA and router exercises; easy to break and reset without involving the household network | No normal external updates or network administration; adding a router/uplink changes the boundary. **Maintenance:** inspect every test NIC and keep any internal addressing/DNS services disposable | The downstream side of the [optional router VM exercise](#a-separate-lab-router-physical-or-virtual), or a fully offline multi-VM topology |
| Disconnected VM / container [none](https://docs.docker.com/engine/network/drivers/none/) | VM has no active network attachment; container has only loopback | Minimal network exposure and clear offline/failure-test behavior | No downloads, remote storage or network administration; shared files and other privileges remain separate concerns. **Maintenance:** retain console/offline inputs and check that no additional attachment is introduced | Network-independent jobs or deliberate link-loss tests |
| Container [user-defined bridge](https://docs.docker.com/engine/network/drivers/bridge/) | Application containers share a host-local network, with selected services published through host ports or a reverse proxy | Good single-host/Compose starting point; application-name DNS and no LAN address needed for every container | Peers can normally reach each other's ports; published bindings and runtime firewall rules determine exposure. **Maintenance:** review network membership, address overlap, port bindings and actual allowed/denied paths after runtime changes | Ordinary self-hosted stacks; keep databases internal and publish only the intended application entry point |
| Container [host networking](https://docs.docker.com/engine/network/drivers/host/) | Workload shares the host network namespace rather than a separate container IP | Supports workloads with a concrete host-network requirement without a bridge/NAT path | Shares host-local networking and ports; port-publishing mappings do not apply, and platform behavior differs. **Maintenance:** audit listening addresses, conflicts and host policy after updates; document why isolation is reduced | A proven protocol or application requirement, not the default fix for a discovery problem |
| Container [macvlan](https://docs.docker.com/engine/network/drivers/macvlan/) | Workload obtains its own IP/MAC presence on the selected physical segment | Useful for a service that genuinely needs a separate LAN identity | Multiple-MAC support and unique address allocation are required; host-to-container communication is restricted by default. **Maintenance:** coordinate addresses with DHCP/IP management, preserve parent/VLAN membership and test the intended host-access path | A specific legacy or direct-LAN requirement that ordinary application publishing cannot meet |
| Container [ipvlan](https://docs.docker.com/engine/network/drivers/ipvlan/) | Direct container addressing using the parent's MAC, with different L2/L3 variants | Useful where upstream MAC limits make separate container MACs undesirable | Host access is restricted by default; L3 requires routes and does not support ordinary L2 broadcast discovery. **Maintenance:** maintain parent/VLAN selection, addresses, routes and variant-specific policy | A deliberately managed underlay with a reason to use direct container addressing |
| Platform-managed overlay or native routing | Workload platform connects applications across hosts; [Docker overlay](https://docs.docker.com/engine/network/drivers/overlay/) and Kubernetes CNI are different implementations | Multi-node communication without manually publishing each internal workload on the LAN | Requires underlay reachability, policy and suitable MTU; encapsulation is not automatically encryption. Docker overlay requires Swarm membership. **Maintenance:** update the chosen platform/plugin, test cross-node paths and maintain routing/tunnel dependencies | A multi-host platform already justified by the [platform design](./platform.md#design-options), not a prerequisite for a single Compose host |

For a small starting point, use ordinary Lab attachment for intended server/node VMs and a user-defined bridge for a single-host application stack. Keep DHCP/router experiments on internal switches, using host-only access only when reaching the host is acceptable. Add direct-LAN containers, routed subnets or overlays only when a requirement justifies the extra maintenance. Actual choices, addresses and test results belong in the private record.

### Text Diagrams for Lab Attachment Choices

These are synthetic alternatives, not the actual home topology. Arrows show potential paths, not permission; `X` marks an absent attachment, not a firewall rule.

**Lab peer VM versus an application inside that VM:**

```text
Home client -- approved application path -- gateway policy -- Lab segment
                                                               |
                                                    hypervisor uplink/bridge
                                                               |
                                                       server VM (Lab IP)
                                                               |
                                                 selected published app port
                                                               |
                                                  container bridge -- app
                                                                   -- database
                                                                      (not published)
```

The VM's bridge and the container's bridge solve different attachment problems. Publishing only the app does not isolate it from its database peers; application authentication and workload policy still matter. Traffic between same-segment peers may not cross the gateway shown for the Home-to-Lab path.

**Disposable routing exercise with a controlled upstream:**

```text
Physical gateway -- restricted Lab uplink -- router VM WAN
                                             |
                                  routing + firewall (+ optional NAT)
                                             |
                                         router VM LAN
                                             |
                                  internal switch -- test VMs
                                             X no direct household uplink

Hypervisor recovery access: outside the router VM's downstream network
```

Only the router VM should provide the exercise's intended upstream path. Check all test NICs so a second attachment does not bypass it. Enforce the physical gateway's Lab-to-household restrictions as well: NAT inside the exercise does not provide that policy.

**Host access versus guest-only communication:**

```text
Host-only:  host interface -- private switch -- test VMs
                                    X no physical uplink by default

Internal:   test VM A ------ private switch -- test VM B
                                    X no host IP interface or physical uplink
```

Do not "fix" an isolated exercise by bridging it into a household or management segment. If external updates are necessary, design a temporary restricted path and remove it afterward. Inspect IPv6 as well as IPv4; an unexpected Router Advertisement or second interface can invalidate the intended isolation.

### Kubernetes Nodes, Pods and Services

Separate the three layers in the [Kubernetes networking model](https://kubernetes.io/docs/concepts/services-networking/):

| Layer | Responsibility | Relationship to the physical network |
|---|---|---|
| Physical/home network | Gateway, switch and AP policy for Home, Management, Lab and other trust roles | The Lab VLAN supplies ordinary connectivity to cluster nodes; hypervisor administration can remain on Management |
| Kubernetes node network | Addresses and connectivity for control-plane/worker machines or VMs | Nodes can all use Lab VLAN 30 in the synthetic six-VLAN example; node-to-node traffic uses this underlying network |
| Kubernetes internal networking | Pod connectivity, stable Service addresses and supported workload policy | The CNI supplies Pod networking; kube-proxy or a compatible replacement implements Service forwarding. Neither the Pod CIDR nor Service CIDR normally needs a corresponding VLAN on the gateway |

```text
Shared gateway/firewall -> Lab VLAN 30 -> node VM A, node VM B, node VM C
                                           |
                                  Kubernetes networking
                                    +-> Pod address space (CNI connectivity)
                                    +-> Service address space (virtual service IPs)
```

The Pod network is normally a logical cluster network, **not another physical VLAN**. A Service CIDR is an address space for virtual service frontends, not a second Ethernet segment. CNI implementations may use overlays or native routing; native/BGP designs can require upstream routes, while overlays require suitable underlay connectivity and MTU. None implies that a Pod CIDR number should become a VLAN ID. Specialized multi-network workloads can have extra interfaces/VLANs, but that is a separate requirement, not a Kubernetes prerequisite.

Choose mutually non-overlapping node, Pod, Service, container, VPN and household ranges before installation; avoid inferring address ranges from VLAN IDs. Keep the actual CIDRs private and verify routes, firewall requirements and MTU for the chosen CNI. See [cluster networking choices](./platform.md#networking-and-persistence) for Cilium, Calico and Flannel responsibilities.

VLAN/gateway rules and [Kubernetes NetworkPolicy](https://kubernetes.io/docs/concepts/services-networking/network-policies/) enforce different boundaries. The gateway can restrict Home-to-Lab and Lab-to-Management traffic, but same-VLAN node traffic and Pod-to-Pod flows may never traverse it. An enforcing CNI can restrict workload ingress/egress, for example frontend-to-backend while denying unrelated Pods access to a database. Namespaces alone do not enforce that policy, and creating NetworkPolicy objects without an enforcing implementation has no effect. Neither mechanism replaces host, API-access or application security.

### Reaching Applications from Home

Household clients normally use a stable application endpoint rather than an ephemeral Pod IP or an otherwise cluster-internal Service address:

```text
Home client -> gateway: permit selected HTTPS endpoint only
                -> Lab-side LoadBalancer address
                   -> Ingress / Gateway API implementation
                      -> Kubernetes Service -> application Pods
```

Reserve a non-conflicting pool for externally reachable LoadBalancer addresses, separate from DHCP and node/static allocations. A DNS record can point to the selected application frontend; DNS does not itself grant firewall access. A LoadBalancer address is not automatically public or internet-reachable. Keep the Kubernetes API, node administration and unrelated application ports blocked from ordinary clients.

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| [MetalLB](https://metallb.io/concepts/) | Allocates addresses from configured pools for LoadBalancer Services and announces them through Layer 2 or BGP | Gives on-premises Services reachable frontends without a cloud load-balancer service | Requires a valid pool and supported announcement topology; not an HTTP ingress controller. **Maintenance:** avoid DHCP/address conflicts, monitor speakers/controllers and test address movement and service reachability after node changes | A separate on-premises LoadBalancer implementation fits the cluster design |
| [Cilium service exposure](https://docs.cilium.io/en/stable/network/l2-announcements/) | Cilium features for address allocation and supported Layer 2/BGP advertisement in a Cilium-based design | Can reuse the selected networking platform instead of adding a separate address-announcement stack | Verify exact feature prerequisites and maturity; the linked Layer 2 announcement feature is documented as beta. **Maintenance:** coordinate compatible Cilium settings, pool ownership and announcement policy; test failover and access controls | The cluster already uses Cilium and the supported feature set fits the required reliability |

Choose one coordinated allocator/announcement design for a pool; do not enable competing components over the same addresses. Service exposure, application routing and DNS remain distinct responsibilities. A same-subnet Lab pool can avoid an additional VLAN; a routed pool is also possible but needs deliberate upstream routing and policy.

### Public Services and the DMZ Boundary

Separate internal and external ingress endpoints can help organize exposure, but do **not** turn Pods sharing nodes, credentials and a control plane into independent infrastructure trust domains. NetworkPolicy, RBAC and workload hardening are still needed, and a cluster/node compromise can cross those intended boundaries. A VLAN called DMZ does not automatically contain a public Pod on a Lab node.

For a deliberate routed boundary, a generic design is:

```text
Internet -> gateway: only selected published ports
             -> DMZ VLAN 60: reverse-proxy VM
                -> gateway: only approved backend address/port
                   -> Lab ingress endpoint -> Service -> Pods
```

This gives the firewall a real DMZ-to-Lab path to filter; it does not isolate the backend application from the rest of its shared cluster. Keep DMZ-to-Home/Management denied and constrain backend permissions. If the public workload itself needs stronger isolation, place it on dedicated DMZ VMs/nodes or a separately administered cluster as justified by the threat model, rather than assuming a reverse proxy or separate ingress controller provides that separation.

Do not build an extra cluster or activate a DMZ just to complete a diagram. Start with the smallest design that meets the actual exposure and trust requirements, then validate both gateway rules and cluster policy. The six-VLAN example reserves the DMZ role for when that boundary is needed.

---

## Equipment and Management Choices

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| ISP / consumer all-in-one | Gateway, switch ports and usually Wi-Fi in one appliance | Few boxes and familiar support | Verify routed VLANs, isolation, port count and radio placement; provider/firmware restrictions vary. **Maintenance:** few components, but less control over update timing/export; test restoration and support/replacement arrangements | Small requirements already met by the available device |
| [UniFi](https://help.ui.com/hc/en-us/categories/6583256751383-UniFi) | Coordinated gateway/switch/AP ecosystem with a Network management application | Consistent configuration and visibility; integrated-console options reduce separate controller operations | Vendor/model dependence; verify local/cloud behavior, console placement, recovery and API limitations. **Maintenance:** lower cross-vendor assembly effort, not zero upkeep; review console/application/device releases, keep protected backups and test restore/re-adoption without relying on lab compute | A coordinated, maintainable network foundation is more valuable than independent device selection |
| [Omada](https://www.tp-link.com/en/support/faq/) | TP-Link managed networking ecosystem with controller options | Coordinated wired/wireless configuration and a range of devices | Check exact gateway/switch/AP support, controller mode and feature compatibility; ecosystem lifecycle remains. **Maintenance:** central management reduces repeated work, but hosted/software/hardware controller placement changes update and recovery obligations; preserve compatible device/controller versions | Similar coordinated-management goals with suitable product/support fit |
| [OPNsense](https://docs.opnsense.org/) | Purpose-built firewall/router OS, commonly on a multi-NIC appliance | Firewall/VPN control and supported extensions without assembling every service on Linux | OS/plugin, firmware and NIC compatibility; switches/APs have separate management and recovery. **Maintenance:** moderate-to-high integration effort depending on plugins; review OS/plugin upgrades, hardware health and backups, plus each separate switch/AP lifecycle | Firewall-focused learning or independent network components |
| [pfSense](https://docs.netgate.com/pfsense/en/latest/) | Firewall/router platform with its own supported package and hardware ecosystem | Integrated routing, firewall and VPN tooling | Edition/support/package choices and hardware lifecycle; separate wireless/switch administration. **Maintenance:** coordinate supported base/package updates, exports and recovery; add-ons increase troubleshooting and upgrade testing | The platform's features/support match firewall and routing requirements |
| [MikroTik RouterOS](https://help.mikrotik.com/docs/) | Routing OS paired with a broad hardware range | Flexible routing/policy and hardware choices | Platform-specific policy/management learning, model capacity and operational responsibility. **Maintenance:** invest in understanding routing/firewall behavior, release channels and export/restore; scripts and advanced routing add ongoing review | Routing practice or requirements fitting its hardware/software ecosystem |
| [OpenWrt](https://openwrt.org/docs/start) | Extensible Linux-based router firmware | Flexible configuration and reuse of supported devices | Exact device/revision support, flash/storage capacity and upgrade/recovery paths matter. **Maintenance:** verify supported images and configuration/package migration on upgrades; custom packages and limited storage can require more hands-on work | Supported hardware and hands-on router customization |
| [VyOS](https://docs.vyos.io/) | Routing-oriented appliance OS with declarative CLI configuration | Routing-focused workflows and reproducible configuration | Image/support choices, learning curve and lifecycle ownership. **Maintenance:** retain versioned configuration and supported images; validate routing, policy and rollback after upgrades; switches/APs remain separate | Routing labs or an appliance-style routing workflow |
| Custom Linux router | General-purpose Linux with forwarding, firewall and DHCP/DNS services assembled explicitly | Deep protocol/service-integration learning and control | Own firewall correctness, dual-stack behavior, updates, service integration and recovery automation. **Maintenance:** high assembly/testing responsibility across packages, kernel, interfaces and boot services; automation reduces repetition but also needs compatibility tests and a manual recovery path | Learning the assembly is the goal, not merely obtaining low-maintenance connectivity |
| Mixed-vendor devices | Standards-based gateway, switches and APs selected independently | Purchasing freedom and fewer single-ecosystem constraints | Multiple updates, management, interoperability and recovery workflows. **Maintenance:** track separate release/support cycles, credentials and exports; test VLAN, PoE and roaming compatibility after changes instead of assuming standards eliminate coordination | Standards compatibility and operational integration are understood |

Check gateway performance with inspection/VPN enabled, not just port speed. Choose switches by access/trunk controls, restricted management, port/uplink capacity and per-port **and total** PoE budget. An unmanaged switch passing tags does not enforce segmentation.

Choose APs and Ethernet/fibre links for actual coverage, length, speed, power and installation requirements. Existing good cable may suffice; use suitable copper/PoE-rated cabling, not copper-clad aluminium. Include power, ventilation and support/return terms in the budget. See [Wi-Fi](#wi-fi).

### Router Platform Trade-offs

The table compares software/management approaches, not interchangeable hardware specifications. For a multi-NIC x86 appliance, inspect chipset support, cooling, firmware, warranty and measured consumption rather than buying from a CPU label alone. For a controller ecosystem, verify recovery without cloud access and do not assume every setting has a supported IaC interface.

### Should You Build Your Own Firewall or Buy an Appliance?

The practical split is **control and customization versus reduced integration and simpler operations**, not "unreliable DIY versus guaranteed appliance uptime." Either can work well; either can fail or be misconfigured. DIY can minimize licensing cost, but new hardware, power and your time may make an appliance cheaper overall. Both approaches have good reasons behind them; choose according to the learning goal, ongoing effort and recovery needs.

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| DIY with [OPNsense](https://docs.opnsense.org/) or [pfSense CE](https://docs.netgate.com/pfsense/en/latest/) | Install a dedicated firewall OS on compatible hardware or a VM | Extensive routing/firewall/VPN control; these community editions can supply core functionality without a mandatory software subscription; reuse suitable hardware | Hardware/driver/firmware compatibility, installation and recovery are yours. Commercial editions, support and some services can cost extra. **Maintenance:** own OS/plugin upgrades, backups, spare/recovery media and downtime; switches/APs may have separate lifecycles | Firewall learning, customization and independent component selection justify the operator effort |
| [FortiGate](https://docs.fortinet.com/product/fortigate) appliance | Vendor-integrated security gateway with commercial support/security-service options | Integrated hardware/software and a supported security-management ecosystem | Purchase/support costs and feature-specific subscriptions; check licensing and entitlement requirements. **Maintenance:** schedule firmware changes, maintain subscriptions where required, review policies and test backups/replacement | Commercial firewall administration or required vendor security/support services |
| [UniFi](https://help.ui.com/hc/en-us/categories/6583256751383-UniFi) gateway appliance | Gateway with coordinated network management for compatible switches/APs | Fewer separate management systems and less hardware/software integration to assemble | Model/ecosystem limits; not every security or support service has the same commercial terms. **Maintenance:** review and apply supported updates, protect accounts/exports and test local recovery; centralized mistakes can affect several devices | A coherent home-and-lab foundation is more valuable than firewall-OS flexibility |
| [Firewalla](https://help.firewalla.com/hc/en-us/articles/4408644783123-Building-Network-Segments) appliance | Packaged gateway/security platform with app-oriented administration | Guided administration and integrated policy/visibility | Verify model, operating mode, segmentation and service terms; it does not replace switch/AP management. **Maintenance:** review updates, alerts, permissions and supported recovery; retain device/account access during an outage | Appliance convenience and its management model fit household requirements |

Buying an appliance does not mean it is ready for the intended segmentation out of the box or that the vendor operates it for you. The vendor supplies supported firmware and, where licensed, threat feeds; the owner still chooses safe settings, applies or supervises updates and handles recovery. DIY distributions also supply integrated interfaces and updates: they are not the same as assembling a custom Linux router.

**Personally, I prefer the appliance approach, for example UniFi, for the stable foundation.** A coordinated gateway/switch/AP platform reduces the integration and separate administration I want to own, leaving more time for hypervisors, VMs and Kubernetes. It still teaches VLANs, subnets, routing and firewall policy. OPNsense can teach those concepts too and offers deeper firewall customization, but selecting and maintaining its hardware/VM, plugins and surrounding network tools can add work and opportunities for mistakes. The preference is for fewer moving parts and a predictable recovery path, not a claim that a brand guarantees greater reliability or removes operator risk.

The ownership difference matters even when configuration is automated. A custom Linux router can make interfaces, DHCP/DNS, forwarding and firewall rules repeatable through a configuration-management tool such as Ansible, but someone still owns their integration, package/kernel changes and dual-stack correctness. A dedicated firewall OS removes much of that assembly, while retaining its own OS/plugin lifecycle and potentially separate switch/AP administration. The issue is the work I want to own, not that those tools are unsuitable.

A physical gateway with supported local management can keep essential routing and administration independent of experimental Proxmox/Kubernetes compute. An integrated ecosystem can also coordinate compatible switches/APs without a controller VM, where the chosen gateway actually hosts that management application. Check the model and recovery behavior rather than assuming every appliance has those properties. I accept vendor dependence and less low-level/plugin flexibility when that reduces essential-network integration work.

Low maintenance is **not** maintenance-free or guaranteed uptime. Supported firmware updates, restricted administration, protected backups, a wired recovery path and [connectivity/isolation verification](#implementation-and-verification) remain necessary. A small documented set of GUI changes is reasonable; building custom automation for every infrequent setting can recreate the maintenance being avoided. Automate supported repeatable work where it helps, rather than turning essential routing into another software project.

### Switching, Power and Cabling

#### Unmanaged, Managed, and Smart

An unmanaged switch forwards traffic with little configuration; a managed switch exposes port/VLAN policies and diagnostics; "smart" models offer a product-specific subset of managed features. For a homelab, choose the least complex model that actually enforces required host/AP VLAN paths, protects administration, and fits uplink and PoE needs. Check the exact model's documentation rather than inferring features from a product category.

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| Unmanaged switch | Basic Layer 2 connectivity without configured port policy | Low cost and little administration | Passing VLAN tags is not enforcing membership; limited management/visibility. **Maintenance:** Very little configuration upkeep; inspect link/power health and aging hardware. Limited diagnostics can increase fault-isolation effort, and replacement does not add VLAN policy controls. | Simple same-trust access with no switch-level segmentation requirement |
| Managed Layer 2 switch | Explicit access/trunk membership and restricted administration | Enforces selected VLAN paths and provides diagnostics | Configuration, updates and recovery; shared uplinks can bottleneck traffic. **Maintenance:** back up port profiles and verify trunks/management access after firmware or cabling changes | Segmented hosts/APs and controlled port roles |
| Smart / web-managed switch, such as the [NETGEAR smart-switch category](https://www.netgear.com/business/wired/switches/smart/) | Limited-management tier between basic unmanaged and more fully managed products; exact scope varies | Often lower cost and simpler UI while retaining selected VLAN/QoS controls | Do not assume advanced ACLs, DHCP snooping, CLI, link aggregation, PoE or routing; verify each requirement. **Maintenance:** update firmware, restrict management, retain port/VLAN backups where supported and recheck isolation after changes | A small segmented network whose complete requirements are met by the exact model |
| Layer 3 switching | Selected routing occurs on the switch | Can reduce gateway transit and support larger/busier networks | Gateway firewall does not automatically inspect those flows; policy ownership and ACL support need planning. **Maintenance:** additional routing/ACL state and route convergence tests; keep policy consistent wherever routing occurs | Demonstrated inter-segment throughput/scale requirements |
| PoE switch | Negotiated IEEE power and data delivered to compatible clients | Central AP power and fewer local adapters | Check per-port standard/delivery and total budget, including growth. **Maintenance:** monitor power draw and re-evaluate budget before adding devices; switch failure can remove both data and AP power | Several powered APs/devices with suitable cabling |
| Supported PoE injector | One powered Ethernet link using a compatible injector | Adds power without replacing the switch | Extra adapter/outlet/cabling; passive PoE is not interchangeable with negotiated IEEE PoE. **Maintenance:** Little configuration work, but inspect power adapters, cable/connector health and supported power requirements after device replacement; keep a compatible spare if AP availability matters. | A few powered devices or an otherwise suitable non-PoE switch |
| Copper Ethernet | Twisted-pair data link, optionally carrying PoE | Familiar termination and power/data on one run; existing good cable may suffice | Speed/length/termination and PoE suitability; avoid copper-clad aluminium. **Maintenance:** usually little ongoing configuration; label/test runs and inspect damaged terminations before replacing active equipment | Ordinary room/building runs within supported limits |
| Fibre | Optical links with compatible transceivers | Useful for distance or electrical separation | Transceivers, termination and compatibility; no PoE to an AP. **Maintenance:** keep connectors clean, respect bend limits and retain compatible spare optics; diagnose optical and electrical paths separately | Longer links, electrical isolation or justified uplink capacity |

Buy managed switching for VLAN membership controls, management restrictions and visibility, not merely a product that passes tagged frames. Count current ports plus realistic growth and uplink needs. Shared gigabit uplinks can bottleneck backups or storage even when a gateway has faster ports; route and switch throughput are different questions.

PoE simplifies AP placement, but check the required standard, maximum per-port delivery and the switch's total budget simultaneously. Include future powered devices and supported injectors where appropriate. Passive PoE is not interchangeable with negotiated IEEE PoE. Keep gateway power requirements distinct from AP power; not every gateway supplies PoE or Wi-Fi.

Choose copper by required speed, installation length, termination quality and PoE suitability. Fibre can suit distance or electrical separation, but adds transceivers and compatibility checks. Avoid replacing good cabling solely for a higher category number. Buying new supported gateway/switch equipment can be justified for household recovery/support, while refurbished compute has a different risk profile; compare warranty and delivered costs rather than declaring one channel always best.

---

## Wi-Fi

Select coverage, backhaul, management and segmentation together with the rest of this network design rather than as a second, independent routing stack. Wireless access is where household devices, guests and limited-trust devices most often join the network, so radio placement and SSID-to-segment mapping are as much part of the [segmentation design](#segmentation-and-access) as the gateway rules are.

> **Scope:** This section evaluates home-scale coverage, equipment and segmentation. Map each SSID to an intended trust role, then verify the access-point, switch and gateway configuration as one path. Keep actual coverage measurements and device settings private.

### Wi-Fi Design Options

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| ISP/all-in-one Wi-Fi | Gateway and radio in one device | Few devices, often already available; economical if centrally located | Provider socket/cabinet may be a poor radio location; isolation and controls vary. **Maintenance:** relatively few settings/devices, but gateway maintenance also interrupts Wi-Fi; recheck guest isolation and credentials after reset/replacement | Coverage and security requirements are modest and verified |
| Consumer mesh | Coordinated multi-unit wireless coverage, with wireless or supported wired backhaul | Convenient coverage expansion and lower setup effort where cabling is impractical | Check router/AP mode, VLAN SSIDs, guest isolation, subscription/cloud dependence and backhaul; mesh is not proof of segmentation. **Maintenance:** coordinated updates can simplify upkeep; still track unit support, app/account recovery and inter-node signal changes after moving furniture/units | Simple coverage is the priority and supported features satisfy policy |
| Managed wired APs, e.g. [UniFi](https://help.ui.com/hc/en-us/categories/6583256751383-UniFi) / [Omada](https://www.tp-link.com/en/support/faq/) | Separate radios with Ethernet uplinks and coordinated SSID/VLAN management | Radio placement independent of routing; predictable uplinks and segmented wireless access | Cabling, controller availability/local recovery and compatible PoE/injectors; ecosystem features vary. **Maintenance:** more initial VLAN/PoE/coverage planning, then centralized profiles and updates; back up controller settings and verify SSID policy/roaming after changes | Maintainable coverage and wireless trust boundaries matter |
| Mixed-vendor wired APs | Independently selected APs using supported networking standards | Purchasing freedom and standards-based flexibility | Several update/configuration workflows; roaming/security consistency needs coordination. **Maintenance:** higher manual coordination for channel/power/security settings and release cycles; retain exports and test client behavior across vendors | Interoperability and operations are understood |
| Wireless repeater | A device retransmits an existing wireless connection | Low installation effort for limited reach extension | Adds a radio hop and airtime contention; cannot fix a poor upstream signal or internet link. **Maintenance:** another firmware/credential lifecycle and failure point; retest uplink quality after placement or upstream SSID changes | A bounded coverage gap with acceptable throughput/latency after testing |

These are general approaches, not recommendations for the owner's installed models. First distinguish a coverage problem from an internet/uplink problem; adding radios does not fix every slow connection.

### Wi-Fi Backhaul and Equipment

| Option | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| Ethernet backhaul | AP/mesh units connect to the network by cable | Predictable uplink; leaves more airtime for clients; can carry compatible PoE | Cable installation, port capacity and power budget. **Maintenance:** initial physical work, then relatively stable links; diagnose cable, port and PoE faults separately | Cabling is feasible and stable capacity is important |
| Wireless backhaul | Units use inter-node radio links | Coverage without new cable runs | Signal/interference and shared airtime constrain capacity; dedicated radios help but do not remove dependency. **Maintenance:** less installation work but potentially more recurring signal/channel diagnosis; review after environmental changes | Installation constraints outweigh performance costs and inter-node links test well |
| 2.4 GHz | Lower-frequency Wi-Fi band used by many legacy/IoT clients | Useful reach and broad compatibility | Crowding and limited channel capacity; compatibility exceptions need containment. **Maintenance:** No separate software lifecycle beyond the AP; periodically review congestion, legacy-client exceptions and channel/power choices without weakening every SSID for one device. | Legacy devices and coverage-focused use |
| 5 GHz | Higher-frequency band with more channel choices than 2.4 GHz in many regions | Capacity/channel-reuse opportunities for supported clients | Range/material losses and local DFS/channel rules; wide channels may reduce reuse. **Maintenance:** Review interference and local DFS/channel behavior after AP updates or placement changes; verify client coverage and roaming rather than maximizing channel width by default. | General modern-client capacity within measured coverage |
| 6 GHz | Additional spectrum for compatible Wi-Fi generations/clients | Additional channel space where permitted | Client, security-mode and local regulatory support; reach remains placement-dependent. **Maintenance:** Keep AP/client firmware and permitted channel/security settings compatible; recheck coverage and local regulatory support as clients or deployment regions change. | Supported clients and justified spectrum/capacity needs |

Mesh products may support wired backhaul, and managed APs may support meshing: compare features, not labels.

Survey placement/building materials before AP count. Compare client support, bands, local spectrum rules, uplink and PoE budget; advertised PHY rates are not application throughput. More APs or maximum transmit power is not automatically better.

An AP needs a good uplink where it is installed; a wireless mesh node at the edge of an already unusable signal cannot manufacture backhaul capacity. Test inter-node signal, not just client signal. Ethernet is predictable and leaves more airtime for clients, while wireless backhaul is a practical trade-off for rooms where installation cost or building constraints dominate.

Use 2.4 GHz for coverage/legacy needs and suitable 5/6 GHz capability for supported clients where range and local rules allow. Wider channels can improve peak rates but reduce channel reuse and may be worse in crowded environments. Walls, floors, antenna orientation and client radios matter; a Wi-Fi-generation badge is not a coverage survey.

Keep SSID count purposeful because each adds management airtime. A separate IoT compatibility SSID can avoid weakening every client network, but must map to real policy. AP management belongs on a restricted path; wireless clients should not gain administration merely because they can associate.

### Wi-Fi Implementation and Verification

Choose bridge/AP versus router mode deliberately to avoid unintended NAT/DHCP. Map required SSIDs to the correct segments, protect administration and use compatible WPA2/WPA3 modes. Limit SSID count and legacy exceptions.

Validate association, addressing, backhaul, intended access/denials, roaming and recovery. Client roaming depends on placement, settings and client behavior; neither mesh branding nor multiple APs guarantees seamless handover.

Measure in the rooms and usage conditions that matter, including latency and stability rather than only a speed test beside an AP. Check that a roaming client retains the expected network/policy and that one radio/client does not create an unintended bridge. Review guest client isolation on wired and wireless paths separately.

Use compatible security modes and a documented update cadence. If legacy clients force an exception, contain it to their segment and record why. Exact SSIDs, AP placement and operational surveys remain private; a public explanation can show the evaluation process without a floor plan or device locations.

---

## Equipment Placement and Accessories

See the [physical chapter](./physical.md) for meter-cabinet trade-offs, safe cabling, mini-PC ventilation, optional racks, rack units and power accessories. The network-specific [switching and cabling criteria](#switching-power-and-cabling) still apply wherever equipment is installed.

---

## Availability, Backups and Recovery

**A shared gateway is an accepted single point of failure when its recovery time fits the household's needs.** Apply [Learn Enterprise Patterns Without Enterprise Availability](../01-foundations/design-principles.md#learn-enterprise-patterns-without-enterprise-availability) directly: learning enterprise patterns does not require paying for enterprise availability. The gateway, switch, ISP handoff and power can each interrupt service. VLANs, multiple VMs and configuration backups do not remove those failures.

Choose a realistic recovery target with the people who depend on the connection. Waiting for a replacement may be acceptable; remote work or accessibility needs may justify a spare or another connection. None of the options below is a promise of seamless failover.

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| Restore after repair/replacement | Recover a supported gateway from protected backups when hardware is available | Lowest spare-hardware cost and few standing dependencies | Delivery, installation and restore time become outage time. **Maintenance:** refresh backups and offline instructions; verify restore compatibility and replacement availability | Downtime is acceptable and selected clients have another way online if needed |
| Cold spare gateway | Keep a compatible unused replacement, for example a UniFi Cloud Gateway, ready for manual swap and supported restore | Avoids waiting for delivery without operating a second live gateway | Purchase cost, aging, firmware/restore compatibility and manual recabling; not automatic HA. **Maintenance:** periodically power up in isolation, update/test as appropriate and verify backup restoration; keep the right power supply/cables | Faster manual recovery is worth buying a spare |
| Temporary [OPNsense VM](https://docs.opnsense.org/manual/virtuals.html) on Proxmox | Manually use existing compute as an emergency gateway until replacement arrives | Can avoid buying another router if suitable compute/NICs already exist | Depends on working host/storage/switching and prepared WAN/LAN access; temporarily loses normal compute independence. UniFi exports cannot be restored into OPNsense. **Maintenance:** stage installation media, a minimal separate policy and recovery instructions; rehearse takeover and rollback | A tested, lower-cash-cost fallback whose setup time and dependencies are acceptable |
| Mobile hotspot / alternate uplink | Give selected clients another internet connection | Simple fallback for communication, downloads or work | Coverage, data limits and tethering support; does not restore LAN services, VLANs or the failed gateway. **Maintenance:** check availability, access credentials and allowances before relying on it | Basic internet access matters more than restoring the whole network immediately |
| Supported HA gateway pair | Compatible gateways with vendor-supported failure detection and failover | Shorter gateway outages within the designed failure scope | More hardware/power, model/software constraints and tested WAN/LAN/state behavior; shared switch/ISP failures remain. **Maintenance:** coordinate updates, synchronization, failover and failback tests | Outage impact justifies a continuously operated redundant pair |
| UPS | Battery-backed power for selected gateway/switch/ISP equipment | Can bridge short power cuts and support orderly shutdown | Finite runtime; does not fix device, firmware or provider failure. **Maintenance:** test load/runtime, replace aging batteries and review all powered dependencies | Short power interruptions are a meaningful source of outages |

A cold spare avoids much powered-use heat and wear, but storage does not stop component aging or guarantee it will boot years later. Keep it dry and within storage limits, and test it without introducing competing DHCP/RA services onto the live LAN.

HA is possible, but it is not the lowest-cost default. For UniFi, verify the current [Shadow Mode requirements](https://help.ui.com/hc/en-us/search?query=Shadow%20Mode): support is model-specific, and budgeting for a supported pair may mean more expensive equipment than a small household gateway. Do not infer HA support from a shared brand or two identical boxes. A second WAN link and redundant switching/power are separate decisions.

### Backups That Survive the Outage

Keep dated supported configuration backups before changes and after validation. Retain an encrypted/access-controlled **local offline copy and an off-site/cloud copy**, with credentials and recovery keys available without the failed gateway, controller or lab storage. A file on the gateway itself is not independent; a cloud-only backup is awkward when internet access is down.

For UniFi, use the supported [System Config Backup and migration workflow](https://help.ui.com/hc/en-us/articles/360008976393) and verify target-model/software compatibility. Record provider settings, physical port roles, necessary DHCP/DNS and firewall intent, local management access and any device re-adoption steps securely. Do not commit raw exports or sensitive recovery inputs to Git. See [backup scope and independent copies](../06-operations/backups.md#independent-copies-and-retention).

Downloading a backup proves only that a file exists. Rehearse restoration on a compatible isolated target where practical, record what was actually tested and measure the manual recovery time before relying on it.

### Temporary Virtual-Gateway Recovery

Treat emergency takeover as a **separate planned exercise**, not as proof that an ordinary downstream lab-router VM can immediately replace the edge. If its WAN remains behind the failed gateway, it has no working internet path. Prepare a supported direct provider handoff or another working uplink, separate WAN/LAN interfaces or carefully restricted VLANs, and a wired hypervisor-management path that does not depend on the VM.

Keep installation media, provider requirements and the minimal recovery policy offline. A Proxmox host without suitable available NICs or safe link separation may require additional hardware; an untested VM is not a zero-cost guaranteed backup. The temporary router also cannot fix a failed shared switch, ISP link or hypervisor.

Use the OPNsense GUI for this one-off recovery if that is simplest; there is no requirement to build an IaC workflow first, for example using Terraform. Recreate only the necessary DHCP/DNS, segmentation and outbound rules; leave public exposure disabled unless explicitly required and validated. Do not try to import another vendor's backup. Existing switch/AP forwarding may continue with saved settings, but their controller is not replaced by OPNsense and that behavior must be verified.

Disconnect the old gateway before the replacement supplies overlapping addresses, DHCP or Router Advertisements. In a scheduled test, verify boot without internet, WAN access, required local services, intended denials and recovery after reboot. Then test the return to the normal physical gateway, removing the temporary router and any temporary permissions. Start on isolated virtual networks; an actual household switchover needs an agreed maintenance window and rollback. Until that exercise passes, label this an **unverified fallback**, not available HA.

---

## Implementation and Verification

After choosing the design, plan addressing, port/SSID roles, provider requirements and a known-good wired recovery path. Stage changes incrementally; verify the replacement access path before removing the old one. Vendor guides own exact installation commands and UI labels.

Test DHCP/DNS, authorized services and intended denials in every enabled IP family. A failed ping does not prove isolation: establish that the service works from an authorized source, then test the same service from the denied source. Check household connectivity with lab compute off when independence is required.

Use the [availability and recovery plan](#availability-backups-and-recovery) to choose restore/replacement versus supported redundancy according to outage impact. Duplicate boxes alone do not provide tested failover. Keep disruptive DHCP/RA/router experiments isolated and raw backups/endpoints protected under the [guidelines](../GUIDELINES.md).

Make changes in a recoverable order: retain a trusted wired bootstrap path, establish the new administration path, apply intended membership/policy and validate before removing the old path. Keep a protected configuration backup and know how to reach local management if internet or cloud sign-in fails.

Use a short acceptance record rather than treating a successful internet speed test as proof of a working design:

| Area | What to establish |
|---|---|
| Dependencies | Identify who supplies routing, DHCP, DNS, switching, Wi-Fi, management and power; turn off experimental compute to test claimed independence |
| Segmentation | Test permitted and denied application traffic, gateway administration and same-VLAN/client isolation separately; repeat in every enabled IP family |
| VM/container attachment | Verify bridge/VLAN and all NIC memberships, address/DHCP ownership, host-to-guest and guest-to-household paths, published bindings and intended denials; confirm isolated DHCP/RA stays inside the exercise and retest after reboot |
| Capacity | Check routed/inspected/VPN traffic and busy backup/storage paths, not just interface link rates; inspect errors, uplink contention and PoE load |
| Recovery | Reach local administration without cloud sign-in; retrieve offline backups/keys and rehearse compatible restoration or the chosen fallback |
| Change safety | Save the last working configuration, change one boundary at a time, verify it and retain a known-good rollback path |

Maintain a private allowed-flow matrix and a short change log recording why exceptions exist, who owns them and when temporary access should be removed. Review firmware/support status and backups periodically. Keep monitoring useful but optional for connectivity: a failed dashboard or controller VM must not silently become a routing/DNS dependency.

Record policy tests with source role, destination service, IP family and expected outcome. Recheck after firmware changes, new exceptions and AP/port changes. Public lessons describe the method; private records hold the actual values and outcomes. Link vendor installation documentation rather than maintaining a click-by-click public imitation of a household setup.

## Further Reading

This chapter summarizes the home-scale design implications; refer to the standards and official project documentation for protocol and implementation details. See [Goals and Scope](../01-foundations/goals-and-scope.md#define-what-is-out-of-scope) for the boundary.

- IEEE 802.1, [IEEE 802.1Q VLAN standard overview](https://1.ieee802.org/maintenance/802-1q-2018/)
- IETF, [Private IPv4 Address Allocation (RFC 1918)](https://www.rfc-editor.org/rfc/rfc1918)
- IETF, [IPv6 addressing architecture (RFC 4291)](https://www.rfc-editor.org/rfc/rfc4291)
- Kubernetes, [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/) and [Services, Load Balancing, and Networking](https://kubernetes.io/docs/concepts/services-networking/)
- Wi-Fi Alliance, [Wi-Fi security](https://www.wi-fi.org/discover-wi-fi/security)
