# Repository Architecture

This repository is one documentation book: the book is the product, not a wrapper around a separate `docs/` tree. Chapters live at the root so the repository landing page is also the book's main entry point.

## Table of Contents

- [Repository scope and ownership](#repository-scope-and-ownership)
- [The book structure](#the-book-structure)
  - [Documentation layers](#documentation-layers)
  - [The storyline](#the-storyline)
  - [Inside a chapter](#inside-a-chapter)
  - [Why this structure](#why-this-structure)
- [Implementation code and examples](#implementation-code-and-examples)

---

## Repository scope and ownership

The public repository owns standalone homelab explanations: goals, requirements, design options, trade-offs, generic examples and reusable lessons. The private homelab repository owns the actual environment, implementation, operational records and recovery details. [DevHub](https://github.com/CollinPoetoehena/DevHub) owns shared personal-project practices, including [relaxation and enjoyment over productivity and pressure](https://github.com/CollinPoetoehena/DevHub/blob/main/docs/GUIDELINES.md#relaxation-and-enjoyment-over-productivity-and-pressure), and reusable tooling. See [Guidelines](./GUIDELINES.md#content-ownership) for detailed ownership and the [publication boundary](./GUIDELINES.md#publication-boundary).

This document describes how the public book is organized; it is not a complete or verified description of the live homelab. For system-design content, see [Part 2 · Architecture](./02-architecture/README.md) and [Part 3 · Platform Services](./03-platform-services/README.md).

## The book structure

The book is written to be **read from start to finish**, following the homelab journey: why it is built, what requirements guide it, how the design meets them, and how it can be implemented, operated and recovered. Chapters build on earlier ones and link back to the foundations when a choice needs justification.

The root [README](./README.md#table-of-contents-of-the-book) is the book's main entry point and complete chapter index. Numbered root folders define reading order, each part README introduces its chapters, and each subject chapter links to neighboring chapters. Readers can also explore individual subjects at their own pace: this is not a completion checklist. The guiding boundary remains [relaxation and enjoyment over productivity and pressure](https://github.com/CollinPoetoehena/DevHub/blob/main/docs/GUIDELINES.md#relaxation-and-enjoyment-over-productivity-and-pressure), with a homelab-specific summary in [Goals and Scope](./01-foundations/goals-and-scope.md#relaxation-and-enjoyment-over-productivity-and-pressure).

### Documentation layers

Four layers organize the reader's journey. Each builds on the reasoning above it:

```text
            +------------------------+
            |      FOUNDATIONS       |
            |  Why are we doing it?  |
            +-----------+------------+
                        |
                        v
            +------------------------+
            |      ARCHITECTURE      |
            | What are we building?  |
            +-----------+------------+
                        |
                        v
            +------------------------+
            |     IMPLEMENTATION     |
            |  How is it realized?   |
            +-----------+------------+
                        |
                        v
            +------------------------+
            |       OPERATIONS       |
            |   How do we run it?    |
            +------------------------+
```

| Layer | Parts | Focus |
|---|---|---|
| Foundations | [`01-foundations/`](./01-foundations/README.md) | Goals, requirements, principles and cost |
| Architecture | [`02-architecture/`](./02-architecture/README.md), [`03-platform-services/`](./03-platform-services/README.md) | System building blocks and shared capabilities |
| Implementation | [`04-automation/`](./04-automation/README.md), [`05-services/`](./05-services/README.md) | Automation, delivery and workloads |
| Operations | [`06-operations/`](./06-operations/README.md) | Observability, backups, recovery and maintenance |

### The storyline

The four layers contain six parts that answer seven questions. Foundations answers both why the lab exists and what constrains its design:

| # | Question | Where it is answered |
|---|---|---|
| 1 | Why am I building this? | [Goals and Scope](./01-foundations/goals-and-scope.md) |
| 2 | What principles and requirements guide it? | [Requirements](./01-foundations/requirements.md), [Design Principles](./01-foundations/design-principles.md) and [Cost](./01-foundations/cost.md) |
| 3 | What architecture solves those requirements? | [Part 2: Architecture](./02-architecture/README.md) |
| 4 | What capabilities make that architecture usable? | [Part 3: Platform Services](./03-platform-services/README.md) |
| 5 | How are those capabilities implemented and delivered? | [Part 4: Automation](./04-automation/README.md) |
| 6 | What services run on it? | [Part 5: Services](./05-services/README.md) |
| 7 | How is it operated, observed and recovered? | [Part 6: Operations](./06-operations/README.md) |

This order is a reasoning path, not a waterfall. A workload's storage needs may send the reader back to host capacity; recovery experience may change a requirement. Those links make feedback part of the story rather than forcing every decision into a one-way sequence.

### Inside a chapter

Each subject tells its own story from high level to low level:

```text
Purpose and requirements
    -> design options and trade-offs
    -> equipment and tool choices
    -> implementation considerations
    -> verification and recovery where relevant
```

Decisions stay with the subject they affect rather than in a separate decisions tree. Option comparisons explain benefits, drawbacks, maintenance responsibilities and when an option fits. Short conceptual context and links to official documentation support the reasoning without turning the book into a vendor manual or an exhaustive knowledge base. The boundary is intentional: this guide explains homelab-specific implications, not full theory or every product's configuration. See [Goals and Scope](./01-foundations/goals-and-scope.md#define-what-is-out-of-scope) and the [documentation scope guidelines](./GUIDELINES.md#documentation-structure--guideline).

The public implementation layer explains approaches and illustrative examples, not the actual private deployment sequence. The private repository records actual selections, configuration, runbooks and results.

### Why this structure

Earlier goals/HLD/LLD and separate design/setup trees split the same subject across several locations. HLD and LLD describe levels of abstraction, not separate subjects: networking has both architectural choices and implementation details, while monitoring has an architecture of its own.

Organizing by the **reader's journey rather than document type** keeps each subject together. One chapter can move from purpose to detailed considerations without sending the reader between overlapping files. Part introductions provide context and navigation; subject chapters own their reasoning. The [documentation structure guidelines](./GUIDELINES.md#documentation-structure--guideline) maintain the detailed writing conventions.

The root layout avoids a redundant `docs/` wrapper because documentation is the whole public product. Project-level references are conventionally uppercase (`README.md`, `ARCHITECTURE.md`, `GUIDELINES.md`); numbered chapter folders and subject files remain lowercase kebab-case.

---

## Implementation code and examples

No implementation code or actual configuration belongs in this public repository. Keep the real implementation and environment-specific configuration in the private homelab repository. Code intended for reuse belongs in an appropriate, separately reviewed package repository, not here by default.

Fenced examples are allowed to explain concepts when they are generic, clearly illustrative and not a deployable mirror of the actual environment. Follow the [sanitization workflow](./GUIDELINES.md#sanitization-workflow) and do not treat a design or example as evidence of deployment.
