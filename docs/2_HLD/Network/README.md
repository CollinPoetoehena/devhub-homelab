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

| Option | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| Flat LAN | Clients share a broadcast domain with ordinary gateway addressing/internet access | Few settings, low initial cost and easy first connectivity | Broad lateral reachability, including management; a different SSID name alone does not isolate clients. **Maintenance:** low configuration effort, but endpoint patching and gateway updates remain; limited isolation can make incident containment harder | Temporary isolated learning without sensitive household dependencies |
| Segmentation on ISP equipment | The provider gateway supplies separate subnets/guest networks and access policy | Fewer appliances; retains familiar provider support | VLAN/routing/firewall features, export and provider control vary; verify actual isolation rather than trusting a guest button. **Maintenance:** relatively low assembly effort; recheck policy after firmware changes and retain a recovery method when exports are limited | The exact device satisfies the required policy and recovery needs |
| Shared gateway with VLANs | One gateway routes multiple trust networks over VLAN-aware switches/APs | One routing, DHCP/DNS, update and recovery lifecycle; explicit cross-segment access; household IoT/guest separation | Shared outage, compromise and bandwidth domain; a policy mistake can affect several segments; no physical independence. **Maintenance:** moderate initial policy/trunk design, then one routing lifecycle; review allowed flows, back up settings and stage updates because mistakes affect household and lab together | Logical isolation and consistent management are sufficient for a maintainable small network |
| Separate downstream lab router | An experimental/independently administered router sits behind the household gateway | Dedicated router-OS practice and a deliberate routing/firewall boundary | Second lifecycle/address plan; coordinated policy, inbound access, VPN and IPv6; possible double NAT; upstream household reach still needs restriction. **Maintenance:** higher recurring effort for two firmware/OS lifecycles, backups and recovery paths; troubleshoot both routing layers and keep cross-boundary changes synchronized | Independent router learning or separate administration is itself a requirement |
| Physically separate network | Separate gateways, switches and cabling implement distinct hardware boundaries | Reduces shared hardware and accidental bridging dependencies | More cost and operations; ISP, power and administrators may remain shared. **Maintenance:** additional devices, spares, updates and physical fault tracing; document remaining shared dependencies instead of assuming complete independence | Logical policy cannot satisfy the experiment/trust requirement |
| Virtual router | A router VM supplies routing for selected virtual/downstream networks | Disposable, reproducible labs without another physical appliance | Household edge operation depends on hypervisor/storage/boot order; an outage can remove the path needed to fix the host; DHCP/RA experiments need isolation. **Maintenance:** repeatable VM resets can simplify experiments, but permanent edge use adds host/bridge/NIC and boot-order coordination; keep offline recovery access | Isolated routing exercises, or an explicitly accepted and recoverable virtual-edge dependency |

A second router is not automatically safer; a shared gateway is not automatically suitable for every experiment. Household DHCP/DNS and local recovery should not depend on disposable lab compute when household availability matters.

Maintenance descriptions are qualitative comparisons of ownership, not measured hours or promises that a product is maintenance-free. More segments, plugins, public services and independent devices increase work; operator familiarity, supported updates and tested recovery can matter more than initial installation speed.

### Flat or ISP-Managed Networks

For the flat/ISP options in the table, test guest-to-LAN and guest-to-guest behavior separately. Check whether the provider gateway can route distinct subnets, enforce chosen cross-network rules and restrict administration; some devices offer only a fixed guest feature.

### One Shared Segmented Gateway

For a shared gateway, define allowed flows before configuring VLANs. Restrict administration, keep essential services off disposable compute and test policy. Logical segmentation is not a safe place for uncontrolled DHCP, RA or bridging experiments.

### A Separate Downstream Lab Router

The separate-router pattern still needs a deliberate upstream boundary. Keep management/recovery reachable and avoid making household Wi-Fi or DNS depend on the experimental router. NAT versus explicit upstream routing is compared below; neither removes the need for policy.

### Physical Separation and Virtual Routers

Define which dependencies must be independent before choosing physical separation. For virtual experiments, constrain the upstream link and prevent experimental DHCP/RA or bridges from reaching household segments. Router hardware placement and router software are separate choices; see the platform table below.

### ISP Handoff, NAT and Remote Access

Provider access may use a modem, optical termination or another access device; replacing routing does not necessarily replace that termination. Bridge/passthrough can avoid duplicate IPv4 NAT, but provider authentication, VLAN tagging, TV/voice and IPv6 delegation may constrain the choice. With double NAT or carrier-grade NAT, do not assume a port forward on the lab gateway creates internet reachability.

