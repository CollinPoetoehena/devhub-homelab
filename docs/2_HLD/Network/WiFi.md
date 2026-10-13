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

---

## Backhaul and Equipment

Ethernet avoids radio-backhaul contention at the cost of installation. Wireless backhaul is useful without cable but depends on inter-node signal/interference; dedicated radios help without removing that dependency. Mesh products may support wired backhaul, and managed APs may support meshing: compare features, not labels.

Survey placement/building materials before AP count. Compare client support, bands, local spectrum rules, uplink and PoE budget; advertised PHY rates are not application throughput. More APs or maximum transmit power is not automatically better.

---

## Implementation and Verification

Choose bridge/AP versus router mode deliberately to avoid unintended NAT/DHCP. Map required SSIDs to the correct segments, protect administration and use compatible WPA2/WPA3 modes. Limit SSID count and legacy exceptions.

Validate association, addressing, backhaul, intended access/denials, roaming and recovery. Client roaming depends on placement, settings and client behavior; neither mesh branding nor multiple APs guarantees seamless handover.
