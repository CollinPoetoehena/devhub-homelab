# Part 5 · Services

> **Layer: Implementation — what runs on it?** · [Book overview](../README.md) · Previous part: [Part 4 · Automation](../04-automation/README.md) · Next part: [Part 6 · Operations](../06-operations/README.md)

Everything so far—the foundations, architecture, platform services and automation—exists so that useful workloads can run. This part answers the storyline's sixth question: **what services run on it?** It explains how to decide which workloads deserve a place in the homelab, what each one needs from the layers below, and how a workload moves through its lifecycle from experiment to household service to retirement.

Services sit in the implementation layer because they are where the architecture is *realized and used*: a service is delivered by the [deployment workflow](../04-automation/deployment.md), runs on the [platform](../02-architecture/platform.md), depends on [platform services](../03-platform-services/README.md) such as DNS, certificates, identity and secrets, and is then observed, backed up and recovered by [operations](../06-operations/README.md).

---

## Table of Contents

- [Chapters in This Part](#chapters-in-this-part)
- [From Requirement to Running Service](#from-requirement-to-running-service)
- [Keep the Service List Small](#keep-the-service-list-small)
- [What Stays Private](#what-stays-private)

---

## Chapters in This Part

| # | Chapter | Question it answers | What you take away |
|---|---|---|---|
| 1 | [Workloads](./workloads.md) | How do I decide what a workload needs and how it is operated? | Workload types and exposure options, service tiers applied to workloads, the service record, promoting experiments and retiring services |
| 2 | [Self-Hosted Services](./self-hosted-services.md) | Which kinds of services are worth running, and what do they demand? | Self-host versus hosted, common service categories with their trade-offs, and a sensible order for adding services |

---

## From Requirement to Running Service

A service should follow the same chain as every other part of the design:

```text
Requirement or learning goal
  -> service tier and service record (workloads.md)
  -> dependencies: DNS, certificates, identity, secrets, storage (Part 3, Part 2)
  -> delivery: CI, GitOps or another approved workflow (Part 4)
  -> verification: health, access, backup and restore (Part 6)
```

Skipping a step is how homelab services become fragile: a service deployed without a backup becomes a data-loss risk once someone relies on it, and a service without a recorded purpose is hard to retire. The [workload chapter](./workloads.md) turns this chain into a short, repeatable routine.

---

## Keep the Service List Small

Every service adds patching, exposure, monitoring and recovery work. The [cost chapter](../01-foundations/cost.md) applies to software as much as to hardware: free-to-download services still cost time, electricity and attention. Prefer a few well-understood services with tested recovery over a long list that nobody can operate after an outage, and remove experiments once they have taught what they were meant to teach.

---

## What Stays Private

This part describes how to evaluate and operate services in general. The actual service catalog—what is installed, its versions, exposure, data locations and service records—belongs in the private homelab record under the [publication boundary](../repository-guidelines.md#publication-boundary).
