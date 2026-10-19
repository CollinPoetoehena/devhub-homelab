# Goals and Scope

> **Part 1 · Foundations** — [Part overview](./README.md) · Next: [Requirements](./requirements.md)

Choose the purpose before buying equipment or installing tools. This is the first chapter of the book because every later decision—how many hosts, which network boundaries, which services, how much automation, how much recovery effort—only makes sense against a clear answer to *why this homelab exists and what it should and should not become*.

The chapter answers the first question of the storyline: **why am I building this?** It describes the kinds of purpose a homelab can serve, why the hands-on exploration is worth it, and how to keep the scope small enough to remain enjoyable and maintainable. The next chapters turn that purpose into [requirements](./requirements.md), [design principles](./design-principles.md) and a [cost boundary](./cost.md).

---

## Table of Contents

- [Design the Purpose](#design-the-purpose)
  - [Why Build a Homelab?](#why-build-a-homelab)
- [Keep the Scope Sustainable](#keep-the-scope-sustainable)
- [Define What Is Out of Scope](#define-what-is-out-of-scope)
- [From Goals to Requirements](#from-goals-to-requirements)

---

## Design the Purpose

A homelab may be a disposable learning environment, a development sandbox, a self-hosted household platform, or a mixture. Learning Kubernetes on demand is different from hosting the only copy of household files. Define what must remain available when experiments fail.

Record learning topics, useful services, data importance, acceptable downtime/data loss, budget, electricity, space/noise, cabling and maintenance time. Decide whether remote access is actually needed and who administers the system.

| Option | What it is | Benefits | Drawbacks / responsibilities | Use cases / when to choose |
|---|---|---|---|---|
| Disposable learning | Short-lived isolated exercises | Fast rebuilds, bounded scope and useful diagnostics | Remove/promote deliberately; not every exercise needs permanent hardware or HA. **Maintenance:** Little permanent upkeep if exercises are removed; refresh rebuild inputs and clean up resources/credentials after each exercise. | Explore tools/protocols without household dependencies |
| Development/testing | Reproducible environments for application integration | Realistic test paths and repeatable development | Test credentials/data must not share production trust; cleanup and capacity still matter. **Maintenance:** Keep test images, dependencies and fixtures current; reset environments and verify that stale credentials or data do not persist. | Build/test applications and infrastructure changes |
| Household services | Self-hosted useful services with ongoing users/data | Practical value and operational learning | Predictable maintenance, backups and usable recovery; household users may not accept lab-style downtime. **Maintenance:** Recurring patching, backup checks and restore exercises are essential; schedule changes around users and retain usable recovery instructions. | You can own the service's reliability/data lifecycle |
| Enterprise-platform practice | Targeted management, upgrade and failure exercises | Experience with specific enterprise concepts | Cost/resources and operational scope; copying enterprise topology is unnecessary for unrelated goals. **Maintenance:** Budget time for supported release paths and management components; retire exercises that no longer serve the learning goal. | Enterprise learning itself is the requirement |

Write requirements as outcomes: "restore important data within the accepted time" or "keep household connectivity working during compute maintenance," not "buy three nodes." Identify must-haves, useful learning goals and deferred ideas. A tool is a means of achieving those outcomes, not the goal unless learning that tool is explicitly the exercise.

### Why Build a Homelab?

A homelab brings many layers of engineering together for hands-on exploration: hardware, networking, storage, software, observability, automation, and much more. An idea does not stay abstract: you can build it, see how its parts depend on one another, find out what happens when something changes, and work through how to make it better. That mix of curiosity, problem-solving, and tangible progress is a large part of the appeal for me. I love engineering, enjoy learning how complex things fit together, and find that time spent experimenting in my lab gives me both a sense of purpose and a way to relax.

It is also a personal playground: a place to try unfamiliar ideas, make mistakes safely, recover, and decide what is worth keeping without turning every experiment into a permanent service. A homelab is not only a way to host things; it is a self-directed space to create, learn, and enjoy the process. And there is always more to it than any short description can capture: every experiment can reveal another connection, another question, or a new direction to explore.

---

## Keep the Scope Sustainable

Prefer a small stable foundation and disposable experiments. One host can teach many concepts; extra machines are justified by capacity, physical-failure practice or actual availability requirements, not an enterprise-rack aesthetic.

Self-hosting adds patching, backup and recovery obligations. Keep essential household routing separate from experimental compute dependencies. Automation is useful when it saves work or supports learning, not when it makes every simple task a development project.

It is easy to overspend on equipment that looks impressive but is unnecessary for your goals; use the central [cost and scope guide](./cost.md) before expanding.

Include time and operational cost as well as purchase cost. Software that is free to download still needs updates and recovery work; repurposed hardware still consumes electricity. Prefer one well-understood service with tested recovery over a long list that nobody can operate after an outage.

Define a stable foundation and an experiment lifecycle: create, observe, learn, then remove or promote deliberately. Automate repeatable rebuilds where supported, and decide what evidence an experiment should produce. Promotion to a household dependency should add ownership, monitoring, access controls and data recovery, not merely a permanent restart policy.

---

## Define What Is Out of Scope

A goal statement is incomplete until it also says what the homelab is **not**. Writing that down protects the budget, the household and spare time just as much as the goals themselves. Typical exclusions are:

- **Not a production data center.** Enterprise availability, redundant everything and vendor support contracts are not targets unless a specific requirement demands them; see [Learn Enterprise Patterns Without Enterprise Availability](./design-principles.md#learn-enterprise-patterns-without-enterprise-availability).
- **Not the only copy of important data.** If the lab stores something valuable, it also owns a [backup and restore process](../06-operations/backups.md); otherwise, keep that data elsewhere.
- **Not a dependency for basic household connectivity.** Internet, Wi-Fi and essential name resolution should keep working while lab compute is off, broken or being rebuilt; the [network chapter](../02-architecture/network.md#design-options) explains how.
- **Not a collection of every interesting tool.** A tool earns its place by serving a goal; evaluating it briefly and removing it again is a successful experiment, not a failure.
- **Not a reason to buy equipment first.** Purchases follow measured needs and the [cost and scope test](./cost.md#the-most-important-constraint-buy-for-a-real-need).

Exclusions can change later—for example, a service may deliberately be promoted to a household dependency—but the change should be a conscious decision with its added obligations, not something that happens because an experiment kept running.

---

## From Goals to Requirements

Goals are intentionally broad. The [requirements chapter](./requirements.md) turns them into measurable outcomes, and the [design principles](./design-principles.md) describe how to choose between options that meet them. Together with the [cost boundary](./cost.md), these foundations are what the [architecture](../02-architecture/README.md) is designed against. Revisit the goals when a choice adds cost or operational work without supporting them.

Personal priorities and the actual decision record remain private under the [repository guidelines](../repository-guidelines.md#content-ownership).

For each major choice, record the requirement it satisfies and the cost/risk accepted. Review goals after changes in workload, available hardware or household reliance. This prevents expanding a cluster simply because a feature exists and helps explain the reasoning in a portfolio without publishing actual network inputs.
