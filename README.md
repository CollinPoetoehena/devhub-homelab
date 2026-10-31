# DevHub Homelab

> **TODO: Final documentation review.** Once most of the homelab is complete, thoroughly review the entire public documentation again. Use practical lessons to correct, expand and improve the generic guidance, not to publish the actual installation. Revisit the book structure (layers, parts and chapters), goals, requirements, option tables, project structure and guidelines; verify official documentation links and update release-dependent features, support/lifecycle assumptions and recommendations to the current supported versions at review time. Recheck completeness, consistency, Table of Contents, anchors and cross-repository `main` links. Review the final public diff against the [publication boundary](./GUIDELINES.md#publication-boundary), keeping actual internals, secrets and operational results private. This is a future review milestone, not a claim that the homelab or documentation is finished now.

**A standalone guide to designing and operating a homelab, written as a book you can read from start to finish.** This repository is part of my personal projects and follows shared engineering standards and best practices, reusable GitHub Actions guidance, and AI customizations maintained in [DevHub](https://github.com/CollinPoetoehena/DevHub). DevHub is not required to read this guide or understand its homelab concepts: the chapters explain their context, options and trade-offs here, with authoritative sources for further detail.

The book describes why I build a homelab, which requirements and principles guide it, which architecture and capabilities solve them, how they can be implemented, and how they can be operated and recovered. This is a documentation book, not an implementation-code repository or a deployment mirror; actual code and configuration stay private or in separately reviewed reusable-package repositories.

This guide is not intended to reproduce networking, security, operating-system or product theory, or to become an exhaustive technical knowledge base. Each chapter gives only the context needed to understand its homelab decisions, explains the local trade-offs, and links to official standards or product documentation for the authoritative detail. See [Goals and Scope](./01-foundations/goals-and-scope.md#define-what-is-out-of-scope) and the [documentation scope guidelines](./GUIDELINES.md#documentation-structure--guideline).

**Start reading:** [Part 1 · Foundations](./01-foundations/README.md). The complete book outline is below; [Architecture](./ARCHITECTURE.md) explains how this repository is organized, and [Guidelines](./GUIDELINES.md) define shared documentation and publication rules.

---

## Table of Contents

- [Why two homelab repositories?](#why-two-homelab-repositories)
- [Reading the Public Repository as a Book](#reading-the-public-repository-as-a-book)
  - [The Documentation Layers](#the-documentation-layers)
  - [The Storyline](#the-storyline)
  - [Table of Contents of the Book](#table-of-contents-of-the-book)
  - [How to Read a Chapter](#how-to-read-a-chapter)
  - [Why the Book Is Organized This Way](#why-the-book-is-organized-this-way)
  - [Guidelines](#guidelines)
- [Why separate from DevHub?](#why-separate-from-devhub)
- [How DevHub and the Homelab Repositories Fit Together](#how-devhub-and-the-homelab-repositories-fit-together)
- [Working Across DevHub and the Homelab Repositories](#working-across-devhub-and-the-homelab-repositories)
- [Task Tracking](#task-tracking)
- [Documentation](#documentation)
- [AI Customizations](#ai-customizations)
  - [AI usage](#ai-usage)

---

## Why two homelab repositories?

The primary reason is **security**. Publishing the internals of a home network can help someone identify devices, locate management services, understand access paths, or target household infrastructure. Credentials are not the only sensitive information: a combination of real topology, addressing, inventory, firewall exceptions, software versions, and operational details can be revealing too. The public repository therefore describes generic patterns, not a redacted map of my actual network.

| Repository | Purpose | What belongs there |
|---|---|---|
| [devhub-homelab](https://github.com/CollinPoetoehena/devhub-homelab) (public) | **The homelab guide:** a standalone, readable account of designing and operating a homelab; also the portfolio and entry point | Goals, requirements and principles; options, selection criteria and trade-offs; generic design examples; recommendations and hints about what I would choose; sanitized setup guidance, reusable lessons and links to reusable components. **Never** the exact architecture or low-level details that could help an attacker—they are not needed to tell the story |
| [devhub-homelab-private](https://github.com/CollinPoetoehena/devhub-homelab-private) (private; access required) | **The actual implementation** behind the book, and its operational record | The real architecture, chosen hardware/software, decision rationale, internal topology, addressing and inventory, Ansible/Terraform code and configuration, runbooks, validation results and recovery details—organized in the same book structure |

For a convenient local setup with all three projects, see [Working Across DevHub and the Homelab Repositories](#working-across-devhub-and-the-homelab-repositories).

One repository forced an awkward compromise: either publish too much, or replace useful operational details with examples and leave the real setup undocumented in my head. The split lets the private record be genuinely useful while the public reference remains safe and reusable.

Two repositories do add a little coordination: links must stay correct and a change may need documentation in both places. But **they make the content easier to maintain**. Public guidance owns the explanations and option comparisons once. Private designs link to that guidance and concentrate on what I chose, why, which relevant alternatives I rejected, and how the actual pieces fit together. There is no need to repeat a full Kubernetes distribution comparison inside my cluster decision record.

The public side can stay curated and focused on material worth sharing, and can be shaped as a readable story from start to finish. It may give hints about what I recommend or would choose, but it never exposes my exact architecture or low-level details such as addressing, inventory, firewall exceptions or management paths: an attacker could use them, and the story does not need them. The private side can contain detailed working notes, incomplete investigations, exact configurations, and recovery procedures without making every note a portfolio item. This removes the pressure to sanitize every operational record or polish everything for an external audience.

**Private does not mean safe to commit secrets.** Both repositories still need deliberate access controls, protected secret storage, patching, and backups. Separation reduces disclosure risk; it is not a substitute for network security. Public changes are explicit publications, never an automatic mirror or a copy of private Git history.

See [Repository guidelines](./GUIDELINES.md) for content ownership, publication review, and secret handling.

---

## Reading the Public Repository as a Book

The public documentation is deliberately designed as a **book with a storyline**, not as a collection of reference pages. It follows four documentation layers and seven questions, from goals through architecture and implementation to operation and recovery.

### The Documentation Layers

The book has four layers, read from top to bottom:

| Layer | Question | Parts |
|---|---|---|
| Foundations | Why are we doing it? | [Part 1 · Foundations](./01-foundations/README.md) |
| Architecture | What are we building? | [Part 2 · Architecture](./02-architecture/README.md), [Part 3 · Platform Services](./03-platform-services/README.md) |
| Implementation | How is it realized? | [Part 4 · Automation](./04-automation/README.md), [Part 5 · Services](./05-services/README.md) |
| Operations | How do we run it? | [Part 6 · Operations](./06-operations/README.md) |

### The Storyline

The parts follow a reasoning path of seven questions:

| # | Question | Where it is answered |
|---|---|---|
| 1 | Why am I building this? | [Goals and Scope](./01-foundations/goals-and-scope.md) |
| 2 | What principles and requirements guide it? | [Requirements](./01-foundations/requirements.md), [Design Principles](./01-foundations/design-principles.md), [Cost](./01-foundations/cost.md) |
| 3 | What architecture solves those requirements? | [Part 2 · Architecture](./02-architecture/README.md) |
| 4 | What capabilities make up that architecture? | [Part 3 · Platform Services](./03-platform-services/README.md) |
| 5 | How are those capabilities implemented? | [Part 4 · Automation](./04-automation/README.md) |
| 6 | What services run on it? | [Part 5 · Services](./05-services/README.md) |
| 7 | How is it operated, observed and recovered? | [Part 6 · Operations](./06-operations/README.md) |

This order is a reasoning path, not a waterfall: a workload's storage needs may send the design back to host capacity, and operational lessons may change requirements.

### Table of Contents of the Book

Each part README introduces the part, and each subject chapter links to the previous and next chapter.

**Part 1 · Foundations** — [Introduction](./01-foundations/README.md)

1. [Goals and Scope](./01-foundations/goals-and-scope.md)
2. [Requirements](./01-foundations/requirements.md)
3. [Design Principles](./01-foundations/design-principles.md)
4. [Cost, Scope and Buying Decisions](./01-foundations/cost.md)

**Part 2 · Architecture** — [Introduction and architecture overview](./02-architecture/README.md)

1. [Physical Environment: Placement and Accessories](./02-architecture/physical.md)
2. [Network](./02-architecture/network.md) (including Wi-Fi)
3. [Compute](./02-architecture/compute.md)
4. [Storage](./02-architecture/storage.md)
5. [Platform: Container Orchestration and Cluster Lifecycle](./02-architecture/platform.md)

**Part 3 · Platform Services** — [Introduction](./03-platform-services/README.md)

1. [DNS and Name Resolution](./03-platform-services/dns.md)
2. [Certificates and TLS](./03-platform-services/certificates.md)
3. [Identity and Access](./03-platform-services/identity.md)
4. [Secrets Management](./03-platform-services/secrets.md)
5. [Scheduling and Power](./03-platform-services/scheduling.md)

**Part 4 · Automation** — [Introduction and ownership model](./04-automation/README.md)

1. [Infrastructure as Code](./04-automation/infrastructure-as-code.md)
2. [Configuration Management](./04-automation/configuration-management.md)
3. [Deployment: Continuous Integration and Delivery](./04-automation/deployment.md)
4. [Project Structure](./04-automation/project-structure.md)

**Part 5 · Services** — [Introduction](./05-services/README.md)

1. [Workloads: Designing and Running Services](./05-services/workloads.md)
2. [Self-Hosted Services](./05-services/self-hosted-services.md)

**Part 6 · Operations** — [Introduction](./06-operations/README.md)

1. [Observability and Monitoring](./06-operations/observability.md)
2. [Backups](./06-operations/backups.md)
3. [Disaster Recovery](./06-operations/disaster-recovery.md)
4. [Maintenance](./06-operations/maintenance.md)

Two themes recur throughout the book. [Cost and scope](./01-foundations/cost.md) are design constraints: start simple and add lifecycle only when a real need justifies it. [Learning enterprise patterns without enterprise availability](./01-foundations/design-principles.md#learn-enterprise-patterns-without-enterprise-availability) makes accepted failure domains and recovery time explicit rather than implying production-grade resilience.

### How to Read a Chapter

Chapters move from the problem and requirements through options and trade-offs to implementation considerations and verification. This public guide is a standalone homelab decision guide, not a vendor installation manual or copy of the private deployment sequence. It provides enough context for homelab-specific choices and links to official standards and product documentation for deeper detail.

### Why the Book Is Organized This Way

Each subject stays in one chapter, moving from high-level design to detailed choices, implementation and verification. Earlier HLD/LLD folders separated the same subjects across documents and obscured the reading path; the current structure follows the reader's journey instead. [Repository guidelines](./GUIDELINES.md#documentation-structure--guideline) maintain the rationale and rules.

### Guidelines

The [repository guidelines](./GUIDELINES.md) are the shared source for content ownership, documentation structure, publication boundaries, sanitization and sensitive-data handling. The private repository uses the same book structure where corresponding private content exists, but records actual choices and operations rather than duplicating public explanations.

---

## Why separate from DevHub?

DevHub is my central hub for personal projects and shared engineering practices. This homelab has its own goals, design, lifecycle and privacy boundary, so it has separate public and private repositories. The public guide stands on its own; it does not depend on DevHub as a source of general theory.

---

## How DevHub and the Homelab Repositories Fit Together

**The homelab is one of my personal projects.** DevHub is where I maintain shared repository practices, reusable engineering workflows, AI customizations and optional project tracking. This guide explains homelab-specific concepts and decisions itself; the private homelab records what I actually chose and implemented. Separate repositories protect publication and operational boundaries.

TODO: remove devhub-sdks later and update with latest results.

| Location | Owns | Examples |
|---|---|---|
| [DevHub](https://github.com/CollinPoetoehena/DevHub) | Shared practices and project tooling for my personal projects | Repository standards, reusable-workflow guidance, AI customizations, task tracking, and a catalog of packages maintained there |
| [devhub-sdks](https://github.com/CollinPoetoehena/devhub-sdks) | Independently versioned Python and Go packages for reusable integrations | SDK packaging and release foundations; concrete capabilities are added from consumer needs, not copies of private homelab configuration |
| Public homelab (this repository) | A standalone homelab design and operations guide | Shared versus separate home/lab gateways, isolated routing exercises, small-host purchasing and capacity, power/noise constraints, equipment placement, and home-scale recovery |
| [Private homelab](https://github.com/CollinPoetoehena/devhub-homelab-private) (access required) | Actual selected design, implementation, and operations | Equipment and topology, confirmed internal settings, decision rationale, runbooks, verification results, and recovery records |

The [public network guide](./02-architecture/network.md) explains home/lab boundaries, gateway and switch choices, wireless access, and recovery in context. It links to standards and vendor documentation where useful; readers do not need DevHub or another repository to follow the reasoning. The private network chapter records the actual configuration and what was verified.

The [compute and storage chapters](./02-architecture/README.md#compute-and-storage-together) explain home-scale capacity, storage, electricity and accepted failure domains. Actual inventory, allocations and operational results stay private. Shared customizations and independently maintained packages remain in their own lifecycle; the [repository guidelines](./GUIDELINES.md#devhub-relationship-and-scope) link to their current entry points.

Each chapter gives enough context to understand its homelab-specific choices without requiring a second reference site. Official standards and product documentation provide deeper detail where needed. The private record can link to shared project practices when relevant, but neither homelab repository mirrors DevHub.

The [repository guidelines](./GUIDELINES.md#content-ownership) turn this relationship into maintenance and publication rules. This separation does not authorize moving private configuration into either public repository.

---

## Working Across DevHub and the Homelab Repositories

For my own workflow, DevHub documents the shared [personal-project workspace](https://github.com/CollinPoetoehena/DevHub/blob/main/docs/USAGE.md#personal-project-workspace) and [AI customizations](https://github.com/CollinPoetoehena/DevHub/blob/main/docs/USAGE.md#ai-customizations). These are optional project practices, not prerequisites for reading or applying the guide. My preferred setup is one focused VS Code workspace containing [DevHub](https://github.com/CollinPoetoehena/DevHub), [devhub-homelab](https://github.com/CollinPoetoehena/devhub-homelab) and [devhub-homelab-private](https://github.com/CollinPoetoehena/devhub-homelab-private), cloned under a shared parent folder. Each clone remains its own Git repository; private access is not required to read or work with the public projects.

**Everything beyond that default set is optional.** Keep only these repositories for a focused workspace, or add consuming applications, separately maintained Ansible roles or collections, Terraform modules, Helm charts, and shared tooling when their context helps the work. Choose whichever set suits the task and your preferences; there is no requirement to clone every related project. Additional repositories can make it easier to inspect a dependency's public contract and coordinate changes without copying its implementation into the homelab.

This shared workspace is useful when I work across related projects: AI tools can use relevant local context, and I can coordinate changes without repeatedly switching windows. A single repository is also suitable for isolated work. For official workspace setup options, see [VS Code multi-root workspaces](https://code.visualstudio.com/docs/editing/workspaces/multi-root-workspaces).

Preserve the repositories' information boundaries: DevHub owns shared project practices and reusable-workflow guidance, component repositories own their implementations, the public homelab owns this general guide, and the private homelab owns actual design and operations. Sharing a workspace does not merge Git histories, release lifecycles or permissions. Private access is not permission to publish private details; public changes still require deliberate review against the [publication boundary](./GUIDELINES.md#publication-boundary). See [AI usage](#ai-usage) for the optional shared customization setup.

---

## Task Tracking

Lightweight task tracking is useful: a small task, visible status, completion check, and next-step note keep follow-ups out of my head and make it easy to pause and resume. It should reduce pressure, not turn the homelab into a delivery obligation.

I track **all my personal projects centrally in DevHub**, including public and private homelab work. The homelab is one part of that hub, not a separate backlog. See [Central DevHub Task Tracking](https://github.com/CollinPoetoehena/DevHub/blob/main/docs/how-to/TASK-TRACKING.md) for the private project, statuses, privacy rules, and workflow.

Keep the homelab's [relaxation, enjoyment, and focus boundary](./01-foundations/goals-and-scope.md#relaxation-and-enjoyment-over-productivity-and-pressure): work when interested, leave a resumption note, and use optional timeboxes as permission to stop, not deadlines. The full shared guidance lives in [DevHub's guidelines](https://github.com/CollinPoetoehena/DevHub/blob/main/docs/GUIDELINES.md#relaxation-and-enjoyment-over-productivity-and-pressure). Follow the [publication boundary](./GUIDELINES.md#publication-boundary); a private project never makes public issue contents private.

---

## Documentation

- The [book table of contents](#table-of-contents-of-the-book) links to every part and chapter.
- [Architecture](./ARCHITECTURE.md) describes the book's repository structure and its public/private ownership boundary.
- [Guidelines](./GUIDELINES.md) are the shared source for documentation conventions, publication review, sanitization and sensitive-data handling.
- [Project Structure](./04-automation/project-structure.md) explains responsibility boundaries for implementation and reusable components.

Examples are illustrative and must be adapted to the reader's requirements; fenced examples are not a deployable mirror. A documented design or checklist is not evidence of a completed deployment. The private repository links to these documents through GitHub `blob/main` URLs; access to it is not required to use the public guidance.

---

<a id="ai-and-copilot-skills"></a>
## AI Customizations

This repository is readable without AI customizations. Its root [agent guidance](./AGENTS.md) summarizes relevant context and links to the [repository guidelines](./GUIDELINES.md). The [APM manifest](./apm.yml) declares the shared common and homelab packages for optional AI-assisted maintenance; reading the guide does not require installing them or a homelab-specific skill.

In my shared project workspace, DevHub provides common Copilot agents, skills and scoped instructions through its APM package. The [DevHub AI customizations guide](https://github.com/CollinPoetoehena/DevHub/blob/main/docs/USAGE.md#ai-customizations) describes that optional setup; the [APM overview's consumer-specific guidance](https://github.com/CollinPoetoehena/DevHub/blob/main/packages/apm/README.md#consumer-specific-guidance) explains how projects add their own context. These customizations support maintenance work but do not supply homelab concepts or replace this guide.

**Documentation owns project policy.** The [repository guidelines](./GUIDELINES.md), including the [Documentation Structure & Guideline](./GUIDELINES.md#documentation-structure--guideline), remain authoritative for both homelab repositories. Follow the documented publication procedure, distinguish plans from verified results and validate changes. Publication and live infrastructure operations still require explicit authorization.

### AI usage

To use the common customizations in a shared workspace, follow DevHub's [AI customizations setup](https://github.com/CollinPoetoehena/DevHub/blob/main/docs/USAGE.md#ai-customizations). A link to the documentation does not install or activate them; follow the current setup steps for the selected host. Workspace setup is optional, and each repository keeps its own instructions and privacy boundary. Access to all three repositories never authorizes publishing private content.
