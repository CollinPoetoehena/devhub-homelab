# Project Structure

> **Part 4 · Automation** — [Part overview](./README.md) · Previous: [Deployment](./deployment.md) · Next: [Part 5 · Services](../05-services/README.md)

A useful structure makes responsibilities and dependencies understandable without publishing site-specific implementation code. This page describes how to organize homelab integration and ends with the maintained high-level structure summary for this project.

The [main README's repository relationship](../../README.md#how-devhub-and-the-homelab-repositories-fit-together) owns the broader explanation of DevHub, public examples, and private operations; this page applies it to integration layout.

---

## Table of Contents

- [Structure by Responsibility](#structure-by-responsibility)
- [Reuse Packages Rather Than Copying Them](#reuse-packages-rather-than-copying-them)
- [Portfolio Without Publishing Implementation Code](#portfolio-without-publishing-implementation-code)
- [Maintaining the Structure Summary with AI](#maintaining-the-structure-summary-with-ai)
- [General Homelab Project Structure](#general-homelab-project-structure)

---

## Structure by Responsibility

Start with a root README explaining purpose, documentation entry points and operational boundaries. Keep the documentation as one book under `docs/`, organized by the [reader's journey](../repository-guidelines.md#the-book-model-layers-and-storyline)—foundations, architecture, platform services, automation, services and operations—with one file per subject that moves from design to implementation and verification. Implementation code lives beside the documentation, not inside it. Code can follow its tool's ecosystem conventions without turning tool names into competing documentation storylines.

Separate provisioning, host configuration and workload reconciliation because they have different inputs, credentials and lifecycles. Define which owns each property. Keep environment-specific composition close to its consumer, reusable components independently maintained, and secrets/state/generated outputs outside ordinary tracked content.

Prefer a small number of meaningful folders over a directory for every idea. Split when ownership or lifecycle genuinely differs. Document prerequisites, input/output contracts and validation beside the relevant integration. Avoid duplicating inventory across several tools without a declared authority and a deliberate handoff.

---

## Reuse Packages Rather Than Copying Them

[DevHub](https://github.com/CollinPoetoehena/DevHub) remains the shared development/reference hub. Its [package guidelines](https://github.com/CollinPoetoehena/DevHub/blob/main/packages/README.md) explain independently versioned reusable components, distribution and shared publishing conventions.

| Option / tool | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| [Ansible roles](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html) | Reusable host/platform task components | Compose tested responsibilities instead of copying tasks | Version/input compatibility and idempotency; private playbooks/inventory supply site values. **Maintenance:** Version and test roles/collections against supported hosts; check convergence/reruns when inputs or operating systems change. | Repeated configuration; follow [DevHub role conventions](https://github.com/CollinPoetoehena/DevHub/blob/main/packages/Ansible.md) |
| [Terraform modules](https://developer.hashicorp.com/terraform/language/modules) | Resource components with input/output contracts | Reusable provisioning without duplicating resource definitions | Provider/version/state and consumer validation; roots supply private inputs/backend. **Maintenance:** Track provider/module contracts and upgrade impact; validate consumer plans and state migrations before promotion. | Repeated provisioning; follow [DevHub module conventions](https://github.com/CollinPoetoehena/DevHub/blob/main/packages/Terraform.md) |
| [Helm charts](https://helm.sh/docs/topics/charts/) | Reusable application resource packaging | Versioned application templates and deployment inputs | Chart/version compatibility; environment values and delivery remain consumer-owned. **Maintenance:** Maintain chart dependencies, API compatibility and documented values; test rendered resources and supported application upgrades. | Kubernetes packaging; follow [DevHub chart conventions](https://github.com/CollinPoetoehena/DevHub/blob/main/packages/Helm.md) |

Pin compatible versions, review upgrades and test the consumer. A reusable component should not embed a household subnet, endpoint or credential. Existing locally housed components can remain until a separately authorized extraction; this structure guide does not move or publish their code.

---

## Portfolio Without Publishing Implementation Code

Publishing actual code is **not required for this homelab portfolio**. The purpose is to explain how the system is designed, how responsibilities connect and how a generic setup is approached. A sanitized structure, options/trade-offs and validation methodology can communicate that without revealing which subnet, secret or live endpoint the implementation uses.

The [publication boundary](../repository-guidelines.md#publication-boundary) explains why even password-free inventory/configuration can expose home-network internals. Keep implementation details private; share genuinely reusable packages only through their own review and lifecycle. Do not misrepresent a structure diagram as proof of deployment, test results or code availability.

---

## Maintaining the Structure Summary with AI

**Update the final section of this page in place** when documentation or integration responsibilities change. Use the shared [devhub-docs](https://github.com/CollinPoetoehena/DevHub/blob/main/.github/skills/devhub-docs/SKILL.md) for documentation maintenance; it discovers local guidance and structure. When the task is authorized preparation of private-derived material for publication, use the self-contained [homelab-sanitize](../../.github/skills/homelab-sanitize/SKILL.md) instead, not a required chain of skills. See [AI usage](../../README.md#ai-usage) for setup. Do not create another drifting structure overview elsewhere.

Inspect only the authorized, relevant top-level structure locally. Summarize responsibility-level folders rather than pasting `tree` output or enumerating hosts, inventories, applications, environment names, secret filenames or generated artifacts. Review the summary against the guideline before publication.

Mark absent/planned areas explicitly. AI must not create implementation folders, claim deployments or infer live values just to make the diagram look complete. Keep the summary at the bottom of this document; earlier sections explain the general conventions.

---

## General Homelab Project Structure

This is an **AI-maintained, sanitized responsibility view**, not an exact directory listing. The public repository contains documentation/customizations; implementation areas belong privately. Paths describe the main organization, not a deployable copy.

```text
Public homelab book and knowledge base
  README.md                 purpose, three-repository relationship and entry point
  .github/                  public-context instructions and sanitization skill
  docs/
    README.md               the book: layers, storyline and table of contents
    repository-guidelines.md
    01-foundations/         goals, requirements, design principles, cost
    02-architecture/        physical, network (incl. Wi-Fi), compute, storage, platform
    03-platform-services/   DNS, certificates, identity, secrets, scheduling
    04-automation/          IaC, configuration management, deployment,
                            this maintained project-structure overview
    05-services/            workload design and self-hosted service options
    06-operations/          observability, backups, disaster recovery, maintenance

Private homelab integration
  README.md                 actual project context and navigation
  .github/                  private-context instructions; no local skills needed
  docs/                     the same book parts (01-06), only where private
                            content exists; actual decisions, setup, checks
                            and recovery inside each subject file
  ansible/                  host/platform configuration composition;
                            reusable roles and protected variable references
  terraform/                provisioning composition and reusable modules;
                            protected runtime inputs and state
  GitOps integration        planned responsibility for workload reconciliation;
                            directory/repository layout not yet established
```

The GitOps line is a planned area, not an existing `gitops/` folder or verified deployment. Reusable packages may have their own repositories linked through DevHub. Credentials, decrypted values, state/plans and raw backups are intentionally not represented as public paths.

TODO: later add the full setup for the private repo, such as (example from ):
```
/
├── ansible/                    # Configuration management playbooks and roles
│   ├── playbooks/              # Ansible playbooks for setup and configuration
│   │   ├── 01-network-setup.yml
│   │   └── 02-security-hardening.yml
│   ├── roles/                  # Reusable Ansible roles
│   │   ├── common/             # Common system configuration
│   │   ├── firewall/           # UFW firewall management
│   │   ├── network-config/     # Network and hostname configuration
│   │   ├── security-hardening/ # SSH, fail2ban, kernel params, auditd
│   │   ├── containerd/         # Container runtime (Article 5)
│   │   └── kubernetes/         # K8s cluster setup (Article 5)
│   ├── inventory/              # Inventory files (gitignored - use examples)
│   │   └── hosts.ini.example   # Example inventory template
│   └── ansible.cfg             # Ansible configuration
├── scripts/                    # Helper scripts for automation
│   ├── proxmox/                # VM creation, snapshot, and management
│   ├── network/                # Network configuration and testing
│   └── kubernetes/             # Kubernetes helper scripts
├── terraform/                  # Infrastructure as Code (Article 10)
│   ├── proxmox-vms/            # VM provisioning configurations
│   └── modules/                # Reusable Terraform modules
├── Kubernetes/                 # K8s manifests and configs (Article 6+)
│   ├── manifests/              # Kubernetes resource definitions
│   └── helm-charts/            # Custom Helm charts
├── docs/                       # Documentation and guides
│   ├── architecture/           # Network design and system architecture
│   └── setup-guides/           # Step-by-step setup instructions
└── examples/                   # Sample applications
```