| Option | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| Provider bridge/passthrough | Retain access termination while handing routing to the chosen gateway | Avoids a second provider IPv4 NAT layer; clearer routing ownership | Provider/model support, authentication, TV/voice and IPv6 delegation must be checked; modes differ. **Maintenance:** retain provider handoff/recovery details and revalidate after replacing access equipment or changing service | Supported handoff with a separately managed gateway |
| Downstream IPv4 NAT | A lab router translates addresses behind an existing router | Convenient outbound connectivity without adding an upstream route | Double NAT complicates inbound services, VPNs and diagnostics; does not solve carrier-grade NAT or IPv6 design. **Maintenance:** synchronize any forwards/address reservations and diagnose translation at both layers; outbound-only use is simpler | Outbound-focused experiments where limitations are acceptable |
| Routed downstream subnet | Upstream routes reach the lab without another translation layer | Clearer end-to-end addressing and fewer NAT layers | Requires supported upstream routes and explicit policy on both routers; ISP equipment may not support it. **Maintenance:** update routes and policy when ranges/interfaces change; test allowed/denied paths in both directions | Routing practice or services needing direct controlled reachability |
| Private/VPN application and admin access | Only authorized local/VPN clients reach services | Reduces public attack exposure; limits management reach | VPN identity, routing, updates and recovery still need maintenance; protect real endpoints. **Maintenance:** patch the entry point, revoke lost clients, manage keys and test a fallback administration path; VPN access does not justify broad internal permissions | Administration and services with no need for anonymous public reachability |
| Deliberate public application exposure | Selected services are reachable from the internet | Supports genuinely public workloads | Requires authentication where appropriate, TLS, updates, narrowly scoped firewall rules and recovery; management UIs should not become public by default. **Maintenance:** ongoing certificate renewal, security updates, exposure review and monitoring; public reach adds obligations beyond merely opening a port | Applications whose purpose requires external public access |

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

Choose APs and Ethernet/fibre links for actual coverage, length, speed, power and installation requirements. Existing good cable may suffice; use suitable copper/PoE-rated cabling, not copper-clad aluminium. Include power, ventilation and support/return terms in the budget. See [Wi-Fi](./WiFi.md).

### Router Platform Trade-offs

The table compares software/management approaches, not interchangeable hardware specifications. For a multi-NIC x86 appliance, inspect chipset support, cooling, firmware, warranty and measured consumption rather than buying from a CPU label alone. For a controller ecosystem, verify recovery without cloud access and do not assume every setting has a supported IaC interface.

### Switching, Power and Cabling

| Option | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| Unmanaged switch | Basic Layer 2 connectivity without configured port policy | Low cost and little administration | Passing VLAN tags is not enforcing membership; limited management/visibility. **Maintenance:** Very little configuration upkeep; inspect link/power health and aging hardware. Limited diagnostics can increase fault-isolation effort, and replacement does not add VLAN policy controls. | Simple same-trust access with no switch-level segmentation requirement |
| Managed Layer 2 switch | Explicit access/trunk membership and restricted administration | Enforces selected VLAN paths and provides diagnostics | Configuration, updates and recovery; shared uplinks can bottleneck traffic. **Maintenance:** back up port profiles and verify trunks/management access after firmware or cabling changes | Segmented hosts/APs and controlled port roles |
| Layer 3 switching | Selected routing occurs on the switch | Can reduce gateway transit and support larger/busier networks | Gateway firewall does not automatically inspect those flows; policy ownership and ACL support need planning. **Maintenance:** additional routing/ACL state and route convergence tests; keep policy consistent wherever routing occurs | Demonstrated inter-segment throughput/scale requirements |
| PoE switch | Negotiated IEEE power and data delivered to compatible clients | Central AP power and fewer local adapters | Check per-port standard/delivery and total budget, including growth. **Maintenance:** monitor power draw and re-evaluate budget before adding devices; switch failure can remove both data and AP power | Several powered APs/devices with suitable cabling |
| Supported PoE injector | One powered Ethernet link using a compatible injector | Adds power without replacing the switch | Extra adapter/outlet/cabling; passive PoE is not interchangeable with negotiated IEEE PoE. **Maintenance:** Little configuration work, but inspect power adapters, cable/connector health and supported power requirements after device replacement; keep a compatible spare if AP availability matters. | A few powered devices or an otherwise suitable non-PoE switch |
| Copper Ethernet | Twisted-pair data link, optionally carrying PoE | Familiar termination and power/data on one run; existing good cable may suffice | Speed/length/termination and PoE suitability; avoid copper-clad aluminium. **Maintenance:** usually little ongoing configuration; label/test runs and inspect damaged terminations before replacing active equipment | Ordinary room/building runs within supported limits |
| Fibre | Optical links with compatible transceivers | Useful for distance or electrical separation | Transceivers, termination and compatibility; no PoE to an AP. **Maintenance:** keep connectors clean, respect bend limits and retain compatible spare optics; diagnose optical and electrical paths separately | Longer links, electrical isolation or justified uplink capacity |

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
