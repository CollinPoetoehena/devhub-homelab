# Network

Design trust boundaries and recovery first. Networking gear and its management software are selected together to implement that design. Broad protocol theory belongs in [DevHub networking](https://github.com/CollinPoetoehena/DevHub/tree/main/reference/network).

---

## Table of Contents

- [Design Options](#design-options)
- [Layered Design](#layered-design)
- [Segmentation and Access](#segmentation-and-access)
- [Equipment and Management Choices](#equipment-and-management-choices)
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

---

## Layered Design

Use **edge, core, distribution and access** as responsibilities, not a requirement to buy an enterprise switch for each layer.

- **Edge:** provider handoff, WAN routing and firewalling.
- **Core:** backbone connectivity between major network parts.
- **Distribution:** aggregates access links and places routing/policy where designed.
- **Access:** classifies wired clients and wireless SSIDs into intended networks.

A small network can collapse core/distribution/access into shared switching. If a Layer 3 switch routes traffic, the gateway firewall does not automatically inspect it. Separate gateway/switch/AP devices improve placement and independent replacement; an all-in-one can suit smaller requirements.

---

## Segmentation and Access

Possible trust roles include administration, trusted household clients, lab workloads, untrusted/IoT devices and visitors. Choose only the segments needed and deny unnecessary new traffic between them; allow specific services and stateful replies.

VLANs separate Layer 2 domains, not availability or all communication. Same-VLAN traffic may never cross a gateway. Protect router services separately from forwarded traffic; namespaces, NAT and SSID names are not firewall isolation.

Use non-overlapping subnets and explicit access/trunk membership. Match native/tagged settings along links and limit host/AP trunks to required segments. Check overlap with ISP/VPN and container/pod/service networks. For IPv6, plan prefix delegation and equivalent policy/control traffic before enabling it; an explicitly documented IPv4-only baseline can be appropriate while that is unresolved.

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

---

## Implementation and Verification

After choosing the design, plan addressing, port/SSID roles, provider requirements and a known-good wired recovery path. Stage changes incrementally; verify the replacement access path before removing the old one. Vendor guides own exact installation commands and UI labels.

Test DHCP/DNS, authorized services and intended denials in every enabled IP family. A failed ping does not prove isolation: establish that the service works from an authorized source, then test the same service from the denied source. Check household connectivity with lab compute off when independence is required.

Choose restore/replacement versus supported redundancy according to outage impact. Duplicate boxes alone do not provide tested failover. Keep disruptive DHCP/RA/router experiments isolated and raw backups/endpoints protected under the [guidelines](../../Repository_Guidelines.md).
