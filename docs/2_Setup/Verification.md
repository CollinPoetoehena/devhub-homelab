# Verification and Troubleshooting

A design is intent; verification is evidence. Keep actual versions, targets, outputs and results privately. Mark each check passed, failed or not tested, with the conditions tested.

| Area | Check |
|---|---|
| Physical | Link speed, cabling, power/PoE, temperature and device health |
| Addressing | Correct DHCP/gateway/DNS, no overlap or rogue DHCP/RA |
| Network policy | Required services reachable only from authorized sources; other paths denied in every enabled address family |
| Wi-Fi | Association, SSID/VLAN mapping, coverage, backhaul, guest isolation and roaming |
| Management | Restricted accounts, trusted TLS, local recovery without unavailable dependencies |
| Compute/cluster | Guest readiness, cluster DNS, pod/service paths, storage and enforcing policy |
| Operations | Monitoring ingestion, alert delivery, usable backups and isolated restoration |
| Independence | Essential household services work with lab compute off where required |

**A failed ping is not proof of a firewall rule.** Confirm the target service is listening and reachable from an authorized client, then test the same address/port from the denied source and inspect appropriate events/counters. Distinguish gateway rules, host firewalls, routing, wrong targets and closed services.

Troubleshoot from physical/link through addressing/routing, DNS, policy and application. For tagged paths inspect both ends' native/tagged VLAN settings and the whole AP/switch/gateway path. For IPv6 inspect advertisements, delegation and family-specific policy. Use [DevHub network diagnostics](https://github.com/CollinPoetoehena/DevHub/tree/main/reference/network) for generic commands.

Change one relevant variable at a time, retain the working management path, and restore the last known-good setting before disruptive resets. Do not permanently disable isolation or TLS verification as a shortcut.

Publish a lesson only after applying the [sanitization workflow](../Repository_Guidelines.md#sanitization-workflow); private output and diagrams are not public evidence bundles.
