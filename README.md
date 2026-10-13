# DevHub Homelab

**The public entry point, knowledge base, and portfolio for my homelab project.** This repository collects homelab-focused design guidance, trade-offs, setup approaches, and reusable lessons. It is not a description of my live home network or a deployable copy of it.

---

## Table of Contents

- [Why two homelab repositories?](#why-two-homelab-repositories)
- [Why separate from DevHub?](#why-separate-from-devhub)
- [Documentation](#documentation)
- [Documentation Structure](#documentation-structure)

---

## Why two homelab repositories?

The primary reason is **security**. Publishing the internals of a home network can help someone identify devices, locate management services, understand access paths, or target household infrastructure. Credentials are not the only sensitive information: a combination of real topology, addressing, inventory, firewall exceptions, software versions, and operational details can be revealing too. The public repository therefore describes generic patterns, not a redacted map of my actual network.

| Repository | Purpose | What belongs there |
|---|---|---|
| [devhub-homelab](https://github.com/CollinPoetoehena/devhub-homelab) (public) | Homelab knowledge base, portfolio, and entry point | Options, selection criteria, generic designs, sanitized setup guidance, reusable lessons, and links to reusable components |
| [devhub-homelab-private](https://github.com/CollinPoetoehena/devhub-homelab-private) (private; access required) | Actual implementation and operational record | Chosen hardware/software, decision rationale, internal topology and inventory, configuration, runbooks, validation results, and recovery details |

One repository forced an awkward compromise: either publish too much, or replace useful operational details with examples and leave the real setup undocumented in my head. The split lets the private record be genuinely useful while the public reference remains safe and reusable.

Two repositories do add a little coordination: links must stay correct and a change may need documentation in both places. But **they make the content easier to maintain**. Public guidance owns the explanations and option comparisons once. Private designs link to that guidance and concentrate on what I chose, why, which relevant alternatives I rejected, and how the actual pieces fit together. There is no need to repeat a full Kubernetes distribution comparison inside my cluster decision record.

The public side can stay curated, concise, and focused on material worth sharing. The private side can contain detailed working notes, incomplete investigations, exact configurations, and recovery procedures without making every note a portfolio item. This removes the pressure to sanitize every operational record or polish everything for an external audience.

**Private does not mean safe to commit secrets.** Both repositories still need deliberate access controls, protected secret storage, patching, and backups. Separation reduces disclosure risk; it is not a substitute for network security. Public changes are explicit publications, never an automatic mirror or a copy of private Git history.

See [Repository guidelines](./docs/Repository_Guidelines.md) for content ownership, publication review, and secret handling.

---

## Why separate from DevHub?

The [DevHub separate-projects explanation](https://github.com/CollinPoetoehena/DevHub#separate-projects) describes the distinction: a homelab is related to development, but is still a distinct project with its own purpose, design, lifecycle, issues, automation, and maintenance.

This public repository is not a second general reference library. It applies knowledge to **homelabs**: buying small always-on hosts, separating household infrastructure from experiments, choosing a cluster platform, providing wireless coverage, and planning recovery at home scale. DevHub continues to own concepts and tooling that apply beyond the homelab, such as [networking fundamentals](https://github.com/CollinPoetoehena/DevHub/tree/main/reference/network), [AI workflows](https://github.com/CollinPoetoehena/DevHub/blob/main/reference/AI.md), and [reusable packages](https://github.com/CollinPoetoehena/DevHub/blob/main/packages/README.md). Link to those resources rather than duplicating them here.

---

## Documentation

- [Repository guidelines](./docs/Repository_Guidelines.md): public/private boundary, writing rules, sanitization, and security.
- [Documentation storyline](./docs/README.md): the entry point and reading order.
- [Goals](./docs/1_Goals/README.md): purpose, constraints and household dependencies.
- [High-level design](./docs/2_HLD/README.md): network, Wi-Fi, host selection, power/capacity, virtualization and host automation.
- [Low-level design](./docs/3_LLD/README.md): container orchestration, services/delivery, monitoring, storage and backups.
- [Copilot skills](.github/skills/README.md): public-focused context and documentation workflows, including an explicit sanitization skill.

Examples are synthetic and must be adapted to the reader's requirements. A documented design or checklist is not evidence of a completed deployment. The private repository references these documents through GitHub `blob/main` URLs; access to it is not required to use the public guidance.

---

## Documentation Structure

The documentation tells one story: **goals → high-level infrastructure → low-level platform and services**. Start by deciding what the lab should achieve, design its network and compute foundation, then decide what runs there and how it will be observed and recovered.

Categories keep connected decisions together. Network architecture needs networking gear and management software; host selection needs hardware, electricity, virtualization and OS choices. Splitting all hardware into one catalog and all software into another makes readers jump between unrelated topics instead of following a decision through.

Likewise, there is **no separate public setup tree**. Each category starts with design and options, then ends with broad implementation/verification considerations. Think before installing. Setup is part of the topic's complete documentation, not a parallel book duplicating its structure. Exact commands and vendor installation details belong in official guides or private runbooks, not an imitation of my own deployment.

High-level design (HLD) describes the foundation and its boundaries: network and physical hosts, including any hypervisor cluster. Low-level design (LLD) describes the platforms and services using it: a Kubernetes/container-orchestration cluster, workload delivery, monitoring and data protection. These categories can feed requirements back to the HLD; the order is a reasoning path, not an inflexible waterfall.

Infrastructure as Code and configuration management are concepts within **host lifecycle**, with Terraform/OpenTofu/Pulumi and Ansible/Puppet/etc. as options. GitOps/CD and CI belong with **services and delivery**, with Argo CD/Flux and CI systems as options. Tool names are not top-level documentation folders.

The private repository mirrors these categories, keeping actual design decisions before detailed setup and results in each category. It does not mirror the public prose. Tables of contents and `---` dividers between second-level sections keep both sides navigable; cross-repository references use GitHub URLs on `main`.
