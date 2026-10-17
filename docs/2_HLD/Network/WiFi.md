# Wi-Fi

Select coverage, backhaul, management and segmentation together with the [network design](./README.md).

> **Scope:** This page evaluates home-scale coverage and equipment. Use [DevHub's network-design principles](https://github.com/CollinPoetoehena/DevHub/blob/main/reference/network/Network_Design.md) for general segmentation/access rules, and the [repository relationship](../../../README.md#how-devhub-and-the-homelab-repositories-fit-together) for where shared explanations and private measurements belong.

---

## Table of Contents

- [Design Options](#design-options)
- [Backhaul and Equipment](#backhaul-and-equipment)
- [Implementation and Verification](#implementation-and-verification)

---

## Design Options

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| ISP/all-in-one Wi-Fi | Gateway and radio in one device | Few devices, often already available; economical if centrally located | Provider socket/cabinet may be a poor radio location; isolation and controls vary. **Maintenance:** relatively few settings/devices, but gateway maintenance also interrupts Wi-Fi; recheck guest isolation and credentials after reset/replacement | Coverage and security requirements are modest and verified |
| Consumer mesh | Coordinated multi-unit wireless coverage, with wireless or supported wired backhaul | Convenient coverage expansion and lower setup effort where cabling is impractical | Check router/AP mode, VLAN SSIDs, guest isolation, subscription/cloud dependence and backhaul; mesh is not proof of segmentation. **Maintenance:** coordinated updates can simplify upkeep; still track unit support, app/account recovery and inter-node signal changes after moving furniture/units | Simple coverage is the priority and supported features satisfy policy |
| Managed wired APs, e.g. [UniFi](https://help.ui.com/hc/en-us/categories/6583256751383-UniFi) / [Omada](https://www.tp-link.com/en/support/faq/) | Separate radios with Ethernet uplinks and coordinated SSID/VLAN management | Radio placement independent of routing; predictable uplinks and segmented wireless access | Cabling, controller availability/local recovery and compatible PoE/injectors; ecosystem features vary. **Maintenance:** more initial VLAN/PoE/coverage planning, then centralized profiles and updates; back up controller settings and verify SSID policy/roaming after changes | Maintainable coverage and wireless trust boundaries matter |
| Mixed-vendor wired APs | Independently selected APs using supported networking standards | Purchasing freedom and standards-based flexibility | Several update/configuration workflows; roaming/security consistency needs coordination. **Maintenance:** higher manual coordination for channel/power/security settings and release cycles; retain exports and test client behavior across vendors | Interoperability and operations are understood |
| Wireless repeater | A device retransmits an existing wireless connection | Low installation effort for limited reach extension | Adds a radio hop and airtime contention; cannot fix a poor upstream signal or internet link. **Maintenance:** another firmware/credential lifecycle and failure point; retest uplink quality after placement or upstream SSID changes | A bounded coverage gap with acceptable throughput/latency after testing |

These are general approaches, not recommendations for the owner's installed models. First distinguish a coverage problem from an internet/uplink problem; adding radios does not fix every slow connection.

---

## Backhaul and Equipment

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

---

## Implementation and Verification

Choose bridge/AP versus router mode deliberately to avoid unintended NAT/DHCP. Map required SSIDs to the correct segments, protect administration and use compatible WPA2/WPA3 modes. Limit SSID count and legacy exceptions.

Validate association, addressing, backhaul, intended access/denials, roaming and recovery. Client roaming depends on placement, settings and client behavior; neither mesh branding nor multiple APs guarantees seamless handover.

Measure in the rooms and usage conditions that matter, including latency and stability rather than only a speed test beside an AP. Check that a roaming client retains the expected network/policy and that one radio/client does not create an unintended bridge. Review guest client isolation on wired and wireless paths separately.

Use compatible security modes and a documented update cadence. If legacy clients force an exception, contain it to their segment and record why. Exact SSIDs, AP placement and operational surveys remain private; a public explanation can show the evaluation process without a floor plan or device locations.
