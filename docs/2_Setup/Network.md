# Network Setup Workflow

Implement the selected [network design](../1_Design/3_Design_Network.md) and [wireless approach](../1_Design/WiFi.md), not a universal device recipe. Exact ports, VLAN IDs, subnets and SSIDs belong in the private runbook.

1. Plan provider handoff/authentication, bridge mode and TV/voice dependencies; verify subnet overlap and recovery before changing WAN settings.
2. Stage the gateway with only trusted setup equipment. Configure WAN, protect administrative accounts and test local access. Remote/cloud administration is a separate explicit choice.
3. Connect and initialize managed switching/APs using vendor-supported procedures. Preserve bootstrap access while moving management; do not connect untrusted clients yet.
4. Create selected networks and DHCP/DNS settings. Verify IPv6 delegation/policy before enabling it; document an IPv4-only baseline if IPv6 is pending.
5. Configure explicit inter-zone and gateway-service permissions. Default-deny unneeded new traffic, preserve stateful replies and necessary DHCP/DNS/IPv6 control traffic, and inspect effective rule order.
6. Assign access/trunk profiles with matching native settings and only required tags. Test the replacement management path before restricting or removing the bootstrap path.
7. Map SSIDs to networks, apply compatible WPA2/WPA3 settings, configure guest/client isolation and validate backhaul. Avoid adding router mode where AP mode is intended.
8. Connect clients/hosts incrementally. Verify reservations, management/guest separation, selected application exceptions and IPv4/IPv6 behavior.
9. Perform the [allowed/denied validation](./Verification.md), including household connectivity with lab compute off when that independence is a requirement.
10. Store dated protected device backups and recovery steps outside public Git. Confirm how to restore/re-adopt, and recheck connectivity after staged updates.

For a **generic UniFi implementation**, consult [initial setup](https://help.ui.com/hc/en-us/articles/4416276882327), [device adoption](https://help.ui.com/hc/en-us/articles/360012622613), [port VLAN assignment](https://help.ui.com/hc/en-us/articles/26136855808919), [SSID creation](https://help.ui.com/hc/en-us/articles/26136823938583), and [backup/migration](https://help.ui.com/hc/en-us/articles/360008976393). UI labels and zone defaults vary by release. Verify gateway-service policy as well as inter-network policy; a built-in zone name is not proof of isolation.

If a management move fails, restore the last known-good path rather than broadly opening every VLAN/firewall. A static address cannot repair an incorrect tag/path. Factory reset is a last resort after confirming backups and the model-specific recovery procedure.
