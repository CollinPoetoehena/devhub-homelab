# The Homelab Book

This documentation is written to be **read like a book, from start to finish**. It tells the story of a homelab: why it is built, what principles and requirements guide it, what architecture solves those requirements, which capabilities make that architecture usable, how it is implemented as code, which services run on it, and how it is operated, observed and recovered. Every chapter builds on the ones before it and links back to the foundations when a choice needs justification.

It is the open-source version of my homelab journey: the reasoning, options, trade-offs and lessons are public, while the actual environment—exact topology, addresses, inventory, configuration and operational records—stays in the private repository. See [why there are two homelab repositories](../README.md#why-two-homelab-repositories).

The book is a reference to explore at your own pace, not a completion checklist. Its guiding boundary is [relaxation and enjoyment over productivity and pressure](./01-foundations/goals-and-scope.md#relaxation-and-enjoyment-over-productivity-and-pressure): the lab should support life, not interrupt work, rest or everyday activities.

---

## Table of Contents

- [The Documentation Layers](#the-documentation-layers)
- [The Storyline](#the-storyline)
- [Table of Contents of the Book](#table-of-contents-of-the-book)
- [How to Read a Chapter](#how-to-read-a-chapter)
- [Why the Book Is Organized This Way](#why-the-book-is-organized-this-way)
- [Guidelines](#guidelines)

---

## The Documentation Layers

The book has four layers, read from top to bottom. Each layer answers one question and depends on the layers above it:

```text
            +----------------------+
            |     FOUNDATIONS      |
            | Why are we doing it? |
            +----------+-----------+
                       |
                       v
            +----------------------+
            |     ARCHITECTURE     |
            | What are we building?|
            +----------+-----------+
                       |
                       v
            +----------------------+
            |    IMPLEMENTATION    |
            | How is it realized?  |
            +----------+-----------+
                       |
                       v
            +----------------------+
            |      OPERATIONS      |
            | How do we run it?    |
            +----------------------+
```

| Layer | Parts | What it contains |
|---|---|---|
| Foundations | [Part 1 · Foundations](./01-foundations/README.md) | Goals and scope, requirements, design principles and cost |
| Architecture | [Part 2 · Architecture](./02-architecture/README.md), [Part 3 · Platform Services](./03-platform-services/README.md) | The building blocks—physical, network, compute, storage, platform—and the shared capabilities that make them usable |
| Implementation | [Part 4 · Automation](./04-automation/README.md), [Part 5 · Services](./05-services/README.md) | How the design is realized as code and delivered, and which workloads run on it |
| Operations | [Part 6 · Operations](./06-operations/README.md) | Observability, backups, disaster recovery and maintenance |

---

## The Storyline

Within the layers, the parts follow a storyline of seven questions:

```text
1. Why am I building this?
        |
2. What principles and requirements guide it?
        |
3. What architecture solves those requirements?
        |
4. What capabilities make up that architecture?
        |
5. How are those capabilities implemented?
        |
6. What services run on it?
        |
7. How is it operated, observed and recovered?
```

| # | Question | Where it is answered |
|---|---|---|
| 1 | Why am I building this? | [Goals and Scope](./01-foundations/goals-and-scope.md) |
| 2 | What principles and requirements guide it? | [Requirements](./01-foundations/requirements.md), [Design Principles](./01-foundations/design-principles.md), [Cost](./01-foundations/cost.md) |
| 3 | What architecture solves those requirements? | [Part 2 · Architecture](./02-architecture/README.md) |
| 4 | What capabilities make up that architecture? | [Part 3 · Platform Services](./03-platform-services/README.md) |
| 5 | How are those capabilities implemented? | [Part 4 · Automation](./04-automation/README.md) |
| 6 | What services run on it? | [Part 5 · Services](./05-services/README.md) |
| 7 | How is it operated, observed and recovered? | [Part 6 · Operations](./06-operations/README.md) |

The order is a reasoning path, not a waterfall: a workload's storage needs can send you back to revisit host capacity, and an operations lesson can change a requirement. Revisiting earlier chapters is part of the story.

---

## Table of Contents of the Book

Each part has a `README.md` that introduces it; each chapter is one file about one subject. Every chapter starts with a navigation line pointing to the previous and next chapter, so the book can be read in order without returning to this page.

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

Two themes run through every part. The [cost and scope guide](./01-foundations/cost.md) is a central design constraint: start simple, count purchase and running costs, and add hardware only when an actual use case justifies the extra lifecycle. And [learning enterprise patterns without enterprise availability](./01-foundations/design-principles.md#learn-enterprise-patterns-without-enterprise-availability) means shared hardware and accepted recovery downtime can be deliberate cost choices, not incomplete enterprise designs; each chapter identifies the resulting failure domains rather than implying production-grade resilience.

---

## How to Read a Chapter

Every subject chapter tells its own small story, from high level to low level:

```text
Purpose and requirements -> design options and trade-offs -> deeper design and equipment/tool choices
    -> implementation considerations -> verification (and recovery where relevant)
```

Option comparisons use detailed tables with the same columns everywhere—what an option is, its benefits, its drawbacks and responsibilities (always including **Maintenance**), and when to choose it—with named tools linking to their official documentation. Surrounding prose explains shared principles, dependencies and how to verify the result. Decisions are part of the chapter they affect; there is no separate decisions section to cross-reference.

This public book is a decision guide, **not a vendor installation manual or a copy of the private deployment sequence**. Follow the [repository relationship in the main README](../README.md#how-devhub-and-the-homelab-repositories-fit-together): DevHub owns general theory and architecture, this book applies them to homelab requirements, and actual selections, detailed runbooks and results belong in the private repository. General protocols, design principles, commands and development workflows stay in [DevHub](https://github.com/CollinPoetoehena/DevHub/tree/main/reference); the book links to them instead of repeating full explanations.

---

## Why the Book Is Organized This Way

> **Previous approach: HLD and LLD folders.** Earlier versions of this documentation were split into goals, a high-level design (HLD) folder and a low-level design (LLD) folder, and before that into separate design and setup trees. HLD and LLD are *levels of abstraction*, but they were used as *filing categories*. As a result, the same subject was spread across folders: storage had a high-level design and separate low-level backup choices, the network had a diagram in one place and setup details in another, and monitoring ended up as "low-level" although it has an architecture of its own. Readers had to jump between overlapping documents, and the documentation did not read as a story.

The current structure organizes the documentation by **the reader's journey instead of by document type**. Each subject lives in exactly one file that moves from high level to low level internally, each part has one README as its entry point, and decisions stay with the subject they belong to. One large chapter—such as the network chapter—is deliberately preferred over many small nested documents: it keeps the story in one place, is easier to maintain and makes it obvious where a change belongs.

The complete rationale and the rules that keep the structure consistent are maintained in the [Documentation Structure & Guideline](./repository-guidelines.md#documentation-structure--guideline), including [why not high-level and low-level design folders](./repository-guidelines.md#why-not-high-level-and-low-level-design-folders) and [one file per subject](./repository-guidelines.md#one-file-per-subject).

---

## Guidelines

Use the [repository guidelines](./repository-guidelines.md) for the documentation structure, the public/private boundary, sensitive values, publication and writing conventions. The [project structure](./04-automation/project-structure.md) connects the documentation to implementation ownership and contains a deliberately high-level homelab structure summary; publishing the private implementation is not required to demonstrate the design.
