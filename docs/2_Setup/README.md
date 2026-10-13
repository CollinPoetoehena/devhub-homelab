# Generic Setup Reference

These are adaptable implementation workflows, not the actual installation record. Concrete inputs, commands executed, tool versions, results and deviations belong privately. Follow the [repository guidelines](../Repository_Guidelines.md); do not copy live configuration here.

| Stage | Guidance |
|---|---|
| Define purpose and compare options | [Design reference](../1_Design/README.md) |
| Inspect/prepare hardware, tools and identities | [Preparation](./Preparation.md) |
| Establish WAN, segments, switching and Wi-Fi | [Network](./Network.md) |
| Install hosts, provision guests and bootstrap cluster | [Compute and cluster](./Compute_Cluster.md) |
| Add delivery, services, monitoring, storage and recovery | [Services and operations](./Services_Operations.md) |
| Validate, troubleshoot and record lessons | [Verification and troubleshooting](./Verification.md) |

Work in dependency order, keep a known-good access path, and verify each stage before relying on it. Commands from a tool's official guide must match the installed version and the intended environment. Generic [DevHub references](https://github.com/CollinPoetoehena/DevHub/tree/main/reference) remain the source for broadly reusable commands and concepts.
