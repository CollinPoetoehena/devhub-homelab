# Provisioning and Configuration Management

Automation belongs to the lifecycle it manages, not a top-level folder named after a selected tool.

---

## Table of Contents

- [Design Ownership](#design-ownership)
- [Tool Options](#tool-options)
- [Implementation and Verification](#implementation-and-verification)

---

## Design Ownership

Infrastructure as Code (IaC) declares resources such as VMs, disks and networks. Configuration management brings existing supported machines to a desired state. Images/templates, node-OS APIs and workload reconciliation have separate ownership. Avoid two tools managing the same property.

Manual steps can be reasonable for a small stable system if documented, backed up and tested. Automate when reproducibility, review or reduced maintenance justifies it.

---

## Tool Options

| Capability | Options | Trade-offs |
|---|---|---|
| Provider-based provisioning | Terraform / OpenTofu | Plans and state support review; protect state and check providers/licensing/compatibility |
| Programming-language IaC | Pulumi | Useful abstractions in familiar languages; review complexity and supported providers |
| Host configuration | Ansible | Agentless supported-host configuration; idempotency/check-mode support depends on implementation |
| Continuous host enforcement | Puppet / Chef / Salt | Other enforcement models, with agents/services or additional learning/operations |
| Image preparation | Packer / supported image workflows | Consistent starting images; complements ongoing configuration |
| Small explicit tasks | Scripts | Simple when bounded; error handling and idempotency are your responsibility |

Ansible may provision too, but its check mode is not equivalent to a resource-state plan. An immutable/API-managed node OS does not automatically accept conventional SSH/package-management roles.

---

## Implementation and Verification

Choose ownership and input contracts first, then version compatible tools/components and test local behavior before a reviewed live change. Keep credentials, state/plans and sensitive endpoints protected under the [guidelines](../../Repository_Guidelines.md). Validation or mocked tests do not prove successful real provisioning.

Reuse independently maintained components through [DevHub packages](https://github.com/CollinPoetoehena/DevHub/blob/main/packages/README.md). The public homelab documents concepts; actual roots, inventories and runbooks remain private.
