# Part 4 · Automation

> **Layer: Implementation — how is it realized?** · [Book overview](../README.md) · Previous part: [Part 3 · Platform Services](../03-platform-services/README.md) · Next part: [Part 5 · Services](../05-services/README.md)

The architecture and platform-services parts describe *what* is built and *why*. This part answers the storyline's fifth question: **how are those capabilities implemented?** In this homelab the answer is code: infrastructure as code (for example Terraform) creates resources, configuration management (for example Ansible) brings machines to a desired state, and a deployment workflow (CI and GitOps) delivers workloads. Together they turn the design into something reproducible, reviewable and recoverable.

Automation is a cross-cutting design principle; its concrete tool choices, ownership boundaries and workflows are implementation decisions. Choose tools in the context of the platforms and services they manage, rather than creating documentation categories for individual products. Follow the [repository relationship](../../README.md#how-devhub-and-the-homelab-repositories-fit-together): shared workflows and reusable components stay in DevHub, this public book explains lab integration, and actual composition, inputs and results stay private.

---

## Table of Contents

- [Chapters in This Part](#chapters-in-this-part)
- [Design Ownership](#design-ownership)
  - [Keep Ownership Clear](#keep-ownership-clear)
- [Code Describes How, Documentation Describes Why](#code-describes-how-documentation-describes-why)
- [Implementation and Verification](#implementation-and-verification)

---

## Chapters in This Part

| # | Chapter | Question it answers | Typical tools discussed |
|---|---|---|---|
| 1 | [Infrastructure as Code](./infrastructure-as-code.md) | How are VMs, disks, networks and other resources created and changed reviewably? | Terraform, OpenTofu, Pulumi |
| 2 | [Configuration Management](./configuration-management.md) | How are hosts, guests and images brought to and kept in a desired state? | Ansible, Puppet, Chef, Salt, Packer, cloud-init, node-OS APIs |
| 3 | [Deployment](./deployment.md) | How does code become an artifact and reach the platform? | GitHub Actions and other CI, Argo CD, Flux, Helm, Kustomize |
| 4 | [Project Structure](./project-structure.md) | How are documentation, provisioning, configuration and delivery organized as a project? | Repository layout, reusable packages, the maintained structure summary |

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

## Code Describes How, Documentation Describes Why

When a responsibility is automated, the code is the authoritative description of *how* it is done. Documentation should not repeat every task: a page that lists "install package X, copy file Y, open port Z" duplicates the playbook, drifts from it and doubles the maintenance. Instead, documentation explains:

- **Why** the component exists and which [requirement](../01-foundations/requirements.md) it serves.
- **What** it is responsible for, which tool owns which properties, and how it relates to other components.
- **Decisions and trade-offs** that are not obvious from the code, including deliberate manual exceptions.
- **How to verify and recover** it, linking to the code rather than copying it.

For example, a hardened jump host can be documented as "the only SSH entry point into the management network, configured by a dedicated role that enforces key-only authentication, a minimal package set, a host firewall and audit logging"—with a link to the role—rather than as a list of the role's tasks. This keeps the documentation stable when implementation details change, and keeps the code the single source of truth for them. The same rule applies to Terraform resources, Kubernetes manifests and pipeline definitions.

---

## Implementation and Verification

Choose ownership and input contracts first, then version compatible tools/components and test local behavior before a reviewed live change. Keep credentials, state/plans and sensitive endpoints protected under the [guidelines](../repository-guidelines.md). Validation or mocked tests do not prove successful real provisioning.

Reuse independently maintained components through [DevHub packages](https://github.com/CollinPoetoehena/DevHub/blob/main/packages/README.md). The public homelab documents concepts; actual roots, inventories and runbooks remain private.

Validate in layers: start with local formatting/schema checks, then focused unit or mocked tests, then an isolated integration target where appropriate. Each chapter in this part describes the checks specific to its tool category—plans and state for [infrastructure as code](./infrastructure-as-code.md#implementation-and-verification), convergence and reruns for [configuration management](./configuration-management.md#implementation-and-verification), and the end-to-end pipeline for [deployment](./deployment.md#implementation-and-verification). For backups, automate job monitoring but still [test restores](../06-operations/backups.md#implementation-and-verification).

Maintain a documented bootstrap path for the tools themselves: a broken automation controller should not prevent recovering the hosts that run it. The [project-structure guide](./project-structure.md) explains where consumers, shared components and documentation belong.
