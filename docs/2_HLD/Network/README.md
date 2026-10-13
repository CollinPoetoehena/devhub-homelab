# Network

Design trust boundaries and recovery first. Networking gear and its management software are selected together to implement that design. Broad protocol theory belongs in [DevHub networking](https://github.com/CollinPoetoehena/DevHub/tree/main/reference/network).

---

## Table of Contents

- [Design Options](#design-options)
  - [Flat or ISP-Managed Networks](#flat-or-isp-managed-networks)
  - [One Shared Segmented Gateway](#one-shared-segmented-gateway)
  - [A Separate Downstream Lab Router](#a-separate-downstream-lab-router)
  - [Physical Separation and Virtual Routers](#physical-separation-and-virtual-routers)
  - [ISP Handoff, NAT and Remote Access](#isp-handoff-nat-and-remote-access)
- [Layered Design](#layered-design)
- [Segmentation and Access](#segmentation-and-access)
  - [Trust Roles and Minimum Access](#trust-roles-and-minimum-access)
  - [Routing, Gateway Services and Local Traffic](#routing-gateway-services-and-local-traffic)
  - [Addressing and VLAN Membership](#addressing-and-vlan-membership)
- [Equipment and Management Choices](#equipment-and-management-choices)
  - [Router Platform Trade-offs](#router-platform-trade-offs)
  - [Switching, Power and Cabling](#switching-power-and-cabling)
- [Implementation and Verification](#implementation-and-verification)

---

## Design Options

| Pattern | Benefits | Drawbacks | Choose when |
|---|---|---|---|
| Flat LAN | Few settings and low initial cost | Untrusted clients and experiments share boundaries | Temporary isolated learning without sensitive household dependencies |
| Segmentation on ISP equipment | Fewer appliances | VLAN/firewall/Wi-Fi features and provider control vary | The exact device meets the required policy |
| Shared gateway with VLANs | One routing lifecycle and selected cross-segment access | Shared failure, security and bandwidth domain | Logical isolation and consistent management are sufficient |
| Separate lab router | Independent routing experiments/boundary | Extra lifecycle, coordinated policies and possibly double NAT | Router independence is a deliberate requirement |
| Physically separate network | More independent hardware boundaries | More cost/operations; upstream or power may still be shared | Logical isolation cannot meet the requirement |

A second router is not automatically safer; a shared gateway is not automatically suitable for every experiment. Household DHCP/DNS and local recovery should not depend on disposable lab compute when household availability matters.

### Flat or ISP-Managed Networks

A flat LAN is easy to begin with: clients share a broadcast domain and ordinary consumer equipment supplies addressing and internet access. The cost is broad lateral reachability, often including management interfaces. A guest SSID can improve this only if the device actually isolates it from the LAN and other guests. A separate SSID name alone creates no security boundary.

Using the ISP gateway for segmentation avoids another router and preserves familiar provider support. Check whether it can route distinct subnets, enforce chosen cross-network rules and keep management restricted; some devices only offer a fixed guest feature. Firmware, provider restrictions and limited configuration export can make it unsuitable for a more deliberate design. Use it when its verified capabilities meet the requirement, not simply because it has a "guest" button.

### One Shared Segmented Gateway

One gateway can route several trust networks over VLAN-aware switching and APs. Home and lab then share hardware while rules permit only the cross-network services that are actually needed. This reduces duplicate routing, DHCP/DNS, updates and recovery workflows; segmentation can benefit household IoT/visitors even without a lab.

The trade-off is shared outage, management and compromise risk. A mistake on that gateway can affect several segments, and VLANs do not create a physically independent perimeter. Restrict its administration, keep essential services off disposable compute and test the policy. This pattern fits a small maintainable network when logical separation is sufficient; it is not a safe place for uncontrolled DHCP, RA or bridging experiments.

### A Separate Downstream Lab Router

A second router can create a deliberate firewall/routing-learning boundary without replacing the household gateway. It adds a device/OS lifecycle and often another address plan. With NAT behind the household router, outbound connectivity can be convenient, but inbound services, VPNs, troubleshooting and IPv6 may require coordination across both layers.

Alternatively, a routed lab subnet without a second NAT layer needs an upstream route and explicit policy on both sides. Some ISP equipment cannot provide that. A lab router's upstream interface still touches a household network unless the upstream design gives it a constrained transit segment; do not assume router count removes every household access path.

Choose this approach when independent router-OS practice or a separately administered boundary is itself a goal. Keep management/recovery reachable and avoid making household Wi-Fi or DNS depend on the experimental router.

### Physical Separation and Virtual Routers

Separate switches, cabling and gateways reduce shared hardware boundaries, but can still share ISP access, power and administrators. True independence requires defining which dependencies must be separate. It costs more equipment and maintenance, and is most useful for experiments or trust requirements that logical policy cannot meet.

A router VM is useful for disposable isolated networks and repeatable routing labs. As the household edge it couples internet access to the hypervisor, storage and boot order; a host maintenance failure can also remove the route needed to fix the host. Dedicated physical gateway hardware avoids that dependency. A hand-built Linux router offers deep protocol learning but makes service integration, firewall correctness, upgrades and recovery your responsibility; a purpose-built router OS removes some of that assembly, not all operations.

### ISP Handoff, NAT and Remote Access

Provider access may use a modem, optical termination or another access device; replacing routing does not necessarily replace that termination. Bridge/passthrough can avoid duplicate IPv4 NAT, but provider authentication, VLAN tagging, TV/voice and IPv6 delegation may constrain the choice. With double NAT or carrier-grade NAT, do not assume a port forward on the lab gateway creates internet reachability.

Use private/VPN access for administration where appropriate rather than exposing management UIs. For deliberately public applications, review authentication, updates, TLS, firewall scope and recovery separately. Store real internet-facing endpoints under the [sensitive-value rules](../../Repository_Guidelines.md#secrets-and-sensitive-artifacts), not in public examples.

---

## Layered Design

Use **edge, core, distribution and access** as responsibilities, not a requirement to buy an enterprise switch for each layer.

- **Edge:** provider handoff, WAN routing and firewalling.
- **Core:** backbone connectivity between major network parts.
- **Distribution:** aggregates access links and places routing/policy where designed.
- **Access:** classifies wired clients and wireless SSIDs into intended networks.

A small network can collapse core/distribution/access into shared switching. If a Layer 3 switch routes traffic, the gateway firewall does not automatically inspect it. Separate gateway/switch/AP devices improve placement and independent replacement; an all-in-one can suit smaller requirements.

The layers make **ownership and traffic paths** visible. Draw where clients enter, where trunks aggregate, where routing happens and which device enforces policy. This is more useful than copying an enterprise three-tier diagram with unused boxes. A small gateway-plus-switch design can implement several responsibilities, while a larger property or busy storage network may justify extra access or aggregation switches.

Separate components let radio placement follow coverage instead of the ISP socket, and let a failed AP or undersized switch be replaced without replacing routing. They add power supplies, cables and update work. An all-in-one is a reasonable small-network choice when coverage, ports and controls suffice; it is not inherently less professional.

---

## Segmentation and Access

Possible trust roles include administration, trusted household clients, lab workloads, untrusted/IoT devices and visitors. Choose only the segments needed and deny unnecessary new traffic between them; allow specific services and stateful replies.

VLANs separate Layer 2 domains, not availability or all communication. Same-VLAN traffic may never cross a gateway. Protect router services separately from forwarded traffic; namespaces, NAT and SSID names are not firewall isolation.

Use non-overlapping subnets and explicit access/trunk membership. Match native/tagged settings along links and limit host/AP trunks to required segments. Check overlap with ISP/VPN and container/pod/service networks. For IPv6, plan prefix delegation and equivalent policy/control traffic before enabling it; an explicitly documented IPv4-only baseline can be appropriate while that is unresolved.

### Trust Roles and Minimum Access

| Role | Why separate it | Typical policy question |
|---|---|---|
| Administration | Network/hypervisor control has a high impact | Which approved clients can administer which devices? |
| Household clients | Trusted users need normal connectivity | Which specific lab applications, if any, should be reachable? |
| Lab workloads | Experiments can be faulty or compromised | What dependencies are required without broad household access? |
| IoT / constrained clients | Limited update/support and different trust | Does a device need internet, a local controller or discovery? |
| Visitors | Internet access without household trust | Can guests reach only necessary gateway services and WAN? |

These are reusable roles, not a prescribed VLAN count or a map of the owner's network. Combine or split roles according to risk and operations. Prefer an explicit allowed-flow matrix over dozens of undocumented exceptions. Default-deny new cross-boundary traffic where feasible, while preserving replies and required infrastructure services.

### Routing, Gateway Services and Local Traffic

Treat traffic **through** a firewall separately from traffic **to** the gateway. An inter-VLAN block may not stop a client reaching the router console; DHCP/DNS permissions should not grant unrestricted management. Same-VLAN flows may bypass the routed firewall entirely, requiring host firewalls, device controls or supported client isolation.

Service discovery such as mDNS is another deliberate cross-boundary feature. Allow it narrowly only when needed, and distinguish discovering a device from permission to connect to it. NAT hides or translates addresses; it does not replace an access policy. Lab host bridges must not accidentally join household and experiment networks.

### Addressing and VLAN Membership

Use a private address plan that will not collide with common VPN sites, existing household ranges or cluster ranges. Size each subnet for its role; the VLAN ID does not have to match an address octet. Reserve infrastructure addresses and keep static allocations outside dynamic pools or represented by appropriate reservations.

Access ports usually carry one untagged client segment. Trunks carry only the tagged networks required by a switch, AP or hypervisor, with an explicitly agreed native/untagged behavior. A native mismatch or unrestricted host trunk can defeat an otherwise good design. Management over tags versus untagged access is a platform/port-design decision, not a universal rule.

IPv6 requires a real plan for delegated prefixes, addressing, DNS, router advertisements and policy. Multiple addresses and global reachability do not imply that all inbound traffic should be allowed. Preserve essential ICMPv6/control traffic and verify both allowed and denied paths; disabling all ICMP is not a safe substitute for understanding it.

---

## Equipment and Management Choices

| Choice | Benefits | Trade-offs / checks |
|---|---|---|
| ISP / consumer all-in-one | Few boxes and familiar support | Verify routed VLANs, wireless isolation, port count and placement |
| Integrated managed ecosystem, e.g. UniFi or Omada | Coordinated gateway/switch/AP configuration | Vendor dependence, model features and local/cloud controller requirements |
| Firewall appliance with OPNsense/pfSense | Router-OS control and extensions | OS/plugin lifecycle plus separate switch/AP management |
| MikroTik, OpenWrt or VyOS | Flexible routing/configuration options | Learning curve, supported hardware/images and operational responsibility |
| Mixed-vendor devices | Standards-based flexibility and purchasing freedom | More configuration, update and recovery workflows |

Check gateway performance with inspection/VPN enabled, not just port speed. Choose switches by access/trunk controls, restricted management, port/uplink capacity and per-port **and total** PoE budget. An unmanaged switch passing tags does not enforce segmentation.

Choose APs and Ethernet/fibre links for actual coverage, length, speed, power and installation requirements. Existing good cable may suffice; use suitable copper/PoE-rated cabling, not copper-clad aluminium. Include power, ventilation and support/return terms in the budget. See [Wi-Fi](./WiFi.md).

### Router Platform Trade-offs

**Integrated ecosystems**, such as UniFi or Omada, can coordinate gateway, switch, AP and client views. This reduces duplicated configuration and is useful when the foundation should be easy to maintain. Verify local versus cloud management, controller placement, recovery without cloud access and the exact model's features. Accept ecosystem dependence and update/API limitations rather than assuming a complete IaC interface exists.

**OPNsense/pfSense appliances** suit firewall-focused control, VPN and extensions with independently selected switches/APs. They integrate much functionality that a custom Linux router would otherwise assemble. You still maintain the OS, supported packages, hardware compatibility, backups and separate wireless/switch administration. Purpose-built multi-NIC x86 systems can fit this role, but inspect NIC chipset support, cooling, firmware, warranty and measured consumption rather than buying from a CPU label alone.

**MikroTik, OpenWrt and VyOS** offer different routing and configuration ecosystems. MikroTik pairs its platform with a broad hardware range; OpenWrt depends on support for the exact device; VyOS follows a routing-oriented appliance model with its own image/support choices. They can provide strong learning and flexibility, at the cost of understanding their policy model and operations. A mixed-vendor standards-based design avoids one management ecosystem but asks you to integrate several.

### Switching, Power and Cabling

Buy managed switching for VLAN membership controls, management restrictions and visibility, not merely a product that passes tagged frames. Count current ports plus realistic growth and uplink needs. Shared gigabit uplinks can bottleneck backups or storage even when a gateway has faster ports; route and switch throughput are different questions.

PoE simplifies AP placement, but check the required standard, maximum per-port delivery and the switch's total budget simultaneously. Include future powered devices and supported injectors where appropriate. Passive PoE is not interchangeable with negotiated IEEE PoE. Keep gateway power requirements distinct from AP power; not every gateway supplies PoE or Wi-Fi.

Choose copper by required speed, installation length, termination quality and PoE suitability. Fibre can suit distance or electrical separation, but adds transceivers and compatibility checks. Avoid replacing good cabling solely for a higher category number. Buying new supported gateway/switch equipment can be justified for household recovery/support, while refurbished compute has a different risk profile; compare warranty and delivered costs rather than declaring one channel always best.

---

## Implementation and Verification

After choosing the design, plan addressing, port/SSID roles, provider requirements and a known-good wired recovery path. Stage changes incrementally; verify the replacement access path before removing the old one. Vendor guides own exact installation commands and UI labels.

Test DHCP/DNS, authorized services and intended denials in every enabled IP family. A failed ping does not prove isolation: establish that the service works from an authorized source, then test the same service from the denied source. Check household connectivity with lab compute off when independence is required.

Choose restore/replacement versus supported redundancy according to outage impact. Duplicate boxes alone do not provide tested failover. Keep disruptive DHCP/RA/router experiments isolated and raw backups/endpoints protected under the [guidelines](../../Repository_Guidelines.md).

Make changes in a recoverable order: retain a trusted wired bootstrap path, establish the new administration path, apply intended membership/policy and validate before removing the old path. Keep a protected configuration backup and know how to reach local management if internet or cloud sign-in fails.

For availability, distinguish a spare device that can be restored from automatic failover with synchronized state, supported upstream behavior and tested client recovery. A UPS can reduce short power interruptions but does not fix firmware errors or provide a second ISP. Spending on redundancy makes sense when the outage impact justifies its lifecycle; a documented replacement/restore process is often enough for learning environments.

Record policy tests with source role, destination service, IP family and expected outcome. Recheck after firmware changes, new exceptions and AP/port changes. Public lessons describe the method; private records hold the actual values and outcomes. Link vendor installation documentation rather than maintaining a click-by-click public imitation of a household setup.
