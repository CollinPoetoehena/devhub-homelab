# Wi-Fi

Select coverage, backhaul, management and segmentation together with the [network design](./README.md).

---

## Table of Contents

- [Design Options](#design-options)
- [Backhaul and Equipment](#backhaul-and-equipment)
- [Implementation and Verification](#implementation-and-verification)

---

## Design Options

| Approach | Benefits | Drawbacks / checks | Choose when |
|---|---|---|---|
| ISP/all-in-one Wi-Fi | Few devices, often already available | Gateway location may be poor for signal; controls vary | Coverage and security requirements are modest and satisfied |
| Consumer mesh | Convenient coordinated coverage | Router/AP mode, VLANs, cloud dependence and backhaul vary | Cabling is difficult and simple coverage is the priority |
| Managed wired APs, e.g. UniFi/Omada | Predictable uplinks and coordinated SSID/VLAN management | Cabling, PoE/injectors and management requirements | Segmented wireless access and maintainable coverage matter |
| Mixed-vendor wired APs | Independent choices and standards-based flexibility | Multiple update/configuration workflows | Interoperability and operations are understood |

**ISP/all-in-one Wi-Fi** is economical when a centrally located device reaches the required rooms. Placement near an access cabinet or outside living areas can make it poor coverage even with a capable radio. Adding repeaters may improve reach but adds another radio hop; first distinguish a coverage problem from an internet/uplink problem.

**Consumer mesh** coordinates multiple units and can reduce setup work where cabling is impractical. Check AP/bridge mode when another gateway already routes, support for wired backhaul, guest isolation, subscription/cloud features and whether VLAN-mapped SSIDs are available. Mesh is a coverage/management approach, not proof of advanced segmentation.

**Managed wired APs** decouple routing from radio placement and can map SSIDs into the chosen trust networks. UniFi/Omada are ecosystem examples, not a recommendation for a particular installed model. Plan controller availability, local recovery and compatible switch/PoE support. Mixed vendors can achieve the same standards-based design, but consistent roaming/security settings and updates need more coordination.

---

## Backhaul and Equipment

Ethernet avoids radio-backhaul contention at the cost of installation. Wireless backhaul is useful without cable but depends on inter-node signal/interference; dedicated radios help without removing that dependency. Mesh products may support wired backhaul, and managed APs may support meshing: compare features, not labels.

Survey placement/building materials before AP count. Compare client support, bands, local spectrum rules, uplink and PoE budget; advertised PHY rates are not application throughput. More APs or maximum transmit power is not automatically better.

An AP needs a good uplink where it is installed; a wireless mesh node at the edge of an already unusable signal cannot manufacture backhaul capacity. Test inter-node signal, not just client signal. Ethernet is predictable and leaves more airtime for clients, while wireless backhaul is a practical trade-off for rooms where installation cost or building constraints dominate.

Use 2.4 GHz for coverage/legacy needs and suitable 5/6 GHz capability for supported clients where range and local rules allow. Wider channels can improve peak rates but reduce channel reuse and may be worse in crowded environments. Walls, floors, antenna orientation and client radios matter; a Wi-Fi-generation badge is not a coverage survey.

Keep SSID count purposeful because each adds management airtime. A separate IoT compatibility SSID can avoid weakening every client network, but must map to real policy. AP management belongs on a restricted path; wireless clients should not gain administration merely because they can associate.

---

## Implementation and Verification

Choose bridge/AP versus router mode deliberately to avoid unintended NAT/DHCP. Map required SSIDs to the correct segments, protect administration and use compatible WPA2/WPA3 modes. Limit SSID count and legacy exceptions.

Validate association, addressing, backhaul, intended access/denials, roaming and recovery. Client roaming depends on placement, settings and client behavior; neither mesh branding nor multiple APs guarantees seamless handover.

Measure in the rooms and usage conditions that matter, including latency and stability rather than only a speed test beside an AP. Check that a roaming client retains the expected network/policy and that one radio/client does not create an unintended bridge. Review guest client isolation on wired and wireless paths separately.

Use compatible security modes and a documented update cadence. If legacy clients force an exception, contain it to their segment and record why. Exact SSIDs, AP placement and operational surveys remain private; a public explanation can show the evaluation process without a floor plan or device locations.
