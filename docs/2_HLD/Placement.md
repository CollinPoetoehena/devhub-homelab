# Equipment Placement and Accessories

Choose a location that makes equipment safe, cool, reachable and maintainable. Space, ventilation, safe electrical access and cable routing matter more than hiding equipment or making the installation look like a data center. This guide is generic; actual placement and purchases belong in the private homelab record.

---

## Table of Contents

- [Choosing a Location](#choosing-a-location)
- [Meter Cabinets and Network Runs](#meter-cabinets-and-network-runs)
- [Mini PCs and Ventilation](#mini-pcs-and-ventilation)
- [Racks and Rack Units](#racks-and-rack-units)
- [Power Accessories](#power-accessories)

---

## Choosing a Location

Keep essential network equipment accessible independently of lab compute. Leave room to reach connectors, power supplies and service controls; label both ends of fixed cables and avoid tight bends or connector strain. A convenient meter cabinet may be suitable for the modem, router and a switch when space, cooling and safe utility access permit it. Do not assume a closed cabinet has adequate ventilation.

Compute can be located with the network equipment or elsewhere nearby. If the meter cabinet is already crowded, too warm, difficult to cable or electrically unsuitable, use a ventilated shelf, desk or equipment rack instead. Never obstruct meters, breakers, isolation controls or required service clearance. Follow local electrical rules and device installation instructions; do not modify electrical installations or drill where concealed wiring could be present.

One possible physical flow is:

```text
ISP service -> modem/ONT -> router/gateway -> managed switch
                                          +-> room Ethernet / Wi-Fi access points
                                          +-> compute host(s) on a suitable shelf or rack
```

The modem/ONT and router may stay in the meter cabinet if the provider handoff and cable layout make that practical. The switch can also stay there to connect wall runs, provided it has adequate space, airflow and power. Compute and accessories can be kept in a rack elsewhere. Alternatively, all equipment can be together in a rack, or a very small setup can use a desk/shelf and avoid a rack entirely. A rack is an organizational option, not a homelab requirement.

If a downstream switch is used near compute, it extends physical connectivity but does not create a security boundary by itself. Configure VLAN membership and firewall policy on appropriate managed equipment; test the full Ethernet path to each room/AP.

---

## Meter Cabinets and Network Runs

Connect a separate router's WAN port to the provider modem/router/ONT handoff and its LAN to the main switch, subject to the ISP's bridge/routing behavior. Check device port speeds, PoE needs and supported uplinks. Use the [Network guide](./Network/README.md) for network equipment and policy choices, and the [Wi-Fi placement guide](./Network/WiFi.md) for AP coverage.

Route fixed Ethernet through suitable conduits with accessible terminations and service slack. Label cable destinations and test runs end-to-end. Locate APs for radio coverage outside a metal or crowded cabinet; a central cabinet is not automatically a good Wi-Fi location.

---

## Mini PCs and Ventilation

Compact mini PCs can sit on a stable shelf without a rack. Leave intake/exhaust vents open, keep power bricks ventilated and leave clearance to remove each unit. Stack only where the manufacturer's guidance and vent placement allow it, using a stable ventilated stand if needed. Recheck CPU and SSD temperatures under representative workloads after moving or enclosing equipment.

For laptops, inspect battery/charger condition, follow the manufacturer's charging and thermal guidance, and do not use a damaged or swollen battery. Confirm that the lid-close setting does not suspend a host expected to run services. A laptop battery is not a substitute for a tested UPS.

---

## Racks and Rack Units

A rack organizes equipment, shelves, patch panels and power accessories, and can make airflow and cable management easier. It can also add cost and consume floor/wall space; small setups may be better on a ventilated shelf. Choose open-frame racks or ventilated cabinets with sufficient depth, load capacity, cable clearance and safe access. Leave clearance around vents, avoid blocking exhaust and monitor device temperatures; a rack does not guarantee cooling by itself. A lock can deter casual access but is not a substitute for physical security.

Rack height is measured in **rack units (U)**: **1U is 1.75 inches / 44.45 mm** (44.45 mm). A 6U rack has six units of nominal vertical mounting space; 9U has nine. Count equipment, shelves and patch panels, then leave practical expansion and service clearance. A shelf-mounted mini PC consumes vertical clearance above its shelf as well as the shelf's mounting space. U does not specify depth, width, weight capacity or cooling. Check the usual 19-inch mounting width, usable depth, rail/shelf compatibility, load limits and manufacturer's installation guidance.

Wall-mounted cabinets must be attached to a suitable structure using an appropriate method and load rating. A wall-mount model is not automatically safe to place freestanding on the floor; follow its instructions and use a stable rated base or select a floor-standing model if it must remain movable.

---

## Power Accessories

### Surge-protected power strip

A grounded strip with enough outlets (often eight or more for a small group) can power compute hosts, networking and accessories together. Choose a suitable cable length and rated load; check device plug and power-adapter clearance. Do not exceed its rating, daisy-chain strips, or mistake surge protection for battery backup. If it is measured through a smart plug, the meter reads the combined load downstream of that plug.

### UPS

A UPS is optional. Consider one when short power interruptions or orderly shutdown matter; size it for the actual load and required runtime, and verify USB/network signaling can request graceful shutdown. Batteries age and runtime is finite. A UPS does not solve failed equipment, provider outages, fire risk or backup requirements. Do not buy one solely because a server rack is present.

### Smart energy-monitoring plug

One appropriately rated metering plug on the common power strip can estimate combined homelab watts and kWh; one plug per device is often unnecessary for a simple setup. Use it to compare idle and active workloads, track trends and estimate monthly costs. A monitoring stack such as Prometheus and Grafana can visualize exported measurements, but the plug may need a supported integration/bridge rather than exporting metrics directly. Verify protocol, data access, privacy and reliability before depending on it.

If the plug uses Wi-Fi, place it in the appropriate IoT/untrusted network and allow only the specific monitoring path needed. Do not put it on an untrusted segment that cannot reach the collector without a deliberate firewall exception, and do not weaken segmentation just to make a dashboard work. The energy monitor is optional: buy it only if measuring energy supports a real goal.
