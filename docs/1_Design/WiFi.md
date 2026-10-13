# Wi-Fi Options for a Homelab

Separate **coverage**, **backhaul**, **management**, and **segmentation**. "Mesh" is not a guarantee of any of them. Several wired APs can share SSIDs and support roaming without wireless backhaul; client decisions, placement and radio settings still determine actual handover.

| Approach | Benefits | Drawbacks / checks | Choose when |
|---|---|---|---|
| ISP/all-in-one Wi-Fi router | Few devices, often already available | Gateway cable location may be poor for radio coverage; VLAN controls vary | A small space with sufficient coverage and security controls |
| ISP extenders/APs | Convenient provider support and compatibility | Ownership, subscription, third-party gateway compatibility and SSID/VLAN controls vary | Requirements fit the exact provider system |
| Consumer mesh (e.g. Deco, Orbi, ASUS systems) | Convenient multi-point coverage and coordinated configuration | Router/AP mode, VLAN mapping, local/cloud management and wired backhaul vary | Cabling is difficult and simple coverage is the priority |
| Managed wired APs (e.g. UniFi or Omada) | Predictable Ethernet uplinks, SSID/VLAN controls, coordinated management | Cable installation, PoE/injectors, controller placement and cost | Segmented Wi-Fi and maintainable coverage are required |
| Mixed-vendor wired APs | Standards-based flexibility, independent hardware choices | Separate configuration/update workflows; features may not align | Interoperability and administration are understood |

Consumer mesh can use Ethernet backhaul, and some systems support suitable segmentation. Managed AP systems may also support wireless meshing. Compare capabilities, not branding.

## Wired versus wireless backhaul

Ethernet avoids radio backhaul contention and provides predictable uplinks, at the cost of installation effort. PoE can simplify powering ceiling/wall APs; verify the standard and total switch budget.

Wireless backhaul suits locations where cable is impractical. It depends on signal and interference between nodes as well as client links; extra hops and shared radios can reduce useful capacity. Dedicated backhaul radios help but do not remove those dependencies. Place a node where it still has a good upstream signal, not only in the final dead zone.

## Choosing and installing

- Survey coverage and building materials before deciding AP count. More radios or transmit power is not always better.
- Check Wi-Fi generation, supported bands, local 6 GHz rules, client compatibility, Ethernet uplink and realistic throughput. PHY rates are not application throughput.
- Verify multiple SSIDs mapped to VLANs, management isolation, and behavior in bridge/AP mode. A branded guest network does not prove routed isolation from every internal segment.
- Avoid an extra router/NAT stack if only wireless access is required. Verify DHCP ownership and all switch/AP VLAN allowances.
- Prefer WPA3 where compatible; confine legacy compatibility to the devices that need it. Limit SSID count and validate channel width/power instead of maximizing every setting.
- Determine which functions need a controller or cloud account and whether existing forwarding continues during their outage.

UniFi can provide cohesive gateway/switch/AP administration with a supported local console. Omada and other managed systems can meet similar needs with different controller/device choices. That convenience trades some flexibility for ecosystem dependence; it is a generic option, not disclosure of the actual installation.

Test association, DHCP/DNS, application access, intended denials, roaming and wired backhaul. Wireless client isolation complements inter-segment firewall rules but does not necessarily cover wired peers. See [network design](./3_Design_Network.md) and [network setup](../2_Setup/Network.md).
