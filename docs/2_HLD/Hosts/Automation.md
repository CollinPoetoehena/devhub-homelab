# Provisioning and Configuration Management

Automation belongs to the lifecycle it manages, not a top-level folder named after a selected tool.

---

## Table of Contents

- [Design Ownership](#design-ownership)
  - [Keep Ownership Clear](#keep-ownership-clear)
- [Tool Options](#tool-options)
  - [Provisioning Choices](#provisioning-choices)
  - [Configuration and Images](#configuration-and-images)
- [Implementation and Verification](#implementation-and-verification)

---

## Design Ownership

Infrastructure as Code (IaC) declares resources such as VMs, disks and networks. Configuration management brings existing supported machines to a desired state. Images/templates, node-OS APIs and workload reconciliation have separate ownership. Avoid two tools managing the same property.

**Ideally automate as much of the repeatable lifecycle as possible:** resource creation, supported host configuration, workload delivery, validation and backup scheduling. A reviewed declaration is easier to reproduce than remembering clicks, and replacing a failed host should not require reconstructing its setup from memory.

Prioritize frequent, error-prone and recovery-critical actions. Automation does not mean unattended destructive changes: keep approval gates, bounded permissions and a tested recovery path. Physical installation, unsupported appliance settings and one-time bootstrap steps may remain manual when safe automation would add more fragility than it removes; document those exceptions explicitly, back them up and revisit them when repeated.

### Keep Ownership Clear

| Layer | Typical owner | Boundary |
|---|---|---|
| Starting OS/image | Supported image workflow / Packer / cloud-init | Prepare a baseline; do not duplicate every ongoing configuration task |
| VM resources | Terraform / OpenTofu / Pulumi | CPU, disks, interfaces and provider-managed objects |
| Conventional host/guest configuration | Ansible / Puppet / Chef / Salt | Users, packages, services and supported platform settings |
| API-managed node OS | Its supported API/provider | Node configuration, bootstrap and upgrade, not arbitrary SSH package tasks |
| Runtime applications | GitOps or another deployment workflow | Workloads and application configuration, not physical host installation |

Agree input/output contracts between layers. Avoid mixing resource creation and unrelated configuration with fragile `local-exec` chains when a separate explicit stage is clearer. If two tools can edit one property, choose one source of truth and document the handoff.

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

### Provisioning Choices

Terraform/OpenTofu use provider schemas, dependency graphs and state to compare declared resources with observed resources. They are useful for reviewing changes to many similar VMs, but imported resources, provider upgrades and state migrations require care. Inspect the actual plan; a version-controlled configuration is not enough if state is lost or a plan proposes replacement.

Pulumi expresses resource declarations through general-purpose languages, which can fit a team already using those tools. Reuse and testing can be convenient, but ordinary program abstractions can also hide infrastructure changes. Compare provider support, state/backend behavior and the review experience for the target platform. Do not switch tools solely because one offers more syntax choices.

### Configuration and Images

Ansible usually connects to supported hosts without a resident agent, making explicit playbook runs convenient for small labs. Prefer role-based tasks and supported modules over shell commands where possible. Idempotency, handlers, check mode and diff output depend on how tasks are written; a successful check run is not proof every change was simulated.

Puppet and Chef often provide agent-based desired-state enforcement; Salt supports several execution models. Continuous convergence can reduce drift across larger fleets but adds controller/agent lifecycle and trust requirements. A small lab can instead use scheduled, reviewed Ansible runs plus explicit drift checks.

Packer or equivalent image workflows reduce repeated install work. Cloud-init can set supported first-boot identity/networking and bootstrap configuration; it is not a replacement for every ongoing maintenance task. Keep images reproducible and refreshed so cloning does not reproduce an obsolete insecure baseline.

Scripts remain useful for bounded glue and diagnostics. Give them explicit inputs, exit statuses and safe rerun behavior rather than hiding failures. Reusable roles/modules should follow [DevHub package conventions](https://github.com/CollinPoetoehena/DevHub/blob/main/packages/README.md); site-specific roots consume them, rather than embedding another divergent copy.

---

## Implementation and Verification

Choose ownership and input contracts first, then version compatible tools/components and test local behavior before a reviewed live change. Keep credentials, state/plans and sensitive endpoints protected under the [guidelines](../../Repository_Guidelines.md). Validation or mocked tests do not prove successful real provisioning.

Reuse independently maintained components through [DevHub packages](https://github.com/CollinPoetoehena/DevHub/blob/main/packages/README.md). The public homelab documents concepts; actual roots, inventories and runbooks remain private.

Start with local formatting/schema checks, then focused unit or mocked tests, then an isolated integration target where appropriate. For configuration, test both first convergence and a rerun; for provisioning, inspect replacement/destruction behavior and state protection. For backups, automate job monitoring but still test restores.

Keep plan/apply separation and environment-specific approvals. Restrict CI runner access, protect secrets and backend keys, and avoid uploading private plans or logs publicly. Maintain a documented bootstrap path for the tools themselves: a broken automation controller should not prevent recovering the hosts that run it. The [project-structure guide](../../Project_Structure.md) explains where consumers, shared components and documentation belong.
