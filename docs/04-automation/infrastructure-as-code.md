# Infrastructure as Code

> **Part 4 · Automation** — [Part overview](./README.md) · Next: [Configuration Management](./configuration-management.md)

Infrastructure as Code (IaC) declares resources—VMs, disks, network interfaces, DNS records, cloud objects—in versioned files, compares that declaration with what exists, and applies the difference after review. In a homelab, IaC is what makes "rebuild the lab VMs from scratch" a reviewed command instead of an afternoon of clicking through a hypervisor interface.

This chapter covers the provisioning tools and how to use them safely. Bringing the created machines to a desired configuration is the next chapter, [Configuration Management](./configuration-management.md); which tool owns which layer is defined in the [part overview](./README.md#keep-ownership-clear).

---

## Table of Contents

- [What IaC Owns](#what-iac-owns)
- [Tool Options](#tool-options)
- [Provisioning Choices](#provisioning-choices)
- [State, Plans and Safety](#state-plans-and-safety)
- [Implementation and Verification](#implementation-and-verification)

---

## What IaC Owns

Provisioning tools own the **existence and shape of resources**: that a VM exists with a given CPU, memory, disks and network interfaces; that a cloned template is the starting point; that a DNS record or cloud bucket exists. They should not also own what happens *inside* the guest operating system—users, packages, services—which belongs to [configuration management](./configuration-management.md), or which applications run on Kubernetes, which belongs to [deployment](./deployment.md).

Hand the result to the next stage through an explicit contract, such as an inventory or output values, instead of chaining configuration through fragile provisioner hooks. Reusable provisioning logic belongs in modules with clear inputs and outputs; environment-specific roots compose them with private values.

---

## Tool Options

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| [Terraform](https://developer.hashicorp.com/terraform/docs) | Provider-schema IaC with a dependency graph, plans and state | Reviews declared versus observed resources; repeatable VM/resource composition | Protect state; inspect replacements/imports/migrations; verify providers, versions and licensing. **Maintenance:** review provider/module upgrades and drift, test state recovery and inspect each plan rather than treating automation as unattended safety | Many provider-managed resources with a clear plan/apply workflow |
| [OpenTofu](https://opentofu.org/docs/) | Provider-based declarative IaC with state and planning | Similar resource-review workflow and an alternative tool lifecycle | Verify provider/backend/version compatibility instead of assuming all Terraform changes are interchangeable; state protection remains. **Maintenance:** Review provider/module/backend upgrades and drift; test state recovery and migration compatibility before changing tool versions. | Provider-based provisioning with suitable ecosystem/governance fit |
| [Pulumi](https://www.pulumi.com/docs/) | Resource declarations in general-purpose programming languages | Familiar abstractions, reuse and language-based testing | Abstractions can hide infrastructure changes; review provider support, state/backend and complexity. **Maintenance:** Maintain language dependencies and providers as well as infrastructure state; review preview output and test abstractions when APIs change. | Language-oriented teams with a clear reviewable infrastructure model |

---

## Provisioning Choices

For the provisioning options in the table, inspect the actual plan and replacement behavior. Version-controlled configuration is not enough if state is lost. Compare provider support, backend behavior and review experience for the target platform; do not switch tools solely for more syntax choices.

For a homelab hypervisor, check which resources the provider actually supports and how it behaves when something is changed outside the tool—for example, a disk resized in the web interface. A provider that cannot express a needed setting is a reason to document a deliberate manual step, not to wrap every gap in scripts.

---

## State, Plans and Safety

IaC tools that track state turn that state into a sensitive artifact. It maps declared resources to real ones, may contain secrets and identifying values, and losing it breaks the tool's ability to manage what exists. Treat it as protected data: store it securely with encryption, access control, locking and backups as appropriate, and keep it out of ordinary Git. A `sensitive` marking redacts some display output; it does not encrypt state. The [repository guidelines](../repository-guidelines.md#secrets-and-sensitive-artifacts) define the rules.

Read every plan. Replacement of a VM, a disk or a network interface can be destructive even when the change looks small, and imports or state moves need the same care as code changes. Keep plan and apply separate, with an approval step for live environments.

---

## Implementation and Verification

Choose ownership and input contracts first, then pin compatible tool and provider versions and reusable modules. Validate locally with formatting and validation commands and, where supported, focused tests with mocked providers; these prove the code is consistent, not that real provisioning will succeed.

Keep plan/apply separation and environment-specific approvals. Restrict CI runner access, protect secrets and backend keys, and avoid uploading private plans or logs publicly. Maintain a documented bootstrap path for the tools themselves: a broken automation controller should not prevent recovering the hosts that run it. The [project-structure guide](./project-structure.md) explains where consumers, shared components and documentation belong.


Before relying on IaC for recovery, rehearse it: destroy and recreate a disposable resource, confirm the plan shows exactly the expected changes, and check that the resulting resource is accepted by the configuration-management stage. Inspect replacement and destruction behavior, and confirm that state can be restored from its backup. Actual roots, variables, state locations and results belong in the private repository.
