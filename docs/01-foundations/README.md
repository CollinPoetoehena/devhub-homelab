# Part 1 · Foundations

> **Layer: Foundations — why are we doing it?** · [Book overview](../README.md) · Next part: [Part 2 · Architecture](../02-architecture/README.md)

Every homelab story starts before the first device is bought or the first VM is created. The foundations answer **why the homelab exists, what it must achieve, how decisions are made and what it may cost**. Everything later in the book—network boundaries, hosts, storage, platforms, services, automation and operations—is designed *against* these foundations, and each later chapter links back here when it needs to justify a choice.

This part answers the first two questions of the [storyline](../README.md#the-storyline):

1. **Why am I building this?** — [Goals and Scope](./goals-and-scope.md)
2. **What principles and requirements guide it?** — [Requirements](./requirements.md), [Design Principles](./design-principles.md) and [Cost, Scope and Buying Decisions](./cost.md)

---

## Table of Contents

- [Chapters in This Part](#chapters-in-this-part)
- [How the Foundations Are Used Later](#how-the-foundations-are-used-later)
- [Cost Is a Foundation, Not an Afterthought](#cost-is-a-foundation-not-an-afterthought)
- [What Stays Private](#what-stays-private)

---

## Chapters in This Part

Read the chapters in order; each builds on the previous one.

| # | Chapter | Question it answers | What you take away |
|---|---|---|---|
| 1 | [Goals and Scope](./goals-and-scope.md) | Why build a homelab, and what should it (not) become? | The purpose options (learning, development, household services, enterprise practice), the personal motivation, a sustainable scope and explicit exclusions |
| 2 | [Requirements](./requirements.md) | What must the homelab achieve? | Outcome-based requirements, requirement areas, service tiers, RPO/RTO, constraints, priorities and traceability |
| 3 | [Design Principles](./design-principles.md) | How do we choose between designs that meet the requirements? | The recurring decision rules of the book, including learning enterprise patterns without enterprise availability |
| 4 | [Cost, Scope and Buying Decisions](./cost.md) | What may it cost, and when is a purchase justified? | The purchase test, full lifecycle cost, bounded experiments, learning at work with approval and the expansion rule |

---

## How the Foundations Are Used Later

The foundations are not an introduction to skim and forget. Each later part refers back to them:

| Later part | Uses from the foundations |
|---|---|
| [Part 2 · Architecture](../02-architecture/README.md) | Requirements and service tiers decide the network boundaries, host capacity, storage roles and platform; principles such as [household independence](./design-principles.md#keep-the-household-foundation-independent-of-the-lab) and [explicit failure domains](./design-principles.md#make-failure-domains-explicit) shape every option table |
| [Part 3 · Platform Services](../03-platform-services/README.md) | Security requirements and [least privilege](./design-principles.md#least-privilege-and-explicit-boundaries) shape DNS, certificates, identity and secrets; running-cost constraints shape [scheduling](../03-platform-services/scheduling.md) |
| [Part 4 · Automation](../04-automation/README.md) | [One owner per responsibility](./design-principles.md#one-owner-per-responsibility) and [automate the repeatable](./design-principles.md#automate-the-repeatable-document-the-exceptions) decide tool ownership and what stays manual |
| [Part 5 · Services](../05-services/README.md) | [Service tiers](./requirements.md#service-tiers-how-much-does-it-matter) classify every workload and define what promotion from experiment to household service requires |
| [Part 6 · Operations](../06-operations/README.md) | [RPO and RTO](./requirements.md#availability-and-recovery-requirements) drive backup frequency and recovery plans; [evidence over assumption](./design-principles.md#evidence-over-assumption) drives observability and verification |

When a later decision cannot be traced back to a goal, requirement or principle, it is either an undocumented assumption or a component without a reason. Both are worth revisiting.

---

## Cost Is a Foundation, Not an Afterthought

Not overspending is one of the most important constraints of this homelab. The [cost chapter](./cost.md) is therefore part of the foundations rather than a buying appendix: the [purchase test](./cost.md#the-most-important-constraint-buy-for-a-real-need), [lifecycle cost](./cost.md#count-purchase-and-operating-costs) and [bounded experiments](./cost.md#learn-advanced-ideas-through-bounded-experiments) are applied throughout the architecture, platform and services chapters. The same reasoning appears as the principle [start simple and expand on evidence](./design-principles.md#start-simple-and-expand-on-evidence).

---

## What Stays Private

This part describes the method and generic examples. Personal priorities, the actual requirement list, the household context and the cost decisions for specific equipment belong in the private homelab record, consistent with the [content ownership rules](../repository-guidelines.md#content-ownership) and the [publication boundary](../repository-guidelines.md#publication-boundary).
