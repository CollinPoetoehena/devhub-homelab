# DevHub Homelab

> **TODO: Final documentation review.** Once most of the homelab is complete, thoroughly review the entire public documentation again. Use practical lessons to correct, expand and improve the generic guidance, not to publish the actual installation. Revisit goals, HLD/LLD categories, option tables, project structure and guidelines; verify official documentation links and update release-dependent features, support/lifecycle assumptions and recommendations to the current supported versions at review time. Recheck completeness, consistency, Table of Contents, anchors and cross-repository `main` links. Review the final public diff against the [publication boundary](./docs/Repository_Guidelines.md#publication-boundary), keeping actual internals, secrets and operational results private. This is a future review milestone, not a claim that the homelab or documentation is finished now.

**The public entry point, knowledge base, and portfolio for my homelab project.** This repository collects homelab-focused design guidance, trade-offs, setup approaches, and reusable lessons. It is not a description of my live home network or a deployable copy of it.

---

## Table of Contents

- [Why two homelab repositories?](#why-two-homelab-repositories)
- [Why separate from DevHub?](#why-separate-from-devhub)
- [Documentation](#documentation)
- [AI and Copilot Skills](#ai-and-copilot-skills)

---

## Why two homelab repositories?

The primary reason is **security**. Publishing the internals of a home network can help someone identify devices, locate management services, understand access paths, or target household infrastructure. Credentials are not the only sensitive information: a combination of real topology, addressing, inventory, firewall exceptions, software versions, and operational details can be revealing too. The public repository therefore describes generic patterns, not a redacted map of my actual network.

| Repository | Purpose | What belongs there |
|---|---|---|
| [devhub-homelab](https://github.com/CollinPoetoehena/devhub-homelab) (public) | Homelab knowledge base, portfolio, and entry point | Options, selection criteria, generic designs, sanitized setup guidance, reusable lessons, and links to reusable components |
| [devhub-homelab-private](https://github.com/CollinPoetoehena/devhub-homelab-private) (private; access required) | Actual implementation and operational record | Chosen hardware/software, decision rationale, internal topology and inventory, configuration, runbooks, validation results, and recovery details |

> **Tip: Open both repositories in one VS Code workspace.** Clone them into a shared parent folder, such as `homelab/`, and open that folder in VS Code, or add both repositories to a multi-root workspace. This makes it easier to give AI chat context from both repositories, use the appropriate public or private Copilot skills, and coordinate documentation changes without switching windows. Explicitly select the relevant files and skills and read their referenced guidelines; opening both repositories does not guarantee that every instruction or skill is automatically loaded. Keep the repositories as separate Git projects and preserve the publication boundary: access to private context is not permission to copy it into public documentation.

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
- [Project structure](./docs/Project_Structure.md): integration responsibilities, package reuse and an AI-maintained sanitized overview.
- [Goals](./docs/1_Goals/README.md): purpose, constraints and household dependencies.
- [High-level design](./docs/2_HLD/README.md): network, Wi-Fi, host selection, power/capacity, virtualization and host automation.
- [Low-level design](./docs/3_LLD/README.md): container orchestration, services/delivery, monitoring, storage and backups.
- [Copilot skills](.github/skills/README.md): public-focused context and documentation workflows, including an explicit sanitization skill.

Examples are synthetic and must be adapted to the reader's requirements. A documented design or checklist is not evidence of a completed deployment. The private repository references these documents through GitHub `blob/main` URLs; access to it is not required to use the public guidance.

---

## AI and Copilot Skills

Use the focused [public skills](./.github/skills/README.md) for this knowledge base and the [private skills](https://github.com/CollinPoetoehena/devhub-homelab-private/blob/main/.github/skills/README.md) for actual implementation records. They add homelab-specific context to reusable DevHub workflows; they are not separate copies of the general development guidance.

**Skills reference policy; documentation owns it.** The [repository guidelines](./docs/Repository_Guidelines.md) are the central policy source, including the [Documentation Structure & Guideline](./docs/Repository_Guidelines.md#documentation-structure--guideline). This is the authoritative documentation guideline for Copilot and for links from READMEs, instructions and other documents. Keep a skill focused on its trigger, scope, execution steps and validation; link the applicable guideline rather than repeating table columns, structure rules or their rationale. This avoids conflicting copies and lets one policy update reach every workflow that reads it.

When a guideline changes, update its references and check the affected skills rather than rewriting the rule in each skill. AI must read the referenced guidance, preserve public/private boundaries, distinguish planned work from verified results and validate its changes; links alone do not guarantee that a workflow was loaded or followed. Publication and live infrastructure operations still require appropriate authorization.

See [DevHub's AI reference](https://github.com/CollinPoetoehena/DevHub/blob/main/reference/AI.md) for broader AI usage and GitHub Copilot guidance, and the [DevHub skills guide](https://github.com/CollinPoetoehena/DevHub/blob/main/.github/skills/README.md) for shared workflows. A GitHub link is a reference, not a skill installation; make generic skills available explicitly or read their linked guidance when accessible.
