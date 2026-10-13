# Homelab Network Design

Design for trust boundaries and recovery before products. [DevHub networking](https://github.com/CollinPoetoehena/DevHub/tree/main/reference/network) owns protocol fundamentals; this document applies them to home-and-lab requirements. All structures below are generic patterns, not the actual home topology.

## Network setup options

| Pattern | Benefits | Drawbacks | Choose when |
|---|---|---|---|
| Flat household/lab LAN | Few settings, low initial cost | Experiments and untrusted clients share broadcast/access boundaries | Isolated temporary learning with no sensitive/shared dependencies; not the default for untrusted devices |
| Segmentation on ISP device | Fewer appliances | Capabilities and provider control vary | Exact model supports the required VLAN/firewall/Wi-Fi policy |
| Shared gateway with VLAN segmentation | One routing lifecycle, straightforward selected cross-segment access | Shared failure/security/bandwidth domain | Logical isolation is sufficient and consistent administration matters |
| Separate downstream lab router | Independent routing experimentation and a distinct boundary | Extra lifecycle, potentially double NAT, coordinated access/recovery | Deliberate router independence or experiments justify it |
| Physically separate networks | Stronger physical/failure separation | More equipment and operations; upstream/power can still be shared | Requirement cannot be met by logical isolation |

VLANs separate Layer 2 domains, not availability. A second router is neither automatically safer nor automatically unnecessary. Keep household DHCP/DNS and recovery independent of disposable lab compute where household availability matters.

## Layered design

Use **edge, core, distribution and access responsibilities** to reason about paths. They are logical roles, not a shopping list of enterprise switches.

```text
Provider handoff -> Edge gateway / WAN firewall
                           |
                 Core / distribution switching
                 (may be collapsed at home scale)
                           |
                  Access ports / wired APs
                           |
                  Clients and lab host uplinks
```

- **Edge:** terminates WAN routing and enforces inbound/outbound policy. Bridge/passthrough on provider equipment may avoid extra NAT where supported; verify voice/TV/authentication and IPv6 delegation first.
- **Core:** moves traffic between major parts of a larger network. A small installation often collapses this backbone role into the gateway and shared switching.
- **Distribution:** aggregates access links and hosts routing/policy where designed. If a Layer 3 switch routes, the gateway firewall does not automatically inspect that traffic.
- **Access:** classifies wired clients and SSIDs into the intended networks; carries only required VLANs on host/AP trunks.

A managed switch can combine core/distribution/access roles at home scale. Separate appliances improve placement and independent upgrades but add cabling, power and configuration. An all-in-one may be sufficient when placement, capacity and segmentation fit. Follow [hardware selection](./1_Design_Hardware_Prerequisites.md) and [Wi-Fi options](./WiFi.md).

## Segmentation and addressing

Choose segments from trust and access needs, not arbitrary device counts. Possible roles include administration, trusted clients, untrusted devices, visitors and lab workloads; combine or split them only with a reason. Keep the actual subnet/VLAN/port map privately.

- Assign distinct non-overlapping subnets and consistent DHCP/reservations. Check ISP LANs, VPNs, container bridges and Kubernetes pod/service CIDRs.
- Access ports classify untagged clients into one VLAN. Trunks carry selected tagged VLANs; native VLAN settings must agree.
- Limit AP/host trunks to needed segments. AP management and SSID traffic may require different tagged/native settings; verify adoption and local recovery behavior.
- VLANs alone are not firewall policy; same-VLAN traffic may never cross a router.

For IPv6, prefer supported prefix delegation with one `/64` per enabled segment. ULA is local addressing, not internet reachability. Global addressing does not imply open inbound access, and NAT is not the security boundary. Apply equivalent IPv4/IPv6 policy and permit necessary IPv6 control traffic. Until delegation and isolation are verified, a documented IPv4-only baseline can be safer than an accidental partial rollout.

## Firewall and management policy

Default-deny new connections between trust zones, then permit required source/destination/service flows and stateful replies. Protect the gateway's own DHCP/DNS/administrative services separately from forwarded traffic. NAT, separate subnets and SSID names do not establish these permissions.

Keep visitor/untrusted clients away from administration. Permit a household client to a particular lab application only when required, not to every lab management interface. Same-VLAN isolation needs supported client isolation or host controls. Scope cross-network discovery carefully; broad multicast reflection can undermine intended boundaries.

Use restricted administrator identities, MFA where supported, no unnecessary WAN administration and a local wired recovery path. A bastion can simplify access but must not be the only way to recover the network. Remote VPN design needs deliberate authentication, reachability, routing and least privilege.

## Availability and experiments

Enterprise redundancy may include independent ISPs, supported firewall HA pairs, redundant switching/links, overlapping wireless coverage and power protection. Failover must actually be configured/tested; common power, cabling, upstream or configuration failures can defeat duplicate devices.

At home scale, documented restore procedures, replacement availability and a limited mobile-internet fallback may be a reasonable alternative. A hotspot does not restore the internal LAN or its normal policy. Choose against actual outage impact, not an enterprise aesthetic.

Run router, DHCP, RA, bridging and firewall experiments inside isolated networks. Validate required **allowed and denied** paths and household connectivity with lab compute off. Preserve a rollback path before changing management VLANs or provider mode. See [network setup](../2_Setup/Network.md) and the [publication guidelines](../Repository_Guidelines.md).
