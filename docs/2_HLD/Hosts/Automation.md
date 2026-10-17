# Provisioning and Configuration Management

Automation belongs to the lifecycle it manages, not a top-level folder named after a selected tool.

This category evaluates automation for lab hosts and recovery. Follow the [repository relationship](../../../README.md#how-devhub-and-the-homelab-repositories-fit-together): shared workflows and reusable components stay in DevHub, public examples explain lab integration, and actual composition, inputs, and results stay private.

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

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| [Terraform](https://developer.hashicorp.com/terraform/docs) | Provider-schema IaC with a dependency graph, plans and state | Reviews declared versus observed resources; repeatable VM/resource composition | Protect state; inspect replacements/imports/migrations; verify providers, versions and licensing. **Maintenance:** review provider/module upgrades and drift, test state recovery and inspect each plan rather than treating automation as unattended safety | Many provider-managed resources with a clear plan/apply workflow |
| [OpenTofu](https://opentofu.org/docs/) | Provider-based declarative IaC with state and planning | Similar resource-review workflow and an alternative tool lifecycle | Verify provider/backend/version compatibility instead of assuming all Terraform changes are interchangeable; state protection remains. **Maintenance:** Review provider/module/backend upgrades and drift; test state recovery and migration compatibility before changing tool versions. | Provider-based provisioning with suitable ecosystem/governance fit |
| [Pulumi](https://www.pulumi.com/docs/) | Resource declarations in general-purpose programming languages | Familiar abstractions, reuse and language-based testing | Abstractions can hide infrastructure changes; review provider support, state/backend and complexity. **Maintenance:** Maintain language dependencies and providers as well as infrastructure state; review preview output and test abstractions when APIs change. | Language-oriented teams with a clear reviewable infrastructure model |
| [Ansible](https://docs.ansible.com/ansible/latest/) | Usually agentless, explicit configuration runs on supported hosts | Roles/modules, handlers and repeatable convergence without a resident agent | Idempotency/check/diff depend on task quality; check mode is not a resource-state plan; API-managed OSes may not support SSH roles. **Maintenance:** keep collections/roles compatible with host releases, test first convergence and reruns, and review drift/credential access | Small fleets and explicit or scheduled reviewed host configuration |
| [Puppet](https://www.puppet.com/docs/) | Desired-state configuration, commonly with agent-based enforcement | Continuous convergence and drift reduction | Agent/controller trust, lifecycle and learning overhead. **Maintenance:** Update agents, server and modules together; monitor failed convergence, certificate expiry and policy regressions. | Fleets needing ongoing enforcement |
| [Chef](https://docs.chef.io/) | Code-driven host configuration and convergence workflows | Reusable policy and continuous configuration management | Client/server or chosen execution-model lifecycle and testing effort. **Maintenance:** Maintain clients, cookbooks and the chosen server/execution model; test convergence and credential renewal before rollout. | Teams already using its policy/tool ecosystem |
| [Salt](https://docs.saltproject.io/) | Configuration and remote-execution platform with several execution models | Flexible orchestration/enforcement across hosts | Choose and secure the controller/agent or agentless model; another operational lifecycle. **Maintenance:** Keep execution components and state definitions compatible; secure keys and monitor failed jobs or disconnected minions. | Fleet execution and convergence needs justify its scope |
| [Packer](https://developer.hashicorp.com/packer/docs) | Automated machine-image build workflows | Consistent, reproducible baselines reduce repeated installation | Images need refresh, testing and compatible builders; not ongoing configuration management. **Maintenance:** rebuild after base-image/security changes and verify boot/integration before promoting templates | Repeated guest/template creation |
| [cloud-init](https://cloudinit.readthedocs.io/en/latest/) | Supported first-boot guest initialization | Establishes identity, networking and bootstrap configuration | Image/platform support varies; not a replacement for all ongoing maintenance. **Maintenance:** Refresh base images and bootstrap data; test a clean first boot after image/platform changes rather than assuming reruns repair existing guests. | Bootstrapping supported cloud/VM images |
| Supported node-OS API | The purpose-built OS's configuration/bootstrap/upgrade interface | Matches immutable/API-managed lifecycle rather than mutable-host assumptions | Explicit credential, compatibility and apply ownership; use the platform's supported tooling. **Maintenance:** Track supported image/API/provider versions, rotate access credentials and test configuration apply, upgrade and rebuild paths. | Dedicated node systems such as [Talos](../../3_LLD/Cluster/README.md#kubernetes-node-lifecycle) |
| Bounded scripts | Explicit glue or diagnostic programs | Simple for small tasks with clear inputs/outputs | Error handling, exit statuses, safe reruns and idempotency are yours. **Maintenance:** Maintain dependencies, error handling and tests; verify safe reruns when platform commands or output formats change. | Tasks not better served by a supported module/provider |

Ansible may provision too, but its check mode is not equivalent to a resource-state plan. An immutable/API-managed node OS does not automatically accept conventional SSH/package-management roles.

### Provisioning Choices

For the provisioning options in the table, inspect the actual plan and replacement behavior. Version-controlled configuration is not enough if state is lost. Compare provider support, backend behavior and review experience for the target platform; do not switch tools solely for more syntax choices.

### Configuration and Images

For configuration tools, prefer reusable roles and supported modules over shell commands. A successful check run is not proof every change was simulated. Choose explicit/scheduled convergence versus continuous enforcement deliberately, and keep images refreshed so cloning does not reproduce an obsolete insecure baseline. Reusable roles/modules should follow [DevHub package conventions](https://github.com/CollinPoetoehena/DevHub/blob/main/packages/README.md); site-specific roots consume them, rather than embedding another divergent copy.

---

## Implementation and Verification

Choose ownership and input contracts first, then version compatible tools/components and test local behavior before a reviewed live change. Keep credentials, state/plans and sensitive endpoints protected under the [guidelines](../../Repository_Guidelines.md). Validation or mocked tests do not prove successful real provisioning.

Reuse independently maintained components through [DevHub packages](https://github.com/CollinPoetoehena/DevHub/blob/main/packages/README.md). The public homelab documents concepts; actual roots, inventories and runbooks remain private.

Start with local formatting/schema checks, then focused unit or mocked tests, then an isolated integration target where appropriate. For configuration, test both first convergence and a rerun; for provisioning, inspect replacement/destruction behavior and state protection. For backups, automate job monitoring but still test restores.

Keep plan/apply separation and environment-specific approvals. Restrict CI runner access, protect secrets and backend keys, and avoid uploading private plans or logs publicly. Maintain a documented bootstrap path for the tools themselves: a broken automation controller should not prevent recovering the hosts that run it. The [project-structure guide](../../Project_Structure.md) explains where consumers, shared components and documentation belong.
